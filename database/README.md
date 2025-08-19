## Базы данных (MongoDB, MySQL)

### MongoDB — практики и примеры
- Проектирование под запросы; валидация на уровне приложения.
- Денормализация разумно; связи — референсы при необходимости.

Индекс под частые запросы
```js
// db.users.createIndex({ email: 1 }, { unique: true })
```

Пример агрегирования
```js
db.posts.aggregate([
  { $match: { published: true } },
  { $group: { _id: '$authorId', total: { $sum: 1 } } },
  { $sort: { total: -1 } }
]);
```

Транзакция (Node.js driver)
```js
const session = client.startSession();
try {
  await session.withTransaction(async () => {
    await users.insertOne({ _id: userId, name }, { session });
    await profiles.insertOne({ userId }, { session });
  });
} finally { await session.endSession(); }
```

### MySQL — практики и примеры
- Нормализация до 3НФ; денормализация целенаправленно.
- Типы данных точные, явные проекции, индексы под фильтры/джоины.

DDL: схема и индексы
```sql
CREATE TABLE users (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  email VARCHAR(255) NOT NULL UNIQUE,
  name VARCHAR(100) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE posts (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  user_id BIGINT NOT NULL,
  title VARCHAR(200) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  INDEX idx_posts_user_id (user_id),
  CONSTRAINT fk_posts_user FOREIGN KEY (user_id) REFERENCES users(id)
);
```

Запрос с явной проекцией
```sql
SELECT p.id, p.title, u.name
FROM posts p
JOIN users u ON u.id = p.user_id
WHERE u.email = ?
ORDER BY p.created_at DESC
LIMIT 20;
```

Транзакция
```sql
START TRANSACTION;
  INSERT INTO users(email, name) VALUES(?, ?);
  INSERT INTO profiles(user_id) VALUES(LAST_INSERT_ID());
COMMIT;
```


