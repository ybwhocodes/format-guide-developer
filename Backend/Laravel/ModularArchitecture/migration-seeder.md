# Database Migration & Seeder Standard — Modular Architecture

> **Status:** Scaled System Standard (Tier 2)  
> **Scope:** Modular / Domain-Driven Architecture in Laravel  
> **Purpose:** Standar pengelolaan migrasi dan seeder yang terlokalisasi di dalam direktori modul (`app/Modules/{Feature}/Database/Migrations`), isolasi domain, dan orchestrator seeder modular.

---

## 1. Lokasi Direktori Per Modul

Setiap modul domain mengelola migrasi, factory, dan seedernya sendiri. Komponen global non-domain diletakkan di `database/` root.

```text
app/
├── Modules/
│   ├── Customer/
│   │   ├── Database/
│   │   │   ├── Migrations/
│   │   │   │   └── 2026_01_01_000000_create_mt_customers_table.php
│   │   │   ├── Factories/
│   │   │   │   └── CustomerFactory.php
│   │   │   └── Seeders/
│   │   │       └── CustomerModuleSeeder.php
│   │   └── Providers/
│   │       └── CustomerServiceProvider.php
│   │
│   ├── Order/
│   │   └── Database/
│   │       ├── Migrations/
│   │       │   ├── 2026_01_02_000000_create_tr_orders_table.php
│   │       │   └── 2026_01_02_000001_create_dt_order_items_table.php
│   │       └── Seeders/
│   │           └── OrderModuleSeeder.php
│   │
│   └── Billing/
│       └── ...
│
database/
├── migrations/          # Hanya untuk migrasi global/core (users, roles, sessions)
└── seeders/
    └── DatabaseSeeder.php # Master orchestrator memanggil module seeders
```

---

## 2. Pendaftaran Migrasi di Module Service Provider

Migrasi modul didaftarkan menggunakan `$this->loadMigrationsFrom()` di dalam method `boot()` Service Provider modul bersangkutan:

```php
namespace App\Modules\Customer\Providers;

use Illuminate\Support\ServiceProvider;

class CustomerServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        $migrationPath = __DIR__ . '/../Database/Migrations';
        if (is_dir($migrationPath)) {
            $this->loadMigrationsFrom($migrationPath);
        }
    }
}
```

---

## 3. Konvensi Tabel & Isolasi Antar Modul

- Setiap tabel domain diawali prefix jenis tabel (`mt_`, `tr_`, `dt_`).
- **Isolasi Database Antar Modul:**
  - Foreign key antar domain tetap boleh menggunakan constraint database (`constrained('mt_customers')`), tetapi **query aplikasi dilarang melakukan direct join lintas domain di luar repository modul pemilik**.
  - Jika tabel transaksi perlu menghapus data detailnya, gunakan `$table->cascadeOnDelete()` pada tabel detail (`dt_`), namun gunakan `restrictOnDelete()` pada relasi lintas modul domain utama.

---

## 4. Standar Module Seeder & Master Orchestration

Setiap modul memiliki seeder sendiri yang bertanggung jawab mengisi data referensi dan dummy modul tersebut:

### Module Seeder (`app/Modules/Customer/Database/Seeders/CustomerModuleSeeder.php`)

```php
namespace App\Modules\Customer\Database\Seeders;

use App\Modules\Customer\Models\Customer;
use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\App;

class CustomerModuleSeeder extends Seeder
{
    public function run(): void
    {
        // Seeder referensi modul
        // Seeder dummy lokal
        if (!App::environment('production')) {
            Customer::factory()->count(10)->create();
        }
    }
}
```

### Master DatabaseSeeder (`database/seeders/DatabaseSeeder.php`)

```php
namespace Database\Seeders;

use App\Modules\Customer\Database\Seeders\CustomerModuleSeeder;
use App\Modules\Order\Database\Seeders\OrderModuleSeeder;
use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    public function run(): void
    {
        // Panggil seeder modul sesuai urutan dependensi domain
        $this->call([
            RoleSeeder::class,               // Core/Global
            CustomerModuleSeeder::class,     // Modul Customer
            OrderModuleSeeder::class,        // Modul Order (bergantung pada customer)
        ]);
    }
}
```
