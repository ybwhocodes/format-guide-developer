# Vue 3 + TypeScript Clean Code Standards

> **Status:** Team Standard
> **Framework:** Vue 3 + TypeScript
> **Pattern:** Composition API dan SFC
> **Purpose:** Menetapkan implementasi Vue yang mudah dirawat, aman, aksesibel, cepat, dan dapat diuji.

## 1. TypeScript dan Composition API

- Aktifkan `strict`.
- Gunakan `unknown` untuk input eksternal, lalu validasi sebelum dipakai.
- Hindari `any`, type assertion berantai, dan generic yang tidak perlu.
- Gunakan `ref` untuk nilai mutable, `reactive` untuk object state yang memang dikelola sebagai satu unit, dan `computed` untuk derived state.
- Hindari `watch` untuk nilai yang bisa dihitung dengan `computed`.
- `watch` harus punya alasan, cleanup, dan sumber dependency yang jelas.
- Composable mengembalikan API kecil dan tidak menyembunyikan side effect.

```ts
export function useSearch(query: Ref<string>) {
  const normalizedQuery = computed(() => query.value.trim().toLowerCase())
  return { normalizedQuery }
}
```

## 2. Template dan komponen

- Gunakan semantic HTML sebelum ARIA.
- Satu component satu tanggung jawab.
- Props untuk data masuk, emits untuk intent keluar.
- Jangan memodifikasi props.
- Hindari logic kompleks, pemanggilan API, dan nested ternary di template.
- Gunakan `:key` dari ID stabil, bukan index untuk list yang dapat berubah.
- Reusable component tidak tahu role bisnis atau endpoint spesifik.
- `v-html` hanya untuk HTML tepercaya yang sudah disanitasi.

## 3. Clean code

- Nama composable berbentuk `use...`, function memakai kata kerja, boolean memakai `is`, `has`, atau `can`.
- Gunakan guard clause dan pure helper.
- Jangan membuat composable untuk satu `ref` lokal.
- Hindari prop explosion; gunakan object config atau slot bila variasi benar-benar diperlukan.
- Komentar hanya menjelaskan constraint, security, trade-off, atau business rule yang tidak terlihat dari kode.
- Hapus dead code, watcher tidak terpakai, dan abstraction speculative.

## 4. Form dan validasi

- Label nyata terhubung ke input.
- Validasi client memberi feedback cepat; backend tetap memvalidasi trust boundary.
- Error field tampil dekat field dan terhubung dengan `aria-describedby`.
- Submit memiliki pending state dan mencegah duplicate action bila perlu.
- Pertahankan input saat submit gagal.
- Fokuskan field error pertama pada form kompleks.
- Tampilkan success state yang dapat dipahami dan error recovery yang jelas.

```vue
<script setup lang="ts">
const email = ref('')
const error = ref('')
const pending = ref(false)

async function submit() {
  error.value = ''
  if (!email.value.includes('@')) {
    error.value = 'Masukkan email yang valid.'
    return
  }
  pending.value = true
  try {
    await subscribe(email.value)
  } catch {
    error.value = 'Pendaftaran gagal. Coba lagi.'
  } finally {
    pending.value = false
  }
}
</script>

<template>
  <form @submit.prevent="submit" novalidate>
    <label for="email">Email</label>
    <input id="email" v-model="email" type="email" :aria-describedby="error ? 'email-error' : undefined">
    <p v-if="error" id="email-error" role="alert">{{ error }}</p>
    <button type="submit" :disabled="pending" :aria-busy="pending">
      {{ pending ? 'Menyimpan...' : 'Daftarkan email' }}
    </button>
  </form>
</template>
```

## 5. Data fetching dan state

- API call berada di service atau composable feature, bukan template.
- Batalkan request stale dengan `AbortController` bila alur mendukungnya.
- Tangani loading, empty, error, retry, dan permission state.
- Hindari waterfall; jalankan request independen secara paralel.
- Cache dan invalidasi server state melalui library yang dipilih proyek.
- Optimistic update hanya bila rollback jelas.

## 6. Aksesibilitas

- Semua kontrol keyboard-accessible dengan focus state terlihat.
- Icon-only button memiliki `aria-label`.
- Dialog mengelola focus masuk, Escape close, dan restore focus.
- Tab, menu, combobox, accordion, dan disclosure mengikuti pola keyboard yang benar.
- Status tidak hanya dibedakan warna; gunakan label atau teks.
- Kontras teks minimal WCAG AA, target sentuh minimal 44px, dan dukung `prefers-reduced-motion`.
- Gunakan `aria-live` untuk hasil async yang perlu diumumkan.

## 7. Responsive dan mobile

- Mobile bukan desktop yang diperkecil. Stack grid saat konten mulai sempit.
- Cegah horizontal overflow, fixed-width child, clipping teks, dan hover-only flow.
- Gunakan CSS untuk layout dan breakpoint; pilih breakpoint saat konten rusak, bukan daftar device.
- Input harus tetap terlihat saat keyboard mobile terbuka.
- Test zoom 200%, viewport sempit, label panjang, dan data kosong.

## 8. Performance

- Lazy-load route dan komponen berat seperti editor, chart, map, dan PDF viewer.
- Hindari watcher dan computed mahal tanpa kebutuhan.
- Jangan deep reactive-kan object besar bila tidak perlu.
- Gunakan stable key dan virtualisasi untuk list besar.
- Optimalkan image dimensions, font loading, dan request waterfall.
- Animasi utamakan `transform` dan `opacity`; ukur sebelum memoization atau micro-optimization.

## 9. Security

- Secret tidak boleh masuk bundle browser, localStorage, URL, atau log.
- Authorization wajib di backend; route guard hanya UX dan defense-in-depth.
- Sanitasi HTML sebelum `v-html`.
- Validasi file upload di server, batasi tipe dan ukuran.
- Cegah open redirect dengan allowlist tujuan navigasi.
- Jangan menampilkan stack trace atau detail internal ke pengguna.

## 10. Testing

- Unit test untuk pure helper dan transform data.
- Component test untuk render, emit, form validation, keyboard, loading, empty, error, dan permission behavior.
- Integration test untuk service adapter dan mutation invalidation.
- E2E untuk login, route guard, form kritis, dan workflow utama.
- Test behavior pengguna, bukan detail implementasi Vue.
- Tambahkan regression test untuk bug yang sudah diperbaiki.

## 11. Quality gate

- [ ] `vue-tsc` atau typecheck proyek berhasil.
- [ ] Lint dan formatter berhasil.
- [ ] Test unit/component/integration relevan berhasil.
- [ ] Build production berhasil.
- [ ] Semua route penting dapat diakses via deep link.
- [ ] Loading, empty, error, forbidden, dan success state diuji.
- [ ] Keyboard, focus, contrast, reduced motion, dan target sentuh diperiksa.
- [ ] Tidak ada `any` tanpa alasan terdokumentasi.
- [ ] Tidak ada `v-html` tanpa sanitasi.
- [ ] Tidak ada secret client-side atau authorization yang hanya bergantung pada UI.
- [ ] Mobile tidak overflow dan form tidak tertutup keyboard.
- [ ] Bundle dan request waterfall tidak memburuk tanpa alasan terukur.

## 12. Exceptions

Pola Options API boleh dipertahankan pada legacy module yang belum dimigrasikan. Jangan campur pola dalam satu SFC tanpa alasan; migrasi bertahap harus menjaga behavior dan memiliki test regression.
