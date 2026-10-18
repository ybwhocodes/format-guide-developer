# Migration & Seeder Standard — Express.js Layered Architecture

> **Status:** Team Standard (Tier 1)  
> **Framework:** Express.js (TypeScript)  
> **Purpose:** Standar migrasi dan seeder database dalam struktur layered Express.js.

---

## 1. Lokasi Direktori

```text
prisma/                    # Jika menggunakan Prisma
├── schema.prisma
├── migrations/
└── seed.ts

# ATAU jika menggunakan Drizzle / Node CLI
db/
├── migrations/
└── seeders/
    └── database.seeder.ts
```

---

## 2. Standar Seeder Idempotent (TypeScript)

```typescript
// prisma/seed.ts
import { PrismaClient } from "@prisma/client";

const prisma = new PrismaClient();

async function main() {
  console.log("🌱 Starting seed...");

  const roles = [
    { code: "SUPERADMIN", name: "Super Administrator" },
    { code: "ADMIN", name: "Administrator" },
    { code: "USER", name: "Standard User" }
  ];

  for (const role of roles) {
    await prisma.role.upsert({
      where: { code: role.code },
      update: { name: role.name },
      create: role
    });
  }

  console.log("✅ Seeding finished successfully.");
}

main()
  .catch((e) => {
    console.error(e);
    process.exit(1);
  })
  .finally(async () => {
    await prisma.$disconnect();
  });
```

Package script di `package.json`:
```json
{
  "prisma": {
    "seed": "tsx prisma/seed.ts"
  }
}
```
