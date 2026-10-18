# Authentication & Authorization Standard — Hono Layered Architecture

> **Status:** Team Standard (Tier 1)  
> **Framework:** Hono (TypeScript)  
> **Purpose:** Standar autentikasi token JWT (`hono/jwt`) dan otorisasi RBAC di Hono.

---

## 1. Komponen Autentikasi Hono

Hono memiliki middleware resmi `hono/jwt` dan `hono/bearer-auth` bawaan yang sangat ringan dan bekerja di semua runtime (Bun, Node, Cloudflare Workers).

---

## 2. Implementasi Middleware Auth & Role Guard

```typescript
// src/middlewares/auth.middleware.ts
import { createMiddleware } from "hono/factory";
import { verify } from "hono/jwt";

export interface UserPayload {
  id: number;
  email: string;
  role: string;
}

type Env = {
  Variables: {
    user: UserPayload;
  };
};

export const jwtAuth = createMiddleware<Env>(async (c, next) => {
  const authHeader = c.req.header("Authorization");

  if (!authHeader || !authHeader.startsWith("Bearer ")) {
    return c.json(
      { meta: { success: false, message: "Unauthorized: Missing Bearer Token", code: 401 }, data: null },
      401
    );
  }

  const token = authHeader.substring(7);

  try {
    const payload = await verify(token, process.env.JWT_SECRET || "default_secret");
    c.set("user", payload as unknown as UserPayload);
    await next();
  } catch (err) {
    return c.json(
      { meta: { success: false, message: "Unauthorized: Invalid Token", code: 401 }, data: null },
      401
    );
  }
});

export const requireRoles = (roles: string[]) => {
  return createMiddleware<Env>(async (c, next) => {
    const user = c.get("user");

    if (!user || !roles.includes(user.role)) {
      return c.json(
        { meta: { success: false, message: "Forbidden: Insufficient Permissions", code: 403 }, data: null },
        403
      );
    }

    await next();
  });
};
```

---

## 3. Penerapan di Route

```typescript
// src/routes/admin.routes.ts
import { Hono } from "hono";
import { jwtAuth, requireRoles } from "../middlewares/auth.middleware";

const adminApp = new Hono();

adminApp.use("*", jwtAuth);
adminApp.use("*", requireRoles(["SUPERADMIN", "ADMIN"]));

adminApp.get("/stats", (c) => {
  const user = c.get("user");
  return c.json({
    meta: { success: true, message: "Admin stats retrieved", code: 200 },
    data: { requestedBy: user.email }
  });
});

export default adminApp;
```
