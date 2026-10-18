# Standar Database PostgreSQL

## Cakupan dan baseline

Gunakan PostgreSQL dengan versi target yang tertulis. Bedakan database, schema, role, dan extension. Schema aplikasi tidak boleh otomatis memakai `public` bila pemisahan ownership atau exposure membutuhkan boundary yang jelas.

## Naming dan schema

- Gunakan lowercase `snake_case`, identifier bisnis yang jelas, dan hindari quoted identifier.
- Pisahkan schema berdasarkan boundary yang nyata, misalnya `app`, `audit`, atau `reporting`. Jangan membuat schema hanya untuk kosmetik.
- Gunakan role terpisah untuk migration, runtime read-write, read-only, reporting, dan administrasi.
- Tabel pivot memiliki foreign key dan unique constraint gabungan bila pasangan harus unik.

## Identity, UUID, dan tipe data

- Gunakan `GENERATED ... AS IDENTITY` untuk ID numerik baru. Hindari serial pada desain baru.
- Gunakan `uuid` bila ID perlu tidak mudah ditebak, dibuat lintas layanan, atau dibagikan sebagai identifier publik. Tetapkan generator dan strategi index.
- Gunakan `timestamptz` untuk waktu absolut. Gunakan `date` untuk tanggal tanpa waktu dan `time` hanya jika domain benar-benar membutuhkannya.
- Gunakan `numeric` untuk uang, bukan floating point.
- Gunakan `text` tanpa batas artifisial kecuali ada aturan domain. Gunakan `varchar(n)` saat batas tersebut memiliki makna bisnis.
- `jsonb` cocok untuk metadata semi-terstruktur dan query containment. Data inti, relasi, constraint, dan field yang sering difilter tetap memakai kolom relasional.
- Enum PostgreSQL cocok untuk vocabulary yang stabil. Untuk status yang dikelola bisnis, tabel referensi memberi fleksibilitas migration dan metadata lebih baik.

## Constraint dan transaksi

- Utamakan `NOT NULL`, `CHECK`, `UNIQUE`, primary key, dan foreign key di database.
- Tentukan `ON DELETE` secara eksplisit. Hindari cascade luas pada data yang wajib dipertahankan.
- Gunakan transaction untuk unit kerja bisnis. Jangan menahan transaction saat melakukan network call.
- Pahami isolation level, row lock, deadlock, dan retry terbatas untuk operasi idempotent.
- Gunakan `DEFERRABLE` hanya ketika constraint memang perlu divalidasi di akhir transaction.
- Untuk migration besar, pecah perubahan, backfill bertahap, dan hindari rewrite tabel tak terukur.

## Index dan query

- Gunakan B-tree untuk equality, range, sort umum. Gunakan GIN untuk kebutuhan `jsonb`, array, atau full-text yang sesuai. Gunakan GiST/BRIN hanya berdasarkan karakter data dan query.
- Partial index dipakai bila predicate bisnis stabil dan mengurangi ukuran/index scan.
- Expression index harus memiliki expression yang konsisten dengan query.
- Gunakan `CREATE INDEX CONCURRENTLY` pada perubahan produksi jika sesuai batasan migration, dan pahami bahwa perintah ini tidak berjalan di transaction biasa.
- Verifikasi dengan `EXPLAIN (ANALYZE, BUFFERS)` pada data representatif. Hindari `SELECT *` pada jalur panas.
- Pantau bloat, vacuum, analyze, locks, slow query, connection saturation, dan replication lag.

## RLS dan akses data

- Aktifkan Row Level Security bila akses baris bergantung tenant, user, organisasi, atau ownership.
- Policy harus fail-closed, memiliki predicate `USING` dan `WITH CHECK` yang tepat, serta diuji sebagai role nyata.
- Jangan menganggap RLS aktif berarti aman bila role bypass RLS, function owner, atau service key masih mengekspos data.
- Pisahkan role aplikasi dan admin. Default privilege harus ditetapkan agar object baru tidak terbuka tanpa sengaja.

## Backup dan operasi

- Tetapkan base backup, WAL archiving atau mekanisme point-in-time recovery, retensi, enkripsi, dan restore target.
- Uji restore berkala. Catat RPO, RTO, ukuran database, waktu recovery, dan dependency extension.
- Migration diberi urutan, review, checksum bila tooling mendukung, dan prosedur rollback/forward-fix.
- Jangan menjalankan maintenance berat tanpa menilai lock, I/O, vacuum, dan dampak replica.

## Keamanan

- TLS untuk koneksi lintas host. Batasi `pg_hba.conf`, network exposure, dan port publik.
- Password, connection string, service key, dan backup key berada di secret manager.
- Audit akses administratif dan perubahan privilege. Redaksi PII dari log dan dump non-produksi.

## Checklist PostgreSQL

- [ ] Schema dan role ownership jelas.
- [ ] Identity/UUID, `timestamptz`, `numeric`, JSONB, dan enum dipilih sesuai domain.
- [ ] Constraint, RLS bila relevan, dan policy diuji.
- [ ] Index dibuktikan dengan execution plan.
- [ ] WAL/PITR, restore test, migration, dan observability tersedia.
