# Бэкенд разработка

## Зачем нужны эти принципы?

Бэкенд — это сердце приложения. Качественный бэкенд должен быть:

- **Надежным** — обрабатывать ошибки и не падать
- **Безопасным** — защищать данные и API
- **Производительным** — быстро обрабатывать запросы
- **Масштабируемым** — расти вместе с нагрузкой
- **Поддерживаемым** — легко развивать и исправлять

## Основные принципы бэкенд разработки

### 1. Чистая архитектура

**Принцип**: разделяй код на слои с четкими границами ответственности.

```php
// ✅ Хорошая архитектура — разделение на слои
// Контроллер (слой представления)
final class UserController extends Controller
{
    public function __construct(
        private readonly UserService $userService,
        private readonly UserValidator $validator
    ) {}

    public function store(CreateUserRequest $request): JsonResponse
    {
        $userData = $request->validated();
        
        $user = $this->userService->createUser($userData);
        
        return response()->json(
            new UserResource($user),
            Response::HTTP_CREATED
        );
    }
}

// Сервис (бизнес-логика)
final class UserService
{
    public function __construct(
        private readonly UserRepository $repository,
        private readonly EmailService $emailService
    ) {}

    public function createUser(array $data): User
    {
        // Бизнес-логика создания пользователя
        $user = $this->repository->create($data);
        
        // Отправка приветственного письма
        $this->emailService->sendWelcomeEmail($user->email);
        
        return $user;
    }
}

// Репозиторий (доступ к данным)
final class UserRepository
{
    public function create(array $data): User
    {
        return User::create($data);
    }
}
```

### 2. Валидация на всех уровнях

**Принцип**: валидируй данные на входе, в бизнес-логике и на выходе.

```php
// ✅ Валидация на уровне запроса
final class CreateUserRequest extends FormRequest
{
    public function rules(): array
    {
        return [
            'name' => ['required', 'string', 'max:255'],
            'email' => ['required', 'email', 'unique:users,email'],
            'password' => ['required', 'string', 'min:8', 'confirmed'],
            'role' => ['sometimes', 'string', 'in:user,admin,moderator'],
        ];
    }

    public function messages(): array
    {
        return [
            'name.required' => 'Имя обязательно для заполнения',
            'email.unique' => 'Пользователь с таким email уже существует',
            'password.min' => 'Пароль должен содержать минимум 8 символов',
        ];
    }
}

// ✅ Валидация в бизнес-логике
final class UserValidator
{
    public function validateUserData(array $data): ValidationResult
    {
        $errors = [];

        // Проверка бизнес-правил
        if ($this->isReservedEmail($data['email'])) {
            $errors['email'] = 'Этот email зарезервирован';
        }

        if ($this->isWeakPassword($data['password'])) {
            $errors['password'] = 'Пароль слишком простой';
        }

        return new ValidationResult(
            isValid: empty($errors),
            errors: $errors
        );
    }

    private function isReservedEmail(string $email): bool
    {
        $reserved = ['admin@example.com', 'test@example.com'];
        return in_array($email, $reserved);
    }

    private function isWeakPassword(string $password): bool
    {
        return !preg_match('/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d).+$/', $password);
    }
}
```

### 3. Обработка ошибок

**Принцип**: обрабатывай ошибки централизованно и предсказуемо.

```php
// ✅ Централизованная обработка ошибок
final class ApiExceptionHandler extends ExceptionHandler
{
    public function register(): void
    {
        $this->reportable(function (Throwable $e) {
            // Логирование ошибок
            Log::error('API Error', [
                'message' => $e->getMessage(),
                'file' => $e->getFile(),
                'line' => $e->getLine(),
                'trace' => $e->getTraceAsString(),
            ]);
        });

        // Маппинг исключений на HTTP коды
        $this->renderable(function (ValidationException $e) {
            return response()->json([
                'message' => 'Validation failed',
                'errors' => $e->errors(),
            ], Response::HTTP_UNPROCESSABLE_ENTITY);
        });

        $this->renderable(function (ModelNotFoundException $e) {
            return response()->json([
                'message' => 'Resource not found',
            ], Response::HTTP_NOT_FOUND);
        });
    }
}

// ✅ Кастомные исключения
final class UserNotFoundException extends Exception
{
    public function __construct(string $userId)
    {
        parent::__construct("User with ID {$userId} not found");
    }
}

final class InsufficientPermissionsException extends Exception
{
    public function __construct(string $action)
    {
        parent::__construct("Insufficient permissions for action: {$action}");
    }
}

// ✅ Использование в сервисах
final class UserService
{
    public function getUserById(string $userId): User
    {
        $user = $this->repository->findById($userId);
        
        if (!$user) {
            throw new UserNotFoundException($userId);
        }
        
        return $user;
    }

    public function deleteUser(string $userId, User $currentUser): void
    {
        $user = $this->getUserById($userId);
        
        if (!$currentUser->canDelete($user)) {
            throw new InsufficientPermissionsException('delete_user');
        }
        
        $this->repository->delete($user);
    }
}
```

## Laravel — современные практики

### 1. Использование современных возможностей PHP 8.2+

```php
// ✅ Используй современный синтаксис PHP
final class UserService
{
    public function __construct(
        private readonly UserRepository $repository,
        private readonly EmailService $emailService,
        private readonly LoggerInterface $logger,
    ) {}

    public function createUser(
        string $name,
        string $email,
        string $password,
        ?string $role = 'user',
    ): User {
        $user = $this->repository->create([
            'name' => $name,
            'email' => $email,
            'password' => Hash::make($password),
            'role' => $role,
        ]);

        $this->logger->info('User created', [
            'user_id' => $user->id,
            'email' => $user->email,
        ]);

        return $user;
    }

    public function updateUser(
        User $user,
        array $data,
    ): User {
        $user->update($data);

        $this->logger->info('User updated', [
            'user_id' => $user->id,
            'changes' => $data,
        ]);

        return $user;
    }
}

// ✅ Использование match выражений
final class UserStatus
{
    public function getStatusText(string $status): string
    {
        return match ($status) {
            'active' => 'Активный',
            'inactive' => 'Неактивный',
            'suspended' => 'Приостановлен',
            'deleted' => 'Удален',
            default => throw new InvalidArgumentException("Unknown status: {$status}"),
        };
    }
}

// ✅ Использование named arguments
$user = $userService->createUser(
    name: 'John Doe',
    email: 'john@example.com',
    password: 'secure_password',
    role: 'admin',
);
```

### 2. Eloquent модели и отношения

```php
// ✅ Хорошо структурированная модель
final class User extends Authenticatable
{
    use HasFactory, HasApiTokens, Notifiable;

    protected $fillable = [
        'name',
        'email',
        'password',
        'role',
        'email_verified_at',
    ];

    protected $hidden = [
        'password',
        'remember_token',
    ];

    protected $casts = [
        'email_verified_at' => 'datetime',
        'settings' => 'array',
        'is_active' => 'boolean',
    ];

    // Отношения
    public function posts(): HasMany
    {
        return $this->hasMany(Post::class);
    }

    public function profile(): HasOne
    {
        return $this->hasOne(UserProfile::class);
    }

    public function roles(): BelongsToMany
    {
        return $this->belongsToMany(Role::class);
    }

    // Аксессоры
    public function getFullNameAttribute(): string
    {
        return "{$this->first_name} {$this->last_name}";
    }

    public function getIsAdminAttribute(): bool
    {
        return $this->role === 'admin';
    }

    // Мутаторы
    public function setEmailAttribute(string $value): void
    {
        $this->attributes['email'] = strtolower($value);
    }

    // Скоупы
    public function scopeActive(Builder $query): void
    {
        $query->where('is_active', true);
    }

    public function scopeByRole(Builder $query, string $role): void
    {
        $query->where('role', $role);
    }

    // Методы
    public function canDelete(User $targetUser): bool
    {
        return $this->isAdmin || $this->id === $targetUser->id;
    }

    public function hasPermission(string $permission): bool
    {
        return $this->roles()
            ->whereHas('permissions', fn($q) => $q->where('name', $permission))
            ->exists();
    }
}
```

### 3. API Resources для сериализации

```php
// ✅ API Resources для контролируемой сериализации
final class UserResource extends JsonResource
{
    public function toArray($request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->when($this->isAdmin(), $this->email),
            'role' => $this->role,
            'created_at' => $this->created_at?->toISOString(),
            'updated_at' => $this->updated_at?->toISOString(),
            
            // Условные поля
            'profile' => $this->whenLoaded('profile', fn() => 
                new UserProfileResource($this->profile)
            ),
            'posts_count' => $this->whenCounted('posts'),
            
            // Дополнительные поля
            'permissions' => $this->when($request->user()?->isAdmin(), 
                fn() => $this->roles->pluck('permissions')->flatten()->pluck('name')
            ),
        ];
    }
}

// ✅ Коллекция ресурсов
final class UserCollection extends ResourceCollection
{
    public function toArray($request): array
    {
        return [
            'data' => $this->collection,
            'meta' => [
                'total' => $this->total(),
                'per_page' => $this->perPage(),
                'current_page' => $this->currentPage(),
                'last_page' => $this->lastPage(),
            ],
            'links' => [
                'first' => $this->url(1),
                'last' => $this->url($this->lastPage()),
                'prev' => $this->previousPageUrl(),
                'next' => $this->nextPageUrl(),
            ],
        ];
    }
}
```

### 4. Middleware для аутентификации и авторизации

```php
// ✅ Кастомный middleware для проверки ролей
final class CheckRole
{
    public function handle(Request $request, Closure $next, string ...$roles): Response
    {
        $user = $request->user();
        
        if (!$user) {
            return response()->json(['message' => 'Unauthorized'], 401);
        }
        
        if (!in_array($user->role, $roles)) {
            return response()->json(['message' => 'Insufficient permissions'], 403);
        }
        
        return $next($request);
    }
}

// ✅ Middleware для логирования запросов
final class LogApiRequests
{
    public function handle(Request $request, Closure $next): Response
    {
        $startTime = microtime(true);
        
        $response = $next($request);
        
        $duration = microtime(true) - $startTime;
        
        Log::info('API Request', [
            'method' => $request->method(),
            'url' => $request->fullUrl(),
            'user_id' => $request->user()?->id,
            'status_code' => $response->getStatusCode(),
            'duration' => round($duration * 1000, 2), // в миллисекундах
        ]);
        
        return $response;
    }
}

// ✅ Регистрация middleware
final class Kernel extends HttpKernel
{
    protected $middlewareGroups = [
        'api' => [
            \App\Http\Middleware\LogApiRequests::class,
            'throttle:api',
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
        ],
    ];

    protected $routeMiddleware = [
        'role' => \App\Http\Middleware\CheckRole::class,
        'permission' => \App\Http\Middleware\CheckPermission::class,
    ];
}
```

## Производительность и оптимизация

### 1. Оптимизация запросов к БД

```php
// ✅ Избегай N+1 проблем
// ❌ Плохо — N+1 запросов
$users = User::all();
foreach ($users as $user) {
    echo $user->posts->count(); // Дополнительный запрос для каждого пользователя
}

// ✅ Хорошо — eager loading
$users = User::with('posts')->get();
foreach ($users as $user) {
    echo $users->posts->count(); // Данные уже загружены
}

// ✅ Условный eager loading
$users = User::when($request->has('with_posts'), function ($query) {
    $query->with('posts');
})->get();

// ✅ Загрузка с подсчетом
$users = User::withCount('posts')->get();
foreach ($users as $user) {
    echo $user->posts_count; // Подсчет уже выполнен
}

// ✅ Загрузка с условиями
$users = User::with(['posts' => function ($query) {
    $query->where('published', true);
}])->get();
```

### 2. Кэширование

```php
// ✅ Кэширование запросов
final class UserService
{
    public function getActiveUsers(): Collection
    {
        return Cache::remember('active_users', 3600, function () {
            return User::active()->with('profile')->get();
        });
    }

    public function getUserById(string $userId): ?User
    {
        return Cache::remember("user_{$userId}", 1800, function () use ($userId) {
            return User::with('profile', 'roles')->find($userId);
        });
    }

    public function updateUser(User $user, array $data): User
    {
        $user->update($data);
        
        // Инвалидация кэша
        Cache::forget("user_{$user->id}");
        Cache::forget('active_users');
        
        return $user;
    }
}

// ✅ Кэширование API ответов
final class UserController extends Controller
{
    public function index(Request $request): JsonResponse
    {
        $cacheKey = 'users_' . md5($request->fullUrl());
        
        return Cache::remember($cacheKey, 300, function () use ($request) {
            $users = User::query()
                ->when($request->search, fn($q, $search) => 
                    $q->where('name', 'like', "%{$search}%")
                )
                ->when($request->role, fn($q, $role) => 
                    $q->where('role', $role)
                )
                ->paginate($request->per_page ?? 15);
            
            return new UserCollection($users);
        });
    }
}
```

### 3. Очереди и фоновые задачи

```php
// ✅ Очереди для тяжелых операций
final class SendWelcomeEmail implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public function __construct(
        private readonly User $user
    ) {}

    public function handle(EmailService $emailService): void
    {
        $emailService->sendWelcomeEmail($this->user->email);
    }

    public function failed(Throwable $exception): void
    {
        Log::error('Failed to send welcome email', [
            'user_id' => $this->user->id,
            'error' => $exception->getMessage(),
        ]);
    }
}

// ✅ Использование в сервисах
final class UserService
{
    public function createUser(array $data): User
    {
        $user = $this->repository->create($data);
        
        // Отправка email в фоне
        SendWelcomeEmail::dispatch($user);
        
        // Создание профиля в фоне
        CreateUserProfile::dispatch($user);
        
        return $user;
    }
}

// ✅ Batch обработка
final class ProcessUsersBatch implements ShouldQueue
{
    use Batchable, Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public function __construct(
        private readonly array $userIds
    ) {}

    public function handle(): void
    {
        foreach ($this->userIds as $userId) {
            ProcessUser::dispatch($userId);
        }
    }
}
```

## Тестирование

### 1. Unit тесты

```php
// ✅ Тестирование сервисов
final class UserServiceTest extends TestCase
{
    use RefreshDatabase;

    private UserService $userService;
    private MockObject $userRepository;
    private MockObject $emailService;

    protected function setUp(): void
    {
        parent::setUp();
        
        $this->userRepository = $this->createMock(UserRepository::class);
        $this->emailService = $this->createMock(EmailService::class);
        
        $this->userService = new UserService(
            $this->userRepository,
            $this->emailService
        );
    }

    public function test_creates_user_successfully(): void
    {
        // Arrange
        $userData = [
            'name' => 'John Doe',
            'email' => 'john@example.com',
            'password' => 'password123',
        ];
        
        $user = new User($userData);
        
        $this->userRepository
            ->expects($this->once())
            ->method('create')
            ->with($userData)
            ->willReturn($user);
        
        $this->emailService
            ->expects($this->once())
            ->method('sendWelcomeEmail')
            ->with($user->email);

        // Act
        $result = $this->userService->createUser($userData);

        // Assert
        $this->assertSame($user, $result);
    }

    public function test_throws_exception_when_user_not_found(): void
    {
        // Arrange
        $userId = 'non-existent-id';
        
        $this->userRepository
            ->expects($this->once())
            ->method('findById')
            ->with($userId)
            ->willReturn(null);

        // Act & Assert
        $this->expectException(UserNotFoundException::class);
        $this->userService->getUserById($userId);
    }
}
```

### 2. Feature тесты

```php
// ✅ Тестирование API эндпоинтов
final class UserApiTest extends TestCase
{
    use RefreshDatabase;

    public function test_can_create_user(): void
    {
        $userData = [
            'name' => 'John Doe',
            'email' => 'john@example.com',
            'password' => 'password123',
            'password_confirmation' => 'password123',
        ];

        $response = $this->postJson('/api/users', $userData);

        $response->assertStatus(201)
            ->assertJsonStructure([
                'data' => [
                    'id',
                    'name',
                    'email',
                    'created_at',
                ]
            ]);

        $this->assertDatabaseHas('users', [
            'name' => 'John Doe',
            'email' => 'john@example.com',
        ]);
    }

    public function test_validates_required_fields(): void
    {
        $response = $this->postJson('/api/users', []);

        $response->assertStatus(422)
            ->assertJsonValidationErrors(['name', 'email', 'password']);
    }

    public function test_prevents_duplicate_email(): void
    {
        User::factory()->create(['email' => 'john@example.com']);

        $response = $this->postJson('/api/users', [
            'name' => 'John Doe',
            'email' => 'john@example.com',
            'password' => 'password123',
            'password_confirmation' => 'password123',
        ]);

        $response->assertStatus(422)
            ->assertJsonValidationErrors(['email']);
    }
}
```

## Безопасность

### 1. Защита от уязвимостей

```php
// ✅ Защита от SQL инъекций (Eloquent делает это автоматически)
$users = User::where('email', $email)->get();

// ✅ Защита от XSS
echo htmlspecialchars($user->name, ENT_QUOTES, 'UTF-8');

// ✅ CSRF защита (Laravel делает это автоматически)
<form method="POST" action="/users">
    @csrf
    <!-- поля формы -->
</form>

// ✅ Rate limiting
Route::middleware(['throttle:60,1'])->group(function () {
    Route::post('/login', [AuthController::class, 'login']);
});

// ✅ Валидация файлов
final class UploadAvatarRequest extends FormRequest
{
    public function rules(): array
    {
        return [
            'avatar' => [
                'required',
                'file',
                'image',
                'mimes:jpeg,png,jpg',
                'max:2048', // 2MB
            ],
        ];
    }
}
```

### 2. Аутентификация и авторизация

```php
// ✅ Использование Sanctum для API
final class AuthController extends Controller
{
    public function login(LoginRequest $request): JsonResponse
    {
        $credentials = $request->validated();
        
        if (!Auth::attempt($credentials)) {
            return response()->json([
                'message' => 'Invalid credentials',
            ], 401);
        }
        
        $user = Auth::user();
        $token = $user->createToken('api-token')->plainTextToken;
        
        return response()->json([
            'user' => new UserResource($user),
            'token' => $token,
        ]);
    }

    public function logout(Request $request): JsonResponse
    {
        $request->user()->currentAccessToken()->delete();
        
        return response()->json([
            'message' => 'Logged out successfully',
        ]);
    }
}

// ✅ Проверка прав доступа
final class UserController extends Controller
{
    public function __construct()
    {
        $this->middleware('auth:sanctum');
        $this->middleware('role:admin')->only(['index', 'destroy']);
    }

    public function update(UpdateUserRequest $request, User $user): JsonResponse
    {
        $this->authorize('update', $user);
        
        $user->update($request->validated());
        
        return response()->json(new UserResource($user));
    }
}
```

## Навигация

- **[Общие принципы](../general/README.md)** — процессы разработки, документация
- **[Фронтенд](../frontend/README.md)** — JavaScript, TypeScript, React, Vue
- **[Базы данных](../database/README.md)** — MongoDB, MySQL


