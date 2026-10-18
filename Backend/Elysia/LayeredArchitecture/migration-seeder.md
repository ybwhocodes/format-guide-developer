# Migration & Seeder Standard — ElysiaJS Layered Architecture

> **Status:** Team Standard (Tier 1)  
> **Framework:** ElysiaJS (TypeScript / Bun)  
> **Purpose:** Standar pengelolaan skema database, migrasi terpusat, dan pengisian data uji (seeder) dalam proyek ElysiaJS.

---

## 1. Lokasi Direktori Terpusat

```text
db/
├── migrations/          # File migrasi tergenerate (SQL / TS)
├── seeders/
│   ├── database.seeder.ts
│   ├── role.seeder.ts
│   └── customer.seeder.ts
└── schema.ts            # Definisi tabel ORM
```

---

## 2. Naming Conventions Database

- Prefix tabel:
  - `mt_` : Master data (`mt_customers`, `mt_users`)
  - `tr_` : Transaksi (`tr_orders`, `tr_invoices`)
  - `dt_` : Detail transaksi (`dt_order_items`)
  - `rf_` : Referensi (`rf_roles`)
- Format kolom: `snake_case` di database, dimapping ke `camelCase` di TypeScript.

---

## 3. Menulis Seeder yang Idempotent

Seeder wajib idempotent (`upsert` / `firstOrCreate` logic) agar dapat dieksekusi berulang kali tanpa memicu duplicate key violation.

```typescript
// db/seeders/role.seeder.ts
export async function seedRoles(db: any) {
  const roles = [
    { code: "SUPERADMIN", name: "Super Administrator" },
    { code: "ADMIN", name: "Administrator" },
    { code: "USER", name: "End User" }
  ];

  for (const role of roles) {
    // Pola Upsert (contoh: Prisma / Drizzle)
    await db.role.upsert({
      where: { code: role.code },
      update: { name: role.name },
      create: role
    });
  }
}
```

```typescript
// db/seeders/database.seeder.ts
import { seedRoles } from "./role.seeder";

export async function runDatabaseSeeder(db: any) {
  console.log("🌱 Running database seeders...");
  await seedRoles(db);
  console.log("✅ Database seeding completed.");
}

// Eksekusi jika dipanggil via CLI Bun: bun run db/seeders/database.seeder.ts
```
