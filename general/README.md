## Общая информация

### Зачем нужен манифест
- **Единые стандарты**: ускоряет онбординг, снижает количество повторяющихся обсуждений.
- **Предсказуемость**: общие принципы архитектуры, качества и процессов.
- **Качество**: контроль технического долга и стабильные релизы.

> Описания конкретных проектов должны вестись в Confuence. Там находится проектная и архитектурная документации по продуктам и проектам.

### Стек и версии (целевые, но есть и legacy)
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

<details>
<summary><strong>S - Single Responsibility Principle (Принцип единственной ответственности)</strong></summary>

**Описание**: Класс должен иметь только одну причину для изменения.

#### Laravel пример

```php
// ❌ Плохо - класс делает слишком много
class UserManager
{
    public function createUser(array $data): User
    {
        // Валидация
        $this->validateUserData($data);
        
        // Создание пользователя
        $user = User::create($data);
        
        // Отправка email
        $this->sendWelcomeEmail($user);
        
        // Логирование
        $this->logUserCreation($user);
        
        return $user;
    }
}

// ✅ Хорошо - каждый класс отвечает за одну задачу
class UserValidator
{
    public function validate(array $data): ValidationResult
    {
        return Validator::make($data, [
            'email' => 'required|email|unique:users',
            'name' => 'required|string|max:255',
        ]);
    }
}

class UserService
{
    public function __construct(
        private UserValidator $validator,
        private EmailService $emailService,
        private Logger $logger
    ) {}

    public function createUser(array $data): User
    {
        $validation = $this->validator->validate($data);
        
        if ($validation->fails()) {
            throw new ValidationException($validation);
        }
        
        $user = User::create($data);
        
        $this->emailService->sendWelcomeEmail($user);
        $this->logger->log('user.created', ['user_id' => $user->id]);
        
        return $user;
    }
}
```

#### React пример

```jsx
// ❌ Плохо - компонент делает слишком много
const UserProfile = ({ user }) => {
    const [isEditing, setIsEditing] = useState(false);
    const [formData, setFormData] = useState(user);
    const [validationErrors, setValidationErrors] = useState({});
    
    const handleSubmit = async (e) => {
        e.preventDefault();
        
        // Валидация
        const errors = validateForm(formData);
        if (Object.keys(errors).length > 0) {
            setValidationErrors(errors);
            return;
        }
        
        // API запрос
        try {
            await updateUser(user.id, formData);
            setIsEditing(false);
        } catch (error) {
            console.error('Failed to update user:', error);
        }
    };
    
    return (
        <div>
            <h2>Профиль пользователя</h2>
            <form onSubmit={handleSubmit}>
                {/* Форма с валидацией */}
            </form>
        </div>
    );
};

// ✅ Хорошо - разделение ответственности
const UserProfile = ({ user }) => {
    const [isEditing, setIsEditing] = useState(false);
    
    return (
        <div>
            <h2>Профиль пользователя</h2>
            {isEditing ? (
                <UserEditForm 
                    user={user} 
                    onSave={() => setIsEditing(false)}
                    onCancel={() => setIsEditing(false)}
                />
            ) : (
                <UserProfileView 
                    user={user} 
                    onEdit={() => setIsEditing(true)}
                />
            )}
        </div>
    );
};

const UserEditForm = ({ user, onSave, onCancel }) => {
    const { formData, errors, handleSubmit, handleChange } = useUserForm(user);
    
    return (
        <form onSubmit={handleSubmit}>
            <FormField
                name="name"
                value={formData.name}
                onChange={handleChange}
                error={errors.name}
            />
            <FormField
                name="email"
                value={formData.email}
                onChange={handleChange}
                error={errors.email}
            />
            <button type="submit">Сохранить</button>
            <button type="button" onClick={onCancel}>Отмена</button>
        </form>
    );
};
```

</details>

<details>
<summary><strong>O - Open/Closed Principle (Принцип открытости/закрытости)</strong></summary>

**Описание**: Программные сущности должны быть открыты для расширения, но закрыты для модификации.

#### Laravel пример

```php
// ❌ Плохо - нужно изменять код при добавлении новых типов
class PaymentProcessor
{
    public function processPayment(string $type, float $amount): bool
    {
        if ($type === 'credit_card') {
            return $this->processCreditCard($amount);
        } elseif ($type === 'paypal') {
            return $this->processPayPal($amount);
        } elseif ($type === 'bank_transfer') {
            return $this->processBankTransfer($amount);
        }
        
        throw new InvalidPaymentTypeException($type);
    }
}

// ✅ Хорошо - используем стратегии
interface PaymentMethodInterface
{
    public function process(float $amount): bool;
}

class CreditCardPayment implements PaymentMethodInterface
{
    public function process(float $amount): bool
    {
        // Логика обработки кредитной карты
        return true;
    }
}

class PayPalPayment implements PaymentMethodInterface
{
    public function process(float $amount): bool
    {
        // Логика обработки PayPal
        return true;
    }
}

class PaymentProcessor
{
    private array $paymentMethods = [];
    
    public function registerPaymentMethod(string $type, PaymentMethodInterface $method): void
    {
        $this->paymentMethods[$type] = $method;
    }
    
    public function processPayment(string $type, float $amount): bool
    {
        if (!isset($this->paymentMethods[$type])) {
            throw new InvalidPaymentTypeException($type);
        }
        
        return $this->paymentMethods[$type]->process($amount);
    }
}

// Использование
$processor = new PaymentProcessor();
$processor->registerPaymentMethod('credit_card', new CreditCardPayment());
$processor->registerPaymentMethod('paypal', new PayPalPayment());
```

#### React пример

```jsx
// ❌ Плохо - нужно изменять компонент при добавлении новых типов
const NotificationBanner = ({ type, message }) => {
    if (type === 'success') {
        return <div className="bg-green-100 text-green-800">{message}</div>;
    } else if (type === 'error') {
        return <div className="bg-red-100 text-red-800">{message}</div>;
    } else if (type === 'warning') {
        return <div className="bg-yellow-100 text-yellow-800">{message}</div>;
    }
    
    return <div className="bg-gray-100 text-gray-800">{message}</div>;
};

// ✅ Хорошо - используем компонентную стратегию
const notificationTypes = {
    success: ({ message }) => (
        <div className="bg-green-100 text-green-800 p-4 rounded">
            <CheckIcon className="w-5 h-5 inline mr-2" />
            {message}
        </div>
    ),
    error: ({ message }) => (
        <div className="bg-red-100 text-red-800 p-4 rounded">
            <XIcon className="w-5 h-5 inline mr-2" />
            {message}
        </div>
    ),
    warning: ({ message }) => (
        <div className="bg-yellow-100 text-yellow-800 p-4 rounded">
            <ExclamationIcon className="w-5 h-5 inline mr-2" />
            {message}
        </div>
    ),
    info: ({ message }) => (
        <div className="bg-blue-100 text-blue-800 p-4 rounded">
            <InformationCircleIcon className="w-5 h-5 inline mr-2" />
            {message}
        </div>
    )
};

const NotificationBanner = ({ type, message }) => {
    const NotificationComponent = notificationTypes[type] || notificationTypes.info;
    return <NotificationComponent message={message} />;
};
```

</details>

<details>
<summary><strong>L - Liskov Substitution Principle (Принцип подстановки Лисков)</strong></summary>

**Описание**: Объекты в программе могут быть заменены их наследниками без изменения корректности программы.

#### Laravel пример

```php
// ❌ Плохо - нарушение LSP
class Bird
{
    public function fly(): void
    {
        // Логика полета
    }
}

class Penguin extends Bird
{
    public function fly(): void
    {
        throw new Exception('Пингвины не летают!');
    }
}

// ✅ Хорошо - правильная иерархия
abstract class Bird
{
    abstract public function move(): void;
}

class FlyingBird extends Bird
{
    public function move(): void
    {
        // Логика полета
    }
}

class WalkingBird extends Bird
{
    public function move(): void
    {
        // Логика ходьбы
    }
}

class Sparrow extends FlyingBird
{
    public function move(): void
    {
        // Специфичная логика полета воробья
    }
}

class Penguin extends WalkingBird
{
    public function move(): void
    {
        // Специфичная логика ходьбы пингвина
    }
}

// Использование - любой Bird можно заменить на его подкласс
function makeBirdMove(Bird $bird): void
{
    $bird->move(); // Работает для любого типа птицы
}
```

#### React пример

```jsx
// ❌ Плохо - нарушение LSP
const Button = ({ onClick, children, ...props }) => (
    <button onClick={onClick} {...props}>
        {children}
    </button>
);

const DisabledButton = ({ onClick, children, ...props }) => (
    <button disabled {...props}>
        {children}
    </button>
    // onClick игнорируется, что нарушает контракт
);

// ✅ Хорошо - правильная подстановка
const Button = ({ onClick, children, disabled = false, ...props }) => (
    <button 
        onClick={disabled ? undefined : onClick} 
        disabled={disabled}
        {...props}
    >
        {children}
    </button>
);

const DisabledButton = ({ children, ...props }) => (
    <Button disabled={true} {...props}>
        {children}
    </Button>
);

// Использование - компоненты взаимозаменяемы
const App = () => {
    const handleClick = () => console.log('Clicked!');
    
    return (
        <div>
            <Button onClick={handleClick}>Обычная кнопка</Button>
            <DisabledButton onClick={handleClick}>Отключенная кнопка</DisabledButton>
        </div>
    );
};
```

</details>

<details>
<summary><strong>I - Interface Segregation Principle (Принцип разделения интерфейса)</strong></summary>

**Описание**: Клиенты не должны зависеть от методов, которые они не используют.

#### Laravel пример

```php
// ❌ Плохо - толстый интерфейс
interface WorkerInterface
{
    public function work(): void;
    public function eat(): void;
    public function sleep(): void;
    public function getSalary(): float;
    public function takeVacation(): void;
}

class Robot implements WorkerInterface
{
    public function work(): void
    {
        // Робот работает
    }
    
    public function eat(): void
    {
        throw new Exception('Роботы не едят!');
    }
    
    public function sleep(): void
    {
        throw new Exception('Роботы не спят!');
    }
    
    public function getSalary(): float
    {
        throw new Exception('Роботы не получают зарплату!');
    }
    
    public function takeVacation(): void
    {
        throw new Exception('Роботы не берут отпуск!');
    }
}

// ✅ Хорошо - разделенные интерфейсы
interface WorkableInterface
{
    public function work(): void;
}

interface EatableInterface
{
    public function eat(): void;
}

interface SleepableInterface
{
    public function sleep(): void;
}

interface PayableInterface
{
    public function getSalary(): float;
}

interface VacationableInterface
{
    public function takeVacation(): void;
}

class Human implements 
    WorkableInterface, 
    EatableInterface, 
    SleepableInterface, 
    PayableInterface, 
    VacationableInterface
{
    public function work(): void { /* ... */ }
    public function eat(): void { /* ... */ }
    public function sleep(): void { /* ... */ }
    public function getSalary(): float { /* ... */ }
    public function takeVacation(): void { /* ... */ }
}

class Robot implements WorkableInterface
{
    public function work(): void { /* ... */ }
}
```

#### React пример

```jsx
// ❌ Плохо - толстый интерфейс пропсов
const UserCard = ({ 
    user, 
    onEdit, 
    onDelete, 
    onShare, 
    onFollow, 
    onBlock,
    showActions = true,
    showStats = true,
    showAvatar = true,
    showBio = true
}) => {
    return (
        <div className="user-card">
            {showAvatar && <UserAvatar user={user} />}
            <UserInfo user={user} showBio={showBio} />
            {showStats && <UserStats user={user} />}
            {showActions && (
                <UserActions 
                    onEdit={onEdit}
                    onDelete={onDelete}
                    onShare={onShare}
                    onFollow={onFollow}
                    onBlock={onBlock}
                />
            )}
        </div>
    );
};

// ✅ Хорошо - разделенные компоненты
const UserCard = ({ user, children }) => (
    <div className="user-card">
        {children}
    </div>
);

const UserAvatar = ({ user }) => (
    <img src={user.avatar} alt={user.name} className="avatar" />
);

const UserInfo = ({ user, showBio = false }) => (
    <div className="user-info">
        <h3>{user.name}</h3>
        {showBio && <p>{user.bio}</p>}
    </div>
);

const UserStats = ({ user }) => (
    <div className="user-stats">
        <span>Постов: {user.postsCount}</span>
        <span>Подписчиков: {user.followersCount}</span>
    </div>
);

const UserActions = ({ onEdit, onDelete, onShare, onFollow, onBlock }) => (
    <div className="user-actions">
        {onEdit && <button onClick={onEdit}>Редактировать</button>}
        {onDelete && <button onClick={onDelete}>Удалить</button>}
        {onShare && <button onClick={onShare}>Поделиться</button>}
        {onFollow && <button onClick={onFollow}>Подписаться</button>}
        {onBlock && <button onClick={onBlock}>Заблокировать</button>}
    </div>
);

// Использование - только нужные части
const SimpleUserCard = ({ user }) => (
    <UserCard user={user}>
        <UserAvatar user={user} />
        <UserInfo user={user} />
    </UserCard>
);

const FullUserCard = ({ user, onEdit, onDelete }) => (
    <UserCard user={user}>
        <UserAvatar user={user} />
        <UserInfo user={user} showBio={true} />
        <UserStats user={user} />
        <UserActions onEdit={onEdit} onDelete={onDelete} />
    </UserCard>
);
```

</details>

<details>
<summary><strong>D - Dependency Inversion Principle (Принцип инверсии зависимостей)</strong></summary>

**Описание**: Зависимости должны строиться на абстракциях, а не на конкретных классах.

#### Laravel пример

```php
// ❌ Плохо - зависимость от конкретного класса
class UserService
{
    private UserRepository $repository;
    
    public function __construct()
    {
        $this->repository = new UserRepository();
    }
    
    public function createUser(array $data): User
    {
        return $this->repository->create($data);
    }
}

// ✅ Хорошо - зависимость от интерфейса
interface UserRepositoryInterface
{
    public function create(array $data): User;
    public function find(int $id): ?User;
    public function update(int $id, array $data): User;
    public function delete(int $id): bool;
}

class UserRepository implements UserRepositoryInterface
{
    public function create(array $data): User
    {
        return User::create($data);
    }
    
    public function find(int $id): ?User
    {
        return User::find($id);
    }
    
    public function update(int $id, array $data): User
    {
        $user = User::findOrFail($id);
        $user->update($data);
        return $user;
    }
    
    public function delete(int $id): bool
    {
        return User::destroy($id) > 0;
    }
}

class UserService
{
    public function __construct(
        private UserRepositoryInterface $repository
    ) {}
    
    public function createUser(array $data): User
    {
        return $this->repository->create($data);
    }
}

// Регистрация в сервис-контейнере
class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->app->bind(UserRepositoryInterface::class, UserRepository::class);
    }
}
```

#### React пример

```jsx
// ❌ Плохо - зависимость от конкретной реализации
const UserList = () => {
    const [users, setUsers] = useState([]);
    
    useEffect(() => {
        // Прямая зависимость от fetch API
        fetch('/api/users')
            .then(response => response.json())
            .then(data => setUsers(data));
    }, []);
    
    return (
        <div>
            {users.map(user => (
                <UserCard key={user.id} user={user} />
            ))}
        </div>
    );
};

// ✅ Хорошо - зависимость от абстракции
// Интерфейс для работы с данными
const useUserService = () => {
    const fetchUsers = async () => {
        const response = await fetch('/api/users');
        return response.json();
    };
    
    const createUser = async (userData) => {
        const response = await fetch('/api/users', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(userData)
        });
        return response.json();
    };
    
    return { fetchUsers, createUser };
};

// Хук для управления состоянием пользователей
const useUsers = (userService) => {
    const [users, setUsers] = useState([]);
    const [loading, setLoading] = useState(false);
    const [error, setError] = useState(null);
    
    const loadUsers = async () => {
        setLoading(true);
        try {
            const data = await userService.fetchUsers();
            setUsers(data);
        } catch (err) {
            setError(err.message);
        } finally {
            setLoading(false);
        }
    };
    
    const addUser = async (userData) => {
        try {
            const newUser = await userService.createUser(userData);
            setUsers(prev => [...prev, newUser]);
        } catch (err) {
            setError(err.message);
        }
    };
    
    return { users, loading, error, loadUsers, addUser };
};

// Компонент с инверсией зависимостей
const UserList = ({ userService }) => {
    const { users, loading, error, loadUsers } = useUsers(userService);
    
    useEffect(() => {
        loadUsers();
    }, []);
    
    if (loading) return <div>Загрузка...</div>;
    if (error) return <div>Ошибка: {error}</div>;
    
    return (
        <div>
            {users.map(user => (
                <UserCard key={user.id} user={user} />
            ))}
        </div>
    );
};

// Использование
const App = () => {
    const userService = useUserService();
    
    return <UserList userService={userService} />;
};
```

</details>

### 5. Явная обработка ошибок

**Принцип**: всегда обрабатывай ошибки явно и предсказуемо.

#### Общие принципы

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

#### Лучшие практики логирования ошибок на фронтенде

**Когда использовать `console.error`:**

```javascript
// ✅ Хорошо - для разработки и отладки
const handleUserFetch = async (id) => {
    try {
        const user = await fetchUser(id);
        return user;
    } catch (error) {
        // Логируем для разработчиков в dev окружении
        if (process.env.NODE_ENV === 'development') {
            console.error('Failed to fetch user:', { 
                id, 
                error: error.message,
                stack: error.stack,
                timestamp: new Date().toISOString()
            });
        }
        
        // Отправляем в систему мониторинга
        logErrorToMonitoring(error, { userId: id, action: 'fetch_user' });
        
        // Показываем пользователю понятное сообщение
        showUserFriendlyError('Не удалось загрузить данные пользователя');
        
        throw error;
    }
};
```

**Структурированное логирование:**

```javascript
// ✅ Хорошо - структурированные логи
const logger = {
    error: (message, context = {}) => {
        const logData = {
            level: 'error',
            message,
            timestamp: new Date().toISOString(),
            userAgent: navigator.userAgent,
            url: window.location.href,
            ...context
        };
        
        // В продакшене отправляем в систему мониторинга
        if (process.env.NODE_ENV === 'production') {
            sendToErrorTracking(logData);
        } else {
            // В разработке выводим в консоль
            console.error('🚨 Error:', logData);
        }
    },
    
    warn: (message, context = {}) => {
        console.warn('⚠️ Warning:', { message, ...context });
    },
    
    info: (message, context = {}) => {
        console.info('ℹ️ Info:', { message, ...context });
    }
};

// Использование
try {
    await riskyOperation();
} catch (error) {
    logger.error('Operation failed', {
        operation: 'riskyOperation',
        error: error.message,
        userId: currentUser?.id
    });
}
```

**Обработка ошибок в React компонентах:**

```jsx
// ✅ Хорошо - Error Boundary для компонентов
class ErrorBoundary extends React.Component {
    constructor(props) {
        super(props);
        this.state = { hasError: false, error: null };
    }

    static getDerivedStateFromError(error) {
        return { hasError: true, error };
    }

    componentDidCatch(error, errorInfo) {
        // Логируем ошибку в систему мониторинга
        logger.error('React Error Boundary caught error', {
            error: error.message,
            componentStack: errorInfo.componentStack,
            errorBoundary: this.constructor.name
        });
    }

    render() {
        if (this.state.hasError) {
            return <ErrorFallback error={this.state.error} />;
        }

        return this.props.children;
    }
}

// ✅ Хорошо - хуки для обработки ошибок
const useAsyncOperation = (asyncFn) => {
    const [state, setState] = useState({
        data: null,
        loading: false,
        error: null
    });

    const execute = useCallback(async (...args) => {
        setState(prev => ({ ...prev, loading: true, error: null }));
        
        try {
            const data = await asyncFn(...args);
            setState({ data, loading: false, error: null });
            return data;
        } catch (error) {
            // Логируем ошибку
            logger.error('Async operation failed', {
                operation: asyncFn.name,
                error: error.message,
                args
            });
            
            setState({ data: null, loading: false, error });
            throw error;
        }
    }, [asyncFn]);

    return { ...state, execute };
};

// Использование в компоненте
const UserProfile = ({ userId }) => {
    const { data: user, loading, error, execute: fetchUser } = useAsyncOperation(fetchUserById);

    useEffect(() => {
        fetchUser(userId);
    }, [userId]);

    if (loading) return <Spinner />;
    if (error) return <ErrorMessage error={error} onRetry={() => fetchUser(userId)} />;
    if (!user) return null;

    return <UserCard user={user} />;
};
```

**Обработка API ошибок:**

```javascript
// ✅ Хорошо - централизованная обработка API ошибок
class ApiError extends Error {
    constructor(message, status, code, details = {}) {
        super(message);
        this.name = 'ApiError';
        this.status = status;
        this.code = code;
        this.details = details;
    }
}

const apiClient = {
    async request(url, options = {}) {
        try {
            const response = await fetch(url, {
                headers: {
                    'Content-Type': 'application/json',
                    ...options.headers
                },
                ...options
            });

            if (!response.ok) {
                const errorData = await response.json().catch(() => ({}));
                
                throw new ApiError(
                    errorData.message || `HTTP ${response.status}`,
                    response.status,
                    errorData.code || 'UNKNOWN_ERROR',
                    errorData
                );
            }

            return await response.json();
        } catch (error) {
            // Логируем только неожиданные ошибки
            if (!(error instanceof ApiError)) {
                logger.error('Unexpected API error', {
                    url,
                    error: error.message,
                    stack: error.stack
                });
            }
            
            throw error;
        }
    }
};

// Обработка в компонентах
const useApi = () => {
    const handleApiError = useCallback((error) => {
        if (error instanceof ApiError) {
            switch (error.status) {
                case 401:
                    // Перенаправляем на логин
                    redirectToLogin();
                    break;
                case 403:
                    showUserFriendlyError('У вас нет прав для выполнения этого действия');
                    break;
                case 404:
                    showUserFriendlyError('Запрашиваемый ресурс не найден');
                    break;
                case 500:
                    showUserFriendlyError('Произошла внутренняя ошибка сервера');
                    break;
                default:
                    showUserFriendlyError('Произошла ошибка при выполнении запроса');
            }
        } else {
            showUserFriendlyError('Произошла непредвиденная ошибка');
        }
    }, []);

    return { apiClient, handleApiError };
};
```

**Что НЕ делать:**

```javascript
// ❌ Плохо - логирование всего подряд
try {
    await fetchUser(id);
} catch (error) {
    console.error('Error:', error); // Слишком общая информация
    console.log('User ID:', id); // console.log для ошибок
    alert('Error!'); // Плохой UX
}

// ❌ Плохо - игнорирование ошибок
try {
    await fetchUser(id);
} catch (error) {
    // Ничего не делаем - ошибка "исчезает"
}

// ❌ Плохо - логирование в продакшене
if (process.env.NODE_ENV === 'production') {
    console.error('Production error:', error); // Засоряет консоль пользователей
}
```

**Рекомендации по логированию:**

1. **В разработке**: используй `console.error` для отладки
2. **В продакшене**: отправляй ошибки в систему мониторинга (Sentry, LogRocket, etc.)
3. **Структурируй данные**: включай контекст (userId, action, timestamp)
4. **Не засоряй консоль**: избегай избыточного логирования
5. **Показывай пользователю**: понятные сообщения об ошибках
6. **Логируй только важное**: не логируй ожидаемые ошибки (404, 401)

## Процессы разработки

### Git Flow и ветвление

Мы используем **GitFlow** с небольшими адаптациями. Полезные материалы в Confluence:

- [Правила оформления README в репозитории для разработчика](https://confluence.eltc.ru/pages/viewpage.action?pageId=156337121)
- [Правила комментирования работы в системе контроля версий](https://confluence.eltc.ru/pages/viewpage.action?pageId=151716241)
- [Правила ветвления в Git-репозитории (GitFlow)](https://confluence.eltc.ru/pages/viewpage.action?pageId=242522134)


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
   - `feature/JIRA-123-<краткое-описание опционально>` — новые фичи
   - `hotfix/JIRA-456-<краткое-описание опционально>` — срочные исправления

3. **Правила именования веток**:
   ```
   feature/JIRA-123
   feature/JIRA-124-починил-валидацию логина
   hotfix/JIRA-125-изменил-отображение-личного-кабинета
   ```

#### Процесс разработки

1. **Создание задачи** → ветка `feature/JIRA-123-<краткое-описание опционально>`
2. **Разработка** → коммиты с описанием изменений
3. **Merge Request** → `feature` → `develop`
4. **Релиз** → `develop` → `master`

### Правила коммитов

#### Формат коммитов

```
JIRA-123: 
- что поделано 1
- что поделано 2
- что поделано 3
```

#### Примеры хороших коммитов

```
JIRA-123: 
- создал EmailValidator класс
- добавил тесты для валидации
- обновил форму регистрации

JIRA-124: 
- исправил баг с отображением аватара
- исправил путь к изображению
- добавил fallback для отсутствующих аватаров

JIRA-125: 
- добавил пагинацию
- оптимизировал SQL запрос
- добавил кэширование
```

#### Что НЕ писать в коммитах

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
 * Создает нового пользователя с валидацией данных
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


