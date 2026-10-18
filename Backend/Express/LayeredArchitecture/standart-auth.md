# Authentication & Authorization Standard — Express.js Layered Architecture

> **Status:** Team Standard (Tier 1)  
> **Framework:** Express.js (TypeScript)  
> **Purpose:** Standar autentikasi JWT stateless dan otorisasi RBAC Express.js.

---

## 1. Type Declaration untuk Express Request

Agar `req.user` terdefinisi dengan kuat di TypeScript tanpa compilation error:

```typescript
// src/types/express.d.ts
export interface AuthenticatedUser {
  id: number;
  email: string;
  role: string;
}

declare global {
  namespace Express {
    interface Request {
      user?: AuthenticatedUser;
    }
  }
}
```

---

## 2. Authentication Middleware (`src/middlewares/auth.middleware.ts`)

```typescript
import { Request, Response, NextFunction } from "express";
import jwt from "jsonwebtoken";
import { AuthenticatedUser } from "../types/express";

const JWT_SECRET = process.env.JWT_SECRET || "change-this-secret";

export const authenticateJWT = (req: Request, res: Response, next: NextFunction) => {
  const authHeader = req.headers.authorization;

  if (!authHeader || !authHeader.startsWith("Bearer ")) {
    return res.status(401).json({
      meta: { success: false, message: "Unauthorized: Missing Bearer token", code: 401 },
      data: null
    });
  }

  const token = authHeader.split(" ")[1];

  try {
    const decoded = jwt.verify(token, JWT_SECRET) as AuthenticatedUser;
    req.user = decoded;
    return next();
  } catch (err) {
    return res.status(401).json({
      meta: { success: false, message: "Unauthorized: Invalid or expired token", code: 401 },
      data: null
    });
  }
};
```

---

## 3. Authorization Middleware (RBAC)

```typescript
// src/middlewares/role.middleware.ts
import { Request, Response, NextFunction } from "express";

export const authorizeRoles = (...allowedRoles: string[]) => {
  return (req: Request, res: Response, next: NextFunction) => {
    if (!req.user) {
      return res.status(401).json({
        meta: { success: false, message: "Unauthorized", code: 401 }
      });
    }

    if (!allowedRoles.includes(req.user.role)) {
      return res.status(403).json({
        meta: { success: false, message: "Forbidden: Access is denied", code: 403 },
        data: null
      });
    }

    return next();
  };
};
```
