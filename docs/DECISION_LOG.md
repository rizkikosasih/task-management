# Architecture Decision Log (Master ADR)

Kompilasi keputusan arsitektur tingkat makro pada Task Management Orchestrator.

---

## ADR-001: Arsitektur Multi-Repository Terisolasi dengan Meta-Workspace

### Konteks
Sistem membutuhkan pemisahan siklus hidup antara backend API (Laravel) dan frontend SPA (React), baik dari segi dependensi, versi pustaka, pipeline CI/CD, maupun penempatan hosting, tanpa mengorbankan pengalaman penelusuran kode dalam satu editor workspace.

### Alternatif
1. Monorepo tunggal dengan package manager monorepo (misal: Turborepo / Nx).
2. Git Submodules.
3. Multi-repository terisolasi yang dihubungkan melalui VS Code Workspace dan diabaikan oleh git root.

### Keputusan
Menerapkan pendekatan multi-repository terisolasi. Repositori root mengelola konfigurasi workspace (`task-management.code-workspace`) serta dokumen spesifikasi makro, sedangkan subdirektori `backend/` dan `frontend/` adalah repositori Git independen yang didaftarkan pada `.gitignore` root.

### Konsekuensi
- Positif: Siklus rilis dan riwayat Git backend dan frontend terpisah total; tidak ada risiko konflik dependensi monorepo.
- Batasan: Perintah Git harus dijalankan secara independen di masing-masing direktori sub-repositori.

### Status Implementasi
Teraplikasi pada basis kode aktif di root workspace.

---

## ADR-002: Standarisasi Protokol Idempotensi Transaksi Mutasi

### Konteks
Koneksi jaringan klien rentan terhadap latensi tinggi atau percobaan pengiriman ulang (*retry*) otomatis yang dapat memicu duplikasi data saat pembuatan entitas proyek atau tugas.

### Alternatif
1. Pengecekan duplikasi manual berbasis nama atau judul pada level aplikasi.
2. Protokol Idempotensi HTTP dengan header `X-Idempotency-Key` dan penyimpanan respon sementara.

### Keputusan
Mengadopsi protokol `X-Idempotency-Key`. Klien frontend (Axios interceptor) menghasilkan UUIDv4 unik untuk setiap request POST mutasi. Middleware backend menyimpan respons transaksi selama masa berlaku kunci dan mengembalikan respons yang sama jika mendeteksi kunci identik.

### Konsekuensi
- Positif: Menjamin resistensi terhadap duplikasi transaksi pada latensi jaringan.
- Batasan: Memerlukan lapisan penyimpanan cache pada backend untuk menampung kunci dan respons sementara.

### Status Implementasi
Teraplikasi pada backend `IdempotencyMiddleware` dan dipersiapkan pada interceptor HTTP frontend.

---

## ADR-003: Model Autentikasi Hibrida Sanctum dan OAuth2 Account Linking

### Konteks
Pengguna memerlukan fleksibilitas masuk ke sistem menggunakan kredensial email-password lokal maupun penyedia identitas pihak ketiga (Google dan GitHub), tanpa menimbulkan duplikasi akun jika alamat email identik.

### Alternatif
1. Autentikasi OAuth eksklusif tanpa akun lokal.
2. Akun lokal dan OAuth terpisah dengan identitas berbeda.
3. Model hibrida dengan penautan akun berbasis kecocokan email terverifikasi.

### Keputusan
Menerapkan model hibrida. Laravel Sanctum mengelola penerbitan Bearer Token untuk seluruh sesi aktif. Laravel Socialite menangani pertukaran kode otorisasi OAuth2. Pengguna dengan email yang cocok ditautkan secara otomatis ke tabel `oauth_providers` yang berelasi dengan tabel `users`. Validasi integritas dan masa aktif token eksternal (Google/GitHub) dipusatkan melalui `OAuthTokenValidatorService` langsung ke endpoint penyedia sebelum akun ditautkan.

### Konsekuensi
- Positif: Integrasi identitas pengguna terpusat dan mulus dengan proteksi integritas kriptografis token eksternal.
- Batasan: Memerlukan toleransi latensi jaringan eksternal (100–300ms) saat verifikasi token ke server penyedia.

### Status Implementasi
Teraplikasi dan tervalidasi pada unit/feature test backend (`OAuthTokenValidationTest`).

---

## ADR-005: Pengendalian Frekuensi API Dinamis Berbasis Token Bucket & Resilient Fallback

### Konteks
Permintaan mutasi data dan endpoint autentikasi rentan terhadap serangan brute force, credential stuffing, dan lonjakan request berlebih yang dapat membebani basis data dan komputasi server.

### Alternatif
1. Rate limiting memori lokal web server (in-memory file limiter).
2. Rate limiting terdistribusi berbasis Redis token bucket dengan mekanisme fallback otomatis ke database cache.

### Keputusan
Mengadopsi named rate limiter bertingkat di Laravel `AppServiceProvider` berbasis Redis (`api-auth` 10/menit, `api-oauth` 15/menit, `api-mutations` 60/menit, `api-reads` 120/menit) dengan resolusi identitas per User ID dan fallback otomatis ke driver database cache apabila server Redis luring. Respon pembatasan distandarisasi ke amplop HTTP 429 berheader `Retry-After`.

### Konsekuensi
- Positif: Mencegah serangan brute force dan isolasi kuota antar pengguna tanpa risiko kegagalan sistem saat Redis tidak aktif.
- Batasan: Memerlukan penanganan resolusi identitas request sebelum eksekusi middleware rate limit.

### Status Implementasi
Teraplikasi pada `AppServiceProvider`, `routes/api.php`, `ApiExceptionTransformer`, dan tervalidasi via `RateLimitingTest`.

---

## ADR-004: Kontrol Konkurensi Transisi Status Kanban Berbasis Pessimistic Locking

### Konteks
Pada lingkungan kolaboratif, dua pengguna dapat memicu pembaruan status tugas secara simultan, memicu kondisi balapan (*race condition*) atau transisi status yang tidak valid (misal: mengembalikan tugas berstatus `done`).

### Alternatif
1. Optimistic locking berbasis kolom versi / timestamp.
2. Pessimistic locking menggunakan kueri SQL `SELECT ... FOR UPDATE`.

### Keputusan
Menerapkan penguncian pesimistis (`lockForUpdate`) pada transaksi perubahan status tugas di backend. Klien frontend menangani respon galat status HTTP 409 (*Conflict*) atau 422 (*Unprocessable Entity*) dengan melakukan pembatalan mutasi visual (*rollback*) pada papan antarmuka.

### Konsekuensi
- Positif: Integritas transisi status terjamin di level database engine.
- Batasan: Menahan baris tabel selama transaksi berlangsung, memerlukan transaksi yang singkat dan cepat.

### Status Implementasi
Teraplikasi pada `TaskController` dan unit test backend.

---

## ADR-006: Standarisasi Efisiensi Pipeline CI Bebas Layanan Kontainer dan Retensi Artefak

### Konteks
Pipeline CI backend sebelumnya memuat service container MySQL 8.0 dan menghasilkan arsip zip artefak build pada setiap Pull Request. Hal ini memboroskan kuota menit komputasi GitHub Actions hingga 2–3 menit per eksekusi dan menguras batas kapasitas penyimpanan repositori cloud, padahal suite pengujian unit dan integrasi backend telah terisolasi penuh menggunakan SQLite in-memory.

### Alternatif
1. Mempertahankan service MySQL dan memperpendek interval health check.
2. Mengeliminasi service container basis data eksternal, meniadakan pembuatan artefak kompilasi untuk pipeline murni evaluasi kode, serta menerapkan mekanisme pembatalan proses komit terdahulu (*cancel-in-progress*) dan cache dependensi Composer.

### Keputusan
Mengadopsi alternatif kedua. Seluruh pengujian otomatis dijalankan murni di atas SQLite in-memory (`:memory:`), menghapus dependensi container MySQL pada alur kerja, menambahkan `actions/cache@v4` untuk dependensi Composer, membatalkan alur kerja yang tertimpa komit baru secara otomatis, serta menghapus tahapan pembuatan dan unggah artefak zip.

### Konsekuensi
- Positif: Memangkas durasi komputasi alur kerja dari ~2-3 menit menjadi ~30-45 detik per eksekusi, mencegah kehabisan kuota menit GitHub Actions, dan mengeliminasi pemborosan ruang penyimpanan repositori.
- Batasan: Perubahan skema basis data spesifik sintaksis MySQL yang tidak didukung SQLite memerlukan pengujian manual terpisah di lingkungan lokal jika ada.

### Status Implementasi
Teraplikasi pada `.github/workflows/laravel-ci.yml` di sub-repositori backend.


