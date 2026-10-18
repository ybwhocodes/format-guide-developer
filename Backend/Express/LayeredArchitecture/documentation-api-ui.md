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

## 2. Integrasi Scalar pada Express.js

Gunakan middleware resmi `@scalar/express-api-reference`.

### 2.1 Instalasi
```bash
npm install @scalar/express-api-reference
```

### 2.2 Konfigurasi
```typescript
import express from "express";
import { apiReference } from "@scalar/express-api-reference";

const app = express();

// Serve spec JSON (contoh)
app.get("/openapi.json", (req, res) => {
  res.sendFile(path.join(__dirname, "openapi.json"));
});

// UI Scalar di /docs
app.use(
  "/docs",
  apiReference({
    spec: {
      url: "/openapi.json"
    },
    theme: "purple"
  })
);
```
