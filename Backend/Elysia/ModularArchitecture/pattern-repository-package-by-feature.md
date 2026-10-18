# Repository Pattern Standard (Package by Feature) — ElysiaJS

> **Status:** Scaled System Standard (Tier 2)  
> **Framework:** ElysiaJS (TypeScript)  
> **Pattern:** Package by Feature / Modular Architecture  
> **Purpose:** Standar penerapan repository per fitur/domain pada ElysiaJS, isolasi data domain, dan batasan komunikasi lintas modul.

---

## 1. Konsep & Struktur Domain

Repository diletakkan **di dalam modul domain masing-masing** (`src/modules/{feature}/{feature}.repository.ts`).

```text
src/modules/
├── customer/
│   ├── customer.repository.ts
│   └── customer.interface.ts
└── order/
    ├── order.repository.ts
    └── order.interface.ts
```

---

## 2. Aturan Akses Antar Domain (Cross-Domain Rules)

1. **Dilarang Direct Cross-Repository:** Modul `Order` tidak boleh mengimpor dan memanggil `CustomerRepository` secara langsung untuk mengubah status pelanggan.
2. **Komunikasi Melalui Service Publik:** Modul `Order` memanggil `CustomerService` atau Domain Interface yang disediakan modul Customer.
3. **Database Boundary:** Tidak diperbolehkan melakukan direct join query terhadap tabel milik modul lain di dalam repository fitur.

---

## 3. Implementasi Domain Repository

```typescript
// src/modules/customer/customer.repository.ts
export interface CustomerRecord {
  id: number;
  name: string;
  email: string;
  phone: string | null;
  createdAt: Date;
}

export class CustomerRepository {
  constructor(private readonly db: any) {}

  async findById(id: number): Promise<CustomerRecord | null> {
    return this.db.customer.findUnique({ where: { id } });
  }

  async findByEmail(email: string): Promise<CustomerRecord | null> {
    return this.db.customer.findUnique({ where: { email } });
  }

  async create(payload: Omit<CustomerRecord, "id" | "createdAt">): Promise<CustomerRecord> {
    return this.db.customer.create({ data: payload });
  }
}
```
