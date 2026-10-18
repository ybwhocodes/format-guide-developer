# Authentication & Authorization Standard — ElysiaJS Modular Architecture

> **Status:** Scaled System Standard (Tier 2)  
> **Framework:** ElysiaJS (TypeScript)  
> **Pattern:** Package by Feature  
> **Purpose:** Standar pembuatan modul dedicated Auth (`src/modules/auth`) dan guards otorisasi yang dapat dipakai lintas modul di ElysiaJS.

---

## 1. Modul Dedicated Auth (`src/modules/auth`)

Otentikasi dikelompokkan penuh dalam satu modul fitur:

```text
src/modules/auth/
├── auth.controller.ts
├── auth.service.ts
├── auth.schema.ts
├── guards/
│   ├── auth.guard.ts
│   └── role.guard.ts
└── index.ts
```

---

## 2. Guard Exportable untuk Modul Lain

Guard dibuat sebagai macro atau plugin Elysia yang diekspor dari modul Auth:

```typescript
// src/modules/auth/guards/auth.guard.ts
import { Elysia } from "elysia";
import { jwt } from "@elysiajs/jwt";
import { bearer } from "@elysiajs/bearer";

export const authCore = new Elysia({ name: "auth.core" })
  .use(
    jwt({
      name: "jwtAuth",
      secret: process.env.JWT_SECRET || "default-secret"
    })
  )
  .use(bearer())
  .macro(({ onBeforeHandle }) => ({
    isAuth(enabled: boolean) {
      if (!enabled) return;
      onBeforeHandle(async ({ jwtAuth, bearer, set }) => {
        if (!bearer) {
          set.status = 401;
          return { meta: { success: false, message: "Missing token", code: 401 }, data: null };
        }
        const payload = await jwtAuth.verify(bearer);
        if (!payload) {
          set.status = 401;
          return { meta: { success: false, message: "Invalid token", code: 401 }, data: null };
        }
      });
    }
  }));
```

---

## 3. Pemakaian Guard di Feature Module Lain (Contoh: Customer)

```typescript
// src/modules/customer/customer.controller.ts
import { Elysia } from "elysia";
import { authCore } from "../auth/guards/auth.guard";

export const customerRoutes = new Elysia()
  .use(authCore)
  .get("/profile", () => ({ meta: { success: true }, data: { status: "Active" } }), {
    isAuth: true // Macro guard aktif
  });
```
