# Panduan Standar Database

## Cara memilih folder

- `Mysql/`: aplikasi yang memakai MySQL, terutama InnoDB.
- `PostgreSQL/`: aplikasi yang memakai PostgreSQL dengan schema, constraint, JSONB, dan transaksi kuat.
- `Redis/`: cache, session, rate limit, queue ringan, lock, dan data berumur pendek.
- `Supabase/`: proyek Supabase yang memakai PostgreSQL terkelola, Auth, Storage, API, dan Edge Functions.

Pilih satu standar engine sebagai sumber aturan teknis. Gunakan standar lintas-engine di bawah sebagai baseline, lalu ikuti aturan khusus engine. Supabase memakai PostgreSQL sebagai fondasi, tetapi kontrol platform, RLS, Auth, Storage, dan migrasi Supabase tetap wajib dibaca.

## Aturan lintas-engine

1. Data memiliki owner, klasifikasi, retensi, dan tingkat sensitivitas.
2. Schema berubah melalui migration yang dapat direview dan dijalankan ulang. Jangan mengubah produksi secara manual tanpa audit trail.
3. Aplikasi memvalidasi input, database menegakkan constraint. Validasi aplikasi bukan pengganti `NOT NULL`, `UNIQUE`, `CHECK`, atau foreign key.
4. Query harus memakai parameter binding. Jangan menyusun SQL dari input pengguna.
5. Least privilege berlaku untuk user aplikasi, migration user, operator, dan service account.
6. Backup dianggap valid hanya setelah restore test berhasil. Catat RPO, RTO, retensi, dan owner.
7. Index dibuat dari query nyata. Ukur dengan execution plan sebelum dan sesudah perubahan.
8. PII, credential, token, dan secret tidak boleh masuk log, fixture publik, repository, atau backup tanpa proteksi.
9. Perubahan berisiko memakai rollout bertahap: tambah kompatibel, backfill terukur, pindahkan pembaca/penulis, lalu hapus setelah masa observasi.
10. Dokumentasikan pengecualian, dampak, cara rollback, dan tanggal review.

## Checklist review

- [ ] Engine dan versi target tertulis.
- [ ] Naming, tipe data, constraint, dan relasi konsisten.
- [ ] Migration memiliki rollback atau prosedur pemulihan yang jelas.
- [ ] Query kritis memiliki index yang dibuktikan execution plan.
- [ ] Transaction boundary dan perilaku retry ditentukan.
- [ ] Backup, restore test, monitoring, dan alert tersedia.
- [ ] Hak akses, enkripsi, secret, dan data exposure ditinjau.
- [ ] Retensi dan purge untuk data sementara/log ditentukan.

## Dokumen utama

- [MySQL](Mysql/mysql-database-standard.md)
- [PostgreSQL](PostgreSQL/postgresql-database-standard.md)
- [Redis](Redis/redis-database-standard.md)
- [Supabase](Supabase/supabase-database-standard.md)
