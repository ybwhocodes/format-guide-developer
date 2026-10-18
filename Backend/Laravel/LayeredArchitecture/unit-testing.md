# Unit & Feature Testing Standard — Layered Architecture

> **Status:** Team Standard (Tier 1)  
> **Scope:** Layered Architecture / Framework Standard Laravel  
> **Purpose:** Standar pengujian otomatis terpusat di `tests/Unit` dan `tests/Feature` untuk menguji Controller, Service, Form Request, dan Repository.

---

## 1. Struktur Folder Testing

```text
tests/
├── TestCase.php
├── CreatesApplication.php
│
├── Unit/
│   ├── Services/
│   │   ├── CustomerServiceTest.php
│   │   └── OrderServiceTest.php
│   └── Formats/
│       └── CurrencyHelperTest.php
│
└── Feature/
    └── Api/
        └── V1/
            ├── AuthTest.php
            ├── CustomerTest.php
            └── OrderTest.php
```

---

## 2. Unit Testing Service (Isolated with Mockery)

Service diuji secara terisolasi tanpa menyentuh koneksi database asli, dengan memalsukan (*mocking*) interface repository.

```php
namespace Tests\Unit\Services;

use App\Interfaces\CustomerRepositoryInterface;
use App\Models\Customer;
use App\Services\CustomerService;
use Mockery;
use Mockery\MockInterface;
use Tests\TestCase;

class CustomerServiceTest extends TestCase
{
    private CustomerService $service;
    private MockInterface $repositoryMock;

    protected function setUp(): void
    {
        parent::setUp();

        $this->repositoryMock = Mockery::mock(CustomerRepositoryInterface::class);
        $this->service = new CustomerService($this->repositoryMock);
    }

    public function test_create_customer_delegates_to_repository(): void
    {
        $payload = [
            'name'  => 'John Doe',
            'email' => 'john@example.com',
        ];

        $expectedCustomer = new Customer($payload);
        $expectedCustomer->id = 1;

        $this->repositoryMock
            ->shouldReceive('create')
            ->once()
            ->with($payload)
            ->andReturn($expectedCustomer);

        $result = $this->service->createCustomer($payload);

        $this->assertSame(1, $result->id);
        $this->assertSame('John Doe', $result->name);
    }

    protected function tearDown(): void
    {
        Mockery::close();
        parent::tearDown();
    }
}
```

---

## 3. Feature Testing API Endpoint (`tests/Feature/Api/V1`)

Menguji endpoint API dari route sampai database in-memory menggunakan trait `RefreshDatabase`.

```php
namespace Tests\Feature\Api\V1;

use App\Models\Customer;
use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Laravel\Sanctum\Sanctum;
use Tests\TestCase;

class CustomerTest extends TestCase
{
    use RefreshDatabase;

    private User $user;

    protected function setUp(): void
    {
        parent::setUp();

        $this->user = User::factory()->create(['is_active' => true]);
        Sanctum::actingAs($this->user);
    }

    public function test_can_list_paginated_customers(): void
    {
        Customer::factory()->count(5)->create();

        $response = $this->getJson('/api/v1/customers');

        $response->assertStatus(200)
            ->assertJsonStructure([
                'data' => [
                    '*' => ['id', 'name', 'email', 'is_active', 'created_at']
                ],
                'links',
                'meta',
            ]);
    }

    public function test_store_customer_persists_in_database(): void
    {
        $payload = [
            'name'  => 'Andi Pratama',
            'email' => 'andi@example.com',
            'phone' => '081234567890',
        ];

        $response = $this->postJson('/api/v1/customers', $payload);

        $response->assertStatus(201)
            ->assertJsonPath('data.name', 'Andi Pratama');

        $this->assertDatabaseHas('mt_customers', [
            'email' => 'andi@example.com',
        ]);
    }
}
```
