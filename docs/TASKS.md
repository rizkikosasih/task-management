# Project Task Backlog (TASKS) - Workspace Orchestrator

Hierarki Standar Pelacakan Tugas Proyek: Priority -> Sub-Poin -> Item Pekerjaan.

---

## Priority 1

### 1. Sinkronisasi Kontrak Kritis & Penanganan Defek Antar-Layanan

- [x] **Penyelarasan Header Idempotensi Backend & Frontend:**
  Menyesuaikan IdempotencyMiddleware pada backend agar membaca header X-Idempotency-Key dan Idempotency-Key secara fleksibel guna memastikan mutasi pembuatan proyek dan tugas terproteksi dari duplikasi jaringan.
- [x] **Perbaikan Alur Redirect & Callback OAuth2 End-to-End:**
  Menyelaraskan properti URL redirect pada SocialAuthController dan LoginForm, menerapkan HTTP redirect 302 dari callback backend ke rute klien /oauth/callback?token=..., serta menangani kasus email nihil dari penyedia eksternal pada AuthService.
- [x] **Penegakan Isolasi Data Hak Akses Daftar Proyek:**
  Menambahkan filter kueri pada ProjectService::getProjects agar pengguna peran member hanya menerima proyek tempat mereka memiliki penugasan tugas aktif, mengeliminasi kebocoran metadata proyek.
- [x] **Penyelarasan Standar Amplop Respon pada PRD:**
  Menyelaraskan dokumentasi PRD.md dengan implementasi riil kode ApiResponse dan tipe TypeScript yang menggunakan properti boolean success.

### 2. Keamanan, Validasi Token & Pengendalian Akses Jaringan Backend

- [ ] **Rate Limiting Dinamis Berbasis Redis per Identitas Pengguna:**
  Menerapkan pembatasan frekuensi mutasi data API memanfaatkan algoritma token bucket Redis berbasis User ID terotentikasi, dengan mekanisme fallback ke driver database cache.
- [ ] **Validasi Integritas Token OAuth Terpusat:**
  Menambahkan verifikasi payload dan status aktif token otentikasi dari penyedia eksternal (Google/GitHub) sebelum finalisasi penautan akun lokal.

### 3. Notifikasi Domain & Infrastruktur Asinkron Backend

- [ ] **Worker Antrean Multi-Kanal Berkinerja Tinggi:**
  Mengarahkan listener SendTaskAssignedNotification ke antrean worker asinkron berbasis Redis dengan integrasi kelas Mailable dan logging kegagalan pengiriman.

---

## Priority 2

### 1. Perluasan Domain Wewenang & Pengelolaan Pengguna

- [ ] **Modul Pengelolaan Pengguna Admin:**
  Membangun endpoint backend dan antarmuka frontend /admin/users untuk melihat daftar pengguna, memodifikasi peran, dan deaktivasi akun.
- [ ] **Skema Keanggotaan Proyek (Project Membership):**
  Merancang migrasi tabel pivot project_user untuk mengikat kolaborator per proyek dan mentransformasikan wewenang RBAC global menjadi wewenang kontekstual per proyek.
- [ ] **Delegasi Penugasan Multipel pada Tugas:**
  Mentransformasikan relasi penugasan tugas menjadi tabel relasional task_assignees guna mendukung kolaborasi tim pada satu tugas.

### 2. Audit Trail & Log Transaksi

- [ ] **Skema Riwayat Perubahan Entitas Asinkron:**
  Membangun tabel log audit terisolasi yang dipicu melalui domain event asinkron guna merekam riwayat perubahan tugas tanpa menambah durasi penguncian baris (lockForUpdate).

### 3. Pelaporan & Transformasi Data

- [ ] **Generator Ekspor CSV Streaming:**
  Menyediakan endpoint streaming pembuatan berkas CSV ringkasan metrik proyek untuk mencegah lonjakan alokasi memori saat memproses dataset besar.

### 4. Standarisasi Dokumentasi Workspace & Tautan Publik

- [ ] **Sinkronisasi Spesifikasi Root README.md:**
  Menyelaraskan versi framework menjadi Laravel 12, mencantumkan SQLite lokal & MySQL produksi, melengkapi ekosistem frontend (TanStack Router, Zustand, dnd-kit, Tailwind v4), serta memasang tautan repositori remote resmi GitHub.
- [ ] **Konsolidasi DEV_LOG.md Tingkat Workspace:**
  Menyelaraskan catatan aktivitas pengembangan frontend terkini (tata letak responsif seluler dan pengujian Vitest/MSW) ke dalam berkas DEV_LOG.md root.

---

## Priority 3

### 1. Manajemen Logging & Observabilitas

- [ ] **Middleware Correlation ID & Monolog JSON Formatter:**
  Menginjeksi header X-Request-ID ke konteks log terstruktur JSON Monolog dan seluruh amplop respon HTTP untuk penelusuran transaksi terdistribusi.
- [ ] **Filter Sanitisasi Log Kontekstual:**
  Mengonfigurasi penanganan eksepsi global agar menyaring kredensial sensitif dari payload request sebelum dicatat pada berkas rotasi harian.

### 2. Strategi Caching & Concurrency Control

- [ ] **Tagged Cache Proyek & Tugas dengan Fallback Guard:**
  Mengimplementasikan mekanisme cache berlabel pada kueri baca proyek dan tugas yang terinvaliasi otomatis saat terjadi mutasi data, disertai penanganan isolasi driver non-Redis.
- [ ] **Distributed Lock Metrik Tugas:**
  Menerapkan Redis atomic locks dengan batasan TTL ketat pada kalkulasi agregasi metrik tugas guna mengeliminasi kondisi cache stampede.

### 3. Skema Basis Data & Optimasi Kueri

- [ ] **Indeks B-Tree Komposit Filter Tugas:**
  Menambahkan migrasi indeks komposit pada pasangan kolom (project_id, status) dan (assigned_to, status) untuk mengeliminasi full table scan pada kueri filter task list.
- [ ] **Penegakan Integritas Tipe Kolom Status:**
  Mengubah tipe kolom status dari VARCHAR(255) bebas menjadi tipe data terbatas yang tervalidasi serta memperketat batasan Foreign Key onDelete restrict/cascade.

### 4. Kesiapan Produksi, Standarisasi Respon & Pengetesan Komprehensif

- [ ] **Endpoint Health Check Probe Terpadu:**
  Menyediakan rute pemantauan /api/health/live dan /api/health/ready yang memverifikasi kesiapan koneksi basis data utama dan kluster Redis.
- [ ] **Header Metadata Paginasi:**
  Menyematkan informasi paginasi terpadu pada header HTTP respon (X-Total-Count, X-Page, X-Per-Page) untuk melengkapi amplop respon JSON.
- [ ] **Pengujian Alur Otentikasi MSW Frontend:**
  Menyusun skenario pengetesan form login dan register terintegrasi dengan Mock Service Worker (MSW).
- [ ] **Audit Aksesibilitas Interaksi Keyboard Langsung:**
  Melakukan inspeksi langsung alur interaksi keyboard pada drag-and-drop kanban (dnd-kit keyboard sensors), navigasi fokus modal, dan eliminasi gaya teks kaku buatan AI pada label antarmuka.
- [ ] **Pipeline CI GitHub Actions Frontend:**
  Membangun workflow otomatisasi GitHub Actions (.github/workflows/frontend-ci.yml) yang mengeksekusi pemeriksaan tipe TypeScript, linter Oxlint, format Prettier, suite pengujian Vitest, dan build bundler Vite pada setiap push dan pull request ke branch main.
- [ ] **Penyempurnaan Skema OpenAPI & Ekspor Swagger UI:**
  Mengonfigurasi skema keamanan Bearer Auth Sanctum dan amplop respon ApiResponse pada Scramble, menambahkan komponen header X-Idempotency-Key pada dokumentasi rute, serta menyediakan generator skema statis docs/api/openapi.json untuk integrasi Postman dan Swagger UI.
