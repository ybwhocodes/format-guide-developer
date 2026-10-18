# ORM Standard: Eloquent (Laravel Modular Architecture)

> **Status:** Team Standard  
> **Framework:** Laravel  
> **Architecture:** Modular Architecture

## 1. Konvensi Eloquent per Module

Gunakan Eloquent sebagai ORM default. Setiap module memiliki model, query concern, repository atau data access service, migration, dan factory yang dekat dengan feature. Module lain tidak boleh mengakses model internal secara langsung.

- Model module berada di boundary module dan memakai `$fillable` atau `$guarded` eksplisit.
- Expose data melalui use case, service, atau contract module.
- Gunakan `DB::transaction()` pada use case yang mengubah beberapa model atau aggregate.
- Definisikan relationship dan local scope di model module.
- Gunakan eager loading dan `select()` untuk mencegah N+1 serta payload berlebih.
- Migration tetap mengikuti urutan deployment terpusat.
- Gunakan event atau contract untuk komunikasi lintas module, bukan query lintas tabel secara langsung.

## 2. Contoh Model Feature

```php
namespace App\Modules\Customer\Infrastructure\Persistence;

class Customer extends Model
{
    protected $table = 'customers';
    protected $fillable = ['name', 'email'];
}
```

Repository module menerjemahkan Eloquent model ke data yang dipakai use case. Route dan controller module tidak boleh membangun query langsung.
