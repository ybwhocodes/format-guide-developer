# Repository Pattern Standard (Package by Layer) — Express.js

> **Status:** Team Standard (Tier 1)  
> **Framework:** Express.js (TypeScript)  
> **Pattern:** Package by Layer  
> **Purpose:** Standar implementasi repository pattern terpusat di `src/repositories/` untuk Express.js.

---

## 1. Konsep & Hirarki File

```text
src/repositories/
├── interfaces/
│   ├── base.interface.ts
│   └── customer.interface.ts
└── implementations/
    ├── prisma-customer.repository.ts
    └── drizzle-customer.repository.ts
```

---

## 2. Implementasi Interface & Concrete Class

```typescript
// src/repositories/interfaces/customer.interface.ts
export interface Customer {
  id: number;
  name: string;
  email: string;
  phone: string | null;
  isActive: boolean;
}

export interface ICustomerRepository {
  findById(id: number): Promise<Customer | null>;
  findByEmail(email: string): Promise<Customer | null>;
  create(payload: Omit<Customer, "id">): Promise<Customer>;
  update(id: number, payload: Partial<Customer>): Promise<Customer>;
  delete(id: number): Promise<boolean>;
}
```

Implementasi concrete Prisma:
```typescript
// src/repositories/implementations/prisma-customer.repository.ts
import { PrismaClient } from "@prisma/client";
import { ICustomerRepository, Customer } from "../interfaces/customer.interface";

export class PrismaCustomerRepository implements ICustomerRepository {
  constructor(private readonly prisma: PrismaClient) {}

  async findById(id: number): Promise<Customer | null> {
    return this.prisma.customer.findUnique({ where: { id } });
  }

  async findByEmail(email: string): Promise<Customer | null> {
    return this.prisma.customer.findUnique({ where: { email } });
  }

  async create(payload: Omit<Customer, "id">): Promise<Customer> {
    return this.prisma.customer.create({ data: payload });
  }

  async update(id: number, payload: Partial<Customer>): Promise<Customer> {
    return this.prisma.customer.update({ where: { id }, data: payload });
  }

  async delete(id: number): Promise<boolean> {
    await this.prisma.customer.delete({ where: { id } });
    return true;
  }
}
```
