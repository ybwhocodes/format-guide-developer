

Tanggal update: 2026-08-22
Agent: NEXORA
Status: Active

## 1. Prinsip Utama

- Pertahankan behavior, layout, responsiveness, dan user flow prototype semirip mungkin.
- Jangan redesign kecuali untuk memperbaiki bug, aksesibilitas, atau konsistensi implementasi.
- Feature-based architecture: logic domain ditempatkan di `src/features/<feature-name>/`.
- `components` hanya untuk reusable UI lintas fitur.
- Jangan menaruh business logic besar di `pages` atau generic component.
- Role, menu, route permission, dan action permission harus terpusat.
- Data entity bisnis tidak boleh hardcode di JSX; gunakan mock JSON melalui `services`.
- Service layer harus bisa diganti dari mock JSON ke HTTP API tanpa refactor besar di component.

## 2. Struktur Repository

```txt
src/
  components/
  config/
  context/
  features/
  hooks/
  icons/
  layout/
  pages/
  routes/
  services/
  stores/
  types/
  utils/
```

## 3. Tanggung Jawab Folder

### `src/components/`

Isi:

- Reusable UI lintas fitur.
- Subfolder `src/components/ui/` berisi primitive/accessible components yang bersumber dari shadcn/ui.
- Tidak tahu domain bisnis spesifik.
- Tidak memanggil service langsung.

Contoh:

```txt
src/components/
  ui/
    Button.tsx
    Card.tsx
    Dialog.tsx
    Input.tsx
    Select.tsx
    Table.tsx
    Tabs.tsx
    Badge.tsx
  DataTable.tsx
  EmptyState.tsx
  Field.tsx
  KpiCard.tsx
  PageIntro.tsx
  Toast.tsx
```

Aturan:

- Komponen generic menerima props data siap-render.
- Tidak ada pengecekan `role === 'fat'` di component generic.
- `DataTable` mendukung search, filter, sort, pagination, empty state, row action render.
- `Modal` wajib accessible: focus trap, Escape close, restore focus.

### `src/config/`

Isi:

- Konfigurasi aplikasi yang stabil.
- Menu, route metadata, permission matrix, status mapping, theme tokens.

Contoh:

```txt
src/config/
  app.ts
  menu.ts
  permissions.ts
  routes.ts
  status.ts
```

Aturan:

- Sidebar, protected route, dan action visibility membaca sumber permission yang sama.
- Hindari string role tersebar di page/component.
- Jangan hardcode union role final seperti `RoleId = 'super' | 'mitra' | 'fat'` sebagai sumber kebenaran permanen di kode frontend. Role dan permission harus didasarkan pada data dinamis yang nantinya bersumber dari backend.
- Siapkan mock JSON data (`roles.json`, `users.json`, `permissions.json`) untuk mendefinisikan role dan mapping permission secara dinamis.
- Gunakan TypeScript interface yang siap mendukung integrasi backend, di mana `RoleId` atau `role` direpresentasikan sebagai entitas dinamis (code/ID string) yang dicocokkan dengan metadata menu/routes.

### `src/context/`

Isi:

- React Context yang benar-benar global.

Contoh:

```txt
src/context/
  AuthContext.tsx
  ToastContext.tsx
  ModalContext.tsx
```

Aturan:

- Jangan gunakan Context untuk server data/list entity.
- Auth/user role boleh Context karena mempengaruhi layout dan route.
- Toast/modal boleh Context jika dipakai lintas app.

### `src/features/`

Isi:

- Domain-specific UI, hooks, helper, service adapter, dan type lokal fitur.

Struktur minimal:

```txt
src/features/
  partnership/
    components/
    hooks/
    services/
    types.ts
  customers/
    components/
    hooks/
    services/
    types.ts
  finance/
    components/
    hooks/
    services/
    types.ts
  network/
    components/
    hooks/
    services/
    types.ts
  radius/
    components/
    hooks/
    services/
    types.ts
```

Aturan:

- Logic spesifik `Data Mitra`, `Registrasi ONU`, `FAT Settlement`, `Topologi` berada di feature masing-masing.
- Helper perhitungan finance yang hanya dipakai finance tetap di `features/finance/helpers.ts`.
- Jangan menaruh component domain seperti `PartnerFinanceOverview` di `components/` global.

### `src/hooks/`

Isi:

- Hooks reusable lintas fitur.

Contoh:

```txt
src/hooks/
  useDebouncedValue.ts
  useMediaQuery.ts
  usePagination.ts
  useRouteQuery.ts
```

Aturan:

- Hook domain-specific tetap di feature.
- Jangan membuat hook untuk satu `useState` lokal.

### `src/icons/`

Isi:

- Icon React components hasil translasi dari `ICONS` inline SVG.

Contoh:

```txt
src/icons/
  BuildingIcon.tsx
  WalletIcon.tsx
  index.ts
```

Aturan:

- Icon menerima props SVG standar.
- Import icon langsung atau dari index kecil yang tidak membloat bundle.

### `src/layout/`

Isi:

- App shell, sidebar, topbar, content container.

Contoh:

```txt
src/layout/
  AppLayout.tsx
  Sidebar.tsx
  Topbar.tsx
  UserSwitcher.tsx
```

Aturan:

- Layout membaca menu dari `config/menu.ts` dan permission dari `config/permissions.ts`.
- Layout tidak memuat data bisnis halaman.
- Sidebar hanya render item yang diizinkan.

### `src/pages/`

Isi:

- Route-level composition.
- Menghubungkan layout route dengan feature component.

Contoh:

```txt
src/pages/
  PartnershipPartnersPage.tsx
  PartnershipUsersPage.tsx
  PartnershipRevenuePage.tsx
  CustomersPage.tsx
  CustomerRegistrationPage.tsx
  PackagesPage.tsx
  FinancePage.tsx
  RadiusMonitoringPage.tsx
  InfrastructureTopologyPage.tsx
  ForbiddenPage.tsx
```

Aturan:

- Page tidak berisi business logic panjang.
- Page boleh membaca route params/search params dan memanggil feature container.
- Page tidak memanipulasi DOM.

### `src/routes/`

Isi:

- Route definitions, protected route, redirect default role.

Contoh:

```txt
src/routes/
  AppRouter.tsx
  ProtectedRoute.tsx
  routeConfig.ts
```

Aturan:

- Direct URL access dicek di route guard.
- Default route role:
  - Super User: `/partnership/mitra`.
  - Admin Mitra: `/customers`.
  - FAT: `/finance/mitra`.
- Hash route prototype boleh diterjemahkan ke path route React, tetapi behavior harus setara.

### `src/services/`

Isi:

- Data access abstraction.
- Mock implementation awal membaca JSON/local data.
- HTTP implementation final bisa mengganti internal service.

Contoh:

```txt
src/services/
  client.ts
  mockDb.ts
  partnerService.ts
  customerService.ts
  packageService.ts
  financeService.ts
  networkService.ts
  radiusService.ts
```

Aturan:

- Component/page hanya memanggil service, bukan import JSON langsung.
- Service return typed response.
- Mutation service bertanggung jawab update mock store sementara.
- Ketika API tersedia, ganti isi service ke HTTP client.

Contoh pattern:

```ts
export const customerService = {
  getList(params: CustomerListParams): Promise<ListResponse<Customer>>,
  create(payload: CustomerCreatePayload): Promise<ApiResponse<Customer>>,
  update(id: string, payload: CustomerUpdatePayload): Promise<ApiResponse<Customer>>,
}
```

### `src/stores/`

Isi:

- Global/client state bila benar-benar diperlukan.

Contoh:

```txt
src/stores/
  authStore.ts
  mockDataStore.ts
```

Aturan:

- Jangan duplikasi server state jika service/query cache sudah cukup.
- Untuk fase mock, `mockDataStore` boleh menyimpan data in-memory agar mutation tercermin di semua page.
- Auth role prototype boleh store/context.

### `src/types/`

Isi:

- Shared TypeScript types/interface.

Contoh:

```txt
src/types/
  api.ts
  auth.ts
  entities.ts
  filters.ts
  permissions.ts
```

Aturan:

- Entity utama ditempatkan di `entities.ts` atau domain file.
- Feature-specific type tetap di feature bila tidak dipakai lintas fitur.
- Schema belum confirmed backend diberi komentar `Proposed` atau `TBD`.

### `src/utils/`

Isi:

- Pure helper functions lintas fitur.

Contoh:

```txt
src/utils/
  format.ts
  ids.ts
  dates.ts
  status.ts
```

Aturan:

- Tidak import React.
- Tidak memanggil service.
- Pure dan mudah dites.

## 4. Role-Based UI Terpusat (Dynamic configuration)

### Role & Permission Models (Proposed/BE-aligned)

Data role dan permissions di-load secara dinamis dari mock JSON (misalnya `roles.json` dan `permissions.json`) yang siap digantikan endpoint API `/auth/me` dan `/roles`.

```ts
export interface Permission {
  key: string;       // e.g., "customer.create"
  label: string;     // Deskripsi permission
}

export interface Role {
  id: string;        // UUID atau ID unik
  code: string;      // e.g., "super", "mitra", "fat"
  name: string;      // Nama role untuk display
  permissions: string[]; // List of permission keys/codes
}

export interface User {
  id: string;
  name: string;
  email: string;
  role_code: string; // relational reference ke Role.code
  partner_id: string | null;
}
```

### Dynamic Menu Config (Proposed/BE-aligned)

Menu sidebar tidak memakai logic hardcoded `isSuperUser()` atau `isFAT()` yang berantakan, melainkan mapping dinamis berdasarkan permission key atau role codes:

```ts
export interface MenuItem {
  key: string;
  path: string;
  label: string;
  icon?: string;
  badgeFn?: string; // string key function lookup
  permissions?: string[]; // permissions required to view
}

export interface MenuGroup {
  group: string;
  items: MenuItem[];
}

export const menuConfig: MenuGroup[] = [
  {
    group: 'Kemitraan',
    items: [
      { key: 'partnership.mitra', path: '/partnership/mitra', label: 'Data Mitra', permissions: ['partnership.mitra.view'] },
      { key: 'partnership.pengguna', path: '/partnership/pengguna', label: 'Manajemen Pengguna', permissions: ['partnership.pengguna.view'] },
      { key: 'partnership.pendapatan', path: '/partnership/pendapatan', label: 'Dashboard Pendapatan', permissions: ['partnership.pendapatan.view'] },
    ],
  },
  {
    group: 'Pelanggan',
    items: [
      { key: 'customer.pelanggan', path: '/customers', label: 'Data Pelanggan', permissions: ['customer.view'] },
      { key: 'customer.registrasi', path: '/customers/registrasi', label: 'Registrasi Pelanggan', permissions: ['customer.registrasi.view'] },
      { key: 'customer.paket', path: '/customers/packages', label: 'Paket Layanan', permissions: ['package.view'] },
    ],
  },
  {
    group: 'Keuangan',
    items: [
      { key: 'keuangan.mitra', path: '/finance/mitra', label: 'Keuangan Mitra', permissions: ['finance.mitra.view'] },
    ],
  },
];
```

Catatan:

- `partnership.pendapatan` untuk FAT adalah direct route permission, bukan sidebar item, mengikuti prototype. Filter item sidebar dilakukan di level runtime menggunakan helper.
- Action permission (seperti tombol verify deposit, delete OLT, dll) harus menggunakan selector helper `hasPermission(user, 'permission.key')`.

### Permission Helpers

```ts
export function hasPermission(user: User | null, permissionKey: string, rolesData: Role[]): boolean
export function getSidebarMenuForUser(user: User | null, menuList: MenuGroup[], rolesData: Role[]): MenuGroup[]
export function getDefaultRouteForUser(user: User | null): string
```

Aturan:

- Jangan lakukan `if (user.role === 'fat')` di banyak component. Gunakan helper `hasPermission`.
- Gunakan hooks seperti `usePermission('customer.create')` di level component untuk menyembunyikan/menonaktifkan action button.

## 5. Translasi Prototype ke React

### Dari HTML string ke component

Prototype pattern:

- Build HTML string.
- Set `innerHTML`.
- Query selector.
- Attach event listener.

React pattern:

- Component render JSX.
- Data/state via props/useState.
- Event via `onClick`, `onChange`, `onSubmit`.
- Modal state via context/store.

Contoh mapping:

| Prototype | React |
| --- | --- |
| `Views['customer.pelanggan']` | `CustomersPage` + `CustomerListFeature` |
| `DataTable(opts)` | `<DataTable rows columns filters />` |
| `Modal.open(...)` | `<Modal open onClose>...</Modal>` atau modal service/context |
| `renderKPIs(items)` | `<KpiGrid items={items} />` |
| `statusBadge(text)` | `<StatusBadge status={status} />` |
| `window.location.hash` | React Router path/search params |
| `CURRENT_USER` | `AuthContext` / `authStore` |

### Jangan Diubah Tanpa Kebutuhan

- Nama menu dan label Bahasa Indonesia.
- Urutan sidebar per role.
- Badge antrian registrasi dan pending finance.
- Modal invoice-style.
- Filter/search/table behavior.
- Layout light neutral saat ini.
- Breakpoint mobile 960px, kecuali sistem CSS React memilih equivalent.

## 6. Mock Data dan Services

### Temporary Mock Data

Disarankan:

```txt
src/services/mock-data/
  partners.json
  users.json
  packages.json
  customers.json
  invoices.json
  payments.json
  depositTopUps.json
  settlements.json
  infrastructure.json
```

Aturan:

- JSON berisi entity seed, bukan logic.
- Logic mutation dan derived data di service/helper.
- Mock store menggabungkan JSON ke in-memory state.

### Service Replacement Strategy

Fase mock:

```ts
const USE_MOCK = true
export const partnerService = USE_MOCK ? mockPartnerService : httpPartnerService
```

Fase API:

- Implement `httpPartnerService` dengan method signature sama.
- Component tidak berubah.
- Error handling memakai `ApiError` standar.

## 7. Proposed API/Data Schema

Referensi detail ada di `docs/FSD.md#6-proposed-typescript-data-models`.

Minimum shared response:

```ts
export interface ApiResponse<T> { data: T; message?: string }
export interface ListResponse<T> { data: T[]; total: number; page: number; pageSize: number }
export interface ApiError { message: string; code?: string; fieldErrors?: Record<string, string> }
```

Filter/pagination:

```ts
export interface PageParams { page?: number; pageSize?: number; search?: string }
export interface PartnerListParams extends PageParams { status?: string; area?: string }
export interface CustomerListParams extends PageParams { partnerId?: string; status?: string; type?: string }
export interface FinanceFilterParams extends PageParams { partnerId?: string; status?: string; dateFrom?: string; dateTo?: string }
```

Status semua contract: `Proposed`, sampai backend disetujui.

## 8. Feature Breakdown Rekomendasi

```txt
src/features/partnership/
  components/PartnerTable.tsx
  components/PartnerForm.tsx
  components/PartnerFinanceOverview.tsx
  components/UserTable.tsx
  hooks/usePartners.ts
  hooks/usePartnerFinanceOverview.ts


src/features/customers/
  components/CustomerTable.tsx
  components/CustomerFormModal.tsx
  components/RegistrationQueue.tsx
  components/RegistrationPage.tsx
  components/OnuScriptPanel.tsx
  components/PackageTable.tsx
  components/PackageFormModal.tsx


src/features/finance/
  components/AdminFinanceView.tsx
  components/FinanceMutationTable.tsx
  components/CustomerPaymentPanel.tsx
  components/FatFinanceView.tsx
  components/DepositVerifyModal.tsx
  components/SettlementProcessModal.tsx
  components/InvoiceModal.tsx
  helpers/financeCalculations.ts


src/features/radius/
  components/RadiusMonitoringTable.tsx
  components/RadiusDetailModal.tsx


src/features/network/
  components/TopologyTree.tsx
  components/TopologyDetailPanel.tsx
  components/NetworkNodeFormModal.tsx
```

## 9. Styling Strategy

- Port CSS token dari `erp-mitra-prototype.html` ke global stylesheet.
- Pertahankan class naming kebab-case jika CSS dipakai ulang.
- React component boleh memakai existing CSS class untuk menjaga tampilan.
- Jangan ubah `--color-accent` tanpa keputusan; `RULES.md` dan CSS aktual konflik.
- Gunakan CSS untuk responsive, hover, focus, transition.

## 10. State Management

Gunakan state lokal untuk:

- Active tab/subtab.
- Modal open/selected record.
- Form draft.
- Table filter/search/sort/pagination.

Gunakan URL state untuk:

- Route-level detail seperti registration `id`.
- Filter yang perlu shareable bila diperlukan di fase lanjut. Prototype belum memakai ini kecuali hash query.

Gunakan store/context untuk:

- Current user/role.
- Toast/modal global.
- Mock DB in-memory bila service mock perlu mutation global.

Jangan gunakan global state untuk:

- Data table lokal.
- Satu modal milik satu page.
- Derived KPI yang bisa dihitung dari data.

## 11. Loading, Empty, Error, Success

- Loading: skeleton/table placeholder saat service async.
- Empty: gunakan `EmptyState` sesuai copy prototype.
- Error: inline error per page + retry; action error via toast dan field error.
- Success: toast setelah mutation.
- Forbidden: `ForbiddenPage` untuk route tidak berizin.

## 12. Accessibility Baseline

- Semua button gunakan `<button type="button">` kecuali submit form.
- Field wajib punya label nyata.
- Dialog fokus ke modal saat open dan kembali ke trigger saat close.
- Tabs/subtabs harus keyboard accessible.
- Icon-only button wajib `aria-label`.
- Status tidak boleh hanya warna; text tetap tampil.
- Toast/async status idealnya `aria-live`.

## 13. Technical Notes Dari Prototype

- Direct DOM manipulation sangat banyak; migrasi harus memecah render function menjadi component.
- `DataTable` melakukan direct property search; React version sebaiknya support accessor.
- `Modal.open` menerima HTML string; React version harus composition-based.
- `window.dispatchEvent('keuangan-refresh')` diganti state invalidation/service refresh.
- `CURRENT_USER` global diganti auth context/store.
- `DB` global diganti service mock + store.
- `navigator.clipboard` tetap dipakai untuk copy script dengan error handling.
- Beberapa legacy views masih ada (`deposit.dashboard`, `settlement.dashboard`, `payment.gateway`) tetapi tidak muncul di sidebar; jangan migrasi sebagai menu utama kecuali dibutuhkan.

## 14. Minimum Quality Gate

- Sidebar sesuai role.
- Direct URL forbidden bekerja.
- Semua menu prototype termigrasi.
- CRUD/mock mutations refresh UI.
- Desktop dan mobile dicek.
- Tidak ada data entity bisnis hardcoded di JSX.
- Permission action tidak tersebar sebagai string role.
- Semua Proposed schema ditandai sampai backend final.

skipped: Redux/global store, add ketika state client lintas fitur benar-benar tidak cukup dengan URL state + TanStack Query.
