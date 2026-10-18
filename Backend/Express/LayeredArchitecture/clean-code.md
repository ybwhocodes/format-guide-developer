# Clean Code Standard — Express.js Layered Architecture

> **Status:** Team Standard (Tier 1)  
> **Framework:** Express.js (TypeScript)  
> **Pattern:** Package by Layer (Layered Architecture)  
> **Purpose:** Standar penulisan clean code TypeScript pada Express.js menggunakan pola Controller-Service-Repository dan validasi Zod.

---

## 1. Struktur Direktori Proyek

```text
src/
├── controllers/
│   └── customer.controller.ts
├── services/
│   └── customer.service.ts
├── repositories/
│   ├── customer.repository.ts
│   └── interfaces/
│       └── customer-repository.interface.ts
├── validations/
│   └── customer.validation.ts
├── middlewares/
│   ├── validate.middleware.ts
│   ├── auth.middleware.ts
│   └── error.middleware.ts
├── routes/
│   ├── customer.routes.ts
│   └── index.ts
└── app.ts
```

---

## 2. Implementasi Layer

### 2.1 Schema & Validasi (Zod)
```typescript
// src/validations/customer.validation.ts
import { z } from "zod";

export const createCustomerSchema = z.object({
  body: z.object({
    name: z.string().min(3).max(150),
    email: z.string().email().max(150),
    phone: z.string().max(25).optional(),
    isActive: z.boolean().default(true).optional()
  })
});

export type CreateCustomerDTO = z.infer<typeof createCustomerSchema>["body"];
```

### 2.2 Validation Middleware
```typescript
// src/middlewares/validate.middleware.ts
import { Request, Response, NextFunction } from "express";
import { AnyZodObject, ZodError } from "zod";

export const validate = (schema: AnyZodObject) => 
  async (req: Request, res: Response, next: NextFunction) => {
    try {
      await schema.parseAsync({
        body: req.body,
        query: req.query,
        params: req.params
      });
      return next();
    } catch (error) {
      if (error instanceof ZodError) {
        return res.status(422).json({
          meta: { success: false, message: "Validation error", code: 422 },
          errors: error.errors
        });
      }
      return next(error);
    }
  };
```

### 2.3 Repository Layer
```typescript
// src/repositories/interfaces/customer-repository.interface.ts
export interface CustomerEntity {
  id: number;
  name: string;
  email: string;
  phone?: string | null;
  isActive: boolean;
  createdAt: Date;
}

export interface ICustomerRepository {
  findById(id: number): Promise<CustomerEntity | null>;
  findByEmail(email: string): Promise<CustomerEntity | null>;
  create(data: Omit<CustomerEntity, "id" | "createdAt">): Promise<CustomerEntity>;
}
```

### 2.4 Service Layer
```typescript
// src/services/customer.service.ts
import { ICustomerRepository, CustomerEntity } from "../repositories/interfaces/customer-repository.interface";
import { CreateCustomerDTO } from "../validations/customer.validation";

export class CustomerService {
  constructor(private readonly customerRepo: ICustomerRepository) {}

  async createCustomer(dto: CreateCustomerDTO): Promise<CustomerEntity> {
    const exists = await this.customerRepo.findByEmail(dto.email);
    if (exists) {
      const error: any = new Error("Email already registered");
      error.statusCode = 409;
      error.errorCode = "CUSTOMER_EMAIL_ALREADY_EXISTS";
      throw error;
    }

    return this.customerRepo.create({
      name: dto.name,
      email: dto.email,
      phone: dto.phone,
      isActive: dto.isActive ?? true
    });
  }

  async getCustomerById(id: number): Promise<CustomerEntity> {
    const customer = await this.customerRepo.findById(id);
    if (!customer) {
      const error: any = new Error("Customer not found");
      error.statusCode = 404;
      throw error;
    }
    return customer;
  }
}
```

### 2.5 Controller Layer
```typescript
// src/controllers/customer.controller.ts
import { Request, Response, NextFunction } from "express";
import { CustomerService } from "../services/customer.service";

export class CustomerController {
  constructor(private readonly customerService: CustomerService) {}

  store = async (req: Request, res: Response, next: NextFunction): Promise<void> => {
    try {
      const customer = await this.customerService.createCustomer(req.body);
      res.status(201).json({
        meta: { success: true, message: "Customer created successfully", code: 201 },
        data: customer
      });
    } catch (error) {
      next(error);
    }
  };

  show = async (req: Request, res: Response, next: NextFunction): Promise<void> => {
    try {
      const customer = await this.customerService.getCustomerById(Number(req.params.id));
      res.status(200).json({
        meta: { success: true, message: "Customer retrieved successfully", code: 200 },
        data: customer
      });
    } catch (error) {
      next(error);
    }
  };
}
```

### 2.6 Routes & Global Error Handler
```typescript
// src/routes/customer.routes.ts
import { Router } from "express";
import { CustomerController } from "../controllers/customer.controller";
import { validate } from "../middlewares/validate.middleware";
import { createCustomerSchema } from "../validations/customer.validation";

export const createCustomerRouter = (controller: CustomerController): Router => {
  const router = Router();
  router.post("/", validate(createCustomerSchema), controller.store);
  router.get("/:id", controller.show);
  return router;
};
```
