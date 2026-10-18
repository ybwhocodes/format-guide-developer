# ORM Standard: Drizzle ORM (Hono Modular)

> **Status:** Team Standard  
> **Framework:** Hono (TypeScript)  
> **Architecture:** Modular / feature-based

## 1. Pilihan ORM

Gunakan Drizzle ORM sebagai default untuk setiap module. Setiap module memiliki schema, repository, dan migration concern sendiri, sementara koneksi database tetap dikelola pada infrastructure layer.

Prisma boleh dipakai bila generated client dan schema-first menjadi kebutuhan utama. Sequelize hanya untuk module legacy yang sudah bergantung padanya. Jangan mencampur ORM dalam satu module.

## 2. Batas Module

- Letakkan schema dan repository dekat feature, misalnya `src/modules/customer/infrastructure/db`.
- Module hanya mengekspor use case atau port, bukan query builder ke module lain.
- Query lintas domain melalui application service atau domain port.
- Migration tetap dikontrol terpusat agar urutan perubahan schema dapat dilacak.
- Gunakan transaction boundary pada use case yang mengubah lebih dari satu aggregate.
- Pilih driver Drizzle sesuai database dan runtime Hono, termasuk PostgreSQL, MySQL, SQLite, atau adapter serverless.

## 3. Contoh Schema Feature

```typescript
import { pgTable, serial, varchar } from "drizzle-orm/pg-core";

export const customers = pgTable("customers", {
  id: serial("id").primaryKey(),
  name: varchar("name", { length: 150 }).notNull(),
  email: varchar("email", { length: 150 }).notNull().unique()
});
```

Repository feature menerjemahkan hasil query ke model yang dipakai use case. Route Hono tidak boleh mengakses tabel secara langsung.
