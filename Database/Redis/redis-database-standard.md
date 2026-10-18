# Standar Database Redis

## Peran Redis

Tentukan apakah Redis dipakai sebagai cache, session, rate limiter, queue, lock, pub/sub, stream, atau source of truth. Default: Redis adalah data turunan yang boleh dibangun ulang, kecuali durability dan recovery sudah disetujui.

## Data structure

- `String`: counter, token sementara, nilai kecil, atau serialized payload dengan batas ukuran.
- `Hash`: object kecil yang sebagian field-nya sering diakses.
- `Set`: membership dan deduplikasi tanpa urutan.
- `Sorted Set`: ranking, jadwal, dan score range.
- `List`/`Stream`: antrean sederhana atau event stream; pilih stream bila butuh consumer group dan replay terukur.
- Jangan menyimpan object besar tanpa batas. Tetapkan ukuran payload dan cardinality.

## Key naming dan TTL

Gunakan pola konsisten, misalnya `{app}:{environment}:{domain}:{id}:{purpose}`. Hindari key yang berasal langsung dari input tanpa normalisasi dan batas panjang.

TTL wajib untuk cache, session sementara, token reset, lock, rate limit, dan data ephemeral. TTL dipilih dari kebutuhan bisnis, bukan angka acak. Tetapkan perilaku cache miss, stale data, dan invalidation. Jangan memakai Redis key tanpa TTL untuk data yang seharusnya sementara.

## Atomicity dan concurrency

- Gunakan command atomic, `INCR`, `SET ... NX EX`, Lua script, atau transaction sesuai kebutuhan.
- Lock memiliki owner token, TTL, dan release yang hanya menghapus lock milik owner. Jangan mengandalkan `DEL` tanpa verifikasi.
- Operasi read-modify-write membutuhkan atomicity eksplisit.
- Idempotency key mencegah retry menggandakan efek.
- Pub/sub tidak menjamin replay. Gunakan Streams atau broker durable untuk event penting.

## Persistence, HA, dan recovery

- Pilih RDB, AOF, atau kombinasi berdasarkan RPO/RTO dan beban tulis.
- Uji restore dari snapshot/AOF, bukan hanya memeriksa file backup.
- Sentinel/Cluster dipilih berdasarkan kebutuhan failover dan sharding. Pahami bahwa HA bukan backup.
- Rencanakan resharding, replica lag, failover, dan client retry dengan backoff.
- Cache stampede ditangani dengan jitter TTL, single-flight/lock terbatas, atau refresh async.

## Memory dan eviction

- Tetapkan `maxmemory`, policy eviction, batas payload, dan alarm memory.
- Pilih eviction policy sesuai workload. Jangan menganggap eviction aman untuk session atau data source of truth.
- Pantau fragmentation, hit/miss ratio, big keys, expired keys, blocked clients, latency, connected clients, dan replication.
- Gunakan `SCAN`, bukan `KEYS`, pada instance produksi.

## Keamanan

- Jangan expose Redis ke internet. Batasi network, TLS, ACL, dan command privilege.
- Secret koneksi di secret manager. Rotasi credential dan audit akses.
- Jangan menyimpan password plaintext, token jangka panjang, atau PII tanpa alasan dan kontrol retensi.

## Checklist Redis

- [ ] Peran Redis dan status source of truth jelas.
- [ ] Data structure, key format, ukuran payload, dan TTL ditetapkan.
- [ ] Atomicity, lock, retry, dan idempotency diuji.
- [ ] Persistence, restore test, HA, dan failover dirancang.
- [ ] Memory limit, eviction, metrics, ACL, TLS, dan network boundary aman.
