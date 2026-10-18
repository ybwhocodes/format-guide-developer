# Migration & Seeder Standard — ElysiaJS Modular Architecture

> **Status:** Scaled System Standard (Tier 2)  
> **Framework:** ElysiaJS (TypeScript / Bun)  
> **Pattern:** Package by Feature  
> **Purpose:** Standar seeder per modul domain dan agregasi master runner di tingkat modular.

---

## 1. Lokasi Direktori Seeder Modular

Setiap modul domain bertanggung jawab atas data awal dan testing modulnya:

```text
src/modules/
├── customer/
│   └── seeders/
│       └── customer.seeder.ts
├── order/
│   └── seeders/
│       └── order.seeder.ts
└── auth/
    └── seeders/
        └── role.seeder.ts

src/core/database/
├── schema.ts            # Skema DB agregat
└── seed.ts              # Root seeder runner
```

---

## 2. Penulisan Feature Seeder

```typescript
// src/modules/customer/seeders/customer.seeder.ts
export async function seedCustomers(db: any) {
  const dummyCustomers = [
    { name: "PT Surya Abadi", email: "surya@example.com", phone: "021-123456" },
    { name: "CV Mandiri Jaya", email: "mandiri@example.com", phone: "022-654321" }
  ];

  for (const c of dummyCustomers) {
    await db.customer.upsert({
      where: { email: c.email },
      update: { name: c.name, phone: c.phone },
      create: c
    });
  }
}
```

---

## 3. Master Seeder Runner (`src/core/database/seed.ts`)

```typescript
import { seedRoles } from "../../modules/auth/seeders/role.seeder";
import { seedCustomers } from "../../modules/customer/seeders/customer.seeder";

export async function runAllSeeders(db: any) {
  console.log("🌱 Starting modular seeding...");
  
  // 1. Core/Auth
  await seedRoles(db);
  
  // 2. Domain Modules
  await seedCustomers(db);

  console.log("✅ All modular seeders applied.");
}
```
