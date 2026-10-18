# Unit & Integration Testing Standard — Hono Modular Architecture

> **Status:** Scaled System Standard (Tier 2)  
> **Framework:** Hono (TypeScript)  
> **Pattern:** Package by Feature  
> **Test Runner:** Vitest atau Bun Test  
> **Purpose:** Standar pengujian terisolasi per feature module di Hono.

---

## 1. Struktur Folder Test Per Modul

```text
src/modules/
├── customer/
│   ├── customer.service.ts
│   ├── index.ts
│   └── tests/
│       ├── customer.service.test.ts
│       └── customer.module.test.ts
```

---

## 2. Contoh Test Sub-App Modul Hono

```typescript
// src/modules/customer/tests/customer.module.test.ts
import { describe, it, expect } from "vitest";
import { buildCustomerModule } from "../index";

describe("Customer Modular Sub-App Test", () => {
  const mockDb = {
    customer: {
      findUnique: async () => null,
      create: async ({ data }: any) => ({ id: 77, ...data })
    }
  };

  const app = buildCustomerModule(mockDb);

  it("POST / berhasil membuat record di sub-app modul", async () => {
    const res = await app.request("/", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        name: "Feature Customer",
        email: "feature@hono.dev"
      })
    });

    expect(res.status).toBe(201);
    const body = await res.json();
    expect(body.data.id).toBe(77);
  });
});
```
