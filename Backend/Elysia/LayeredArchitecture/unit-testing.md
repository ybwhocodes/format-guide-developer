# Unit & Integration Testing Standard — ElysiaJS Layered Architecture

> **Status:** Team Standard (Tier 1)  
> **Framework:** ElysiaJS (TypeScript)  
> **Test Runner:** Bun Test (`bun:test`)  
> **Purpose:** Standar pengujian endpoint HTTP dan unit service di ElysiaJS.

---

## 1. Tooling & Test Runner

ElysiaJS didesain native untuk runtime Bun, sehingga menggunakan **`bun:test`** bawaan:
- Eksekusi instan tanpa konfigurasi compiler eksternal.
- Endpoint testing dilakukan via method `.handle()` bawaan instance Elysia tanpa perlu membuka port TCP fisik.

---

## 2. Unit Testing Service (Isolated Mock)

```typescript
// tests/unit/customer.service.test.ts
import { describe, expect, it, mock } from "bun:test";
import { CustomerService } from "../../src/services/customer.service";
import type { ICustomerRepository } from "../../src/repositories/interfaces/customer-repository.interface";

describe("CustomerService Unit Test", () => {
  it("harus melempar error jika email sudah terdaftar", async () => {
    const mockRepo: ICustomerRepository = {
      findById: mock(),
      findByEmail: mock(async () => ({
        id: 1,
        name: "Existing",
        email: "test@example.com",
        phone: null,
        isActive: true,
        createdAt: new Date()
      })),
      create: mock(),
      findAll: mock()
    };

    const service = new CustomerService(mockRepo);

    expect(
      service.registerCustomer({
        name: "New User",
        email: "test@example.com"
      })
    ).rejects.toThrow("CUSTOMER_EMAIL_ALREADY_EXISTS");
  });
});
```

---

## 3. Feature / Endpoint Testing via `app.handle()`

```typescript
// tests/feature/customer.controller.test.ts
import { describe, expect, it } from "bun:test";
import { Elysia } from "elysia";
import { customerController } from "../../src/controllers/customer.controller";

describe("Customer API Endpoint Test", () => {
  // Mock service
  const mockCustomerService: any = {
    registerCustomer: async (input: any) => ({
      id: 99,
      ...input,
      isActive: true,
      createdAt: new Date()
    })
  };

  const app = new Elysia().use(customerController(mockCustomerService));

  it("POST /customers valid payload mengembalikan status 201", async () => {
    const response = await app.handle(
      new Request("http://localhost/customers", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          name: "Budi Santoso",
          email: "budi@example.com"
        })
      })
    );

    expect(response.status).toBe(201);
    const json = await response.json();
    expect(json.meta.success).toBe(true);
    expect(json.data.id).toBe(99);
  });

  it("POST /customers invalid body mengembalikan status 422", async () => {
    const response = await app.handle(
      new Request("http://localhost/customers", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          email: "not-an-email" // name missing, email invalid
        })
      })
    );

    expect(response.status).toBe(422);
  });
});
```
