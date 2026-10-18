# Unit & Integration Testing Standard — ElysiaJS Modular Architecture

> **Status:** Scaled System Standard (Tier 2)  
> **Framework:** ElysiaJS (TypeScript)  
> **Pattern:** Package by Feature  
> **Test Runner:** Bun Test (`bun:test`)  
> **Purpose:** Standar penulisan test terisolasi per feature module di ElysiaJS.

---

## 1. Struktur Folder Test Terdistribusi

Setiap modul domain mengelola file test-nya sendiri di dalam direktori modul:

```text
src/modules/
├── customer/
│   ├── customer.service.ts
│   ├── customer.controller.ts
│   └── tests/
│       ├── customer.service.test.ts
│       └── customer.module.test.ts
```

---

## 2. Contoh Test Modul Terisolasi

```typescript
// src/modules/customer/tests/customer.module.test.ts
import { describe, expect, it } from "bun:test";
import { Elysia } from "elysia";
import { customerModule } from "../index";

describe("Customer Feature Module Test", () => {
  // Mock DB minimal
  const mockDb = {
    customer: {
      findUnique: async () => null,
      create: async ({ data }: any) => ({ id: 101, ...data, createdAt: new Date() })
    }
  };

  const app = new Elysia().use(customerModule(mockDb));

  it("POST /api/v1/customers berhasil membuat customer", async () => {
    const res = await app.handle(
      new Request("http://localhost/api/v1/customers", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          name: "Customer Modular",
          email: "modular@example.com"
        })
      })
    );

    expect(res.status).toBe(201);
    const body = await res.json();
    expect(body.data.id).toBe(101);
  });
});
```
