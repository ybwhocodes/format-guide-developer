# Authentication & Authorization Standard — Layered Architecture

> **Status:** Team Standard (Tier 1)  
> **Scope:** Layered Architecture / Framework Standard Laravel  
> **Purpose:** Standar autentikasi token Sanctum dan otorisasi terpusat pada layer standar Laravel (`app/Http/Controllers/Api/V1/AuthController.php`, `app/Policies`).

---

## 1. Struktur Folder Auth & Otorisasi

```text
app/
├── Http/
│   ├── Controllers/
│   │   └── Api/
│   │       └── V1/
│   │           └── AuthController.php
│   ├── Requests/
│   │   └── Auth/
│   │       ├── LoginRequest.php
│   │       └── RegisterRequest.php
│   └── Resources/
│       └── UserResource.php
│
├── Models/
│   └── User.php
│
├── Policies/
│   └── CustomerPolicy.php
│
└── Providers/
    └── AuthServiceProvider.php

routes/
└── api.php
```

---

## 2. Implementasi AuthController

Menggunakan Laravel Sanctum untuk autentikasi API stateless dengan response meta standar.

```php
namespace App\Http\Controllers\Api\V1;

use App\Http\Controllers\Controller;
use App\Http\Requests\Auth\LoginRequest;
use App\Http\Resources\UserResource;
use App\Models\User;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Illuminate\Validation\ValidationException;

class AuthController extends Controller
{
    public function login(LoginRequest $request): JsonResponse
    {
        $user = User::query()
            ->where('email', $request->input('email'))
            ->where('is_active', true)
            ->first();

        if (!$user || !Hash::check($request->input('password'), $user->password)) {
            throw ValidationException::withMessages([
                'email' => ['Kredensial yang diberikan tidak cocok dengan data kami.'],
            ]);
        }

        $token = $user->createToken('api-token', ['*'], now()->addDays(7))->plainTextToken;

        return response()->json([
            'meta' => [
                'success' => true,
                'message' => 'Login berhasil',
                'code'    => 200,
            ],
            'data' => [
                'token'      => $token,
                'token_type' => 'Bearer',
                'user'       => new UserResource($user),
            ],
        ]);
    }

    public function me(Request $request): JsonResponse
    {
        return response()->json([
            'meta' => [
                'success' => true,
                'message' => 'User profile retrieved',
                'code'    => 200,
            ],
            'data' => new UserResource($request->user()),
        ]);
    }

    public function logout(Request $request): JsonResponse
    {
        $request->user()->currentAccessToken()->delete();

        return response()->json([
            'meta' => [
                'success' => true,
                'message' => 'Logout berhasil',
                'code'    => 200,
            ],
            'data' => null,
        ]);
    }
}
```

---

## 3. Otorisasi Terpusat via Policy (`app/Policies`)

Gunakan Policy untuk memeriksa otorisasi tindakan pengguna terhadap entitas model.

```php
namespace App\Policies;

use App\Models\Customer;
use App\Models\User;
use Illuminate\Auth\Access\Response;

class CustomerPolicy
{
    public function viewAny(User $user): bool
    {
        return $user->hasRole('admin') || $user->hasRole('staff');
    }

    public function update(User $user, Customer $customer): Response
    {
        if ($user->hasRole('admin')) {
            return Response::allow();
        }

        return $customer->created_by === $user->id
            ? Response::allow()
            : Response::deny('Anda tidak memiliki hak untuk mengubah data pelanggan ini.');
    }
}
```

### Pemanggilan di Controller
```php
public function update(UpdateCustomerRequest $request, int $id): JsonResponse
{
    $customer = $this->customerService->getCustomerById($id);

    $this->authorize('update', $customer);

    $updated = $this->customerService->updateCustomer($id, $request->validated());

    return (new CustomerResource($updated))->response();
}
```
