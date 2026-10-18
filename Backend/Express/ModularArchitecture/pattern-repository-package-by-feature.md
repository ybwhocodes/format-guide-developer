# Repository Pattern Standard (Package by Feature) — Express.js

> **Status:** Scaled System Standard (Tier 2)  
> **Framework:** Express.js (TypeScript)  
> **Pattern:** Package by Feature  
> **Purpose:** Standar implementasi repository terlokalisasi di dalam modul fitur Express.js.

---

## 1. Konsep & Batasan Domain

- Setiap modul domain memiliki repository-nya sendiri di `src/modules/{feature}/{feature}.repository.ts`.
- Repository hanya mengakses tabel yang menjadi ownership domain modulnya.
- Modul lain yang memerlukan data pelanggan **tidak boleh** menginstansiasi `CustomerRepository` secara langsung, melainkan harus melalui `CustomerService` publik.

---

## 2. Implementasi Feature Repository

```typescript
// src/modules/customer/customer.repository.ts
export interface CustomerModel {
  id: number;
  name: string;
  email: string;
  phone: string | null;
  isActive: boolean;
  createdAt: Date;
}

export class CustomerRepository {
  constructor(private readonly db: any) {}

  async findById(id: number): Promise<CustomerModel | null> {
    return this.db.customer.findUnique({ where: { id } });
  }

  async findByEmail(email: string): Promise<CustomerModel | null> {
    return this.db.customer.findUnique({ where: { email } });
  }

  async create(data: { name: string; email: string; phone?: string }): Promise<CustomerModel> {
    return this.db.customer.create({ data });
  }
}
```
