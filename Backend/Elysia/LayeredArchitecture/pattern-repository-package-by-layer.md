# Repository Pattern Standard (Package by Layer) — ElysiaJS

> **Status:** Team Standard (Tier 1)  
> **Framework:** ElysiaJS (TypeScript)  
> **Pattern:** Package by Layer  
> **Purpose:** Standar pemisahan abstraksi akses data (Repository Pattern) secara terpusat pada layer standar framework ElysiaJS.

---

## 1. Konsep & Arsitektur

Pada **Package by Layer**, seluruh antarmuka dan implementasi repository disimpan terpusat di folder `src/repositories/`. 

```text
src/
├── repositories/
│   ├── interfaces/
│   │   ├── base-repository.interface.ts
│   │   ├── customer-repository.interface.ts
│   │   └── order-repository.interface.ts
│   ├── customer.repository.ts
│   └── order.repository.ts
```

- **Service** hanya bergantung pada `ICustomerRepository`, bukan pada detail ORM.
- Memungkinkan pergantian engine ORM (Prisma/Drizzle/Sequelize) tanpa mengubah kode Service.

---

## 2. Generic Base Repository Contract

```typescript
// src/repositories/interfaces/base-repository.interface.ts
export interface IBaseRepository<T, CreateDTO, UpdateDTO> {
  findById(id: number | string): Promise<T | null>;
  findAll(limit: number, offset: number): Promise<T[]>;
  create(data: CreateDTO): Promise<T>;
  update(id: number | string, data: UpdateDTO): Promise<T>;
  delete(id: number | string): Promise<boolean>;
}
```

---

## 3. Spesifik Repository Implementation

```typescript
// src/repositories/interfaces/customer-repository.interface.ts
import type { IBaseRepository } from "./base-repository.interface";

export interface Customer {
  id: number;
  name: string;
  email: string;
  phone: string | null;
  isActive: boolean;
  createdAt: Date;
}

export type CreateCustomerDTO = Omit<Customer, "id" | "createdAt">;
export type UpdateCustomerDTO = Partial<CreateCustomerDTO>;

export interface ICustomerRepository extends IBaseRepository<Customer, CreateCustomerDTO, UpdateCustomerDTO> {
  findByEmail(email: string): Promise<Customer | null>;
  countActive(): Promise<number>;
}
```

Implementasi menggunakan Dependency Injection sederhana pada instansiasi aplikasi:

```typescript
// src/repositories/customer.repository.ts
import type { ICustomerRepository, Customer, CreateCustomerDTO, UpdateCustomerDTO } from "./interfaces/customer-repository.interface";

export class CustomerRepository implements ICustomerRepository {
  constructor(private readonly db: any) {}

  async findById(id: number): Promise<Customer | null> {
    return this.db.customer.findUnique({ where: { id } });
  }

  async findByEmail(email: string): Promise<Customer | null> {
    return this.db.customer.findUnique({ where: { email } });
  }

  async findAll(limit: number, offset: number): Promise<Customer[]> {
    return this.db.customer.findMany({ take: limit, skip: offset });
  }

  async create(data: CreateCustomerDTO): Promise<Customer> {
    return this.db.customer.create({ data });
  }

  async update(id: number, data: UpdateCustomerDTO): Promise<Customer> {
    return this.db.customer.update({ where: { id }, data });
  }

  async delete(id: number): Promise<boolean> {
    await this.db.customer.delete({ where: { id } });
    return true;
  }

  async countActive(): Promise<number> {
    return this.db.customer.count({ where: { isActive: true } });
  }
}
```
