Tanggal update: 2026-08-22
Agent: NEXORA
Status: Active

## 1. Frontend Coding Standards

Standar ini khusus frontend React.js + TypeScript. Fokusnya UI, UX, accessibility, maintainability, dan konsumsi API Laravel.

- **TypeScript wajib** untuk komponen, hooks, service response, form schema, dan shared types.
- **Feature-based structure**: logic domain tetap di `src/features/<feature-name>/`.
- **Reusable UI di components**: `src/components/` hanya untuk komponen lintas fitur.
- **No business logic besar di page**: page hanya orchestration dan composition.
- **No hardcode role/menu/action**: gunakan `config`, `permissions`, `AuthContext`, dan helper terpusat.
- **Service-first data access**: component tidak import JSON langsung; component memanggil service.
- **Sanitize rendered HTML**: semua TipTap/HTML harus disanitasi sebelum render.
- **Accessibility**: keyboard support, label nyata, focus state, aria hanya bila perlu.
- **Mobile first**: layout harus aman di viewport kecil.

## 2. Component Quality

- Satu component satu tanggung jawab.
- Props kecil dan jelas.
- Komponen generic menerima data siap-render.
- Jangan tambahkan abstraction untuk satu kasus.
- Modal, dialog, tabs, table, dan form harus mendukung empty/loading/error state.

## 3. State and Data Rules

- Gunakan local state untuk UI lokal.
- Gunakan URL state untuk filter/shareable state.
- Gunakan TanStack Query untuk server state.
- Gunakan context/store hanya bila state lintas layout memang perlu.
- Jangan copy server state ke global store tanpa alasan kuat.

## 4. Forms

- Labels nyata, bukan placeholder-only.
- Error per field.
- Pending/disabled state saat submit.
- Validasi client-side untuk UX.
- Validasi backend tetap wajib.

## 5. API Client

- Service return typed response.
- Error handling konsisten.
- Retry hanya jika aman.
- Abort stale request bila perlu.
- Component memanggil service, bukan fetch langsung ke JSON lokal.

## 6. TipTap and Content Render

- Editor: `@tiptap/react`.
- Extensi: `StarterKit`, `Placeholder`, `Youtube`.
- Content lesson disimpan sebagai JSON TipTap.
- Render preview/readonly pakai sanitasi aman.
- Jangan inject HTML mentah tanpa sanitasi.

## 7. Clean Code Patterns

- Guard clause lebih baik dari nested `if`.
- Nama function harus menjelaskan aksi.
- Hindari utility generik yang dipakai satu kali.
- Hindari hook untuk satu `useState` lokal.
- Komposisi lebih baik dari prop explosion.
- Komentar hanya untuk tradeoff yang tidak jelas dari kode.

## 8. Accessibility Baseline

- Semantic HTML dulu.
- Keyboard navigable.
- Focus visible.
- Touch target cukup.
- Icon-only button wajib `aria-label`.
- Status tidak boleh hanya dibedakan warna.
- Toast/async message idealnya `aria-live`.

## 9. Testing Standard

- Unit test untuk pure helper.
- Component test untuk flow UI penting.
- Integration test untuk form dan service adapter.
- E2E test untuk login, course flow, assessment flow, dan editor flow kritis.
- Test loading, empty, error, forbidden state.

## 10. Minimum Quality Gate

- Tidak ada hardcode role/menu di JSX.
- Tidak ada business logic besar di page.
- Tidak ada HTML mentah tanpa sanitasi.
- Mobile aman.
- Akses keyboard aman.
- State server tidak dobel di store.
- Component reusable tetap kecil.

---

skipped: lint rules spesifik per folder, add ketika repo frontend sudah punya standar ESLint/Prettier final.
