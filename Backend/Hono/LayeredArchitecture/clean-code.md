# Clean Code Standard — Hono Layered Architecture

> **Status:** Team Standard (Tier 1)  
> **Framework:** Hono (TypeScript)  
> **Pattern:** Package by Layer (Layered Architecture)  
> **Purpose:** Standar penulisan clean code Hono berbasis TypeScript, Zod OpenAPI Validator (@hono/zod-validator), dan Service-Repository pattern.

---

## 1. Struktur Direktori Proyek

```text
src/
├── controllers/
│   └── customer.controller.ts
├── services/
│   └── customer.service.ts
├── repositories/
│   ├── customer.repository.ts
│   └── interfaces/
│       └── customer-repository.interface.ts
├── schemas/
│   └── customer.schema.ts
├── middlewares/
│   ├── auth.middleware.ts
│   └── error.middleware.ts
└── index.ts
```

---

## 2. Implementasi Tiap Layer

### 2.1 Skema & Validasi (`src/schemas/customer.schema.ts`)
```typescript
import { z } from "zod";

export const CreateCustomerSchema = z.object({
  name: z.string().min(3).max(150),
  email: z.string().email().max(150),
  phone: z.string().max(25).optional(),
  isActive: z.boolean().default(true).optional()
});

export const CustomerIdParamSchema = z.object({
  id: z.coerce.number()
});

export type CreateCustomerDTO = z.infer<typeof CreateCustomerSchema>;
```

### 2.2 Repository Interface & Implementation (`src/repositories/`)
```typescript
// interfaces/customer-repository.interface.ts
export interface CustomerEntity {
  id: number;
  name: string;
  email: string;
  phone: string | null;
  isActive: boolean;
  createdAt: Date;
}

export interface ICustomerRepository {
  findById(id: number): Promise<CustomerEntity | null>;
  findByEmail(email: string): Promise<CustomerEntity | null>;
  create(payload: { name: string; email: string; phone?: string; isActive?: boolean }): Promise<CustomerEntity>;
}
```

```typescript
// customer.repository.ts
import { ICustomerRepository, CustomerEntity } from "./interfaces/customer-repository.interface";

export class CustomerRepository implements ICustomerRepository {
  constructor(private readonly db: any) {}

  async findById(id: number): Promise<CustomerEntity | null> {
    return this.db.customer.findUnique({ where: { id } });
  }

  async findByEmail(email: string): Promise<CustomerEntity | null> {
    return this.db.customer.findUnique({ where: { email } });
  }

  async create(payload: any): Promise<CustomerEntity> {
    return this.db.customer.create({ data: payload });
  }
}
```

### 2.3 Service Layer (`src/services/customer.service.ts`)
```typescript
import { ICustomerRepository, CustomerEntity } from "../repositories/interfaces/customer-repository.interface";
import { CreateCustomerDTO } from "../schemas/customer.schema";

export class CustomerService {
  constructor(private readonly customerRepo: ICustomerRepository) {}

  async register(dto: CreateCustomerDTO): Promise<CustomerEntity> {
    const existing = await this.customerRepo.findByEmail(dto.email);
    if (existing) {
      throw new Error("CUSTOMER_EMAIL_ALREADY_EXISTS");
    }

    return this.customerRepo.create({
      name: dto.name,
      email: dto.email,
      phone: dto.phone,
      isActive: dto.isActive ?? true
    });
  }

  async getById(id: number): Promise<CustomerEntity> {
    const customer = await this.customerRepo.findById(id);
    if (!customer) {
      throw new Error("CUSTOMER_NOT_FOUND");
    }
    return customer;
  }
}
```

### 2.4 Controller Layer (`src/controllers/customer.controller.ts`)
Menggunakan `@hono/zod-validator`:
```typescript
import { Hono } from "hono";
import { zValidator } from "@hono/zod-validator";
import { CustomerService } from "../services/customer.service";
import { CreateCustomerSchema, CustomerIdParamSchema } from "../schemas/customer.schema";

export const createCustomerController = (customerService: CustomerService) => {
  const router = new Hono();

  router.post(
    "/",
    zValidator("json", CreateCustomerSchema, (result, c) => {
      if (!result.success) {
        return c.json(
          { meta: { success: false, message: "Validation failed", code: 422 }, errors: result.error.errors },
          422
        );
      }
    }),
    async (c) => {
      const body = c.req.valid("json");
      const customer = await customerService.register(body);

      return c.json(
        { meta: { success: true, message: "Customer created", code: 201 }, data: customer },
        201
      );
    }
  );

  router.get(
    "/:id",
    zValidator("param", CustomerIdParamSchema),
    async (c) => {
      const { id } = c.req.valid("param");
      const customer = await customerService.getById(id);

      return c.json(
        { meta: { success: true, message: "Customer retrieved", code: 200 }, data: customer },
        200
      );
    }
  );

  return router;
};
```
