# Backend Engineering Format Guide

> Panduan penggunaan standar engineering pada folder `Format/`.
>
> **Scope:** backend JavaScript/TypeScript, API contract, runtime, ORM, authentication, testing, migration, documentation, clean code, dan architecture.
> **Status:** Working team guide

---

## 1. Tujuan

Folder ini menjadi sumber standar implementasi backend. Dokumen di dalamnya dipilih berdasarkan tiga keputusan utama:

1. Framework atau runtime yang dipakai.
2. Gaya arsitektur aplikasi.
3. Topik engineering yang sedang dikerjakan.

Jangan menyalin semua standar ke setiap proyek. Pilih standar yang sesuai stack dan kebutuhan proyek, lalu catat keputusan bila ada penyimpangan.

---

## 2. Struktur Folder

```text
Format/
├── Backend/
│   ├── universal-api-contract.md
│   ├── Elysia/
│   │   ├── documentation-api-ui.md
│   │   ├── runtime-standart.md
│   │   ├── standart-orm.md
│   │   ├── LayeredArchitecture/
│   │   └── ModularArchitecture/
│   ├── Express/
│   │   ├── documentation-api-ui.md
│   │   ├── runtime-standart.md
│   │   ├── standart-orm.md
│   │   ├── LayeredArchitecture/
│   │   └── ModularArchitecture/
│   ├── Hono/
│   │   ├── LayeredArchitecture/
│   │   └── ModularArchitecture/
│   └── Laravel/
│       ├── LayeredArchitecture/
│       └── ModularArchitecture/
├── Database/
├── Frontend/
├── Fullstack/
└── Mobile/
```

Folder `Database/` memuat standar engine dan platform database. Pilih `Mysql/`, `PostgreSQL/`, `Redis/`, atau `Supabase/` sesuai sistem yang dipakai, lalu baca `Database/README.md` untuk aturan lintas-engine. Folder `Frontend/` memiliki baseline lintas framework dan standar khusus React/Vue. Folder `Fullstack/` dan `Mobile/` disiapkan untuk standar domain berikutnya. Jangan menaruh dokumen backend ke folder tersebut.

### Frontend

```text
Frontend/
├── README.md
├── React/
│   ├── frontend-react.js-typescript-repository-architecture.md
│   └── frontend-react.js-typescript-clean-code-standards.md
└── Vue/
    ├── vue-typescript-repository-architecture.md
    └── vue-typescript-clean-code-standards.md
```

Aturan pemilihan:

1. Selalu mulai dari `Frontend/README.md` untuk baseline TypeScript, feature boundary, state, form, accessibility, mobile, security, dan quality gate.
2. Pilih tepat satu folder framework utama. Gunakan `React/` untuk React/JSX/hooks dan `Vue/` untuk Vue 3, SFC, Composition API, serta `<script setup>`.
3. Baca dokumen arsitektur untuk folder, routing, service, dan state. Baca dokumen clean code untuk implementasi, testing, performance, security, dan quality gate.
4. Jangan mencampur contoh, API, atau simbol antar-framework. Proyek hybrid harus menetapkan boundary per app/package dan mencatat pengecualian.
5. Pertahankan dokumen React yang sudah ada kecuali ada koreksi yang jelas dan terverifikasi.

Gunakan standar API, authentication, design system, dan browser support proyek sebagai referensi tambahan. Baseline frontend tidak menggantikan kontrak backend.

---

## 3. Aturan Penamaan

### 3.1 Folder

- Gunakan nama domain atau stack yang jelas: `Backend`, `Database`, `Frontend`.
- Gunakan nama framework dengan casing resmi: `Elysia`, `Express`, `Hono`, `Laravel`.
- Gunakan nama arsitektur konsisten: `LayeredArchitecture` atau `ModularArchitecture`.
- Jangan membuat folder berdasarkan nama orang, ticket, atau tanggal.

### 3.2 File

- Gunakan lowercase kebab-case.
- Gunakan nama topik, bukan nama implementasi sementara.
- Pertahankan istilah standar yang sudah dipakai repository, termasuk `standart-orm.md` dan `runtime-standart.md`, agar link yang ada tidak rusak.
- Satu file membahas satu standar utama.

Contoh:

```text
standart-auth.md
migration-seeder.md
unit-testing.md
pattern-repository-package-by-layer.md
pattern-repository-package-by-feature.md
```

---

## 4. Cara Memilih Dokumen

### 4.1 Standar universal

Selalu mulai dari:

```text
Backend/universal-api-contract.md
```

Dokumen ini mengatur naming endpoint, HTTP status, response JSON, error code, pagination, query parameter, versioning, dan idempotency.

### 4.2 Standar framework

Pilih satu folder framework:

| Framework | Gunakan ketika |
|---|---|
| `Elysia` | Proyek memakai ElysiaJS dan TypeScript |
| `Express` | Proyek memakai Express.js |
| `Hono` | Proyek memakai Hono |
| `Laravel` | Proyek memakai Laravel/PHP |

Jangan mencampur contoh implementasi antar-framework tanpa menandai adaptasinya.

### 4.3 Standar arsitektur

Pilih satu pendekatan utama:

| Folder | Kapan dipakai |
|---|---|
| `LayeredArchitecture` | Struktur dipisah berdasarkan layer seperti controller, service, repository, schema |
| `ModularArchitecture` | Struktur dipisah berdasarkan feature/domain seperti customer, order, dan payment |

`LayeredArchitecture` cocok untuk service kecil sampai menengah dengan batas layer yang mudah dipetakan. `ModularArchitecture` cocok untuk domain yang berkembang dan membutuhkan boundary antar-feature.

---

## 5. Peta Dokumen

### API

```text
Backend/universal-api-contract.md
```

Gunakan untuk semua service backend. Aturan utama: noun plural pada endpoint, HTTP status sesuai kondisi, response konsisten, error code eksplisit, pagination terdefinisi, API versioning, dan idempotency untuk transaksi.

### Runtime

```text
{Framework}/runtime-standart.md
```

Gunakan untuk memilih Bun.js atau Node.js, package manager, scripts, test runner, TypeScript configuration, dan environment execution.

### ORM

```text
{Framework}/standart-orm.md
```

Gunakan saat menentukan akses database, model/schema, query convention, transaction, dan batasan repository.

### Authentication

```text
{Framework}/{Architecture}/standart-auth.md
```

Gunakan untuk login, token, session, password hashing, middleware, authorization, dan proteksi endpoint.

### Testing

```text
{Framework}/{Architecture}/unit-testing.md
```

Gunakan untuk unit test service, repository boundary, controller behavior, mock, fixture, dan coverage.

### Migration dan seeder

```text
{Framework}/{Architecture}/migration-seeder.md
```

Gunakan untuk perubahan schema, migration ordering, rollback, seed data, dan environment setup.

### Clean code

```text
{Framework}/{Architecture}/clean-code.md
```

Gunakan untuk naming, strict typing, validasi input, batas layer, error handling, dependency direction, dan pola implementasi.

### Repository by layer

```text
{Framework}/LayeredArchitecture/pattern-repository-package-by-layer.md
```

Gunakan bila repository berada pada layer global seperti `src/repositories/`.

### Repository by feature

```text
{Framework}/ModularArchitecture/pattern-repository-package-by-feature.md
```

Gunakan bila repository tinggal di modul domain seperti `src/modules/customer/customer.repository.ts`.

### Dokumentasi API dan UI

```text
{Framework}/documentation-api-ui.md
```

Gunakan untuk dokumentasi endpoint, schema, contoh request/response, dan integrasi UI.

---

## 6. Template Dokumen Standar

Setiap dokumen baru wajib memakai struktur minimum berikut:

```markdown
# [Nama Standar]

> **Status:** Draft | Team Standard | Scaled System Standard
> **Framework:** [Framework atau Universal]
> **Pattern:** [Layered Architecture | Modular Architecture | Universal]
> **Purpose:** [Tujuan dalam satu kalimat]

---

## 1. Scope

[Bagian yang dicakup dan tidak dicakup.]

## 2. Rule

- [Aturan yang wajib.]
- [Aturan yang dilarang.]

## 3. Struktur atau Alur

```text
[Contoh struktur]
```

## 4. Implementasi

```typescript
// Contoh minimal yang dapat dijalankan atau diadaptasi.
```

## 5. Validation Checklist

- [ ] [Pemeriksaan pertama]
- [ ] [Pemeriksaan kedua]

## 6. Exceptions

[Alasan pengecualian dan batas penggunaannya.]
```

### Aturan isi

- Tulis tujuan dokumen secara spesifik.
- Bedakan aturan wajib, rekomendasi, dan contoh.
- Sertakan contoh struktur atau kode jika aturan sulit dipahami tanpa contoh.
- Gunakan contoh yang konsisten dengan framework dokumen.
- Jangan menyimpan secret, token asli, data pelanggan, atau credential.
- Gunakan `WARNING` untuk konsekuensi yang dapat menyebabkan kerusakan data atau celah keamanan.
- Tambahkan checklist agar standar bisa diverifikasi saat review.

---

## 7. Checklist Implementasi Proyek

### Pemilihan standar

- [ ] Framework sudah dipilih.
- [ ] Runtime sudah dipilih.
- [ ] Arsitektur sudah dipilih.
- [ ] ORM dan database sudah dipilih.
- [ ] Dokumen universal API sudah diterapkan.

### Struktur kode

- [ ] Controller tidak menjalankan query database langsung.
- [ ] Business rule berada di service/domain layer.
- [ ] Repository menjadi batas akses database.
- [ ] Modul tidak mengubah data modul lain melalui repository langsung.
- [ ] Validasi request dilakukan di trust boundary.
- [ ] TypeScript strict aktif bila proyek menggunakan TypeScript.

### API

- [ ] Endpoint menggunakan noun plural dan kebab-case.
- [ ] Status HTTP sesuai kondisi.
- [ ] Error memiliki bentuk konsisten.
- [ ] Validation error memakai struktur field error.
- [ ] Pagination memiliki batas maksimum.
- [ ] Endpoint transaksi menggunakan idempotency key.

### Quality

- [ ] Unit test tersedia untuk business rule penting.
- [ ] Migration dapat dijalankan pada environment kosong.
- [ ] Seeder tidak menggunakan data sensitif.
- [ ] Dokumentasi API sesuai implementasi aktual.
- [ ] Lint, format, test, dan build berhasil.

---

## 8. Aturan Perubahan Dokumen

1. Cari dokumen yang sudah ada sebelum membuat file baru.
2. Perbarui standar lama bila topiknya sama.
3. Buat file baru hanya bila scope berbeda jelas.
4. Jangan mengubah nama file tanpa memperbarui link dan index.
5. Setelah perubahan, cek semua contoh kode dan path folder.
6. Catat breaking change pada bagian `Exceptions` atau changelog proyek.

---

## 9. Preview

Preview seluruh dokumen tersedia di:

```text
Format/index.html
```

File tersebut membaca markdown dari folder `Format/` saat dibuka melalui web server lokal. Preview menampilkan tree folder, pencarian, filter stack/arsitektur, dan isi dokumen dengan tema visual yang mengikuti shadcn: netral, border halus, radius kecil, dan fokus pada keterbacaan.
