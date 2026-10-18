# Authentication & Authorization Standard — Express.js Modular Architecture

> **Status:** Scaled System Standard (Tier 2)  
> **Framework:** Express.js (TypeScript)  
> **Pattern:** Package by Feature  
> **Purpose:** Standar pembuatan modul terisolasi Auth (`src/modules/auth`) di Express.js.

---

## 1. Struktur Modul Auth

```text
src/modules/auth/
├── auth.controller.ts
├── auth.service.ts
├── auth.router.ts
├── middlewares/
│   ├── authenticate.middleware.ts
│   └── authorize.middleware.ts
└── index.ts # Ekspor router dan guard untuk dipakai app/modul lain
```

---

## 2. Ekspor Auth Guard dari Modul Auth

```typescript
// src/modules/auth/middlewares/authenticate.middleware.ts
import { Request, Response, NextFunction } from "express";
import jwt from "jsonwebtoken";

export const requireAuth = (req: Request, res: Response, next: NextFunction) => {
  const token = req.headers.authorization?.replace("Bearer ", "");
  if (!token) {
    return res.status(401).json({
      meta: { success: false, message: "Token missing", code: 401 }
    });
  }

  try {
    const user = jwt.verify(token, process.env.JWT_SECRET || "secret");
    req.user = user as any;
    return next();
  } catch {
    return res.status(401).json({
      meta: { success: false, message: "Invalid token", code: 401 }
    });
  }
};
```

---

## 3. Pemakaian di Modul Lain

```typescript
// src/modules/customer/customer.router.ts
import { Router } from "express";
import { requireAuth } from "../auth/middlewares/authenticate.middleware";

const router = Router();
router.use(requireAuth); // Proteksi seluruh endpoint customer
```
