## Бэкенд (PHP, Laravel)

### Цели
- Предсказуемые, безопасные и производительные API/сервисы с чёткими контрактами.

### Технологии и поддержка версий
- **PHP**: 8.2+ (строгие типы), при необходимости поддержка 8.0/8.1.
- **Laravel**: 10+ (при необходимости поддержка 8/9, с учётом отличий миграций/очередей/маршрутизации).
- **БД**: MongoDB, MySQL (подробности — раздел БД).

### Архитектура слоёв
- Контроллеры → Сервисы/Юзкейсы → Доменные объекты → Доступ к данным (репозитории при необходимости).
- Валидация: FormRequest на входе; бизнес‑инварианты — в домене.
- Сериализация: API Resources.
- Ошибки: централизованный Handler, маппинг исключений на HTTP‑коды.

Пример: контроллер, FormRequest, сервис и ресурс
```php
// app/Http/Controllers/UserController.php
declare(strict_types=1);

namespace App\Http\Controllers;

use App\Http\Requests\StoreUserRequest;
use App\Http\Resources\UserResource;
use App\Services\UserService;

final class UserController extends Controller
{
    public function store(StoreUserRequest $request, UserService $service): UserResource
    {
        $user = $service->create($request->validated());
        return new UserResource($user);
    }
}
```

```php
// app/Http/Requests/StoreUserRequest.php
declare(strict_types=1);
namespace App\Http\Requests;
use Illuminate\Foundation\Http\FormRequest;

final class StoreUserRequest extends FormRequest
{
    public function rules(): array
    {
        return [
            'name' => ['required','string','max:100'],
            'email' => ['required','email','unique:users,email'],
        ];
    }
}
```

```php
// app/Services/UserService.php
declare(strict_types=1);
namespace App\Services;
use App\Models\User;

final class UserService
{
    public function create(array $data): User
    {
        return User::create([
            'name' => $data['name'],
            'email' => $data['email'],
        ]);
    }
}
```

```php
// app/Http/Resources/UserResource.php
declare(strict_types=1);
namespace App\Http\Resources;
use Illuminate\Http\Resources\Json\JsonResource;

final class UserResource extends JsonResource
{
    public function toArray($request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
        ];
    }
}
```

### Производительность и данные
- Использовать `select`‑проекции, избегать N+1 (`with`/`load`).
- Индексы и ограничения в миграциях; транзакции для атомарности.

Пример: выборка без N+1
```php
$users = User::query()->with(['posts:id,user_id,title'])->select(['id','name'])->get();
```

### Очереди и идемпотентность
```php
// app/Jobs/SendReport.php
final class SendReport implements ShouldQueue { /* handle() с ретраями и идемпотентностью */ }
```

### Тестирование (PHPUnit)
```php
public function test_user_created(): void
{
    $payload = ['name' => 'Test', 'email' => 't@example.com'];
    $resp = $this->postJson('/api/users', $payload)->assertCreated();
    $resp->assertJsonPath('data.email', 't@example.com');
}
```

### Процессы
- API‑контракты: OpenAPI, версионирование `/v1`.
- Миграции: атомарные и обратимые; данные — через сидеры/фабрики.
- Конфигурация: через `.env` и `config/*`, без секретов в коде.
- Наблюдаемость: структурированные логи, трассировки, метрики.


