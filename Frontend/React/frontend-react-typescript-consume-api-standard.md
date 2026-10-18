# React + TypeScript API Consumption Standard

> **Status:** Team Standard
> **Framework:** React + TypeScript
> **Scope:** HTTP client, typed service, server state, mutation, error, retry, cache, dan cancellation.

## 1. Boundary API

- Komponen tidak memanggil `fetch` atau Axios langsung.
- API call berada di `src/services/` untuk client bersama atau `src/features/{feature}/services/` untuk domain-specific.
- Service mengembalikan data typed dan melempar error terstruktur.
- Adapter API boleh diganti dari mock ke HTTP tanpa mengubah komponen.
- Semua response mengikuti `Backend/{Framework}/universal-api-contract.md`.

```text
src/
├── services/
│   ├── api-client.ts
│   ├── api-error.ts
│   └── http.ts
└── features/
    └── customers/
        ├── services/
        │   └── customer-api.ts
        ├── hooks/
        │   └── use-customers.ts
        └── types.ts
```

## 2. Typed API Client

Gunakan satu client untuk base URL, header, credential, timeout, dan parsing error.

```typescript
export type ApiMeta = {
  success: boolean;
  message: string;
  code: number;
  error_code?: string;
};

export type ApiResponse<T> = {
  meta: ApiMeta;
  data: T;
};

export async function apiRequest<T>(
  path: string,
  init: RequestInit = {}
): Promise<ApiResponse<T>> {
  const response = await fetch(`${import.meta.env.VITE_API_URL}${path}`, {
    ...init,
    headers: {
      Accept: "application/json",
      "Content-Type": "application/json",
      ...init.headers
    }
  });

  const body = await response.json().catch(() => null);
  if (!response.ok) {
    throw new Error(body?.meta?.error_code ?? `HTTP_${response.status}`);
  }

  return body as ApiResponse<T>;
}
```

Aturan:

- Base URL berasal dari environment variable.
- Jangan menaruh token, secret, atau credential server di bundle frontend.
- Jangan menganggap HTTP `200` selalu berarti business operation sukses; periksa envelope dan status.
- Validasi response eksternal bila data masuk ke bagian sensitif atau bentuk response tidak sepenuhnya dipercaya.

## 3. Service Feature

```typescript
import { apiRequest } from "@/services/api-client";
import type { Customer } from "../types";

export function getCustomers(signal?: AbortSignal) {
  return apiRequest<Customer[]>("/api/v1/customers", { signal });
}

export function createCustomer(input: { name: string; email: string }) {
  return apiRequest<Customer>("/api/v1/customers", {
    method: "POST",
    body: JSON.stringify(input)
  });
}
```

- Service tidak mengatur loading state UI.
- Service tidak menampilkan toast.
- Service tidak membaca atau memanipulasi DOM.
- Mutation transaksi memakai idempotency key bila diwajibkan API.

## 4. Server State dengan TanStack Query

Gunakan TanStack Query untuk cache, loading, error, invalidation, dan mutation server state.

```typescript
import { useQuery } from "@tanstack/react-query";
import { getCustomers } from "../services/customer-api";

export function useCustomers() {
  return useQuery({
    queryKey: ["customers"],
    queryFn: ({ signal }) => getCustomers(signal),
    staleTime: 30_000
  });
}
```

Aturan:

- Query key mencerminkan parameter request.
- Jangan menyalin hasil query ke Context atau global store tanpa alasan kuat.
- Invalidate query setelah mutation yang mengubah data terkait.
- Optimistic update hanya bila rollback tersedia.
- Request stale menerima `AbortSignal` bila client mendukungnya.
- Retry default hanya untuk error transient dan method aman. Jangan retry mutation finansial tanpa idempotency.

## 5. UI State

Setiap konsumsi API wajib menangani:

- Loading state.
- Empty state.
- Error state dan retry.
- Forbidden atau unauthorized state bila relevan.
- Success state setelah mutation.
- Disabled atau pending state untuk tombol submit.

```tsx
if (query.isPending) return <LoadingState label="Memuat customer" />;
if (query.isError) return <ErrorState onRetry={() => query.refetch()} />;
if (query.data.data.length === 0) return <EmptyState label="Belum ada customer" />;
return <CustomerTable rows={query.data.data} />;
```

## 6. Pagination, Filter, dan Search

- Filter, sort, search, dan pagination yang dapat dibagikan disimpan di URL search params.
- Query key harus memasukkan seluruh parameter.
- Debounce search sebelum request.
- Reset page ke `1` saat filter utama berubah.
- Tampilkan total dan batas halaman dari metadata API.
- Jangan mengambil seluruh dataset untuk difilter di browser kecuali ukuran dan alasannya jelas.

## 7. Authentication dan Error

- Access token dikelola sesuai keputusan security proyek.
- Jangan menyimpan credential sensitif di URL atau log.
- `401` memicu alur session expired, bukan retry tanpa batas.
- `403` menampilkan forbidden state, bukan menganggap data kosong.
- `422` dipetakan ke error field form.
- `409` menampilkan konflik bisnis yang dapat dipahami pengguna.
- `429` menghormati retry hint bila tersedia.
- `5xx` menampilkan recovery action tanpa membocorkan stack trace.

## 8. Testing dan Quality Gate

- Test service untuk URL, method, body, header, response, dan error mapping.
- Test hook/query untuk loading, success, empty, error, retry, dan invalidation.
- Test mutation untuk pending, success, validation, conflict, dan duplicate submit.
- Mock network di boundary HTTP, bukan meniru internal TanStack Query.
- Pastikan request tidak berjalan setelah component unmount bila cancellation dibutuhkan.

Checklist:

- [ ] Tidak ada `fetch` langsung di JSX/component.
- [ ] Semua service memiliki tipe input dan output.
- [ ] Base URL berasal dari environment.
- [ ] Query key memasukkan parameter request.
- [ ] Loading, empty, error, forbidden, dan success state tersedia.
- [ ] Retry dan mutation aman terhadap duplicate action.
- [ ] Token, secret, dan response sensitif tidak masuk log.
- [ ] Test service dan alur UI kritis berhasil.
