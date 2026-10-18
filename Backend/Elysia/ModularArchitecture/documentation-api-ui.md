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

## 2. Integrasi Scalar pada ElysiaJS

ElysiaJS memiliki plugin resmi `@elysiajs/swagger` yang mendukung Scalar sebagai provider tampilan UI.

### 2.1 Instalasi
```bash
bun add @elysiajs/swagger
```

### 2.2 Konfigurasi
```typescript
import { Elysia } from "elysia";
import { swagger } from "@elysiajs/swagger";

export const app = new Elysia()
  .use(
    swagger({
      provider: "scalar", // Menggunakan UI Scalar
      documentation: {
        info: {
          title: "Dasaria Core API Documentation",
          version: "1.0.0",
          description: "API Standard documentation powered by ElysiaJS and Scalar"
        },
        tags: [
          { name: "Auth", description: "Authentication and authorization endpoints" },
          { name: "Customers", description: "Customer management operations" }
        ],
        components: {
          securitySchemes: {
            bearerAuth: {
              type: "http",
              scheme: "bearer",
              bearerFormat: "JWT"
            }
          }
        }
      },
      path: "/docs" // Akses UI di http://localhost:3000/docs
    })
  );
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
