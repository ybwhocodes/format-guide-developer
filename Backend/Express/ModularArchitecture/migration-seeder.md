# Migration & Seeder Standard — Express.js Modular Architecture

> **Status:** Scaled System Standard (Tier 2)  
> **Framework:** Express.js (TypeScript)  
> **Pattern:** Package by Feature  
> **Purpose:** Standar seeder per feature module pada arsitektur modular Express.js.

---

## 1. Lokasi Direktori Seeder Modular

```text
src/modules/
├── customer/
│   └── seeders/
│       └── customer.seeder.ts
├── auth/
│   └── seeders/
│       └── role-user.seeder.ts
└── order/
    └── seeders/
        └── order.seeder.ts

src/core/database/
└── run-seeders.ts
```

---

## 2. Implementasi Feature Seeder

```typescript
// src/modules/customer/seeders/customer.seeder.ts
export async function seedCustomerModule(db: any) {
  const defaultCustomers = [
    { email: "corp@domain.com", name: "Corporate Client", isActive: true }
  ];

  for (const c of defaultCustomers) {
    await db.customer.upsert({
      where: { email: c.email },
      update: { name: c.name },
      create: c
    });
  }
}
```

---

## 3. Modular Runner (`src/core/database/run-seeders.ts`)

```typescript
import { seedAuthModule } from "../../modules/auth/seeders/role-user.seeder";
import { seedCustomerModule } from "../../modules/customer/seeders/customer.seeder";

export async function runAllSeeders(db: any) {
  console.log("🚀 Executing feature-based seeders...");
  await seedAuthModule(db);
  await seedCustomerModule(db);
  console.log("✨ Seeding completed.");
}
```
