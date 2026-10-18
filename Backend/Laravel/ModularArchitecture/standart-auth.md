# Authentication & Authorization Standard — Modular Architecture

> **Status:** Scaled System Standard (Tier 2)  
> **Scope:** Modular / Domain-Driven Architecture in Laravel  
> **Purpose:** Standar penempatan autentikasi sebagai Modul Inti (`app/Modules/Auth`), otorisasi domain terisolasi per feature module, dan kontrak cross-module guard/policy.

---

## 1. Penempatan Auth Sebagai Dedicated Module

Otentikasi, manajemen token, dan lifecycle akun ditempatkan ke dalam modul mandiri `app/Modules/Auth`:

```text
app/
├── Modules/
│   ├── Auth/
│   │   ├── Controllers/
│   │   │   ├── LoginController.php
│   │   │   └── LogoutController.php
│   │   ├── DTOs/
│   │   │   └── LoginDTO.php
│   │   ├── Requests/
│   │   │   └── LoginRequest.php
│   │   ├── Resources/
│   │   │   └── AuthTokenResource.php
│   │   ├── Services/
│   │   │   └── AuthService.php
│   │   ├── Providers/
│   │   │   └── AuthModuleServiceProvider.php
│   │   └── routes.php
│   │
│   └── Customer/
│       ├── Policies/
│       │   └── CustomerPolicy.php
│       └── Providers/
│           └── CustomerAuthServiceProvider.php
```

---

## 2. Implementasi Auth Service & Controller di Modul Auth

### Login Controller (`app/Modules/Auth/Controllers/LoginController.php`)

```php
namespace App\Modules\Auth\Controllers;

use App\Core\Controllers\BaseApiController;
use App\Modules\Auth\DTOs\LoginDTO;
use App\Modules\Auth\Requests\LoginRequest;
use App\Modules\Auth\Resources\AuthTokenResource;
use App\Modules\Auth\Services\AuthService;
use Illuminate\Http\JsonResponse;

class LoginController extends BaseApiController
{
    public function __construct(
        private readonly AuthService $authService
    ) {}

    public function __invoke(LoginRequest $request): JsonResponse
    {
        $dto = LoginDTO::fromRequest($request);
        $authData = $this->authService->authenticate($dto);

        return response()->json([
            'meta' => [
                'success' => true,
                'message' => 'Authentication successful',
                'code'    => 200,
            ],
            'data' => new AuthTokenResource($authData),
        ], 200);
    }
}
```

---

## 3. Otorisasi Terlokalisasi di Tiap Modul Domain

Setiap modul memiliki folder `Policies` sendiri untuk mengisolasi aturan hak akses domain tersebut. Modul tidak bergantung pada logic otorisasi modul lain.

### Customer Policy (`app/Modules/Customer/Policies/CustomerPolicy.php`)

```php
namespace App\Modules\Customer\Policies;

use App\Models\User;
use App\Modules\Customer\Models\Customer;
use Illuminate\Auth\Access\Response;

class CustomerPolicy
{
    public function viewAny(User $user): bool
    {
        return $user->hasPermission('customer.view');
    }

    public function update(User $user, Customer $customer): Response
    {
        if ($user->hasPermission('customer.manage-all')) {
            return Response::allow();
        }

        return $user->branch_id === $customer->branch_id
            ? Response::allow()
            : Response::deny('Cross-branch customer modification is forbidden.');
    }
}
```

### Registrasi Policy di Module Provider

```php
namespace App\Modules\Customer\Providers;

use App\Modules\Customer\Models\Customer;
use App\Modules\Customer\Policies\CustomerPolicy;
use Illuminate\Support\Facades\Gate;
use Illuminate\Support\ServiceProvider;

class CustomerAuthServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        Gate::policy(Customer::class, CustomerPolicy::class);
    }
}
```
