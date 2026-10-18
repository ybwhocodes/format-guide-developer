# Database Migration & Seeder Standard — Layered Architecture

> **Status:** Team Standard (Tier 1)  
> **Scope:** Layered Architecture / Framework Standard Laravel  
> **Purpose:** Menjaga integritas skema database, konvensi tabel/kolom terpusat di direktori global Laravel (`database/migrations` dan `database/seeders`).

---

## 1. Lokasi Direktori (Centralized)

Dalam Layered Architecture, seluruh skema dan pengisian data diletakkan terpusat pada folder bawaan Laravel:

```text
database/
├── factories/
│   ├── CustomerFactory.php
│   └── UserFactory.php
├── migrations/
│   ├── 2026_01_01_000000_create_rf_roles_table.php
│   ├── 2026_01_01_000001_create_mt_users_table.php
│   ├── 2026_01_01_000002_create_mt_customers_table.php
│   └── 2026_01_01_000003_create_tr_orders_table.php
└── seeders/
    ├── DatabaseSeeder.php
    ├── RoleSeeder.php
    └── UserSeeder.php
```

---

## 2. Konvensi Penamaan Tabel & Kolom

### Prefix Tabel
- `mt_` : Master data inti (`mt_users`, `mt_customers`, `mt_products`).
- `tr_` : Transaksi time-based (`tr_orders`, `tr_invoices`, `tr_payments`).
- `dt_` : Detail/child transaksi (`dt_order_items`, `dt_invoice_items`).
- `rf_` : Referensi/lookup statis (`rf_roles`, `rf_order_statuses`).
- `lg_` : Log & audit trail (`lg_activity_logs`, `lg_webhook_logs`).
- `cfg_` : Parameter/konfigurasi sistem (`cfg_settings`).
- `tmp_` : Staging/temporary import (`tmp_import_records`).

### Aturan Kolom
- Gunakan `snake_case`.
- Foreign Key: `<singular_entity>_id` (contoh: `customer_id`, `order_id`).
- Boolean: Gunakan predikat kondisi (`is_active`, `is_verified`, `can_cancel`).
- Tanggal/Waktu Domain: Gunakan suffix `_at` (`paid_at`, `approved_at`, `cancelled_at`).
- Soft Delete: Kolom standar `deleted_at`.

---

## 3. Standar Penulisan Migration

- **Reversible:** Method `down()` wajib ada dan benar-benar membalikkan perubahan `up()`.
- **Foreign Key Constraints:** Wajib deklarasikan referensi tabel target, `cascadeOnUpdate()`, dan restriksi hapus (`restrictOnDelete()` untuk master/transaksi).
- **Index:** Wajib tambahkan index untuk kolom pencarian, filter status, foreign key, dan sorting.

### Template Migration (Contoh: `mt_customers`)

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('mt_customers', function (Blueprint $table) {
            $table->id();
            $table->string('name', 150);
            $table->string('email', 150)->unique();
            $table->string('phone', 25)->nullable();
            $table->boolean('is_active')->default(true)->index();
            $table->timestamps();
            $table->softDeletes();

            $table->index(['created_at', 'is_active']);
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('mt_customers');
    }
};
```

---

## 4. Standar Penulisan Seeder

- **Pemisahan Lingkungan:** Pisahkan seeder data dasar/wajib produksi (`RoleSeeder`) dari data testing/dummy dev.
- **Idempotent:** Seeder data referensi wajib aman dijalankan berulang kali (`updateOrInsert` / `firstOrCreate`).

```php
namespace Database\Seeders;

use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\DB;

class RoleSeeder extends Seeder
{
    public function run(): void
    {
        $roles = [
            ['code' => 'SUPERADMIN', 'name' => 'Super Administrator'],
            ['code' => 'ADMIN',      'name' => 'Administrator'],
            ['code' => 'STAFF',      'name' => 'Staff Operasional'],
        ];

        foreach ($roles as $role) {
            DB::table('rf_roles')->updateOrInsert(
                ['code' => $role['code']],
                [
                    'name'       => $role['name'],
                    'updated_at' => now(),
                    'created_at' => now(),
                ]
            );
        }
    }
}
```

### DatabaseSeeder

```php
namespace Database\Seeders;

use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\App;

class DatabaseSeeder extends Seeder
{
    public function run(): void
    {
        $this->call([
            RoleSeeder::class,
        ]);

        if (!App::environment('production')) {
            $this->call([
                CustomerSeeder::class,
            ]);
        }
    }
}
```
