# API Documentation UI Standard: Scalar — Modular Architecture

> **Status:** Scaled System Standard (Tier 2)  
> **Framework:** Laravel (Modular / Domain-Driven Architecture)  
> **Scope:** Multi-Domain OpenAPI aggregation using Scalar  
> **Purpose:** Standar dokumentasi API interaktif menggunakan Scalar pada sistem modular, mendukung pengelompokan tag per-domain/fitur atau multi-spec per-modul.

---

## 1. Konsep Modular API Documentation

Pada arsitektur modular, dokumentasi API harus mencerminkan batasan domain:
- **Domain Tagging:** Setiap endpoint dari modul (`app/Modules/{Feature}`) dikelompokkan dalam Tag Domain OpenAPI yang jelas (contoh: `Customer`, `Billing`, `Inventory`, `Auth`).
- **Unified Portal atau Multi-Spec:** Scalar dapat menyajikan satu portal gabungan seluruh domain atau dropdown pemilihan spec per-modul domain.

---

## 2. Struktur Modul & Penempatan Route Docs

Dokumentasi disajikan melalui Core Module atau router web terpusat:

```text
app/
├── Core/
│   ├── Docs/
│   │   ├── Controllers/
│   │   │   └── ApiDocsController.php
│   │   └── Views/
│   │       └── scalar-portal.blade.php
│
├── Modules/
│   ├── Customer/
│   │   └── routes.php # Di-tag ke Tag "Customer"
│   ├── Billing/
│   │   └── routes.php # Di-tag ke Tag "Billing"
│   └── Auth/
│       └── routes.php # Di-tag ke Tag "Auth"
```

---

## 3. Implementasi Scalar di Laravel Modular

### 3.1 Otomatisasi dengan Scramble (Multi-Domain Tags)

Jika menggunakan `dedoc/scramble`, konfigurasikan tag generator agar membaca struktur direktori `app/Modules/`:

```php
// config/scramble.php
return [
    'ui' => [
        'title' => 'Enterprise Modular API Reference',
        'theme' => 'deepSpace',
        'hide_try_it' => false,
    ],
    'info' => [
        'version' => '2.0.0',
        'description' => 'Comprehensive multi-domain API reference for modular backend services.',
    ],
    // Mengelompokkan routes modular ke masing-masing domain tags
    'routes' => [
        'api/v1/*',
    ],
];
```

---

### 3.2 Custom Multi-Spec Scalar Portal (Blade View)

Jika setiap modul menghasilkan file OpenAPI spec sendiri (`/api/v1/customer/openapi.json`, `/api/v1/billing/openapi.json`), sajikan view Scalar terpadu:

```php
// app/Core/Docs/Controllers/ApiDocsController.php
namespace App\Core\Docs\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\View\View;

class ApiDocsController extends Controller
{
    public function __invoke(): View
    {
        return view('core::scalar-portal', [
            'modules' => [
                ['name' => 'All Modules', 'url' => '/docs/openapi.json'],
                ['name' => 'Customer Module', 'url' => '/docs/customer-openapi.json'],
                ['name' => 'Billing Module', 'url' => '/docs/billing-openapi.json'],
            ],
        ]);
    }
}
```

```html
<!-- resources/views/scalar-portal.blade.php -->
<!doctype html>
<html>
  <head>
    <title>Enterprise API Reference - Scalar</title>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <style>
      body { margin: 0; }
    </style>
  </head>
  <body>
    <script
      id="api-reference"
      data-url="/docs/openapi.json"
      data-configuration='{
        "theme": "deepSpace",
        "layout": "modern",
        "showSidebar": true
      }'>
    </script>
    <script src="https://cdn.jsdelivr.net/npm/@scalar/api-reference"></script>
  </body>
</html>
```

---

## 4. Otorisasi Akses Portal Modular

Endpoint dokumentasi modular dilindungi middleware otorisasi:

```php
Route::get('/docs/api', ApiDocsController::class)
    ->middleware(['web', 'auth', 'can:view-api-docs']);
```
