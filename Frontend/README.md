# Frontend Standards

> Standar lintas framework untuk frontend. Pilih folder framework sesuai teknologi proyek, lalu gunakan baseline di dokumen ini.

## Folder

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

## Aturan pemilihan

1. Baca `Frontend/README.md` sebagai baseline.
2. Pilih tepat satu folder framework utama: `React/` atau `Vue/`.
3. Gunakan `React/` untuk React dan ekosistem JSX/hooks. Gunakan `Vue/` untuk Vue 3, Single-File Component, Composition API, dan composable.
4. Gunakan dokumen arsitektur untuk struktur repository, boundary feature, routing, service, dan state.
5. Gunakan dokumen clean code untuk implementasi komponen, TypeScript, form, aksesibilitas, testing, performance, security, dan quality gate.
6. Jangan mencampur contoh atau simbol antar-framework. Standar lintas framework hanya mengatur prinsip, bukan API framework.
7. Jika proyek hybrid, tetapkan boundary per package/app dan dokumentasikan pengecualian. Jangan memindahkan pola React ke Vue atau sebaliknya tanpa adaptasi eksplisit.

## Baseline lintas framework

- TypeScript strict dan runtime validation di trust boundary.
- Feature/domain menjadi boundary utama; shared component tidak memuat business rule.
- Server state berada pada cache/query layer, bukan global UI store.
- Route, permission, dan action harus konsisten dengan backend.
- Form memiliki label nyata, validasi, error field, pending, dan success state.
- Semantic HTML, keyboard navigation, visible focus, contrast WCAG AA, dan target sentuh minimal 44px.
- Layout mobile reflow tanpa horizontal overflow, clipping, atau hover-only interaction.
- Async view memiliki loading, empty, error, dan forbidden state bila relevan.
- Jangan expose secret ke browser, render HTML mentah tanpa sanitasi, atau menganggap validasi client sebagai security boundary.
- Test behavior pengguna dan business rule penting. Jalankan lint, typecheck, test, dan build sebelum merge.

## Cara memakai standar

```text
1. Baseline ini
2. Dokumen arsitektur framework
3. Dokumen clean code framework
4. Standar API, auth, dan design system proyek
5. Quality gate sebelum merge
```

Dokumen ini tidak menggantikan aturan proyek yang lebih ketat. Bila terjadi konflik, keamanan, aksesibilitas, kontrak API, dan keputusan proyek yang tercatat menjadi prioritas.
