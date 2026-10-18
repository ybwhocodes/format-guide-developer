# ORM Standard: Eloquent (Laravel Layered Architecture)

> **Status:** Team Standard  
> **Framework:** Laravel  
> **Architecture:** Layered Architecture

## 1. Konvensi Eloquent

Gunakan Eloquent sebagai ORM default Laravel. Model berada pada layer domain/data, sedangkan controller memanggil service dan repository sesuai kebutuhan kompleksitas. Query Eloquent tidak ditempatkan langsung di controller.

- Satu model mewakili satu tabel dan mendefinisikan `$fillable` atau `$guarded` secara eksplisit.
- Gunakan casts, accessors, relationships, scopes, dan Form Request untuk aturan yang sesuai tempatnya.
- Gunakan route model binding hanya untuk resource yang memang diekspos endpoint.
- Gunakan `DB::transaction()` pada service untuk perubahan multi-model.
- Migration menjadi sumber perubahan schema; seeder hanya untuk data awal atau fixture.
- Gunakan eager loading untuk mencegah N+1 dan `select()` untuk membatasi kolom.

## 2. Contoh Model

```php
class Customer extends Model
{
    protected $fillable = ['name', 'email'];

    protected function casts(): array
    {
        return ['email_verified_at' => 'datetime'];
    }
}
```

## 3. Batas Layer

Controller meneruskan input tervalidasi ke service. Service mengatur transaksi dan aturan bisnis. Repository dipakai saat query perlu diabstraksikan atau dipakai lintas service, bukan sebagai wrapper otomatis untuk setiap method Eloquent.
