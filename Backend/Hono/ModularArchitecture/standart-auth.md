# Authentication & Authorization Standard — Hono Modular Architecture

> **Status:** Scaled System Standard (Tier 2)  
> **Framework:** Hono (TypeScript)  
> **Pattern:** Package by Feature  
> **Purpose:** Standar pembuatan modul dedicated Auth (`src/modules/auth`) pada Hono modular.

---

## 1. Struktur Modul Auth

```text
src/modules/auth/
├── auth.controller.ts
├── auth.service.ts
├── middlewares/
│   └── auth-guard.middleware.ts
└── index.ts # Menghasilkan sub-app auth dan mengekspor guard
```

---

## 2. Guard Ekspor dari Modul Auth

```typescript
// src/modules/auth/middlewares/auth-guard.middleware.ts
import { createMiddleware } from "hono/factory";
import { verify } from "hono/jwt";

export const requireAuth = createMiddleware(async (c, next) => {
  const token = c.req.header("Authorization")?.replace("Bearer ", "");
  if (!token) {
    return c.json({ meta: { success: false, message: "Token missing", code: 401 } }, 401);
  }

  try {
    const payload = await verify(token, process.env.JWT_SECRET || "secret");
    c.set("jwtPayload", payload);
    await next();
  } catch {
    return c.json({ meta: { success: false, message: "Invalid token", code: 401 } }, 401);
  }
});
```

---

## 3. Pemakaian di Feature Module Lain

```typescript
// src/modules/customer/index.ts
import { Hono } from "hono";
import { requireAuth } from "../auth/middlewares/auth-guard.middleware";

export const buildCustomerModule = (db: any) => {
  const app = new Hono();
  app.use("*", requireAuth); // Proteksi seluruh endpoint module ini

  app.get("/me", (c) => c.json({ data: "My Profile" }));
  return app;
};
```
