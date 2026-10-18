# Unit & Integration Testing Standard — Hono Layered Architecture

> **Status:** Team Standard (Tier 1)  
> **Framework:** Hono (TypeScript)  
> **Test Runner:** Bun Test atau Vitest  
> **Purpose:** Standar pengujian Service dan API endpoint Hono via method native `app.request()`.

---

## 1. Native Testing via `app.request()`

Hono dibangun di atas standar Web Fetch API (`Request` dan `Response`). Untuk mengetes endpoint, gunakan method bawaan `app.request()` tanpa memerlukan external HTTP client seperti Supertest.

---

## 2. Unit Testing Service (Isolated Mock)

```typescript
// tests/unit/customer.service.test.ts
import { describe, it, expect, vi } from "vitest";
import { CustomerService } from "../../src/services/customer.service";
import { ICustomerRepository } from "../../src/repositories/interfaces/customer-repository.interface";

describe("CustomerService Unit Test", () => {
  it("melempar error jika email duplikat", async () => {
    const mockRepo: ICustomerRepository = {
      findById: vi.fn(),
      findByEmail: vi.fn().mockResolvedValue({ id: 1, email: "duplicate@test.com" }),
      create: vi.fn()
    };

    const service = new CustomerService(mockRepo);

    await expect(
      service.register({ name: "Testing", email: "duplicate@test.com" })
    ).rejects.toThrow("CUSTOMER_EMAIL_ALREADY_EXISTS");
  });
});
```

---

## 3. Integration Testing Endpoint (`app.request()`)

```typescript
// tests/integration/customer.test.ts
import { describe, it, expect } from "vitest";
import { Hono } from "hono";
import { createCustomerController } from "../../src/controllers/customer.controller";

describe("Customer API Integration", () => {
  const mockService: any = {
    register: async (input: any) => ({ id: 10, ...input }),
    getById: async (id: number) => ({ id, name: "Customer 10" })
  };

  const app = new Hono().route("/customers", createCustomerController(mockService));

  it("POST /customers berhasil mengembalikan 201", async () => {
    const res = await app.request("/customers", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        name: "Hono Tester",
        email: "hono@test.com"
      })
    });

    expect(res.status).toBe(201);
    const body = await res.json();
    expect(body.meta.success).toBe(true);
    expect(body.data.id).toBe(10);
  });

  it("POST /customers validasi gagal mengembalikan 422", async () => {
    const res = await app.request("/customers", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        email: "invalid-email" // missing name
      })
    });

    expect(res.status).toBe(422);
  });
});
```
