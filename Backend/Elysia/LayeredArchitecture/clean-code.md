# Clean Code Standard — ElysiaJS Layered Architecture

> **Status:** Team Standard (Tier 1)  
> **Framework:** ElysiaJS (TypeScript)  
> **Pattern:** Package by Layer (Layered Architecture)  
> **Purpose:** Standar penulisan kode bersih, pemisahan layer, TypeBox schema validation, dan service-repository pattern di ElysiaJS.

---

## 1. Struktur Direktori

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
├── models/ / db/
│   └── schema.ts
├── middlewares/
│   ├── error.middleware.ts
│   └── auth.middleware.ts
└── index.ts
```

---

## 2. Prinsip Layering & TypeScript Rules

- **Strict Type Checking:** `strict: true` di `tsconfig.json`. Dilarang menggunakan type `any`.
- **Elysia Schema Validation (`t` dari `elysia`):** Validasi request body, params, dan query menggunakan TypeBox bawaan Elysia untuk inferensi tipe otomatis.
- **Controller Layer:** Bertugas menangani routing HTTP, validasi skema via hook, dan memanggil Service. Tidak boleh berisi query DB.
- **Service Layer:** Menangani domain logic, business rules, hashing, dan transaksi data.
- **Repository Layer:** Abstraksi akses database (Drizzle, Prisma, atau Sequelize).

---

## 3. Implementasi Kode Tiap Layer

### 3.1 Skema & Validasi (`src/schemas/customer.schema.ts`)
```typescript
import { t } from "elysia";

export const CreateCustomerSchema = t.Object({
  name: t.String({ minLength: 3, maxLength: 150 }),
  email: t.String({ format: "email", maxLength: 150 }),
  phone: t.Optional(t.String({ maxLength: 25 })),
  isActive: t.Optional(t.Boolean({ default: true }))
});

export const CustomerParamSchema = t.Object({
  id: t.Numeric()
});

export type CreateCustomerInput = typeof CreateCustomerSchema.static;
```

### 3.2 Repository Interface & Implementation (`src/repositories/`)
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
  create(data: { name: string; email: string; phone?: string; isActive?: boolean }): Promise<CustomerEntity>;
  findAll(limit: number, offset: number): Promise<CustomerEntity[]>;
}
```

```typescript
// customer.repository.ts
import type { ICustomerRepository, CustomerEntity } from "./interfaces/customer-repository.interface";

export class CustomerRepository implements ICustomerRepository {
  // Contoh menggunakan instance DB (Drizzle / Prisma)
  constructor(private readonly db: any) {}

  async findById(id: number): Promise<CustomerEntity | null> {
    return this.db.query.customers.findFirst({
      where: (customers: any, { eq }: any) => eq(customers.id, id)
    });
  }

  async findByEmail(email: string): Promise<CustomerEntity | null> {
    return this.db.query.customers.findFirst({
      where: (customers: any, { eq }: any) => eq(customers.email, email)
    });
  }

  async create(data: { name: string; email: string; phone?: string; isActive?: boolean }): Promise<CustomerEntity> {
    const [created] = await this.db.insert(this.db.customers).values(data).returning();
    return created;
  }

  async findAll(limit = 20, offset = 0): Promise<CustomerEntity[]> {
    return this.db.query.customers.findMany({ limit, offset });
  }
}
```

### 3.3 Service Layer (`src/services/customer.service.ts`)
```typescript
import type { ICustomerRepository, CustomerEntity } from "../repositories/interfaces/customer-repository.interface";
import type { CreateCustomerInput } from "../schemas/customer.schema";

export class CustomerService {
  constructor(private readonly customerRepo: ICustomerRepository) {}

  async registerCustomer(input: CreateCustomerInput): Promise<CustomerEntity> {
    const existing = await this.customerRepo.findByEmail(input.email);
    if (existing) {
      throw new Error("CUSTOMER_EMAIL_ALREADY_EXISTS");
    }

    return this.customerRepo.create({
      name: input.name,
      email: input.email,
      phone: input.phone,
      isActive: input.isActive ?? true
    });
  }

  async getCustomer(id: number): Promise<CustomerEntity> {
    const customer = await this.customerRepo.findById(id);
    if (!customer) {
      throw new Error("CUSTOMER_NOT_FOUND");
    }
    return customer;
  }
}
```

### 3.4 Controller Layer (`src/controllers/customer.controller.ts`)
```typescript
import { Elysia } from "elysia";
import { CustomerService } from "../services/customer.service";
import { CreateCustomerSchema, CustomerParamSchema } from "../schemas/customer.schema";

export const customerController = (customerService: CustomerService) =>
  new Elysia({ prefix: "/customers" })
    .post(
      "/",
      async ({ body, set }) => {
        const result = await customerService.registerCustomer(body);
        set.status = 201;
        return {
          meta: { success: true, message: "Customer created", code: 201 },
          data: result
        };
      },
      {
        body: CreateCustomerSchema
      }
    )
    .get(
      "/:id",
      async ({ params: { id }, set }) => {
        const result = await customerService.getCustomer(id);
        set.status = 200;
        return {
          meta: { success: true, message: "Customer retrieved", code: 200 },
          data: result
        };
      },
      {
        params: CustomerParamSchema
      }
    );
```
