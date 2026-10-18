

## 1. Purpose

Dokumen ini menetapkan standar wajib penamaan database, struktur tabel, kolom, primary key, foreign key, pivot table, index, constraint, relationship, timestamp, flag, dan default value untuk LMS.

Tujuannya:

- membuat struktur database konsisten;
- mudah dipahami developer, DB engineer, analyst, dan AI assistant;
- mudah dipelihara jangka panjang;
- mudah dipetakan ke ORM seperti Laravel Eloquent;
- mudah didokumentasikan di ERD, migration, API contract, dan technical spec;
- mengurangi kesalahan akibat naming yang tidak seragam.

Standar ini menjadi default kecuali ada pengecualian proyek yang disetujui.

---

## 2. Scope

### 2.1 Roles Covered

Standar ini berlaku untuk:

- Backend Developers Laravel.
- Frontend/Full-stack Developers yang berinteraksi dengan contract data.
- Database Engineers.
- System Analysts.
- Software Architects.
- Technical Leads.
- Reviewers migrasi/schema.
- AI coding assistants.

### 2.2 Systems Covered

Standar ini dipakai untuk:

- sistem baru;
- modul internal LMS;
- refactor schema;
- relasi master/transaction/reference/log;
- database reporting dan analytics;
- fitur auth, course, enrollment, assessment, certificate, notification, audit.

Untuk legacy schema, jangan rename buta jika memicu breaking change. Review dependency dulu.

---

## 3. Definitions

| Term | Definition |
| --- | --- |
| Database | Basis data logis untuk aplikasi atau domain. |
| Schema | Namespace logis di dalam database (di MySQL, `SCHEMA` identik dengan `DATABASE`). |
| Table | Struktur data untuk menyimpan record. |
| Column | Atribut data di dalam table. |
| Primary Key | Identifier unik utama record. |
| Foreign Key | Kolom relasi ke table lain. |
| Constraint | Aturan integritas data. |
| Index | Struktur untuk mempercepat query. |
| Pivot Table | Table penghubung many-to-many. |
| Master Data | Data inti yang stabil dan sering direferensikan. |
| Transaction Data | Data event/proses bisnis yang terus bertambah. |
| Reference Data | Data lookup/enumerasi yang jarang berubah. |
| Log Data | Audit, activity, diagnostic, event, integration log. |
| Prefix | Awalan klasifikasi table seperti `mt_` atau `tr_`. |
| snake_case | Huruf kecil dan underscore. |
| Soft Delete | Penghapusan logis memakai `deleted_at`. |
| Flag | Kolom boolean seperti `is_active`. |

---

## 4. General Naming Principles

### 4.1 Lowercase

Semua nama object database memakai lowercase.

Benar:

```text
customers
tr_course_enrollments
created_at
```

Salah:

```text
Customers
TrCourseEnrollments
CreatedAt
```

### 4.2 snake_case

Gunakan `snake_case` untuk semua nama multiword.

Benar:

```text
course_enrollments
quiz_attempts
certificate_templates
```

Salah:

```text
courseEnrollments
CourseEnrollments
course-enrollments
```

### 4.3 Descriptive Name

Nama harus jelas secara bisnis.

Benar:

```text
payment_status
total_amount
lesson_progress
```

Salah:

```text
value
info
data1
flag1
```

### 4.4 Avoid Unclear Abbreviations

Hindari singkatan yang tidak umum.

Benar:

```text
phone_number
quantity
description
```

Salah:

```text
phone_no
qty
desc
```

### 4.5 Avoid Reserved Keywords

Hindari keyword yang rawan konflik.

Salah:

```text
order
group
user
select
```

Pilih yang lebih jelas seperti:

```text
orders
user_accounts
reference_values
```

---

## 5. Database Naming Standards

Database name harus:

- lowercase;
- snake_case;
- mencerminkan domain atau service;
- tanpa spasi dan simbol khusus.

Contoh:

```text
lms_service
lms_system
lms_platform
```

Format rekomendasi:

```text
{domain}_{service_or_system}
```

Contoh:

```text
hr_lms_service
learning_management_system
lms_service
```

---

## 6. Table Naming Standards

### 6.1 Basic Rule

Table harus:

- plural;
- lowercase;
- snake_case;
- berupa noun;
- bukan verb.

Benar:

```text
users
courses
course_enrollments
quiz_attempts
```

Salah:

```text
user
courseEnrollment
ManageCourse
```

### 6.2 Table Prefix Standard

LMS mengikuti prefix classification berikut:

| Prefix | Purpose                |
| ------ | ---------------------- |
| `mt_`  | Master data            |
| `tr_`  | Transaction data       |
| `dt_`  | Detail data            |
| `rf_`  | Reference data         |
| `lg_`  | Log data               |
| `cfg_` | Config data            |
| `tmp_` | Temporary/staging data |

### 6.3 LMS Table Classification

Rekomendasi klasifikasi table LMS:

- `mt_users`
- `mt_courses`
- `mt_programs`
- `mt_batches`
- `mt_quizzes`
- `mt_assignments`
- `mt_certificate_templates`
- `rf_roles`
- `rf_course_categories`
- `rf_statuses`
- `tr_course_enrollments`
- `tr_program_enrollments`
- `tr_quiz_attempts`
- `tr_assignment_submissions`
- `tr_certificates`
- `tr_batch_sessions`
- `tr_batch_announcements`
- `dt_chapters`
- `dt_lessons`
- `dt_course_instructors`
- `dt_program_courses`
- `dt_quiz_questions`
- `dt_quiz_options`
- `dt_quiz_answers`
- `dt_submission_files`
- `dt_assessment_links`
- `lg_webhook_events`
- `lg_learning_activities`
- `lg_audit_trails`
- `cfg_settings`
- `cfg_notification_preferences`
- `cfg_certificate_templates`

Catatan:

- `mt_` dipakai untuk entity inti yang stabil.
- `tr_` dipakai untuk data historis atau event bisnis.
- `dt_` dipakai untuk child/detail dari master atau transaction.
- `rf_` dipakai untuk data lookup/enumerasi.
- `lg_` dipakai untuk log dan audit.
- `cfg_` dipakai untuk konfigurasi sistem.
- `tmp_` dipakai untuk staging/import sementara.

### 6.4 Consistency Rule

Jika proyek sudah memakai prefix classification, maka semua table baru harus konsisten.

Tidak boleh campur:

```text
mt_users
courses
tr_quiz_attempts
settings
```

Harus konsisten seperti:

```text
mt_users
mt_courses
tr_quiz_attempts
cfg_settings
```

---

## 7. Column Naming Standards

### 7.1 General Rule

- lowercase
- snake_case
- jelas
- tidak ambigu
- satu kolom satu makna

Benar:

```text
full_name
phone_number
payment_status
total_amount
```

Salah:

```text
fullname
phoneNo
status_flag
```

### 7.2 Common Column Standards

| Purpose | Column Name |
| --- | --- |
| Primary key | `id` |
| Foreign key | `{singular_entity}_id` |
| Created timestamp | `created_at` |
| Updated timestamp | `updated_at` |
| Deleted timestamp | `deleted_at` |
| Boolean flag | `is_active`, `is_featured`, `has_passed` |

### 7.3 Primary Key

Default primary key:

```text
id
```

Gunakan UUID/ULID hanya jika dibutuhkan oleh desain sistem, misalnya:

- public identifier;
- security;
- distributed system;
- data merge antar sistem.

### 7.4 Foreign Key

Format:

```text
{singular_entity}_id
```

Contoh:

```text
user_id
course_id
lesson_id
batch_id
```

Contoh LMS:

- `tr_course_enrollments.user_id -> mt_users.id`
- `tr_course_enrollments.course_id -> mt_courses.id`
- `dt_lessons.chapter_id -> dt_chapters.id`
- `dt_program_courses.program_id -> mt_programs.id`

### 7.5 Boolean / Flag

Boolean harus terbaca sebagai kondisi.

Benar:

```text
is_active
is_verified
is_featured
is_self_enrollment_enabled
is_preview
has_passed
can_cancel
```

Salah:

```text
active
flag
status_flag
```

### 7.6 Timestamp Domain

Gunakan suffix `_at` untuk waktu kejadian.

Contoh:

```text
published_at
enrolled_at
completed_at
started_at
submitted_at
graded_at
issued_at
expires_at
```

Untuk date-only gunakan `_date`.

Contoh:

```text
start_date
end_date
due_date
```

---

## 8. Database Structure Standards

### 8.1 Column Order

Urutan disarankan:

1. `id`
2. foreign keys
3. core attributes
4. optional metadata
5. flags/status
6. `created_at`
7. `updated_at`
8. `deleted_at`

Contoh:

```text
id
course_id
user_id
order_number
total_amount
status
is_active
created_at
updated_at
deleted_at
```

### 8.2 Data Type Standards

Gunakan tipe data sesuai makna bisnis.

Contoh:

- `string/varchar` untuk text pendek.
- `text` untuk konten panjang.
- `integer/bigint` untuk ID atau counter.
- `decimal` untuk nominal uang.
- `boolean` untuk flag.
- `date` untuk tanggal.
- `timestamp/datetime` untuk waktu.
- `json` untuk metadata fleksibel.

Jangan pakai `varchar(255)` untuk semua kolom tanpa alasan.

### 8.3 Numeric Types

- integer untuk quantity, sort order, attempt count.
- decimal untuk `price`, `amount`, `balance`, `tax`.
- Jangan pakai floating point untuk nilai finansial.

### 8.4 Default Values

Default harus bermakna.

Contoh:

```text
is_active -> true
quantity -> 0
total_amount -> 0
```

Jangan pakai default hanya untuk menghindari null kalau null punya makna bisnis.

### 8.5 Nullability

Gunakan `NOT NULL` untuk data wajib.

Gunakan `NULL` jika:

- data opsional;
- data belum tersedia;
- relasi boleh kosong;
- null punya makna bisnis.

---

## 9. Relationship and Constraint Standards

### 9.1 Foreign Key Constraint

Format:

```text
fk_{child_table}_{parent_table}
```

Contoh:

```text
fk_tr_course_enrollments_mt_users
fk_tr_course_enrollments_mt_courses
```

### 9.2 Unique Constraint

Format:

```text
uq_{table}_{column}
```

Contoh:

```text
uq_mt_users_nip
uq_mt_users_email
uq_tr_certificates_certificate_no
```

### 9.3 Index Naming

Format:

```text
idx_{table}_{column}
```

Contoh:

```text
idx_tr_course_enrollments_user_id_status
idx_mt_courses_status_published_at
```

### 9.4 Cascade Rules

- `restrictOnDelete()` untuk data penting dan historis.
- `cascadeOnDelete()` hanya untuk child yang tidak bermakna tanpa parent.
- Audit, log, sertifikat, transaksi, dan data historis jangan casual cascade.

### 9.5 Master-Detail Rules

- gunakan `RESTRICT` bila child mencegah parent dihapus;
- gunakan `SET NULL` bila child tetap valid tanpa parent;
- gunakan `CASCADE` hanya jika memang aman.

---

## 10. MySQL Specific Configurations

### 10.1 Engine, Character Set, and Collation

- Engine default: `InnoDB` (dukung ACID, foreign key, row-level locking).
- Charset default: `utf8mb4` (dukung 4-byte unicode / emoji).
- Collation: `utf8mb4_unicode_ci` atau `utf8mb4_0900_ai_ci` (MySQL 8.0+).

```sql
CREATE DATABASE lms_service
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_0900_ai_ci;
```

### 10.2 Timezone Alignment

MySQL disetel ke UTC.

- `my.cnf`: `default-time-zone = '+00:00'`
- Kolom timestamp: gunakan `DATETIME` untuk waktu independen session atau `TIMESTAMP` (auto UTC convert, rentang s/d 2038).
- Laravel: `config/database.php` set `'timezone' => '+00:00'`.

### 10.3 UUID

- Internal relation: tetap `BIGINT UNSIGNED AUTO_INCREMENT` / `$table->id()`.
- Public identifier: gunakan UUID string (`char(36)`) atau binary (`BINARY(16)` via `orderedUuid()` di Laravel untuk performa B-Tree index).

### 10.4 JSON Column

Gunakan native type `JSON` (tersedia sejak MySQL 5.7+):

- metadata sertifikat;
- snapshot result quiz;
- dynamic form submission / TipTap content;
- payload event webhook / log.

Untuk query performa tinggi pada field JSON, gunakan **Generated Column + B-Tree Index**:

```sql
ALTER TABLE tr_certificates
  ADD COLUMN cert_hash VARCHAR(64) AS (json_unquote(json_extract(metadata_json, '$.hash'))) STORED,
  ADD INDEX idx_tr_certificates_cert_hash (cert_hash);
```

### 10.5 Indexing

#### B-Tree (Default InnoDB)
Gunakan untuk:
- foreign key;
- unique lookup;
- sorting (`ORDER BY`);
- equality & range query (`=`, `<`, `>`, `BETWEEN`, `LIKE 'prefix%'`).

#### Full-Text (FULLTEXT)
Gunakan untuk:
- pencarian teks course / lesson content (`MATCH(...) AGAINST(...)`).
- InnoDB FULLTEXT didukung native di MySQL 5.6+.

#### Composite Index
Gunakan untuk kombinasi kolom query rutin:
- `(user_id, status)` pada enrollment;
- `(course_id, status)` pada course listing.

Ikuti aturan **Leftmost Prefix**: tempatkan kolom equality check di urutan depan, range/sort di belakang.

---

## 11. LMS Migration Standards

### 11.1 Example Migration Rule

```php
Schema::create('tr_course_enrollments', function (Blueprint $table) {
    $table->id();
    $table->uuid('uuid')->unique();

    $table->foreignId('user_id')
        ->constrained('mt_users')
        ->cascadeOnUpdate()
        ->restrictOnDelete();

    $table->foreignId('course_id')
        ->constrained('mt_courses')
        ->cascadeOnUpdate()
        ->restrictOnDelete();

    $table->string('status', 30)->default('active');
    $table->timestamp('completed_at')->nullable();
    $table->timestamps();

    $table->index(['user_id', 'status']);
});
```

### 11.2 Migration Rules

- reversible jika memungkinkan;
- aman untuk data existing;
- punya index yang diperlukan;
- foreign key dibuat sadar;
- jangan campur terlalu banyak concern dalam satu migration;
- pertimbangkan lock untuk tabel besar.

---

## 12. LMS Table Examples

### 12.1 Users

```text
mt_users
id
nip
name
email
is_active
created_at
updated_at
deleted_at
```

### 12.2 Courses

```text
mt_courses
id
slug
title
short_intro
description
category_id
image_path
preview_video_url
status
is_featured
is_self_enrollment_enabled
published_at
created_at
updated_at
deleted_at
```

### 12.3 Chapters

```text
dt_chapters
id
course_id
title
position
created_at
updated_at
```

### 12.4 Lessons

```text
dt_lessons
id
chapter_id
title
slug
content_type
content
video_url
attachment_path
position
is_preview
created_at
updated_at
```

### 12.5 Enrollments

```text
tr_course_enrollments
id
course_id
user_id
status
enrolled_at
completed_at
progress_percent
created_at
updated_at
```

### 12.6 Certificates

```text
tr_certificates
id
uuid
user_id
certifiable_type
certifiable_id
template_id
certificate_no
issued_at
expires_at
metadata_json
pdf_path
created_at
updated_at
```

---

## 13. AI Validation Checklist

Sebelum schema final, pastikan:

- database lowercase dan snake_case;
- table plural dan konsisten prefix;
- column snake_case;
- primary key `id`;
- foreign key `{singular}_id`;
- timestamp konsisten `_at`;
- boolean pakai awalan kondisi;
- pivot table singular dan alfabetis;
- constraint `fk_`, `uq_`, `idx_` konsisten;
- type data sesuai domain;
- default value semantik;
- cascade aman;
- log/audit tidak mudah hilang;
- index sesuai query nyata;
- semua object mudah dipahami tanpa baca code.

---

## 14. Conflict and Legacy Rules

- Ikuti requirement proyek yang eksplisit dulu.
- Jangan rename legacy schema tanpa review dampak.
- Konsistensi lebih penting dari preferensi personal.
- Kalau ada exception, harus jelas, disetujui, dan terdokumentasi.

---

## 15. Summary for LMS

Rekomendasi final LMS:

```text
Database name: lms_service

Master: mt_users, mt_courses, mt_programs, mt_batches, mt_quizzes, mt_assignments
Transaction: tr_course_enrollments, tr_program_enrollments, tr_quiz_attempts, tr_assignment_submissions, tr_certificates
Detail: dt_chapters, dt_lessons, dt_program_courses, dt_quiz_questions, dt_quiz_options, dt_quiz_answers
Reference: rf_roles, rf_course_categories, rf_statuses
Log: lg_webhook_events, lg_learning_activities, lg_audit_trails
Config: cfg_settings, cfg_notification_preferences
Temporary: tmp_import_users
```

---

skipped: physical schema partitioning for audit tables, add ketika volume log melewati batas operasional nyata.