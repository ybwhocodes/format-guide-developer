# Unit & Integration Testing Standard — Express.js Layered Architecture

> **Status:** Team Standard (Tier 1)  
> **Framework:** Express.js (TypeScript)  
> **Test Runner:** Vitest & Supertest  
> **Purpose:** Standar pengujian Service dan API routes pada Express.js.

---

## 1. Tooling

- Test Runner: **Vitest** (Cepat, native ESM/TS, kompatibel Jest API).
- HTTP Integration Assertion: **Supertest**.

---

## 2. Unit Testing Service (Vitest)

```typescript
// tests/unit/customer.service.test.ts
import { describe, it, expect, vi, beforeEach } from "vitest";
import { CustomerService } from "../../src/services/customer.service";
import { ICustomerRepository } from "../../src/repositories/interfaces/customer-repository.interface";

describe("CustomerService Unit Tests", () => {
  let customerService: CustomerService;
  let mockRepository: ICustomerRepository;

  beforeEach(() => {
    mockRepository = {
      findById: vi.fn(),
      findByEmail: vi.fn(),
      create: vi.fn()
    };
    customerService = new CustomerService(mockRepository);
  });

  it("berhasil membuat customer jika email belum ada", async () => {
    vi.mocked(mockRepository.findByEmail).mockResolvedValue(null);
    vi.mocked(mockRepository.create).mockResolvedValue({
      id: 1,
      name: "Budi",
      email: "budi@test.com",
      isActive: true,
      createdAt: new Date()
    });

    const result = await customerService.createCustomer({
      name: "Budi",
      email: "budi@test.com"
    });

    expect(result.id).toBe(1);
    expect(mockRepository.create).toHaveBeenCalledOnce();
  });

  it("melempar 409 jika email sudah terdaftar", async () => {
    vi.mocked(mockRepository.findByEmail).mockResolvedValue({
      id: 2,
      name: "Existing",
      email: "exist@test.com",
      isActive: true,
      createdAt: new Date()
    });

    await expect(
      customerService.createCustomer({ name: "Baru", email: "exist@test.com" })
    ).rejects.toThrow("Email already registered");
  });
});
```

---

## 3. Integration Testing Endpoint (Supertest)

```typescript
// tests/integration/customer.routes.test.ts
import { describe, it, expect } from "vitest";
import request from "supertest";
import { app } from "../../src/app";

describe("Customer API Routes", () => {
  it("POST /api/v1/customers mengembalikan 201 dengan payload valid", async () => {
    const res = await request(app)
      .post("/api/v1/customers")
      .send({
        name: "Ahmad Dahlan",
        email: "ahmad@example.com"
      });

    expect(res.status).toBe(201);
    expect(res.body.meta.success).toBe(true);
    expect(res.body.data.email).toBe("ahmad@example.com");
  });

  it("POST /api/v1/customers mengembalikan 422 jika format email salah", async () => {
    const res = await request(app)
      .post("/api/v1/customers")
      .send({
        name: "Ahmad",
        email: "bukan-email"
      });

    expect(res.status).toBe(422);
    expect(res.body.meta.code).toBe(422);
  });
});
```
