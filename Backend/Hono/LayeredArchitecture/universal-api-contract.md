# Universal API Contract Standard

> **Status:** Team Standard  
> **Scope:** RESTful API Response, Status Code, & Error Specification  
> **Purpose:** Menjamin konsistensi kontrak response, format error, metadata, pagination, dan konvensi endpoint API di seluruh layanan backend.

---

## 1. REST Endpoint Naming

- Gunakan **noun (kata benda)** jamak, bukan verb (kata kerja).
- Gunakan kebab-case untuk path URL multi-kata.

### Pola CRUD Standar
```text
GET    /api/v1/customers          # Mengambil daftar data
POST   /api/v1/customers          # Membuat data baru
GET    /api/v1/customers/{id}     # Mengambil detail satu data
PUT    /api/v1/customers/{id}     # Mengganti / update penuh satu data
PATCH  /api/v1/customers/{id}     # Update parsial satu data
DELETE /api/v1/customers/{id}     # Menghapus data
```

### Action Bisnis Spesifik
Gunakan sub-resource noun atau verb eksplisit di akhir path:
```text
POST   /api/v1/invoices/{id}/cancel
POST   /api/v1/orders/{id}/confirm
POST   /api/v1/payments/{id}/refund
```

Hindari pola RPC pada REST:
```text
POST /api/v1/createCustomer        # SALAH
GET  /api/v1/getCustomerList       # SALAH
```

---

## 2. HTTP Status Code Standard

| Status Code | Kondisi Penggunaan |
|---|---|
| `200 OK` | Query sukses, update sukses, action non-creation sukses |
| `201 Created` | Resource baru berhasil dibuat |
| `204 No Content` | Delete sukses tanpa body kembalian |
| `400 Bad Request` | Format request invalid / malformed payload |
| `401 Unauthorized` | Autentikasi gagal / token absen / token expired |
| `403 Forbidden` | Terautentikasi tetapi tidak punya izin akses (permission) |
| `404 Not Found` | Resource / URL tidak ditemukan |
| `409 Conflict` | Konflik state bisnis (misal: invoice sudah lunas, idempotency clash) |
| `422 Unprocessable Entity` | Kegagalan validasi payload (validation error) |
| `429 Too Many Requests` | Melebihi rate limit |
| `500 Internal Server Error` | Kesalahan internal server yang tidak tertangani |

> [!WARNING]
> Dilarang mengembalikan status HTTP `200 OK` untuk payload yang berstatus error.

---

## 3. Format Response Standard

Gunakan **Meta JSON** sebagai kontrak default untuk seluruh API produksi. Gunakan **Simple JSON** hanya untuk service internal atau service sederhana yang sudah mendapat persetujuan eksplisit. Satu service tidak boleh mencampur format tanpa keputusan kontrak tertulis.

### 3.1 Simple JSON Response (Khusus Internal / Service Sederhana)
Gunakan hanya untuk endpoint internal atau service sederhana yang telah disetujui eksplisit; format ini bukan default produksi.

**Success:**
```json
{
  "success": true,
  "message": "Customer created successfully",
  "data": {
    "id": 1,
    "name": "John Doe",
    "email": "john@example.com"
  }
}
```

**Error:**
```json
{
  "success": false,
  "message": "Customer not found",
  "data": null
}
```

---

### 3.2 Meta JSON Response (Standard Production / Enterprise API)
Gunakan untuk sistem modular, API publik, dashboard, dan layanan dengan kebutuhan metadata/tracing.

**Single Resource Success:**
```json
{
  "meta": {
    "success": true,
    "message": "Customer retrieved successfully",
    "code": 200
  },
  "data": {
    "id": 1,
    "name": "John Doe",
    "email": "john@example.com"
  }
}
```

**Paginated Collection Success:**
```json
{
  "meta": {
    "success": true,
    "message": "Customers retrieved successfully",
    "code": 200,
    "pagination": {
      "current_page": 1,
      "per_page": 20,
      "total": 125,
      "last_page": 7
    }
  },
  "data": [
    {
      "id": 1,
      "name": "John Doe"
    }
  ]
}
```

---

## 4. Error Response Standard

### 4.1 Validation Error (`422 Unprocessable Entity`)
```json
{
  "meta": {
    "success": false,
    "message": "Validation failed",
    "code": 422
  },
  "errors": {
    "email": [
      "The email field is required."
    ],
    "phone": [
      "The phone format is invalid."
    ]
  }
}
```

### 4.2 Business Rule Error (`409 Conflict` / `400 Bad Request`)
Sertakan `error_code` berupa enum snake/screaming-snake-case agar klien/frontend dapat melakukan handling spesifik:
```json
{
  "meta": {
    "success": false,
    "message": "Invoice cannot be cancelled after settlement",
    "code": 409,
    "error_code": "INVOICE_ALREADY_SETTLED"
  },
  "data": null
}
```

**Daftar Error Code Standar:**
```text
AUTH_INVALID_CREDENTIALS
AUTH_TOKEN_EXPIRED
FORBIDDEN_ACTION

CUSTOMER_NOT_FOUND
CUSTOMER_INACTIVE
CUSTOMER_EMAIL_ALREADY_EXISTS

ORDER_NOT_FOUND
ORDER_ALREADY_CONFIRMED
ORDER_ALREADY_CANCELLED

INVOICE_NOT_FOUND
INVOICE_ALREADY_PAID
INVOICE_ALREADY_CANCELLED

PAYMENT_FAILED
PAYMENT_DUPLICATE
PAYMENT_ALREADY_REFUNDED

STOCK_NOT_AVAILABLE
STOCK_RESERVATION_FAILED
```

---

## 5. Parameter Query: Filter, Sort, & Search

Format URL standar:
```text
GET /api/v1/customers?search=andi&status=active&sort=-created_at&page=1&per_page=20
```

- `search`: Keyword pencarian teks bebas.
- `sort`: Prefix `-` untuk descending (`sort=-created_at`), tanpa prefix untuk ascending (`sort=created_at`).
- `page`: Nomor halaman (1-indexed).
- `per_page`: Jumlah record per request (beri batas atas tegas, misal: max 100).

---

## 6. API Versioning & Idempotency

- **URL Versioning:** `/api/v1/...`. Naikkan versi (`/api/v2/`) hanya jika terdapat breaking change pada kontrak.
- **Idempotency Key:** Untuk endpoint transaksi finansial, checkout order, atau pembayaran, wajib sediakan header:
  ```text
  Idempotency-Key: <unique-uuid>
  ```
  Backend menyimpan status pemrosesan key tersebut untuk mencegah eksekusi ganda saat re-try network.
