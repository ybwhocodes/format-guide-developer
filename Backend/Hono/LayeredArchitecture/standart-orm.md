# ORM Standard: Drizzle ORM (Hono)

> **Status:** Team Standard  
> **Framework:** Hono (TypeScript)  
> **Scope:** ORM, schema, query, transaction, dan repository boundary pada Hono.

## 1. Pilihan ORM

Gunakan Drizzle ORM sebagai default. Drizzle cocok untuk Hono karena SQL-like, type-safe, ringan, dan mendukung adapter database yang dipakai Hono seperti PostgreSQL, MySQL, SQLite, serta serverless drivers.

Prisma boleh dipakai jika kebutuhan schema-first dan generated client lebih penting. Sequelize hanya untuk pemeliharaan sistem existing yang sudah menggunakannya. Jangan memilih ORM berdasarkan framework saja.

## 2. Konvensi

- Simpan schema di `src/db/schema` dan koneksi di `src/db/client`.
- Pisahkan repository dari Hono route handler.
- Pilih driver sesuai runtime dan platform deployment.
- Jalankan migration melalui Drizzle Kit di pipeline deployment.
- Gunakan transaction untuk operasi lintas tabel.
- Validasi input di route boundary sebelum memanggil repository.
- Jangan menaruh objek database global pada Cloudflare Workers; buat client sesuai batasan adapter.

## 3. Contoh Schema

```typescript
import { pgTable, serial, varchar } from "drizzle-orm/pg-core";

export const customers = pgTable("customers", {
  id: serial("id").primaryKey(),
  name: varchar("name", { length: 150 }).notNull(),
  email: varchar("email", { length: 150 }).notNull().unique()
});
```

## 4. Repository Boundary

Repository menerima dependency database dan mengembalikan domain data. Handler Hono hanya mengatur HTTP request, response, status, dan binding context.
