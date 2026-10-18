# API Documentation UI Standard: Scalar — Layered Architecture

> **Status:** Team Standard (Tier 1)  
> **Framework:** Laravel (Layered Architecture)  
> **Scope:** OpenAPI / Swagger UI replacement using Scalar  
> **Purpose:** Standar integrasi dokumentasi API interaktif berbasis OpenAPI menggunakan Scalar di Laravel dengan skema controller terpusat.

---

## 1. Mengapa Scalar di Laravel?

Scalar memberikan antarmuka dokumentasi API modern pengganti Swagger UI:
- Desain modern, cepat, dan responsif.
- Built-in API testing client (REST Playground terintegrasi).
- Dukungan OpenAPI 3.0 & 3.1 (JSON/YAML).
- Mudah diintegrasikan dengan generator OpenAPI Laravel (`dedoc/scramble` atau `l5-swagger`).

---

## 2. Pilihan Integrasi di Laravel

Terdapat 2 pendekatan standar untuk menyajikan Scalar di Laravel:

### Pendekatan A: Menggunakan Package `dedoc/scramble` (Direkomendasikan - Auto Generate)
`scramble` secara otomatis men-generate dokumentasi OpenAPI dari route, Form Request, dan Resource Laravel tanpa anotasi manual, serta mendukung UI Scalar secara native.

#### 1. Instalasi
```bash
composer require dedoc/scramble
```

#### 2. Konfigurasi UI Scalar (`config/scramble.php`)
Pastikan tema dan konfigurasi Scalar diaktifkan:
```php
return [
    'ui' => [
        'title' => 'Dasaria Core API Documentation',
        'theme' => 'purple', // default, moon, purple, solarized
        'hide_try_it' => false,
    ],
    'info' => [
        'version' => '1.0.0',
        'description' => 'API documentation for Layered Laravel Architecture powered by Scalar.',
    ],
];
```

Akses dokumentasi langsung di:
```text
GET /docs/api
```

---

### Pendekatan B: Custom Blade View + Scalar CDN (OpenAPI JSON Manual/L5-Swagger)
Jika menggunakan file OpenAPI statis (`openapi.json` di `storage/` atau `public/`) atau hasil generate `l5-swagger`:

#### 1. Controller Route (`routes/web.php` atau `routes/api.php`)
```php
use Illuminate\Support\Facades\Route;

Route::get('/docs', function () {
    return view('scalar');
});

Route::get('/docs/openapi.json', function () {
    return response()->file(storage_path('api-docs/openapi.json'));
});
```

#### 2. Blade View (`resources/views/scalar.blade.php`)
```html
<!doctype html>
<html>
  <head>
    <title>API Documentation - Scalar</title>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <style>
      body {
        margin: 0;
      }
    </style>
  </head>
  <body>
    <script
      id="api-reference"
      data-url="/docs/openapi.json"
      data-configuration='{
        "theme": "purple",
        "layout": "modern"
      }'>
    </script>
    <script src="https://cdn.jsdelivr.net/npm/@scalar/api-reference"></script>
  </body>
</html>
```

---

## 3. Keamanan Endpoint Dokumentasi

Dokumentasi API pada staging dan production wajib diproteksi menggunakan Gate/Middleware otorisasi:

```php
// app/Providers/AuthServiceProvider.php atau route middleware
use App\Models\User;
use Illuminate\Support\Facades\Gate;

Gate::define('viewApiDocs', function (?User $user) {
    if (app()->environment('local')) {
        return true;
    }
    return $user !== null && $user->hasRole('admin');
});
```
