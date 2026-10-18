# Laravel Repository Pattern — Package by Layer (Layered Architecture)

> **Status:** Framework Standard Pattern (Tier 1)  
> **Classification:** Package by Layer / Horizontal Slicing  
> **Scope:** Laravel Standard MVC + Service-Repository Layer  
> **Best For:** Aplikasi skala kecil-menengah, MVP, microservice tunggal, atau sistem dengan 1–2 domain utama.

---

## 1. Konsep & Filosofi

Semua komponen dikelompokkan berdasarkan **jenis kelas teknisnya (role/layer)**:
- Semua Controller berada di satu folder `app/Http/Controllers`.
- Semua Service berada di folder `app/Services`.
- Semua Repository berada di folder `app/Repositories`.
- Semua Model berada di folder `app/Models`.

Pola ini merupakan arsitektur bawaan/standar komunitas Laravel saat menambahkan Service-Repository layer tanpa memecah modul per fitur.

---

## 2. Struktur Direktori Proyek

```text
app/
├── Http/
│   ├── Controllers/
│   │   └── Api/
│   │       └── V1/
│   │           ├── AuthController.php
│   │           ├── CustomerController.php
│   │           └── OrderController.php
│   ├── Requests/
│   │   ├── Customer/
│   │   │   ├── StoreCustomerRequest.php
│   │   │   └── UpdateCustomerRequest.php
│   │   └── Order/
│   └── Resources/
│       ├── CustomerResource.php
│       └── OrderResource.php
│
├── Services/
│   ├── CustomerService.php
│   └── OrderService.php
│
├── Interfaces/
│   ├── CustomerRepositoryInterface.php
│   └── OrderRepositoryInterface.php
│
├── Repositories/
│   ├── CustomerRepository.php
│   └── OrderRepository.php
│
├── Models/
│   ├── Customer.php
│   └── Order.php
│
├── Providers/
│   └── RepositoryServiceProvider.php
│
└── Exceptions/
    └── DomainException.php

routes/
└── api.php
```

---

## 3. Alur Eksekusi (Request Lifecycle)

```text
HTTP Request
     ↓
routes/api.php
     ↓
Form Request (Validation & Authorization)
     ↓
Controller (Thin Layer, Orchestrator)
     ↓
Service (Business Logic & DB Transaction)
     ↓
Repository Interface (Contract)
     ↓
Repository (Eloquent Query & Data Retrieval)
     ↓
Eloquent Model
     ↓
Database
     ↓
Eloquent Model / Collection
     ↓
API Resource (DTO / Output Transformer)
     ↓
JSON Response
```

---

## 4. Tanggung Jawab & Contoh Implementasi Tiap Layer

### 4.1 Form Request (`app/Http/Requests`)
Hanya bertugas memvalidasi format input dan otorisasi level request.

```php
namespace App\Http\Requests\Customer;

use Illuminate\Foundation\Http\FormRequest;

class StoreCustomerRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'name'  => ['required', 'string', 'max:150'],
            'email' => ['required', 'email', 'unique:mt_customers,email'],
            'phone' => ['nullable', 'string', 'max:20'],
        ];
    }
}
```

---

### 4.2 Controller (`app/Http/Controllers/Api/V1`)
Controller wajib tipis (*thin controller*). Dilarang menulis query database, kalkulasi pajak, atau alur transaksi di sini.

```php
namespace App\Http\Controllers\Api\V1;

use App\Http\Controllers\Controller;
use App\Http\Requests\Customer\StoreCustomerRequest;
use App\Http\Resources\CustomerResource;
use App\Services\CustomerService;
use Illuminate\Http\JsonResponse;

class CustomerController extends Controller
{
    public function __construct(
        private readonly CustomerService $customerService
    ) {}

    public function store(StoreCustomerRequest $request): JsonResponse
    {
        $customer = $this->customerService->registerCustomer($request->validated());

        return (new CustomerResource($customer))
            ->response()
            ->setStatusCode(201);
    }
}
```

---

### 4.3 Service Layer (`app/Services`)
Tempat seluruh logika bisnis, workflow, validasi aturan bisnis, kalkulasi angka, dan database transaction boundary.

```php
namespace App\Services;

use App\Interfaces\CustomerRepositoryInterface;
use App\Models\Customer;
use Illuminate\Support\Facades\DB;

class CustomerService
{
    public function __construct(
        private readonly CustomerRepositoryInterface $customerRepository
    ) {}

    public function registerCustomer(array $payload): Customer
    {
        return DB::transaction(function () use ($payload) {
            $customer = $this->customerRepository->create($payload);

            // Logika bisnis tambahan (misal: trigger welcome notification, default ledger account)

            return $customer;
        });
    }
}
```

---

### 4.4 Repository Interface (`app/Interfaces`)
Mendefinisikan kontrak akses data agar Service terlepas dari detail implementasi ORM.

```php
namespace App\Interfaces;

use App\Models\Customer;
use Illuminate\Pagination\LengthAwarePaginator;

interface CustomerRepositoryInterface
{
    public function paginate(int $perPage = 20): LengthAwarePaginator;
    public function findById(int $id): ?Customer;
    public function create(array $payload): Customer;
    public function update(Customer $customer, array $payload): Customer;
    public function delete(Customer $customer): void;
}
```

---

### 4.5 Repository Implementation (`app/Repositories`)
Hanya menangani query database, filter query, sorting, eager loading, dan persistensi data. Dilarang mengambil keputusan bisnis.

```php
namespace App\Repositories;

use App\Interfaces\CustomerRepositoryInterface;
use App\Models\Customer;
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

    public function delete(Customer $customer): void
    {
        $customer->delete();
    }
}
```

---

### 4.6 Service Provider Binding (`app/Providers/RepositoryServiceProvider.php`)
Mendaftarkan interface ke implementasinya pada IoC container Laravel:

```php
namespace App\Providers;

use App\Interfaces\CustomerRepositoryInterface;
use App\Repositories\CustomerRepository;
use Illuminate\Support\ServiceProvider;

class RepositoryServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->app->bind(CustomerRepositoryInterface::class, CustomerRepository::class);
    }
}
```

---

### 4.7 API Resource (`app/Http/Resources`)
Memformat model/entity ke output JSON agar kontrak API stabil dan field sensitif tidak bocor.

```php
namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class CustomerResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id'         => $this->id,
            'name'       => $this->name,
            'email'      => $this->email,
            'is_active'  => (bool) $this->is_active,
            'created_at' => $this->created_at?->toISOString(),
        ];
    }
}
```

---

## 5. Kelebihan & Batasan

### Kelebihan
- Cepat dipahami oleh developer yang terbiasa dengan struktur standar Laravel.
- Sangat efisien untuk tim kecil dengan sprint cepat.
- Setup tooling, linter, dan auto-discovery class langsung berjalan tanpa konfigurasi ekstra.

### Kapan Harus Migrasi ke Package by Feature?
- Folder `app/Services` atau `app/Repositories` sudah berisi puluhan hingga ratusan file.
- Sering terjadi *merge conflict* karena beberapa developer mengedit folder teknis yang sama.
- Mulai sulit mengidentifikasi relasi antar kelas ketika satu domain bisnis membutuhkan perubahan.
