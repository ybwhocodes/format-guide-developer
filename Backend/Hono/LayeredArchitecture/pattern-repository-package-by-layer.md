# Repository Pattern Standard (Package by Layer) — Hono

> **Status:** Team Standard (Tier 1)  
> **Framework:** Hono (TypeScript)  
> **Pattern:** Package by Layer  
> **Purpose:** Standar implementasi repository pattern terpusat pada framework Hono.

---

## 1. Konsep & Hirarki File

```text
src/
├── repositories/
│   ├── interfaces/
│   │   ├── base.interface.ts
│   │   └── customer.interface.ts
│   └── customer.repository.ts
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
  createdAt: Date;
}

export interface ICustomerRepository {
  findById(id: number): Promise<Customer | null>;
  findByEmail(email: string): Promise<Customer | null>;
  create(payload: { name: string; email: string; phone?: string; isActive?: boolean }): Promise<Customer>;
}
```

Implementasi concrete Drizzle / Prisma:
```typescript
// src/repositories/customer.repository.ts
import { ICustomerRepository, Customer } from "./interfaces/customer.interface";

export class CustomerRepository implements ICustomerRepository {
  constructor(private readonly db: any) {}

  async findById(id: number): Promise<Customer | null> {
    return this.db.customer.findUnique({ where: { id } });
  }

  async findByEmail(email: string): Promise<Customer | null> {
    return this.db.customer.findUnique({ where: { email } });
  }

  async create(payload: any): Promise<Customer> {
    return this.db.customer.create({ data: payload });
  }
}
```
