# Clean Code & Module Standard — ElysiaJS Modular Architecture

> **Status:** Scaled System Standard (Tier 2)  
> **Framework:** ElysiaJS (TypeScript)  
> **Pattern:** Package by Feature / Modular Architecture  
> **Purpose:** Standar penulisan kode modular di ElysiaJS, isolasi domain per folder, sub-plugin routing, dan typed contract per modul.

---

## 1. Konsep & Struktur Direktori

Setiap domain fitur dibungkus menjadi modul plugin Elysia mandiri:

```text
src/
├── core/
│   ├── database/
│   ├── middlewares/
│   └── errors/
│
├── modules/
│   ├── auth/
│   │   ├── auth.controller.ts
│   │   ├── auth.service.ts
│   │   ├── auth.schema.ts
│   │   └── index.ts
│   │
│   ├── customer/
│   │   ├── customer.controller.ts
│   │   ├── customer.service.ts
│   │   ├── customer.repository.ts
│   │   ├── customer.schema.ts
│   │   ├── dtos/
│   │   │   └── create-customer.dto.ts
│   │   └── index.ts # Sub-plugin Elysia untuk module customer
│   │
│   └── order/
│       └── ...
│
└── index.ts # Root Elysia app yang me-mount setiap module plugin
```

---

## 2. Implementasi Feature Module Plugin

### 2.1 Schema & DTO (`src/modules/customer/customer.schema.ts`)
```typescript
import { t } from "elysia";

export const CreateCustomerSchema = t.Object({
  name: t.String({ minLength: 2, maxLength: 100 }),
  email: t.String({ format: "email" }),
  phone: t.Optional(t.String())
});

export const UpdateCustomerSchema = t.Partial(CreateCustomerSchema);
export const CustomerIdParamSchema = t.Object({ id: t.Numeric() });

export type CreateCustomerDTO = typeof CreateCustomerSchema.static;
```

### 2.2 Module Service (`src/modules/customer/customer.service.ts`)
```typescript
import type { CustomerRepository } from "./customer.repository";
import type { CreateCustomerDTO } from "./customer.schema";

export class CustomerService {
  constructor(private readonly repo: CustomerRepository) {}

  async create(dto: CreateCustomerDTO) {
    const exists = await this.repo.findByEmail(dto.email);
    if (exists) throw new Error("CUSTOMER_DUPLICATE_EMAIL");
    return this.repo.create(dto);
  }

  async getById(id: number) {
    const customer = await this.repo.findById(id);
    if (!customer) throw new Error("CUSTOMER_NOT_FOUND");
    return customer;
  }
}
```

### 2.3 Module Plugin Mounting (`src/modules/customer/index.ts`)
Setiap modul mengekspor satu Elysia plugin yang siap dimount ke app root:

```typescript
import { Elysia } from "elysia";
import { CustomerRepository } from "./customer.repository";
import { CustomerService } from "./customer.service";
import { CreateCustomerSchema, CustomerIdParamSchema } from "./customer.schema";

export const customerModule = (db: any) => {
  const repo = new CustomerRepository(db);
  const service = new CustomerService(repo);

  return new Elysia({ prefix: "/api/v1/customers", name: "module.customer" })
    .post(
      "/",
      async ({ body, set }) => {
        const data = await service.create(body);
        set.status = 201;
        return { meta: { success: true, message: "Customer created", code: 201 }, data };
      },
      { body: CreateCustomerSchema }
    )
    .get(
      "/:id",
      async ({ params: { id }, set }) => {
        const data = await service.getById(id);
        set.status = 200;
        return { meta: { success: true, message: "Customer retrieved", code: 200 }, data };
      },
      { params: CustomerIdParamSchema }
    );
};
```

### 2.4 Root Application Mounting (`src/index.ts`)
```typescript
import { Elysia } from "elysia";
import { customerModule } from "./modules/customer";
import { authModule } from "./modules/auth";

const app = new Elysia()
  .use(authModule)
  .use(customerModule(/* db instance */))
  .listen(3000);

console.log(`🦊 Elysia server running at ${app.server?.hostname}:${app.server?.port}`);
```
