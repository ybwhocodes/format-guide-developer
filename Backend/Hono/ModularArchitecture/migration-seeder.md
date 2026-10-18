# Migration & Seeder Standard — Hono Modular Architecture

> **Status:** Scaled System Standard (Tier 2)  
> **Framework:** Hono (TypeScript)  
> **Pattern:** Package by Feature  
> **Purpose:** Standar seeder per feature module pada arsitektur modular Hono.

---

## 1. Struktur Seeder Per Modul

```text
src/modules/
├── customer/
│   └── seeders/
│       └── customer.seeder.ts
├── auth/
│   └── seeders/
│       └── auth.seeder.ts

src/core/database/
└── master-seed.ts
```

---

## 2. Feature Seeder & Master Runner

```typescript
// src/modules/customer/seeders/customer.seeder.ts
export async function seedCustomerModule(db: any) {
  const data = [{ name: "Modular Client", email: "client@modular.io" }];
  for (const item of data) {
    await db.customer.upsert({
      where: { email: item.email },
      update: { name: item.name },
      create: item
    });
  }
}
```

```typescript
// src/core/database/master-seed.ts
import { seedAuthModule } from "../../modules/auth/seeders/auth.seeder";
import { seedCustomerModule } from "../../modules/customer/seeders/customer.seeder";

export async function runAllSeeders(db: any) {
  console.log("🌱 Executing modular Hono seeders...");
  await seedAuthModule(db);
  await seedCustomerModule(db);
  console.log("✅ Modular seeders executed.");
}
```
