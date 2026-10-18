# Laravel Repository Pattern — Package by Feature (Modular Architecture)

> **Status:** Scaled System Pattern (Tier 2)  
> **Classification:** Package by Feature / Vertical Slicing / Modular Architecture  
> **Scope:** Domain-Driven Modular Structure in Laravel  
> **Best For:** Aplikasi enterprise, ERP, SaaS multi-domain, sistem dengan lifecycle panjang, atau tim dengan banyak developer.

---

## 1. Konsep & Filosofi

Semua komponen dikelompokkan berdasarkan **batasan bisnis atau domain fitur**:
- Setiap modul/fitur membungkus Controller, Service, Repository, Model, DTO, Request, dan Route-nya sendiri.
- Komponen generik non-bisnis dipusatkan di folder `Core` atau `Shared`.
- Modul berkomunikasi antar-domain melalui **Contract/Service Interface**, bukan direct database query atau coupling model langsung.

Tujuan utamanya adalah menjaga *high cohesion* di dalam modul dan *low coupling* antar modul.

---

## 2. Struktur Direktori Proyek

```text
app/
├── Core/
│   ├── Controllers/
│   │   └── BaseApiController.php
│   ├── Exceptions/
│   │   └── DomainException.php
│   └── Contracts/
│
├── Modules/
│   ├── Customer/
│   │   ├── Controllers/
│   │   │   └── CustomerController.php
│   │   ├── DTOs/
│   │   │   └── CreateCustomerDTO.php
│   │   ├── Interfaces/
│   │   │   └── CustomerRepositoryInterface.php
│   │   ├── Models/
│   │   │   └── Customer.php
│   │   ├── Repositories/
│   │   │   └── CustomerRepository.php
│   │   ├── Requests/
│   │   │   └── StoreCustomerRequest.php
│   │   ├── Resources/
│   │   │   └── CustomerResource.php
│   │   ├── Services/
│   │   │   └── CustomerService.php
│   │   ├── Providers/
│   │   │   └── CustomerModuleServiceProvider.php
│   │   └── routes.php
│   │
│   ├── Billing/
│   │   ├── Controllers/
│   │   ├── Services/
│   │   ├── Repositories/
│   │   ├── Models/
│   │   └── routes.php
│   │
│   └── Inventory/
│       └── ...
│
└── Shared/
    ├── DTOs/
    ├── Repositories/
    │   └── ReferenceRepository.php
    └── Support/
        └── PaginationHelper.php
```

---

## 3. Komunikasi Antar-Domain (Cross-Domain Boundaries)

### Rule of Thumb
- **DILARANG:** Modul `Billing` melakukan query langsung terhadap model `Inventory`:
  ```php
  // SALAH: Direct query antar domain
  \App\Modules\Inventory\Models\Stock::where('item_id', $id)->decrement('qty', 1);
  ```
- **BENAR:** Panggil Service atau Interface yang disediakan oleh modul terkait:
  ```php
  // BENAR: Lewat Domain Contract / Service
  $this->inventoryService->reserveStock($itemId, $qty);
  ```

### Domain vs Shared
| Kategori | Kriteria | Contoh |
|---|---|---|
| **Domain Module** | Logic spesifik untuk unit bisnis tertentu | `Customer`, `Billing`, `Invoice`, `Stock`, `Order` |
| **Shared** | Komponen referensi umum / utilitas lintas domain | Nilai mata uang, master cabang, helper pagination global |

---

## 4. Contoh Implementasi Modular

### 4.1 Data Transfer Object (`app/Modules/Customer/DTOs/CreateCustomerDTO.php`)
Menyediakan kontrak tipe data yang aman antara Request dan Service tanpa ketergantungan pada objek HTTP Request.

```php
namespace App\Modules\Customer\DTOs;

readonly class CreateCustomerDTO
{
    public function __construct(
        public string $name,
        public string $email,
        public ?string $phone = null,
    ) {}

    public static function fromArray(array $data): self
    {
        return new self(
            name: $data['name'],
            email: $data['email'],
            phone: $data['phone'] ?? null,
        );
    }
}
```

---

### 4.2 Module Controller (`app/Modules/Customer/Controllers/CustomerController.php`)

```php
namespace App\Modules\Customer\Controllers;

use App\Core\Controllers\BaseApiController;
use App\Modules\Customer\DTOs\CreateCustomerDTO;
use App\Modules\Customer\Requests\StoreCustomerRequest;
use App\Modules\Customer\Resources\CustomerResource;
use App\Modules\Customer\Services\CustomerService;
use Illuminate\Http\JsonResponse;

class CustomerController extends BaseApiController
{
    public function __construct(
        private readonly CustomerService $customerService
    ) {}

    public function store(StoreCustomerRequest $request): JsonResponse
    {
        $dto = CreateCustomerDTO::fromArray($request->validated());
        $customer = $this->customerService->registerCustomer($dto);

        return response()->json([
            'meta' => [
                'success' => true,
                'message' => 'Customer registered successfully',
                'code'    => 201,
            ],
            'data' => new CustomerResource($customer),
        ], 201);
    }
}
```

---

### 4.3 Domain Service (`app/Modules/Customer/Services/CustomerService.php`)

```php
namespace App\Modules\Customer\Services;

use App\Modules\Customer\DTOs\CreateCustomerDTO;
use App\Modules\Customer\Interfaces\CustomerRepositoryInterface;
use App\Modules\Customer\Models\Customer;
use Illuminate\Support\Facades\DB;

class CustomerService
{
    public function __construct(
        private readonly CustomerRepositoryInterface $customerRepository
    ) {}

    public function registerCustomer(CreateCustomerDTO $dto): Customer
    {
        return DB::transaction(function () use ($dto) {
            return $this->customerRepository->create([
                'name'  => $dto->name,
                'email' => $dto->email,
                'phone' => $dto->phone,
            ]);
        });
    }
}
```

---

### 4.4 Module Repository & Interface

```php
namespace App\Modules\Customer\Interfaces;

use App\Modules\Customer\Models\Customer;

interface CustomerRepositoryInterface
{
    public function create(array $payload): Customer;
    public function findById(int $id): ?Customer;
}
```

```php
namespace App\Modules\Customer\Repositories;

use App\Modules\Customer\Interfaces\CustomerRepositoryInterface;
use App\Modules\Customer\Models\Customer;

class CustomerRepository implements CustomerRepositoryInterface
{
    public function create(array $payload): Customer
    {
        return Customer::query()->create($payload);
    }

    public function findById(int $id): ?Customer
    {
        return Customer::query()->find($id);
    }
}
```

---

### 4.5 Module Routes (`app/Modules/Customer/routes.php`)
Setiap modul mengelola endpoint-nya secara terisolasi tanpa membebani `routes/api.php` utama.

```php
use App\Modules\Customer\Controllers\CustomerController;
use Illuminate\Support\Facades\Route;

Route::prefix('api/v1/customers')
    ->middleware(['api', 'auth:sanctum'])
    ->group(function () {
        Route::get('/', [CustomerController::class, 'index']);
        Route::post('/', [CustomerController::class, 'store']);
        Route::get('/{customer}', [CustomerController::class, 'show']);
    });
```

---

### 4.6 Module Service Provider (`app/Modules/Customer/Providers/CustomerModuleServiceProvider.php`)
Memuat route modul dan mendaftarkan dependency binding secara mandiri.

```php
namespace App\Modules\Customer\Providers;

use App\Modules\Customer\Interfaces\CustomerRepositoryInterface;
use App\Modules\Customer\Repositories\CustomerRepository;
use Illuminate\Support\ServiceProvider;

class CustomerModuleServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->app->bind(CustomerRepositoryInterface::class, CustomerRepository::class);
    }

    public function boot(): void
    {
        if (file_exists(__DIR__ . '/../routes.php')) {
            $this->loadRoutesFrom(__DIR__ . '/../routes.php');
        }
    }
}
```

---

## 5. Kelebihan & Aturan Transisi

### Kapan Menggunakan Modular Architecture?
- Terdapat lebih dari 3 domain bisnis independen (contoh: Auth, Customer, Billing, Inventory).
- Lebih dari 3 developer bekerja aktif di repositori yang sama secara paralel.
- Route API sudah sangat padat dan sulit dilacak jika hanya terpusat di satu file route.
- Rencana jangka panjang mendukung pemisahan menjadi sub-layanan / service mandiri di masa depan.
