# Standar Database MySQL

## Cakupan dan baseline

Gunakan MySQL dengan InnoDB sebagai engine tabel default. Target versi harus ditetapkan per proyek. Semua contoh memakai lowercase `snake_case`, tanpa mengandalkan mode SQL yang berbeda antar-environment.

## Naming dan struktur

- Database dan tabel memakai nama domain yang jelas, lowercase, `snake_case`, dan plural bila konvensi proyek menggunakannya.
- Kolom foreign key memakai `{entity}_id`. Waktu memakai `created_at`, `updated_at`, dan suffix kejadian seperti `published_at`.
- Gunakan prefix `mt_`, `tr_`, `dt_`, `rf_`, `lg_`, `cfg_` hanya bila proyek sudah menetapkannya. Jangan mencampur prefix tanpa alasan.
- Hindari reserved word dan identifier yang perlu backtick.
- Tabel pivot memiliki foreign key ke kedua parent dan unique constraint gabungan bila pasangan tidak boleh berulang.

## InnoDB dan integritas

- Semua tabel transaksional memakai InnoDB.
- Primary key wajib ada. Foreign key wajib ada untuk relasi yang memang harus konsisten.
- Tentukan `ON DELETE` dan `ON UPDATE` eksplisit. Hindari cascade pada relasi yang dapat menghapus data bisnis secara luas.
- Gunakan `NOT NULL` bila nilai kosong tidak valid. Gunakan `CHECK` pada versi yang mendukung dan tetap uji perilakunya di target versi.
- `ENUM` hanya untuk nilai yang benar-benar stabil dan dikendalikan engine. Untuk status bisnis yang dapat berubah, gunakan tabel referensi atau `VARCHAR` dengan constraint aplikasi/database.

## Tipe data

- `BIGINT UNSIGNED` untuk ID numerik dengan pertumbuhan besar. Samakan signedness antara parent dan foreign key.
- UUID tidak otomatis lebih aman. Jika perlu ID publik atau distribusi lintas sistem, pilih format UUID yang didukung aplikasi, simpan `BINARY(16)` bila tooling dan operasi tim siap, lalu tetapkan strategi generate dan index.
- Gunakan `DECIMAL(precision, scale)` untuk uang, bukan `FLOAT` atau `DOUBLE`.
- Gunakan `DATETIME(6)` atau `TIMESTAMP(6)` sesuai kebutuhan timezone dan batas range. Tetapkan timezone aplikasi dan koneksi.
- Gunakan `JSON` hanya untuk atribut fleksibel yang belum layak menjadi kolom relasional. Field yang sering difilter sebaiknya menjadi kolom terindeks.
- Default string dan collation harus konsisten. Gunakan `utf8mb4`, bukan `utf8` lama. Pilih collation yang sesuai kebutuhan pencarian dan perbandingan.

## Index dan query

- Primary key, unique key, dan foreign key mengikuti kebutuhan integritas serta query.
- Index komposit mengikuti urutan filter, join, dan sort yang nyata. Jangan menambah index untuk setiap kolom.
- Hindari index redundant, index pada kolom ber-cardinality rendah tanpa bukti, dan index terlalu panjang pada text.
- Review `EXPLAIN` untuk query berat. Pantau slow query log, rows examined, lock wait, dan perubahan setelah release.
- Foreign key dan kolom pencarian utama harus memiliki index yang sesuai. Unique constraint menjadi index sekaligus aturan integritas.

## Transaksi dan konkurensi

- Transaction boundary mengikuti satu unit kerja bisnis, bukan satu request secara otomatis.
- Jangan menahan transaksi saat memanggil API eksternal, mengunggah file, atau menunggu interaksi pengguna.
- Gunakan isolation level default kecuali ada alasan terukur. Pahami deadlock dan siapkan retry terbatas untuk deadlock yang aman diulang.
- Kunci baris hanya saat diperlukan. Konsistenkan urutan akses tabel/baris untuk mengurangi deadlock.
- Operasi idempotent memakai unique key atau request ID, bukan pemeriksaan aplikasi saja.

## Backup dan operasi

- Tentukan full backup, incremental/binlog atau mekanisme point-in-time recovery, retensi, enkripsi, dan lokasi terpisah.
- Uji restore berkala ke environment terisolasi. Ukur RPO dan RTO aktual.
- DDL besar dianalisis dampaknya pada lock, ukuran tabel, replication lag, dan waktu maintenance.
- Migration produksi harus backward-compatible bila aplikasi di-deploy bertahap.
- Pantau storage, connection count, buffer pool, replication, slow queries, deadlock, error rate, dan backup freshness.

## Keamanan

- User aplikasi tidak memakai akun root dan tidak memiliki hak DDL bila tidak perlu.
- Secret berada di secret manager/environment aman, bukan migration atau repository.
- TLS digunakan untuk koneksi lintas host. Batasi network access, bind address, dan exposure publik.
- Masking wajib untuk PII pada log, dump developer, dan observability.

## Checklist MySQL

- [ ] InnoDB dan `utf8mb4` ditetapkan.
- [ ] BIGINT/UUID dipilih berdasarkan akses dan skala nyata.
- [ ] Constraint, index, dan transaction boundary direview.
- [ ] Backup restore test dan binlog/PITR tersedia bila dibutuhkan.
- [ ] User privilege, TLS, secret, dan exposure jaringan aman.
