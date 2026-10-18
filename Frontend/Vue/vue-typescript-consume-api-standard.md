# Vue 3 + TypeScript API Consumption Standard

> **Status:** Team Standard
> **Framework:** Vue 3 + TypeScript
> **Scope:** HTTP client, typed service, composable, server state, mutation, error, retry, cache, dan cancellation.

## 1. Boundary API

- Template dan component tidak memanggil `fetch` atau Axios langsung.
- API call berada di `src/services/` untuk client bersama atau `src/features/{feature}/services/` untuk domain-specific.
- Composable menghubungkan service dengan state UI.
- Service mengembalikan data typed dan melempar error terstruktur.
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
        ├── composables/
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
- Jangan menganggap HTTP `200` selalu berarti business operation sukses.
- Validasi response eksternal bila bentuk response tidak sepenuhnya dipercaya.

## 3. Service dan Composable

Service hanya menangani transport dan mapping data.

```typescript
import { apiRequest } from "@/services/api-client";
import type { Customer } from "../types";

export function getCustomers(signal?: AbortSignal) {
  return apiRequest<Customer[]>("/api/v1/customers", { signal });
}
```

Composable mengelola state yang digunakan component.

```typescript
import { ref } from "vue";
import { getCustomers } from "../services/customer-api";

export function useCustomers() {
  const data = ref<Customer[]>([]);
  const pending = ref(false);
  const error = ref<Error | null>(null);

  async function load(signal?: AbortSignal) {
    pending.value = true;
    error.value = null;
    try {
      data.value = (await getCustomers(signal)).data;
    } catch (cause) {
      error.value = cause instanceof Error ? cause : new Error("Request failed");
    } finally {
      pending.value = false;
    }
  }

  return { data, pending, error, load };
}
```

Aturan:

- Service tidak mengatur toast, router, atau DOM.
- Composable memiliki API kecil dan side effect yang jelas.
- Gunakan TanStack Query untuk Vue atau library server-state yang disetujui bila cache dan invalidation kompleks.
- Jangan menyalin server state ke Pinia tanpa kebutuhan lintas halaman yang jelas.
- Mutation transaksi memakai idempotency key bila diwajibkan API.

## 4. UI State di SFC

Setiap konsumsi API wajib menangani:

- Loading state.
- Empty state.
- Error state dan retry.
- Forbidden atau unauthorized state bila relevan.
- Success state setelah mutation.
- Disabled atau pending state pada submit.

```vue
<script setup lang="ts">
const { data, pending, error, load } = useCustomers()
await load()
</script>

<template>
  <LoadingState v-if="pending" label="Memuat customer" />
  <ErrorState v-else-if="error" @retry="load" />
  <EmptyState v-else-if="data.length === 0" label="Belum ada customer" />
  <CustomerTable v-else :rows="data" />
</template>
```

## 5. Pagination, Filter, dan Search

- Filter, sort, search, dan pagination yang dapat dibagikan disimpan di query URL melalui Vue Router.
- Query key atau cache key harus memasukkan seluruh parameter request.
- Debounce search sebelum request.
- Reset page ke `1` saat filter utama berubah.
- Tampilkan total dan batas halaman dari metadata API.
- Jangan mengambil seluruh dataset untuk difilter di browser tanpa alasan terukur.

## 6. Authentication dan Error

- Access token dikelola sesuai keputusan security proyek.
- Jangan menyimpan credential sensitif di URL atau log.
- `401` memicu alur session expired, bukan retry tanpa batas.
- `403` menampilkan forbidden state, bukan menganggap data kosong.
- `422` dipetakan ke error field form.
- `409` menampilkan konflik bisnis yang dapat dipahami pengguna.
- `429` menghormati retry hint bila tersedia.
- `5xx` menampilkan recovery action tanpa stack trace.

## 7. Cancellation dan Retry

- Teruskan `AbortSignal` ke request yang dapat menjadi stale.
- Batalkan request ketika route atau parameter berubah bila request lama tidak lagi relevan.
- Retry hanya untuk error transient dan method aman.
- Jangan retry mutation finansial tanpa idempotency.
- Hindari request waterfall; jalankan request independen secara paralel.

## 8. Testing dan Quality Gate

- Test service untuk URL, method, body, header, response, dan error mapping.
- Test composable untuk loading, success, empty, error, retry, dan cancellation.
- Test component untuk render state, event retry, form pending, dan permission behavior.
- Mock network di boundary HTTP, bukan detail internal reactive state.
- Test duplicate submit dan mutation invalidation.

Checklist:

- [ ] Tidak ada `fetch` langsung di template/component.
- [ ] Semua service memiliki tipe input dan output.
- [ ] Base URL berasal dari environment.
- [ ] Cache key memasukkan parameter request.
- [ ] Loading, empty, error, forbidden, dan success state tersedia.
- [ ] Retry dan mutation aman terhadap duplicate action.
- [ ] Request stale dapat dibatalkan bila diperlukan.
- [ ] Token, secret, dan response sensitif tidak masuk log.
- [ ] Test service, composable, dan alur UI kritis berhasil.
