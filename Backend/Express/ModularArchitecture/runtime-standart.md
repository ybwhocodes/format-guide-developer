# Runtime Standard: Bun.js vs Node.js

> **Status:** Team Standard  
> **Scope:** Backend JavaScript / TypeScript Execution Runtime  
> **Purpose:** Standar pemilihan runtime, manajemen dependensi, script eksekusi, dan konfigurasi environment.

---

## 1. Rekomendasi Runtime Express.js

Gunakan Node.js LTS sebagai runtime produksi Express.js. Pilih `tsx` untuk development TypeScript dan `vitest` atau `jest` untuk pengujian.

## 2. Pemilihan Runtime: Bun.js vs Node.js

| Fitur | Bun.js | Node.js (LTS 20+) |
|---|---|---|
| **Eksekusi TypeScript** | Native bawaan (tanpa `ts-node` / `tsx` / `tsc`) | Butuh runtime bundler (`tsx` / `ts-node` / `esbuild`) |
| **Package Manager** | `bun install` (sangat cepat) | `npm` / `pnpm` / `yarn` |
| **Test Runner** | `bun test` built-in | `vitest` / `jest` |
| **Framework** | Express.js |
| **Kelebihan** | Startup instan, performa I/O tinggi | Ekosistem dependensi mature, dukungan vendor cloud luas |

---

## 2. Standard Konfigurasi Bun.js

### 2.1 `package.json` Scripts
```json
{
  "name": "backend-service",
  "module": "src/index.ts",
  "type": "module",
  "scripts": {
    "dev": "bun --watch src/index.ts",
    "start": "bun src/index.ts",
    "test": "bun test",
    "test:coverage": "bun test --coverage",
    "lint": "biome check src",
    "format": "biome format --write src"
  }
}
```

### 2.2 `tsconfig.json` Bun
```json
{
  "compilerOptions": {
    "lib": ["ESNext"],
    "target": "ESNext",
    "module": "ESNext",
    "moduleDetection": "force",
    "jsx": "react-jsx",
    "allowJs": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "verbatimModuleSyntax": true,
    "noEmit": true,
    "strict": true,
    "skipLibCheck": true,
    "noFallthroughCasesInSwitch": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noPropertyAccessFromIndexSignature": true
  }
}
```

---

## 3. Standard Konfigurasi Node.js (v20+)

### 3.1 `package.json` Scripts (Menggunakan `pnpm` + `tsx`)
```json
{
  "name": "backend-service-node",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "build": "tsup src/index.ts --format esm --dts --clean",
    "start": "node dist/index.js",
    "test": "vitest run",
    "test:watch": "vitest",
    "lint": "eslint src --ext .ts",
    "format": "prettier --write \"src/**/*.ts\""
  }
}
```

### 3.2 `tsconfig.json` Node.js
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "outDir": "dist"
  },
  "include": ["src/**/*"]
}
```
