# ORM Standard: Prisma, Drizzle, & Sequelize (TypeScript)

> **Status:** Team Standard  
> **Scope:** TypeScript Backend ORM Selection & Implementation  
> **Purpose:** Menyamakan standar implementasi 3 ORM utama: Prisma, Drizzle ORM, dan Sequelize dengan TypeScript strict.

---

## 1. Perbandingan & Rekomendasi Pemilihan ORM

| Aspek | Drizzle ORM | Prisma ORM | Sequelize |
|---|---|---|---|
| **Paradigma** | SQL-like, Zero-overhead, Type-safe | Schema-first, Generator-based client | Traditional Active Record / OOP |
| **Performa & Cold Start** | Sangat cepat (cocok untuk Bun/Serverless) | Menengah (engine query Rust) | Cukup lambat / heavy |
| **Type Safety** | 100% inferred dari TS schema | Generated dari `schema.prisma` | Butuh decorators / `sequelize-typescript` |
| **Posisi pada Express** | Pilihan utama untuk query type-safe dan ringan | Pilihan untuk schema-first dan generated client | Pilihan untuk aplikasi Express legacy |

---

## 2. Standard Implementasi: Drizzle ORM

### 2.1 Schema Definition (`src/db/schema.ts`)
```typescript
import { pgTable, serial, varchar, boolean, timestamp } from "drizzle-orm/pg-core";

export const customers = pgTable("mt_customers", {
  id: serial("id").primaryKey(),
  name: varchar("name", { length: 150 }).notNull(),
  email: varchar("email", { length: 150 }).notNull().unique(),
  phone: varchar("phone", { length: 25 }),
  isActive: boolean("is_active").default(true).notNull(),
  createdAt: timestamp("created_at").defaultNow().notNull(),
  updatedAt: timestamp("updated_at").defaultNow().notNull()
});

export type Customer = typeof customers.$inferSelect;
export type NewCustomer = typeof customers.$inferInsert;
```

### 2.2 Repository Query
```typescript
import { eq } from "drizzle-orm";
import { customers, type Customer, type NewCustomer } from "./schema";

export class DrizzleCustomerRepository {
  constructor(private readonly db: any) {}

  async findById(id: number): Promise<Customer | null> {
    const [result] = await this.db.select().from(customers).where(eq(customers.id, id));
    return result || null;
  }

  async create(data: NewCustomer): Promise<Customer> {
    const [result] = await this.db.insert(customers).values(data).returning();
    return result;
  }
}
```

---

## 3. Standard Implementasi: Prisma ORM

### 3.1 Schema Definition (`prisma/schema.prisma`)
```prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

model Customer {
  id        Int      @id @default(autoincrement())
  name      String   @db.VarChar(150)
  email     String   @unique @db.VarChar(150)
  phone     String?  @db.VarChar(25)
  isActive  Boolean  @default(true) @map("is_active")
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt @map("updated_at")

  @@map("mt_customers")
}
```

### 3.2 Repository Query
```typescript
import { PrismaClient, type Customer } from "@prisma/client";

export class PrismaCustomerRepository {
  constructor(private readonly prisma: PrismaClient) {}

  async findById(id: number): Promise<Customer | null> {
    return this.prisma.customer.findUnique({ where: { id } });
  }

  async create(data: { name: string; email: string; phone?: string; isActive?: boolean }): Promise<Customer> {
    return this.prisma.customer.create({ data });
  }
}
```

---

## 4. Standard Implementasi: Sequelize (TypeScript)

### 4.1 Model Definition
```typescript
import { Table, Column, Model, DataType, PrimaryKey, AutoIncrement } from "sequelize-typescript";

@Table({ tableName: "mt_customers", timestamps: true, underscored: true })
export class Customer extends Model {
  @PrimaryKey
  @AutoIncrement
  @Column(DataType.INTEGER)
  id!: number;

  @Column({ type: DataType.STRING(150), allowNull: false })
  name!: string;

  @Column({ type: DataType.STRING(150), allowNull: false, unique: true })
  email!: string;

  @Column({ type: DataType.STRING(25), allowNull: true })
  phone?: string;

  @Column({ type: DataType.BOOLEAN, defaultValue: true })
  isActive!: boolean;
}
```

### 4.2 Repository Query
```typescript
export class SequelizeCustomerRepository {
  async findById(id: number): Promise<Customer | null> {
    return Customer.findByPk(id);
  }

  async create(data: { name: string; email: string; phone?: string; isActive?: boolean }): Promise<Customer> {
    return Customer.create(data as any);
  }
}
```
