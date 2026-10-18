# Clean Code & Implementation Standard — Modular Architecture

> **Status:** Scaled System Standard (Tier 2)  
> **Classification:** Package by Feature / Modular Architecture  
> **Scope:** Domain-Driven Modular Laravel Backend  
> **Purpose:** Standar penulisan kode modular, pemisahan domain vs shared, kontrak DTO, pemisahan router per modul, dan spesifikasi class di setiap feature module.

---

## 1. Prinsip Batasan Domain (Domain Boundaries)

1. **High Cohesion, Low Coupling**: Semua hal yang berkaitan dengan satu fitur/domain dikumpulkan dalam satu direktori modul (`app/Modules/{Feature}`).
2. **No Direct Inter-Domain DB Access**: Modul dilarang query langsung tabel milik modul lain. Semua interaksi lintas domain wajib melalui Service publik atau Contract Interface modul terkait.
3. **DTO as Data Contract**: Data yang berpindah dari HTTP Controller ke Domain Service wajib dibungkus dalam typed Data Transfer Object (DTO), bukan array bebas atau objek request HTTP.
4. **Isolated Routes**: Setiap modul memiliki file `routes.php` sendiri yang dimuat oleh Module Service Provider terkait.

---

## 2. Standar Penulisan Module Controller

Controller modular mewarisi `BaseApiController` dan memetakan input Request ke DTO sebelum memanggil Service.

```php
namespace App\Modules\Customer\Controllers;

use App\Core\Controllers\BaseApiController;
use App\Modules\Customer\DTOs\CreateCustomerDTO;
use App\Modules\Customer\DTOs\UpdateCustomerDTO;
use App\Modules\Customer\Requests\StoreCustomerRequest;
use App\Modules\Customer\Requests\UpdateCustomerRequest;
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
        $dto = CreateCustomerDTO::fromRequest($request);
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

    public function show(int $id): JsonResponse
    {
        $customer = $this->customerService->getCustomerById($id);

        return response()->json([
            'meta' => [
                'success' => true,
                'message' => 'Customer retrieved successfully',
                'code'    => 200,
            ],
            'data' => new CustomerResource($customer),
        ], 200);
    }

    public function update(UpdateCustomerRequest $request, int $id): JsonResponse
    {
        $dto = UpdateCustomerDTO::fromRequest($request);
        $customer = $this->customerService->updateCustomer($id, $dto);

        return response()->json([
            'meta' => [
                'success' => true,
                'message' => 'Customer updated successfully',
                'code'    => 200,
            ],
            'data' => new CustomerResource($customer),
        ], 200);
    }
}
```

---

## 3. Standar Penulisan DTO (Data Transfer Object)

- Gunakan class `readonly`.
- Berikan static constructor (`fromRequest`, `fromArray`).
- Typed properties tanpa ketergantungan pada framework HTTP di level signature domain.

```php
namespace App\Modules\Customer\DTOs;

use App\Modules\Customer\Requests\StoreCustomerRequest;

readonly class CreateCustomerDTO
{
    public function __construct(
        public string $name,
        public string $email,
        public ?string $phone = null,
        public bool $isActive = true,
    ) {}

    public static function fromRequest(StoreCustomerRequest $request): self
    {
        $validated = $request->validated();

        return new self(
            name: $validated['name'],
            email: $validated['email'],
            phone: $validated['phone'] ?? null,
            isActive: $validated['is_active'] ?? true,
        );
    }

    public function toArray(): array
    {
        return [
            'name'      => $this->name,
            'email'     => $this->email,
            'phone'     => $this->phone,
            'is_active' => $this->isActive,
        ];
    }
}
```

---

## 4. Standar Penulisan Domain Service

- Menerima DTO pada method mutasi.
- Mengontrol transaksi database (`DB::transaction`).
- Mempublikasikan Domain Event (jika modul lain perlu bereaksi).

```php
namespace App\Modules\Customer\Services;

use App\Core\Exceptions\EntityNotFoundException;
use App\Modules\Customer\DTOs\CreateCustomerDTO;
use App\Modules\Customer\DTOs\UpdateCustomerDTO;
use App\Modules\Customer\Events\CustomerRegisteredEvent;
use App\Modules\Customer\Interfaces\CustomerRepositoryInterface;
use App\Modules\Customer\Models\Customer;
use Illuminate\Support\Facades\DB;

class CustomerService
{
    public function __construct(
        private readonly CustomerRepositoryInterface $customerRepository
    ) {}

    public function getCustomerById(int $id): Customer
    {
        $customer = $this->customerRepository->findById($id);

        if (!$customer) {
            throw new EntityNotFoundException("Customer with ID {$id} not found.");
        }

        return $customer;
    }

    public function registerCustomer(CreateCustomerDTO $dto): Customer
    {
        return DB::transaction(function () use ($dto) {
            $customer = $this->customerRepository->create($dto->toArray());

            event(new CustomerRegisteredEvent($customer));

            return $customer;
        });
    }

    public function updateCustomer(int $id, UpdateCustomerDTO $dto): Customer
    {
        $customer = $this->getCustomerById($id);

        return DB::transaction(function () use ($customer, $dto) {
            return $this->customerRepository->update($customer, $dto->toArray());
        });
    }
}
```

---

## 5. Standar Penulisan Module Model

Model diletakkan di dalam folder `Models` pada modul bersangkutan.

```php
namespace App\Modules\Customer\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Customer extends Model
{
    use HasFactory, SoftDeletes;

    protected $table = 'mt_customers';

    protected $fillable = [
        'name',
        'email',
        'phone',
        'is_active',
    ];

    protected $casts = [
        'is_active'  => 'boolean',
        'created_at' => 'datetime',
        'updated_at' => 'datetime',
        'deleted_at' => 'datetime',
    ];
}
```

---

## 6. Standar Penulisan Module Repository & Interface

### Interface (`app/Modules/Customer/Interfaces/CustomerRepositoryInterface.php`)
```php
namespace App\Modules\Customer\Interfaces;

use App\Modules\Customer\Models\Customer;
use Illuminate\Pagination\LengthAwarePaginator;

interface CustomerRepositoryInterface
{
    public function paginate(int $perPage = 20): LengthAwarePaginator;
    public function findById(int $id): ?Customer;
    public function create(array $payload): Customer;
    public function update(Customer $customer, array $payload): Customer;
}
```

### Implementasi (`app/Modules/Customer/Repositories/CustomerRepository.php`)
```php
namespace App\Modules\Customer\Repositories;

use App\Modules\Customer\Interfaces\CustomerRepositoryInterface;
use App\Modules\Customer\Models\Customer;
use Illuminate\Pagination\LengthAwarePaginator;

class CustomerRepository implements CustomerRepositoryInterface
{
    public function paginate(int $perPage = 20): LengthAwarePaginator
    {
        return Customer::query()
            ->latest('id')
            ->paginate($perPage);
    }

    public function findById(int $id): ?Customer
    {
        return Customer::query()->find($id);
    }

    public function create(array $payload): Customer
    {
        return Customer::query()->create($payload);
    }

    public function update(Customer $customer, array $payload): Customer
    {
        $customer->update($payload);
        return $customer;
    }
}
```

---

## 7. Standar Penulisan Module Provider & Routes

### Module Provider (`app/Modules/Customer/Providers/CustomerServiceProvider.php`)
```php
namespace App\Modules\Customer\Providers;

use App\Modules\Customer\Interfaces\CustomerRepositoryInterface;
use App\Modules\Customer\Repositories\CustomerRepository;
use Illuminate\Support\ServiceProvider;

class CustomerServiceProvider extends ServiceProvider
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

### Module Route (`app/Modules/Customer/routes.php`)
```php
use App\Modules\Customer\Controllers\CustomerController;
use Illuminate\Support\Facades\Route;

Route::prefix('api/v1/customers')
    ->middleware(['api', 'auth:sanctum'])
    ->group(function () {
        Route::get('/', [CustomerController::class, 'index']);
        Route::post('/', [CustomerController::class, 'store']);
        Route::get('/{customer}', [CustomerController::class, 'show']);
        Route::put('/{customer}', [CustomerController::class, 'update']);
        Route::delete('/{customer}', [CustomerController::class, 'destroy']);
    });
```
