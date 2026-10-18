# Unit & Integration Testing Standard — Express.js Modular Architecture

> **Status:** Scaled System Standard (Tier 2)  
> **Framework:** Express.js (TypeScript)  
> **Pattern:** Package by Feature  
> **Test Runner:** Vitest  
> **Purpose:** Standar pengujian terisolasi per feature module di Express.js.

---

## 1. Struktur Folder Test Per Modul

```text
src/modules/
├── customer/
│   ├── customer.controller.ts
│   ├── customer.service.ts
│   └── tests/
│       ├── customer.service.test.ts
│       └── customer.router.test.ts
```

---

## 2. Contoh Test Sub-Router Modul

```typescript
// src/modules/customer/tests/customer.router.test.ts
import { describe, it, expect } from "vitest";
import express from "express";
import request from "supertest";
import { buildCustomerModule } from "../customer.router";

describe("Customer Module Integration Test", () => {
  const mockDb = {
    customer: {
      findUnique: async () => null,
      create: async ({ data }: any) => ({ id: 50, ...data, createdAt: new Date() })
    }
  };

  const app = express();
  app.use(express.json());
  app.use("/customers", buildCustomerModule(mockDb));

  it("POST /customers valid input mengembalikan status 201", async () => {
    const res = await request(app)
      .post("/customers")
      .send({
        name: "Test Customer",
        email: "test@modular.com"
      });

    expect(res.status).toBe(201);
    expect(res.body.data.id).toBe(50);
  });
});
```
