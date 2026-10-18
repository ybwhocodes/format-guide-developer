# Clean Code & Module Standard — Express.js Modular Architecture

> **Status:** Scaled System Standard (Tier 2)  
> **Framework:** Express.js (TypeScript)  
> **Pattern:** Package by Feature / Modular Architecture  
> **Purpose:** Standar penulisan Express.js modular, isolasi domain per folder modul, sub-router independen, dan DTO contract.

---

## 1. Struktur Folder Modular

```text
src/
├── core/
│   ├── database/
│   ├── middlewares/
│   │   ├── error-handler.middleware.ts
│   │   └── not-found.middleware.ts
│   └── errors/
│
├── modules/
│   ├── auth/
│   │   ├── auth.controller.ts
│   │   ├── auth.service.ts
│   │   ├── auth.router.ts
│   │   └── dtos/
│   │
│   ├── customer/
│   │   ├── customer.controller.ts
│   │   ├── customer.service.ts
│   │   ├── customer.repository.ts
│   │   ├── customer.router.ts
│   │   ├── customer.validation.ts
│   │   └── dtos/
│   │       └── create-customer.dto.ts
│   │
│   └── order/
│       └── ...
│
└── app.ts # Root loader yang me-mount module router
```

---

## 2. Implementasi Feature Module

### 2.1 Module Router (`src/modules/customer/customer.router.ts`)
```typescript
import { Router } from "express";
import { CustomerController } from "./customer.controller";
import { CustomerService } from "./customer.service";
import { CustomerRepository } from "./customer.repository";
import { validate } from "../../core/middlewares/validate.middleware";
import { createCustomerSchema } from "./customer.validation";

export const buildCustomerModule = (db: any): Router => {
  const router = Router();
  const repo = new CustomerRepository(db);
  const service = new CustomerService(repo);
  const controller = new CustomerController(service);

  router.post("/", validate(createCustomerSchema), controller.store);
  router.get("/:id", controller.show);

  return router;
};
```

### 2.2 App Root Mount (`src/app.ts`)
```typescript
import express from "express";
import { buildCustomerModule } from "./modules/customer/customer.router";
import { buildAuthModule } from "./modules/auth/auth.router";

export const app = express();
app.use(express.json());

// Mount modular sub-routers
app.use("/api/v1/auth", buildAuthModule(/* db */));
app.use("/api/v1/customers", buildCustomerModule(/* db */));
```
