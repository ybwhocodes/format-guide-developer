# API Documentation UI Standard: Scalar

> **Status:** Team Standard  
> **Scope:** Interactive OpenAPI / Swagger UI replacement using Scalar  
> **Purpose:** Standar integrasi dokumentasi API berbasis OpenAPI Spec dengan antarmuka modern Scalar.

---

## 1. Mengapa Scalar?

Scalar menggantikan Swagger UI tradisional dengan:
- Tampilan antarmuka modern, cepat, dan responsif.
- Dukungan testing request terintegrasi (built-in REST client).
- Integrasi mudah dengan OpenAPI 3.0 / 3.1.
- Tema visual yang bersih dan mudah dikustomisasi.

---

## 2. Integrasi Scalar pada Hono Modular

Hono menyajikan OpenAPI JSON melalui route module atau route dokumentasi terpusat. Scalar membaca spec gabungan dari endpoint tersebut.

### 2.1 Instalasi
```bash
npm add @scalar/hono-api-reference
```

### 2.2 Konfigurasi
```typescript
import { Hono } from "hono";
import { apiReference } from "@scalar/hono-api-reference";

const app = new Hono();

app.get("/openapi.json", (c) => c.json(openApiDocument));
app.get("/docs", apiReference({ spec: { url: "/openapi.json" } }));

export default app;
```

---

## 3. Integrasi Standar Menggunakan OpenAPI Spec JSON/YAML

Jika menggunakan Express atau framework lain, Scalar dapat disajikan via HTML CDN atau middleware `@scalar/express-api-reference`:

```typescript
import { apiReference } from "@scalar/express-api-reference";

app.use(
  "/reference",
  apiReference({
    spec: {
      url: "/openapi.json"
    },
    theme: "purple"
  })
);
```
