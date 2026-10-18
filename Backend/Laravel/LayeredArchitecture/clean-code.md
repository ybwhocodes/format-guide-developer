# Clean Code & Implementation Standard — Layered Architecture

> **Status:** Team Standard (Tier 1)  
> **Scope:** Laravel Standard / Layered Architecture  
> **Purpose:** Standar penulisan kode bersih, spesifikasi teknis tiap layer (Controller, Service, Repository, Model, Request, Resource, Interface), dan aturan implementasi konkret.

---

## 1. Tanggung Jawab & Boundary Tiap Layer

| Layer | Boleh Dilakukan | Dilarang Keras |
|---|---|---|
| **Controller** | Menerima request, memanggil service, mengembalikan HTTP response, memanggil Resource | Query Eloquent, kalkulasi bisnis, DB transaction, raw external HTTP call |
| **Form Request** | Validasi input data, otorisasi level request, normalisasi payload ringan | Query database selain rule validasi, manipulasi state domain |
| **Service** | Aturan bisnis, kalkulasi finansial/stok, DB transaction, koordinasi antar repository | Mengakses `Request` object langsung, return view/response HTTP |
| **Repository** | Query Eloquent/DB, filtering, pagination, eager loading, aggregate summary | Pengambilan keputusan bisnis, validasi logika domain |
| **Model** | Casts, relasi Eloquent (`hasMany`, dll), scope query murni, mutator/accessor format | Menjalankan alur proses transaksi antar tabel, HTTP logic |
| **Resource** | Transformasi output JSON, masking field rahasia, formatting tanggal | Menjalankan query berat di dalam loop resource (N+1 query) |

---

## 2. Standar Penulisan Controller

- Wajib tipis (*thin controller*).
- Dependency injection via constructor (gunakan `readonly`).
- Mengembalikan `JsonResponse` atau instance `JsonResource`.
- Gunakan type-hint Form Request spesifik.

```php
namespace App\Http\Controllers\Api\V1;

use App\Http\Controllers\Controller;
use App\Http\Requests\Customer\StoreCustomerRequest;
use App\Http\Requests\Customer\UpdateCustomerRequest;
use App\Http\Resources\CustomerResource;
use App\Services\CustomerService;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Resources\Json\AnonymousResourceCollection;

class CustomerController extends Controller
{
    public function __construct(
        private readonly CustomerService $customerService
    ) {}

    public function index(): AnonymousResourceCollection
    {
        $customers = $this->customerService->getPaginatedCustomers();

        return CustomerResource::collection($customers);
    }

    public function store(StoreCustomerRequest $request): JsonResponse
    {
        $customer = $this->customerService->createCustomer($request->validated());

        return (new CustomerResource($customer))
            ->response()
            ->setStatusCode(201);
    }

    public function show(int $id): JsonResponse
    {
        $customer = $this->customerService->getCustomerById($id);

        return (new CustomerResource($customer))
            ->response()
            ->setStatusCode(200);
    }

    public function update(UpdateCustomerRequest $request, int $id): JsonResponse
    {
        $customer = $this->customerService->updateCustomer($id, $request->validated());

        return (new CustomerResource($customer))
            ->response()
            ->setStatusCode(200);
    }

    public function destroy(int $id): JsonResponse
    {
        $this->customerService->deleteCustomer($id);

        return response()->json(null, 204);
    }
}
```

---

## 3. Standar Penulisan Form Request

- Selalu gunakan array syntax untuk rules validation.
- Gunakan nama tabel sesuai konvensi prefix database (`mt_`, `tr_`).

```php
namespace App\Http\Requests\Customer;

use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Validation\Rule;

class StoreCustomerRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'name'      => ['required', 'string', 'max:150'],
            'email'     => ['required', 'email', 'max:150', Rule::unique('mt_customers', 'email')->whereNull('deleted_at')],
            'phone'     => ['nullable', 'string', 'max:20'],
            'is_active' => ['sometimes', 'boolean'],
        ];
    }
}
```

---

## 4. Standar Penulisan Service Layer

- Menerima parameter berupa `array` valid atau DTO primitif, bukan objek `Illuminate\Http\Request`.
- Membungkus proses multi-tabel dalam `DB::transaction()`.
- Melempar domain exception jika melanggar invariant bisnis.

```php
namespace App\Services;

use App\Exceptions\CustomerNotFoundException;
use App\Interfaces\CustomerRepositoryInterface;
use App\Models\Customer;
use Illuminate\Pagination\LengthAwarePaginator;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Log;

class CustomerService
{
    public function __construct(
        private readonly CustomerRepositoryInterface $customerRepository
    ) {}

    public function getPaginatedCustomers(int $perPage = 20): LengthAwarePaginator
    {
        return $this->customerRepository->paginate($perPage);
    }

    public function getCustomerById(int $id): Customer
    {
        $customer = $this->customerRepository->findById($id);

        if (!$customer) {
            throw new CustomerNotFoundException("Customer with ID {$id} not found.");
        }

        return $customer;
    }

    public function createCustomer(array $validatedData): Customer
    {
        return DB::transaction(function () use ($validatedData) {
            $customer = $this->customerRepository->create($validatedData);

            Log::info('Customer created successfully', [
                'customer_id' => $customer->id,
                'email'       => $customer->email,
            ]);

            return $customer;
        });
    }

    public function updateCustomer(int $id, array $validatedData): Customer
    {
        $customer = $this->getCustomerById($id);

        return DB::transaction(function () use ($customer, $validatedData) {
            return $this->customerRepository->update($customer, $validatedData);
        });
    }

    public function deleteCustomer(int $id): void
    {
        $customer = $this->getCustomerById($id);

        DB::transaction(function () use ($customer) {
            $this->customerRepository->delete($customer);
        });
    }
}
```

---

## 5. Standar Penulisan Repository & Interface

### Interface (`app/Interfaces/CustomerRepositoryInterface.php`)
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

### Implementasi (`app/Repositories/CustomerRepository.php`)
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
            ->select(['id', 'name', 'email', 'phone', 'is_active', 'created_at'])
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

## 6. Standar Penulisan Eloquent Model

- Definisikan `$table` eksplisit (patuhi prefix `mt_`, `tr_`, dll).
- Wajib proteksi kolom via `$fillable` (jangan gunakan `$guarded = []`).
- Wajib ketik `$casts` untuk tipe data primitif, boolean, dan datetime.
- Relasi wajib memiliki return type deklaratif (`HasMany`, `BelongsTo`).

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;
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

    public function orders(): HasMany
    {
        return $this->hasMany(Order::class, 'customer_id', 'id');
    }
}
```

---

## 7. Standar Penulisan API Resource

- Format output tanggal standar ISO8601 string.
- Hindari menyertakan password hash, token privat, atau secret keys.

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
            'phone'      => $this->phone,
            'is_active'  => $this->is_active,
            'created_at' => $this->created_at?->toISOString(),
            'updated_at' => $this->updated_at?->toISOString(),
        ];
    }
}
```
