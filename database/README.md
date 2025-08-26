# Работа с базами данных

## Зачем нужны эти принципы?

База данных — это основа твоего приложения. Качественная работа с БД обеспечивает:

- **Производительность** — быстрые запросы и отклик приложения
- **Надежность** — целостность данных и отсутствие потерь
- **Масштабируемость** — рост вместе с данными
- **Безопасность** — защита от потери и несанкционированного доступа

## Основные принципы работы с БД

### 1. Нормализация данных

**Принцип**: проектируй схему БД так, чтобы избежать дублирования и аномалий.

```sql
-- ❌ Плохо — денормализованная структура
CREATE TABLE orders (
    id INT PRIMARY KEY,
    customer_name VARCHAR(255),
    customer_email VARCHAR(255),
    customer_phone VARCHAR(20),
    product_name VARCHAR(255),
    product_price DECIMAL(10,2),
    order_date DATE
);

-- ✅ Хорошо — нормализованная структура
CREATE TABLE customers (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    phone VARCHAR(20),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE products (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE orders (
    id INT PRIMARY KEY AUTO_INCREMENT,
    customer_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL DEFAULT 1,
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (customer_id) REFERENCES customers(id),
    FOREIGN KEY (product_id) REFERENCES products(id)
);
```

### 2. Индексы для производительности

**Принцип**: создавай индексы для полей, по которым часто ищешь и сортируешь.

```sql
-- ✅ Индексы для часто используемых запросов
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_created_at ON users(created_at);
CREATE INDEX idx_posts_user_id_created_at ON posts(user_id, created_at);

-- ✅ Уникальные индексы
CREATE UNIQUE INDEX idx_users_email ON users(email);
CREATE UNIQUE INDEX idx_products_sku ON products(sku);

-- ✅ Составные индексы
CREATE INDEX idx_orders_customer_date ON orders(customer_id, order_date DESC);
CREATE INDEX idx_posts_status_created ON posts(status, created_at DESC);
```

### 3. Транзакции для целостности

**Принцип**: используй транзакции для операций, которые должны быть атомарными.

```sql
-- ✅ Транзакция для создания заказа
START TRANSACTION;

INSERT INTO orders (customer_id, product_id, quantity) 
VALUES (1, 5, 2);

UPDATE products 
SET stock = stock - 2 
WHERE id = 5 AND stock >= 2;

-- Проверяем, что обновление прошло успешно
IF ROW_COUNT() = 0 THEN
    ROLLBACK;
    SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Insufficient stock';
END IF;

COMMIT;
```

## MySQL — современные практики

### 1. Оптимизация запросов

```sql
-- ✅ Используй EXPLAIN для анализа запросов
EXPLAIN SELECT u.name, COUNT(p.id) as post_count
FROM users u
LEFT JOIN posts p ON u.id = p.user_id
WHERE u.created_at >= '2024-01-01'
GROUP BY u.id, u.name
HAVING post_count > 5
ORDER BY post_count DESC;

-- ✅ Избегай SELECT *
SELECT id, name, email, created_at 
FROM users 
WHERE is_active = 1;

-- ✅ Используй LIMIT для больших выборок
SELECT id, name, email 
FROM users 
WHERE created_at >= '2024-01-01'
ORDER BY created_at DESC
LIMIT 100;

-- ✅ Пагинация с OFFSET
SELECT id, name, email 
FROM users 
ORDER BY created_at DESC
LIMIT 20 OFFSET 40; -- страница 3 (20 записей на страницу)
```

### 2. Сложные запросы и JOIN

```sql
-- ✅ Правильное использование JOIN
SELECT 
    u.name as user_name,
    p.title as post_title,
    c.content as comment_content,
    c.created_at as comment_date
FROM users u
INNER JOIN posts p ON u.id = p.user_id
LEFT JOIN comments c ON p.id = c.post_id
WHERE u.is_active = 1
    AND p.status = 'published'
    AND c.created_at >= DATE_SUB(NOW(), INTERVAL 7 DAY)
ORDER BY c.created_at DESC;

-- ✅ Подзапросы для агрегации
SELECT 
    u.name,
    u.email,
    (SELECT COUNT(*) FROM posts WHERE user_id = u.id) as posts_count,
    (SELECT COUNT(*) FROM comments WHERE user_id = u.id) as comments_count
FROM users u
WHERE u.created_at >= '2024-01-01';

-- ✅ Window functions для аналитики
SELECT 
    user_id,
    post_title,
    created_at,
    ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY created_at DESC) as post_rank,
    LAG(created_at) OVER (PARTITION BY user_id ORDER BY created_at) as prev_post_date
FROM posts
WHERE status = 'published';
```

### 3. Миграции и версионирование схемы

```sql
-- ✅ Создание таблицы с правильными типами данных
CREATE TABLE users (
    id BIGINT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
    uuid CHAR(36) NOT NULL UNIQUE,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    email_verified_at TIMESTAMP NULL,
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    settings JSON NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    INDEX idx_users_email (email),
    INDEX idx_users_created_at (created_at),
    INDEX idx_users_active_created (is_active, created_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- ✅ Добавление индекса (миграция)
ALTER TABLE posts 
ADD INDEX idx_posts_user_status_date (user_id, status, created_at DESC);

-- ✅ Изменение типа данных
ALTER TABLE users 
MODIFY COLUMN settings JSON NOT NULL DEFAULT ('{}');

-- ✅ Добавление внешнего ключа
ALTER TABLE posts 
ADD CONSTRAINT fk_posts_user 
FOREIGN KEY (user_id) REFERENCES users(id) 
ON DELETE CASCADE;
```

### 4. Оптимизация производительности

```sql
-- ✅ Использование покрывающих индексов
CREATE INDEX idx_users_active_email_name ON users(is_active, email, name);

-- Запрос использует только индекс, не обращается к таблице
SELECT email, name FROM users WHERE is_active = 1;

-- ✅ Партиционирование больших таблиц
CREATE TABLE posts (
    id BIGINT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    title VARCHAR(255) NOT NULL,
    content TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) PARTITION BY RANGE (YEAR(created_at)) (
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025),
    PARTITION p2025 VALUES LESS THAN (2026),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);

-- ✅ Оптимизация текстового поиска
CREATE TABLE posts (
    id BIGINT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    content TEXT,
    FULLTEXT(title, content)
);

-- Поиск по тексту
SELECT id, title, MATCH(title, content) AGAINST('search term') as relevance
FROM posts 
WHERE MATCH(title, content) AGAINST('search term' IN BOOLEAN MODE)
ORDER BY relevance DESC;
```

## MongoDB — современные практики

### 1. Проектирование документов

```javascript
// ✅ Хорошо структурированный документ пользователя
{
  "_id": ObjectId("507f1f77bcf86cd799439011"),
  "email": "john@example.com",
  "name": "John Doe",
  "profile": {
    "avatar": "https://example.com/avatar.jpg",
    "bio": "Software developer",
    "location": "New York"
  },
  "settings": {
    "theme": "dark",
    "notifications": {
      "email": true,
      "push": false
    }
  },
  "createdAt": ISODate("2024-01-15T10:30:00Z"),
  "updatedAt": ISODate("2024-01-20T14:45:00Z"),
  "isActive": true
}

// ✅ Вложенные документы для связанных данных
{
  "_id": ObjectId("507f1f77bcf86cd799439012"),
  "title": "My First Post",
  "content": "This is my first blog post...",
  "author": {
    "_id": ObjectId("507f1f77bcf86cd799439011"),
    "name": "John Doe",
    "email": "john@example.com"
  },
  "tags": ["programming", "javascript"],
  "publishedAt": ISODate("2024-01-15T10:30:00Z"),
  "status": "published"
}
```

### 2. Индексы в MongoDB

```javascript
// ✅ Создание индексов
// Простой индекс
db.users.createIndex({ "email": 1 }, { unique: true });

// Составной индекс
db.posts.createIndex({ "author._id": 1, "createdAt": -1 });

// Текстовый индекс
db.posts.createIndex({ "title": "text", "content": "text" });

// Геопространственный индекс
db.places.createIndex({ "location": "2dsphere" });

// Индекс для массивов
db.posts.createIndex({ "tags": 1 });

// ✅ Уникальный составной индекс
db.users.createIndex(
  { "email": 1, "isActive": 1 }, 
  { unique: true, partialFilterExpression: { "isActive": true } }
);
```

### 3. Агрегация данных

```javascript
// ✅ Сложная агрегация
db.posts.aggregate([
  // Фильтрация
  {
    $match: {
      status: "published",
      publishedAt: { $gte: new Date("2024-01-01") }
    }
  },
  
  // Объединение с пользователями
  {
    $lookup: {
      from: "users",
      localField: "author._id",
      foreignField: "_id",
      as: "authorDetails"
    }
  },
  
  // Разворачивание массива
  { $unwind: "$authorDetails" },
  
  // Группировка по автору
  {
    $group: {
      _id: "$author._id",
      authorName: { $first: "$authorDetails.name" },
      postsCount: { $sum: 1 },
      totalViews: { $sum: "$views" },
      avgViews: { $avg: "$views" },
      latestPost: { $max: "$publishedAt" }
    }
  },
  
  // Сортировка
  { $sort: { postsCount: -1 } },
  
  // Ограничение результатов
  { $limit: 10 }
]);

// ✅ Агрегация с условной логикой
db.orders.aggregate([
  {
    $addFields: {
      orderValue: {
        $multiply: ["$quantity", "$unitPrice"]
      },
      discount: {
        $cond: {
          if: { $gte: ["$orderValue", 100] },
          then: { $multiply: ["$orderValue", 0.1] },
          else: 0
        }
      }
    }
  },
  {
    $addFields: {
      finalValue: { $subtract: ["$orderValue", "$discount"] }
    }
  }
]);
```

### 4. Транзакции в MongoDB

```javascript
// ✅ Транзакция для создания заказа
const session = db.getMongo().startSession();

try {
  session.startTransaction();
  
  // Создание заказа
  const order = {
    userId: ObjectId("507f1f77bcf86cd799439011"),
    items: [
      {
        productId: ObjectId("507f1f77bcf86cd799439013"),
        quantity: 2,
        price: 29.99
      }
    ],
    total: 59.98,
    status: "pending",
    createdAt: new Date()
  };
  
  const orderResult = await db.orders.insertOne(order, { session });
  
  // Обновление остатков товара
  const updateResult = await db.products.updateOne(
    { _id: ObjectId("507f1f77bcf86cd799439013") },
    { $inc: { stock: -2 } },
    { session }
  );
  
  if (updateResult.modifiedCount === 0) {
    throw new Error("Insufficient stock");
  }
  
  await session.commitTransaction();
  console.log("Order created successfully");
  
} catch (error) {
  await session.abortTransaction();
  console.error("Transaction failed:", error);
} finally {
  await session.endSession();
}
```

### 5. Оптимизация запросов

```javascript
// ✅ Проекция для уменьшения объема данных
db.users.find(
  { isActive: true },
  { 
    name: 1, 
    email: 1, 
    profile: 1,
    _id: 0 
  }
);

// ✅ Использование covered queries
// Индекс покрывает все поля в запросе
db.users.createIndex({ "email": 1, "name": 1, "isActive": 1 });

db.users.find(
  { email: "john@example.com" },
  { name: 1, isActive: 1, _id: 0 }
);

// ✅ Пагинация с курсором
const pageSize = 20;
const lastId = ObjectId("507f1f77bcf86cd799439011");

db.posts.find({
  _id: { $gt: lastId },
  status: "published"
})
.sort({ _id: 1 })
.limit(pageSize);

// ✅ Агрегация с оптимизацией
db.posts.aggregate([
  // Ранняя фильтрация
  { $match: { status: "published" } },
  
  // Сортировка с индексом
  { $sort: { publishedAt: -1 } },
  
  // Лимит для уменьшения объема данных
  { $limit: 100 },
  
  // Lookup только после фильтрации
  {
    $lookup: {
      from: "users",
      localField: "author._id",
      foreignField: "_id",
      as: "authorDetails"
    }
  }
]);
```

## Безопасность данных

### 1. Защита от SQL инъекций

```php
// ✅ Использование подготовленных запросов
$stmt = $pdo->prepare("SELECT * FROM users WHERE email = ? AND is_active = ?");
$stmt->execute([$email, $isActive]);

// ✅ В Laravel Eloquent (автоматическая защита)
$users = User::where('email', $email)
    ->where('is_active', $isActive)
    ->get();

// ✅ В MongoDB (автоматическая защита)
$users = $collection->find([
    'email' => $email,
    'isActive' => $isActive
]);
```

### 2. Шифрование чувствительных данных

```sql
-- ✅ Шифрование паролей (Laravel делает это автоматически)
UPDATE users 
SET password = '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi'
WHERE id = 1;

-- ✅ Шифрование других чувствительных данных
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    email VARCHAR(255),
    ssn_encrypted VARBINARY(255), -- Зашифрованный SSN
    created_at TIMESTAMP
);
```

```javascript
// ✅ Шифрование в MongoDB
// Используй MongoDB Field Level Encryption
const schema = {
  bsonType: "object",
  encryptMetadata: {
    keyId: [keyId],
    algorithm: "AEAD_AES_256_CBC_HMAC_SHA_512_Deterministic"
  },
  properties: {
    ssn: {
      encrypt: {
        bsonType: "string"
      }
    }
  }
};
```

### 3. Резервное копирование

```bash
# ✅ Резервное копирование MySQL
mysqldump -u username -p database_name > backup_$(date +%Y%m%d_%H%M%S).sql

# ✅ Восстановление из резервной копии
mysql -u username -p database_name < backup_20240115_143000.sql

# ✅ Резервное копирование MongoDB
mongodump --db database_name --out /backup/path

# ✅ Восстановление MongoDB
mongorestore --db database_name /backup/path/database_name
```

## Мониторинг и оптимизация

### 1. Мониторинг производительности

```sql
-- ✅ Анализ медленных запросов MySQL
SHOW VARIABLES LIKE 'slow_query_log';
SHOW VARIABLES LIKE 'long_query_time';

-- Включение slow query log
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 2;

-- Анализ slow query log
mysqldumpslow /var/log/mysql/slow.log
```

```javascript
// ✅ Мониторинг MongoDB
// Включение profiler
db.setProfilingLevel(1, { slowms: 100 });

// Просмотр профиля
db.system.profile.find().sort({ ts: -1 }).limit(10);

// Анализ статистики коллекций
db.stats();
db.collection_name.stats();
```

### 2. Оптимизация индексов

```sql
-- ✅ Анализ использования индексов MySQL
SHOW INDEX FROM users;
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';

-- Удаление неиспользуемых индексов
DROP INDEX idx_unused_index ON users;
```

```javascript
// ✅ Анализ индексов MongoDB
// Просмотр индексов
db.users.getIndexes();

// Анализ использования индексов
db.users.aggregate([
  { $indexStats: {} }
]);

// Удаление неиспользуемых индексов
db.users.dropIndex("unused_index_name");
```

## Навигация

- **[Общие принципы](../general/README.md)** — процессы разработки, документация
- **[Фронтенд](../frontend/README.md)** — JavaScript, TypeScript, React, Vue
- **[Бэкенд](../backend/README.md)** — PHP, Laravel


