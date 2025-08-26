## Общая информация

### Зачем нужен манифест
- **Единые стандарты**: ускоряет онбординг, снижает количество повторяющихся обсуждений.
- **Предсказуемость**: общие принципы архитектуры, качества и процессов.
- **Качество**: контроль технического долга и стабильные релизы.

### Команда и роли (Отдел ПО Веб)
- **Начальник отдела ПО**: архитектура на уровне подразделения, приоритезация, roadmap, координация.
- **Фронтенд разработчики**: UI/UX реализация, производительность, доступность, сборка и инфраструктура фронта.
- **Бэкенд разработчики**: доменная логика, API, интеграции, работа с данными и производительностью сервисов.

Смежные подразделения:
- **Отдел автоматизации и эксплуатации (DevOps)**: CI/CD, инфраструктура, наблюдаемость, безопасность.
- **Отдел тестирования (QA)**: стратегия тестирования, автотесты, качество релизов, тестовая документация.

> Описания конкретных проектов из этого раздела убраны. Ведите проектную информацию в репозиториях проектов и Confluence.

### Стек и версии (целевые)
- **Фронтенд**: JavaScript (ES2022+), TypeScript (4.9+), React 18+/Vue 3+
- **Бэкенд**: PHP 8.2+, Laravel 10+
- **БД**: MongoDB 6+/MySQL 8+

# Общие принципы разработки

## Зачем нужны эти принципы?

Эти принципы — основа нашей работы. Они помогают:

- **Писать качественный код** — понятный, поддерживаемый и масштабируемый
- **Работать эффективно** — четкие процессы и стандарты
- **Избегать ошибок** — проверенные практики и подходы
- **Быстро онбордить новичков** — все правила в одном месте

## Основные принципы хорошего программирования

### 1. Читаемость превыше всего

**Принцип**: код читается чаще, чем пишется. Делай его понятным для других разработчиков.

```javascript
// ❌ Плохо
const x = a.filter(b => b.c > 10).map(d => d.e).reduce((f, g) => f + g, 0);

// ✅ Хорошо
const totalPrice = products
  .filter(product => product.price > 10)
  .map(product => product.price)
  .reduce((sum, price) => sum + price, 0);
```

```php
// ❌ Плохо
function f($a) { return $a > 0 ? $a * 2 : 0; }

// ✅ Хорошо
function calculateDiscount($price): float
{
    if ($price <= 0) {
        return 0;
    }
    
    return $price * 2;
}
```

### 2. Принцип KISS (Keep It Simple, Stupid)

**Принцип**: выбирай самое простое решение, которое работает.

```javascript
// ❌ Сложно
const isUserActive = user.status === 'active' || 
                    (user.status === 'pending' && user.verifiedAt) ||
                    (user.status === 'suspended' && user.reactivatedAt > Date.now() - 86400000);

// ✅ Просто
const isUserActive = user.isActive; // Логика вынесена в геттер модели
```

### 3. DRY (Don't Repeat Yourself)

**Принцип**: не дублируй код и знания.

```javascript
// ❌ Дублирование
const validateEmail = (email) => {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(email);
};

const validateUserEmail = (email) => {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(email);
};

// ✅ Переиспользование
const EMAIL_REGEX = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

const validateEmail = (email) => EMAIL_REGEX.test(email);
const validateUserEmail = validateEmail; // Алиас для ясности
```

### 4. SOLID принципы

**Принцип**: следуй принципам SOLID для создания гибкого и расширяемого кода.

```php
// ✅ Single Responsibility Principle
class UserService
{
    public function createUser(array $data): User
    {
        // Только создание пользователя
    }
}

class UserValidator
{
    public function validate(array $data): ValidationResult
    {
        // Только валидация
    }
}

// ✅ Dependency Inversion Principle
interface UserRepositoryInterface
{
    public function save(User $user): void;
}

class UserService
{
    public function __construct(
        private UserRepositoryInterface $repository
    ) {}
}
```

### 5. Явная обработка ошибок

**Принцип**: всегда обрабатывай ошибки явно и предсказуемо.

```javascript
// ❌ Плохо
async function fetchUser(id) {
    const response = await fetch(`/api/users/${id}`);
    return response.json(); // Может упасть без обработки ошибок
}

// ✅ Хорошо
async function fetchUser(id) {
    try {
        const response = await fetch(`/api/users/${id}`);
        
        if (!response.ok) {
            throw new Error(`HTTP ${response.status}: ${response.statusText}`);
        }
        
        return await response.json();
    } catch (error) {
        console.error('Failed to fetch user:', { id, error });
        throw new Error('USER_FETCH_FAILED');
    }
}
```

## Процессы разработки

### Git Flow и ветвление

Мы используем **GitFlow** с небольшими адаптациями:

#### Структура веток

```
master (main)     ← релизы
    ↑
develop          ← интеграция фич
    ↑
feature/JIRA-123 ← разработка
```

#### Правила ветвления

1. **Основные ветки**:
   - `master` — продакшн код, только релизы
   - `develop` — интеграция всех фич

2. **Ветки разработки**:
   - `feature/JIRA-123-краткое-описание` — новые фичи
   - `hotfix/JIRA-456-описание` — срочные исправления

3. **Правила именования веток**:
   ```
   feature/JIRA-123-add-user-registration
   feature/JIRA-124-fix-login-validation
   hotfix/JIRA-125-security-patch
   ```

#### Процесс разработки

1. **Создание задачи** → ветка `feature/JIRA-123-описание`
2. **Разработка** → коммиты с описанием изменений
3. **Merge Request** → `feature` → `develop`
4. **Релиз** → `develop` → `master`

### Правила коммитов

#### Формат коммитов

```
JIRA-123: что поделано 1
- что поделано 2
- что поделано 3
```

#### Примеры хороших коммитов

```
JIRA-123: добавить валидацию email при регистрации
- создать EmailValidator класс
- добавить тесты для валидации
- обновить форму регистрации

JIRA-124: исправить баг с отображением аватара
- исправить путь к изображению
- добавить fallback для отсутствующих аватаров

JIRA-125: оптимизировать загрузку списка пользователей
- добавить пагинацию
- оптимизировать SQL запрос
- добавить кэширование
```

#### Что НЕ делать в коммитах

```
❌ "fix"
❌ "update"
❌ "wip"
❌ "добавил фичу"
❌ "исправил баг"
```

### Merge Request (MR)

#### Правила создания MR

1. **Размер**: небольшие MR легче ревьюить
2. **Описание**: четко опиши что изменилось и зачем
3. **Тесты**: добавь тесты для новой функциональности
4. **Документация**: обнови README если нужно

#### Шаблон описания MR

```markdown
## Что изменилось
- Краткое описание изменений

## Зачем это нужно
- Обоснование изменений

## Что поделано
- [ ] Добавлена валидация email
- [ ] Написаны тесты
- [ ] Обновлена документация

## Тестирование
- [ ] Локально протестировано
- [ ] Автотесты проходят
- [ ] Проверено в dev окружении

## Скриншоты (если применимо)
[Добавь скриншоты UI изменений]

## Ссылки
- Задача: JIRA-123
- Документация: [ссылка на Confluence]
```

### Code Review

#### Правила ревью

1. **Обязательность**: каждый MR должен пройти ревью
2. **Время**: отвечай на ревью в течение 24 часов
3. **Тон**: конструктивная критика, без личных нападок
4. **Фокус**: на код, а не на автора

#### Что проверять в ревью

- [ ] Код соответствует стандартам команды
- [ ] Логика корректна и понятна
- [ ] Тесты покрывают изменения
- [ ] Нет дублирования кода
- [ ] Обработка ошибок
- [ ] Производительность
- [ ] Безопасность

## Документация

### Принципы документирования

1. **Документируй как можно раньше** — не откладывай на потом
2. **Пиши для других** — представь, что читаешь документацию через полгода
3. **Обновляй вместе с кодом** — документация должна быть актуальной
4. **Используй примеры** — код лучше слов

### Типы документации

#### 1. README проекта

Каждый проект должен иметь README.md в корне:

```markdown
# Название проекта

## Описание
Краткое описание что делает проект и зачем он нужен.

## Технологии
- Frontend: React 18, TypeScript
- Backend: Laravel 10, PHP 8.2
- Database: MySQL 8.0

## Установка и запуск

### Требования
- Node.js 18+
- PHP 8.2+
- MySQL 8.0+

### Установка
```bash
# Клонирование
git clone <repository-url>
cd project-name

# Backend
composer install
cp .env.example .env
php artisan key:generate
php artisan migrate

# Frontend
npm install
npm run dev
```

### Запуск
```bash
# Backend
php artisan serve

# Frontend
npm run dev
```

## Структура проекта
```
project/
├── app/           # Backend код
├── resources/     # Frontend код
├── database/      # Миграции и сидеры
├── tests/         # Тесты
└── docs/          # Документация
```

## API документация
- Swagger UI: http://localhost:8000/api/documentation
- Postman Collection: [ссылка]

## Разработка
- Ветка разработки: `develop`
- Стандарты кода: см. манифест команды
- Тесты: `npm test` / `php artisan test`
```

#### 2. Документация API

Используй OpenAPI/Swagger для документирования API:

```yaml
openapi: 3.0.0
info:
  title: User API
  version: 1.0.0
paths:
  /api/users:
    get:
      summary: Получить список пользователей
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            default: 1
      responses:
        '200':
          description: Успешно
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                    items:
                      $ref: '#/components/schemas/User'
```

#### 3. Архитектурная документация

Создавай ADR (Architecture Decision Records) для важных решений:

```markdown
# ADR-001: Выбор базы данных

## Статус
Принято

## Контекст
Нужно выбрать БД для нового проекта.

## Решение
Используем MySQL 8.0 для реляционных данных.

## Последствия
- ✅ Хорошая поддержка транзакций
- ✅ Знакомая команде
- ❌ Сложность масштабирования
```

### Где хранить документацию

1. **README.md** — основная информация о проекте
2. **docs/** — детальная документация
3. **Confluence** — проектная документация, процессы
4. **Комментарии в коде** — для сложной логики

### Правила комментирования кода

#### Когда комментировать

```javascript
// ✅ Комментируй сложную бизнес-логику
// Проверяем, что пользователь может редактировать пост
// только если он автор или модератор
const canEdit = post.authorId === userId || user.role === 'moderator';

// ✅ Комментируй неочевидные решения
// Используем setTimeout вместо setInterval для избежания
// накопления задержек при медленном выполнении
setTimeout(updateData, 5000);

// ❌ НЕ комментируй очевидное
const name = user.name; // имя пользователя
```

#### Формат комментариев

```javascript
/**
 * Создает новый пользователя с валидацией данных
 * @param {Object} userData - Данные пользователя
 * @param {string} userData.email - Email пользователя
 * @param {string} userData.name - Имя пользователя
 * @returns {Promise<User>} Созданный пользователь
 * @throws {ValidationError} Если данные некорректны
 */
async function createUser(userData) {
    // Валидация email
    if (!isValidEmail(userData.email)) {
        throw new ValidationError('Invalid email format');
    }
    
    // Создание пользователя
    const user = await User.create(userData);
    
    // Отправка приветственного письма
    await sendWelcomeEmail(user.email);
    
    return user;
}
```

## Качество кода

### Линтеры и форматирование

#### Frontend
```json
// .eslintrc.json
{
  "extends": [
    "eslint:recommended",
    "@typescript-eslint/recommended"
  ],
  "rules": {
    "no-console": "warn",
    "prefer-const": "error"
  }
}
```

#### Backend
```json
// .php-cs-fixer.php
<?php
return (new PhpCsFixer\Config())
    ->setRules([
        '@PSR12' => true,
        'strict_param' => true,
        'array_syntax' => ['syntax' => 'short'],
    ]);
```

### Тестирование

#### Принципы тестирования

1. **Покрытие**: минимум 70% для критических модулей
2. **Пирамида тестов**: много unit, меньше integration, мало e2e
3. **Изоляция**: тесты не должны зависеть друг от друга
4. **Читаемость**: тесты должны быть понятными

#### Примеры тестов

```javascript
// Unit тест
describe('UserService', () => {
  it('should create user with valid data', async () => {
    const userData = {
      email: 'test@example.com',
      name: 'Test User'
    };
    
    const user = await userService.create(userData);
    
    expect(user.email).toBe(userData.email);
    expect(user.name).toBe(userData.name);
  });
  
  it('should throw error for invalid email', async () => {
    const userData = {
      email: 'invalid-email',
      name: 'Test User'
    };
    
    await expect(userService.create(userData))
      .rejects
      .toThrow('Invalid email format');
  });
});
```

## Безопасность

### Основные принципы

1. **Не доверяй входным данным** — всегда валидируй
2. **Принцип наименьших привилегий** — давай минимум прав
3. **Защита в глубину** — несколько уровней защиты
4. **Регулярные обновления** — следи за уязвимостями

### Примеры безопасного кода

```php
// ✅ Безопасная валидация
public function store(Request $request)
{
    $validated = $request->validate([
        'email' => 'required|email|unique:users',
        'name' => 'required|string|max:255',
    ]);
    
    $user = User::create($validated);
    
    return response()->json($user, 201);
}

// ✅ Защита от SQL инъекций (Laravel делает это автоматически)
$users = User::where('email', $email)->get();

// ✅ Экранирование вывода
echo htmlspecialchars($user->name, ENT_QUOTES, 'UTF-8');
```

## Наблюдаемость

### Логирование

#### Структурированные логи

```javascript
// ✅ Хорошо
logger.info('User registered', {
  userId: user.id,
  email: user.email,
  source: 'web',
  timestamp: new Date().toISOString()
});

// ❌ Плохо
console.log('User registered: ' + user.email);
```

#### Обязательные поля

```javascript
const logContext = {
  traceId: req.headers['x-trace-id'],
  userId: req.user?.id,
  requestId: req.id,
  service: 'user-service',
  timestamp: new Date().toISOString()
};

logger.info('API request', {
  ...logContext,
  method: req.method,
  path: req.path,
  statusCode: res.statusCode
});
```

### Метрики

Отслеживай ключевые метрики:

- **Ошибки**: количество и типы ошибок
- **Латентность**: время ответа API
- **RPS**: запросы в секунду
- **Ресурсы**: CPU, память, диск

## Навигация

- **[Фронтенд](../frontend/README.md)** — JavaScript, TypeScript, React, Vue
- **[Бэкенд](../backend/README.md)** — PHP, Laravel
- **[Базы данных](../database/README.md)** — MongoDB, MySQL


