# Clean Code & Module Standard — Hono Modular Architecture

> **Status:** Scaled System Standard (Tier 2)  
> **Framework:** Hono (TypeScript)  
> **Pattern:** Package by Feature / Modular Architecture  
> **Purpose:** Standar penulisan Hono modular, isolasi sub-app per folder domain, dan route grouping mandiri.

---

## 1. Konsep & Struktur Direktori

Setiap modul domain membuat sub-instance `Hono` yang didelegasikan ke root app melalui method `.route()`:

```text
src/
├── core/
│   ├── database/
│   └── middlewares/
│
├── modules/
│   ├── auth/
│   │   ├── auth.controller.ts
│   │   ├── auth.service.ts
│   │   └── index.ts
│   │
│   ├── customer/
│   │   ├── customer.controller.ts
│   │   ├── customer.service.ts
│   │   ├── customer.repository.ts
│   │   ├── customer.schema.ts
│   │   └── index.ts # Sub-app Hono untuk feature customer
│   │
│   └── order/
│       └── ...
│
└── index.ts # Root Hono app
```

---

## 2. Implementasi Feature Module (`src/modules/customer/index.ts`)

```typescript
import { Hono } from "hono";
import { zValidator } from "@hono/zod-validator";
import { CustomerRepository } from "./customer.repository";
import { CustomerService } from "./customer.service";
import { CreateCustomerSchema } from "./customer.schema";

export const buildCustomerModule = (db: any) => {
  const customerApp = new Hono();
  const repo = new CustomerRepository(db);
  const service = new CustomerService(repo);

  customerApp.post(
    "/",
    zValidator("json", CreateCustomerSchema),
    async (c) => {
      const body = c.req.valid("json");
      const result = await service.create(body);
      return c.json({ meta: { success: true, message: "Created", code: 201 }, data: result }, 201);
    }
  );

  customerApp.get("/:id", async (c) => {
    const id = Number(c.req.param("id"));
    const result = await service.getById(id);
    return c.json({ meta: { success: true, code: 200 }, data: result }, 200);
  });

  return customerApp;
};
```

---

## 3. Root Application Mounting (`src/index.ts`)

```typescript
import { Hono } from "hono";
import { buildCustomerModule } from "./modules/customer";
import { buildAuthModule } from "./modules/auth";

const app = new Hono();

// Mount modular sub-apps
app.route("/api/v1/auth", buildAuthModule(/* db */));
app.route("/api/v1/customers", buildCustomerModule(/* db */));

export default app;
```
