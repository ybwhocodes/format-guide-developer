# Authentication & Authorization Standard — ElysiaJS Layered Architecture

> **Status:** Team Standard (Tier 1)  
> **Framework:** ElysiaJS (TypeScript)  
> **Pattern:** Package by Layer  
> **Purpose:** Standar autentikasi token JWT (@elysiajs/jwt) dan otorisasi role-based access control (RBAC) pada ElysiaJS.

---

## 1. Komponen Autentikasi

- **Plugin:** `@elysiajs/jwt` & `@elysiajs/bearer`.
- **Stateless:** Validasi token Bearer pada header request.
- **Hook:** Menggunakan lifecycle hook `derive` atau `beforeHandle` Elysia untuk ekstraksi user session.

---

## 2. Implementasi Auth Guard Middleware

```typescript
// src/middlewares/auth.middleware.ts
import { Elysia } from "elysia";
import { jwt } from "@elysiajs/jwt";
import { bearer } from "@elysiajs/bearer";

export const authPlugin = new Elysia({ name: "auth" })
  .use(
    jwt({
      name: "jwtAuth",
      secret: process.env.JWT_SECRET || "super-secret-key-change-in-env",
      exp: "7d"
    })
  )
  .use(bearer())
  .derive(async ({ jwtAuth, bearer, set }) => {
    if (!bearer) {
      return { user: null };
    }

    const payload = await jwtAuth.verify(bearer);
    if (!payload) {
      return { user: null };
    }

    return {
      user: {
        id: payload.sub as number,
        role: payload.role as string,
        email: payload.email as string
      }
    };
  });

export const requireAuth = new Elysia({ name: "require-auth" })
  .use(authPlugin)
  .onBeforeHandle(({ user, set }) => {
    if (!user) {
      set.status = 401;
      return {
        meta: { success: false, message: "Unauthorized: Invalid or missing token", code: 401 },
        data: null
      };
    }
  });

export const requireRole = (allowedRoles: string[]) =>
  new Elysia({ name: "require-role" })
    .use(requireAuth)
    .onBeforeHandle(({ user, set }) => {
      if (!user || !allowedRoles.includes(user.role)) {
        set.status = 403;
        return {
          meta: { success: false, message: "Forbidden: Insufficient privileges", code: 403 },
          data: null
        };
      }
    });
```

---

## 3. Penerapan di Controller

```typescript
// src/controllers/protected.controller.ts
import { Elysia } from "elysia";
import { requireRole } from "../middlewares/auth.middleware";

export const adminController = new Elysia({ prefix: "/admin" })
  .use(requireRole(["SUPERADMIN", "ADMIN"]))
  .get("/dashboard", ({ user }) => {
    return {
      meta: { success: true, message: "Dashboard accessed", code: 200 },
      data: { stats: "Confidential Data", accessedBy: user.email }
    };
  });
```
