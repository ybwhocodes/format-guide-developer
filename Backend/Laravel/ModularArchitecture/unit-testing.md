# Unit & Feature Testing Standard — Modular Architecture

> **Status:** Scaled System Standard (Tier 2)  
> **Scope:** Modular / Domain-Driven Architecture in Laravel  
> **Purpose:** Standar pengujian otomatis terdistribusi di level modul domain (`app/Modules/{Feature}/Tests`), isolasi pengujian antar-domain, dan pipeline testing modular.

---

## 1. Struktur Folder Pengujian Modular

Pengujian didekatkan ke modul pemilik fitur. Setiap modul domain memiliki sub-direktori `Tests/Unit` dan `Tests/Feature`:

```text
app/
├── Modules/
│   ├── Customer/
│   │   ├── Tests/
│   │   │   ├── Unit/
│   │   │   │   └── CustomerServiceTest.php
│   │   │   └── Feature/
│   │   │       ├── CreateCustomerApiTest.php
│   │   │       └── CustomerAuthorizationTest.php
│   │   └── ...
│   │
│   └── Order/
│       └── Tests/
│           ├── Unit/
│           │   └── OrderPricingCalculatorTest.php
│           └── Feature/
│               └── OrderCheckoutApiTest.php
│
tests/
└── TestCase.php # Base test case kelas induk
```

---

## 2. Konfigurasi `phpunit.xml` untuk Test Suite Modular

Daftarkan modul ke dalam konfigurasi test suite agar dapat dieksekusi per-modul atau sekaligus:

```xml
<testsuites>
    <testsuite name="CustomerModule">
        <directory>app/Modules/Customer/Tests</directory>
    </testsuite>
    <testsuite name="OrderModule">
        <directory>app/Modules/Order/Tests</directory>
    </testsuite>
</testsuites>
```

Menjalankan tes khusus satu modul:
```bash
php artisan test --testsuite=CustomerModule
```

---

## 3. Contoh Unit Test Modular (`app/Modules/Customer/Tests/Unit/CustomerServiceTest.php`)

Menguji business logic murni Service dan integritas DTO tanpa interaksi HTTP atau database eksternal:

```php
namespace App\Modules\Customer\Tests\Unit;

use App\Modules\Customer\DTOs\CreateCustomerDTO;
use App\Modules\Customer\Interfaces\CustomerRepositoryInterface;
use App\Modules\Customer\Models\Customer;
use App\Modules\Customer\Services\CustomerService;
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

    public function test_register_customer_creates_record_via_dto(): void
    {
        $dto = new CreateCustomerDTO(
            name: 'Budi Santoso',
            email: 'budi@example.com',
            phone: '08123456789'
        );

        $expectedCustomer = new Customer($dto->toArray());
        $expectedCustomer->id = 10;

        $this->repositoryMock
            ->shouldReceive('create')
            ->once()
            ->with($dto->toArray())
            ->andReturn($expectedCustomer);

        $result = $this->service->registerCustomer($dto);

        $this->assertSame(10, $result->id);
        $this->assertSame('Budi Santoso', $result->name);
    }

    protected function tearDown(): void
    {
        Mockery::close();
        parent::tearDown();
    }
}
```

---

## 4. Contoh Feature Test Modular (`app/Modules/Customer/Tests/Feature/CreateCustomerApiTest.php`)

Menguji endpoint API modul secara menyeluruh, termasuk validasi DTO, otentikasi Sanctum, dan Policy domain:

```php
namespace App\Modules\Customer\Tests\Feature;

use App\Models\User;
use App\Modules\Customer\Models\Customer;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Laravel\Sanctum\Sanctum;
use Tests\TestCase;

class CreateCustomerApiTest extends TestCase
{
    use RefreshDatabase;

    private User $adminUser;

    protected function setUp(): void
    {
        parent::setUp();

        $this->adminUser = User::factory()->create([
            'is_active' => true,
        ]);

        Sanctum::actingAs($this->adminUser);
    }

    public function test_can_create_customer_via_modular_endpoint(): void
    {
        $payload = [
            'name'  => 'PT Solusi Bangsa',
            'email' => 'contact@solusi.com',
            'phone' => '0215551234',
        ];

        $response = $this->postJson('/api/v1/customers', $payload);

        $response->assertStatus(201)
            ->assertJsonPath('meta.code', 201)
            ->assertJsonPath('data.name', 'PT Solusi Bangsa');

        $this->assertDatabaseHas('mt_customers', [
            'email' => 'contact@solusi.com',
        ]);
    }
}
```
