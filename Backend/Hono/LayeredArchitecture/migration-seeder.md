# Migration & Seeder Standard — Hono Layered Architecture

> **Status:** Team Standard (Tier 1)  
> **Framework:** Hono (TypeScript / Bun / Node)  
> **Purpose:** Standar migrasi skema dan seeder database terpusat untuk aplikasi Hono.

---

## 1. Lokasi Direktori

```text
db/
├── migrations/
├── seeders/
│   ├── database.seeder.ts
│   └── role.seeder.ts
└── schema.ts
```

---

## 2. Implementasi Seeder (TypeScript)

```typescript
// db/seeders/role.seeder.ts
export async function seedRoles(db: any) {
  const defaultRoles = [
    { code: "ADMIN", name: "Administrator" },
    { code: "STAFF", name: "Staff Member" }
  ];

  for (const r of defaultRoles) {
    await db.role.upsert({
      where: { code: r.code },
      update: { name: r.name },
      create: r
    });
  }
}
```

```typescript
// db/seeders/database.seeder.ts
import { seedRoles } from "./role.seeder";

export async function runSeeder(db: any) {
  console.log("🌱 Seeding database...");
  await seedRoles(db);
  console.log("✅ Seeding complete.");
}
```
