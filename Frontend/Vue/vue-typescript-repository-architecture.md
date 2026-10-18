# Vue 3 + TypeScript Repository Architecture

> **Status:** Team Standard
> **Framework:** Vue 3 + TypeScript
> **Pattern:** Feature-based architecture
> **Purpose:** Menjaga boundary feature, UI, state, dan integrasi API tetap jelas pada aplikasi Vue.

## 1. Scope

Berlaku untuk Vue 3 dengan Composition API, Single-File Component (SFC), `<script setup>`, Vue Router, Pinia, dan server-state library yang dipilih proyek.

## 2. Struktur repository

```text
src/
├── app/
│   ├── App.vue
│   ├── router.ts
│   └── providers/
├── components/
│   ├── ui/
│   └── shared/
├── composables/
├── config/
├── features/
│   └── customers/
│       ├── api/
│       ├── components/
│       ├── composables/
│       ├── forms/
│       ├── pages/
│       ├── types.ts
│       └── index.ts
├── layouts/
├── pages/
├── stores/
├── services/
├── types/
├── utils/
└── assets/
```

## 3. Boundary folder

- `features/`: UI dan logic domain. Feature boleh mengimpor shared code, bukan detail internal feature lain.
- `components/ui/`: primitive reusable, semantic, dan accessible. Tidak memanggil API.
- `components/shared/`: komposisi lintas domain yang sudah terbukti reusable.
- `pages/`: route composition; tipis, tanpa business rule panjang.
- `layouts/`: shell navigasi dan struktur halaman.
- `services/`: HTTP client, adapter, serializer, dan error mapping.
- `composables/`: stateful behavior reusable; composable domain tetap di feature.
- `stores/`: client state lintas halaman, session, preference, atau workflow yang benar-benar global.
- `utils/`: pure helper tanpa Vue, router, store, atau network.

## 4. SFC dan `<script setup>`

SFC menjadi unit utama. Urutan disarankan: `<script setup lang="ts">`, `<template>`, lalu `<style scoped>` atau styling system proyek. Satu SFC memiliki satu tanggung jawab.

```vue
<script setup lang="ts">
import { computed } from 'vue'

type Customer = { id: string; name: string; active: boolean }

const props = defineProps<{ customer: Customer }>()
const emit = defineEmits<{ archive: [id: string] }>()
const statusLabel = computed(() => (props.customer.active ? 'Aktif' : 'Arsip'))
</script>

<template>
  <article aria-labelledby="customer-name">
    <h2 id="customer-name">{{ customer.name }}</h2>
    <p>{{ statusLabel }}</p>
    <button type="button" @click="emit('archive', customer.id)">Arsipkan</button>
  </article>
</template>
```

Aturan: props immutable, emit untuk event ke parent, hindari `any`, gunakan `computed` untuk derived state, dan jangan menaruh fetch langsung di template.

## 5. Routing

- Definisikan route di `app/router.ts` atau convention proyek.
- Gunakan lazy import untuk halaman besar.
- Guard hanya untuk autentikasi dan authorization route-level; backend tetap menjadi enforcement.
- Simpan filter, tab, sort, dan pagination yang perlu dibagikan di query string.
- Pastikan deep link, back/forward, 404, forbidden, dan redirect diuji.

```ts
const routes = [{
  path: '/customers',
  component: () => import('@/features/customers/pages/CustomersPage.vue'),
  meta: { requiresAuth: true, permission: 'customer.view' },
}]
```

## 6. Pinia dan server state

- Pinia untuk auth session, preference, modal global, atau client workflow lintas halaman.
- Jangan menyalin response API ke Pinia bila server-state library sudah menangani cache, invalidation, retry, dan loading.
- Query key harus spesifik. Mutation menginvalidasi query yang terdampak.
- Service API berada di `features/<name>/api` atau `services` bila lintas feature.

```ts
export const useSessionStore = defineStore('session', () => {
  const user = ref<User | null>(null)
  const isSignedIn = computed(() => user.value !== null)
  return { user, isSignedIn }
})
```

## 7. Quality boundary

API response divalidasi di service dengan schema runtime bila tersedia. Component hanya menerima model siap-render. Jangan import JSON mock langsung ke SFC. Mock service dan HTTP service memakai kontrak method sama.

## 8. Validation checklist

- [ ] Feature domain tidak bocor ke shared component.
- [ ] SFC memakai `<script setup lang="ts">` bila tidak ada alasan jelas untuk pola lain.
- [ ] Router memiliki lazy route, guard, forbidden, dan 404 yang sesuai.
- [ ] Pinia tidak menyimpan duplikasi server state.
- [ ] Service mengisolasi fetch, mapping, error, dan validasi response.
- [ ] Tidak ada business logic panjang di page/layout/template.
- [ ] Loading, empty, error, dan permission state tersedia.
- [ ] Typecheck dan test berjalan.

## 9. Exceptions

Komponen kecil boleh berada langsung di feature tanpa subfolder. Store Pinia boleh menyimpan server data hanya bila library query tidak tersedia dan invalidation manual terdokumentasi.
