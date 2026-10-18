# Repository Pattern Standard (Package by Feature) — Hono

> **Status:** Scaled System Standard (Tier 2)  
> **Framework:** Hono (TypeScript)  
> **Pattern:** Package by Feature  
> **Purpose:** Standar implementasi repository per fitur domain pada arsitektur modular Hono.

---

## 1. Konsep & Batasan

- Repository berada di dalam modul fitur: `src/modules/{feature}/{feature}.repository.ts`.
- Hanya memuat query untuk entitas yang dimiliki modul tersebut.
- Modul lain mengakses data melalui service interface, dilarang mengakses repository internal modul lain.

---

## 2. Implementasi Feature Repository

```typescript
// src/modules/customer/customer.repository.ts
export interface CustomerRecord {
  id: number;
  name: string;
  email: string;
  phone: string | null;
  isActive: boolean;
}

export class CustomerRepository {
  constructor(private readonly db: any) {}

  async findById(id: number): Promise<CustomerRecord | null> {
    return this.db.customer.findUnique({ where: { id } });
  }

  async findByEmail(email: string): Promise<CustomerRecord | null> {
    return this.db.customer.findUnique({ where: { email } });
  }

  async create(data: { name: string; email: string; phone?: string }): Promise<CustomerRecord> {
    return this.db.customer.create({ data });
  }
}
```
