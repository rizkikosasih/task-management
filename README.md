# Task Management Workspace

Sistem manajemen proyek dan koordinasi tugas kolaboratif berbasis arsitektur client-server terpisah. Repositori ini berfungsi sebagai workspace root orchestrator yang mengoordinasikan implementasi backend RESTful API dan frontend Single Page Application (SPA).

---

## Repositori Terkait

- **Backend API:** [backend-task-management](https://github.com/rizkikosasih/backend-task-management)
- **Frontend SPA:** [frontend-task-management](https://github.com/rizkikosasih/frontend-task-management)

---

## Arsitektur Sistem

Sistem menerapkan pemisahan tanggung jawab yang tegas antara penyedia layanan data dan antarmuka pengguna:

- **Frontend SPA (Port 5101):** Dibangun menggunakan React 19 dan Vite, menyediakan manajemen navigasi berbasis tipe dengan TanStack Router, sinkronisasi state server melalui TanStack Query v5, manajemen sesi otentikasi via Zustand, dan visualisasi kanban drag-and-drop melalui dnd-kit. Seluruh komunikasi HTTP diarahkan melalui Axios dengan interceptor otomatis untuk Bearer Token dan idempotensi transaksi.
- **Backend REST API (Port 8101):** Dibangun menggunakan Laravel 12, menyediakan otentikasi hibrida berbasis Laravel Sanctum dan OAuth2 (Google, GitHub via Laravel Socialite), otorisasi dua lapis (Role-Based Access Control via Spatie Permission dan Attribute-Based Access Control via Laravel Policy), mitigasi duplikasi transaksi melalui Idempotency Middleware (`X-Idempotency-Key`), serta pencegahan race condition melalui penguncian pesimistis database (`lockForUpdate`).
- **Basis Data:** Mendukung SQLite untuk pengujian dan pengembangan lokal, serta MySQL untuk lingkungan produksi.

---

## Matriks Kontrol Akses (RBAC & ABAC)

| Aksi Sistem | Admin (Pemilik) | Admin (Lain) | Member (Assignee) | Member (Lain) |
| :--- | :---: | :---: | :---: | :---: |
| Pembuatan Proyek | Diizinkan | Diizinkan | Ditolak | Ditolak |
| Modifikasi / Penghapusan Proyek | Diizinkan | Ditolak | Ditolak | Ditolak |
| Pembuatan Tugas | Diizinkan | Ditolak | Ditolak | Ditolak |
| Modifikasi Rincian Tugas | Diizinkan | Ditolak | Ditolak | Ditolak |
| Transisi Status Tugas | Diizinkan | Ditolak | Diizinkan | Ditolak |
| Penghapusan Tugas | Diizinkan | Ditolak | Ditolak | Ditolak |
| Pembuatan Komentar | Diizinkan | Diizinkan | Diizinkan | Diizinkan* |
| Penghapusan Komentar | Pemilik Komentar | Pemilik Komentar | Pemilik Komentar | Pemilik Komentar |

Keterangan: Member lain hanya dapat melihat atau menambahkan komentar apabila terdaftar memiliki penugasan pada proyek atau tugas terkait.

---

## Panduan Menjalankan Lingkungan Lokal

### 1. Menjalankan Backend Service

```bash
cd backend
composer install
cp .env.example .env
php artisan key:generate
php artisan migrate --seed
php artisan serve --port=8101
```

Backend dapat diakses pada alamat `http://localhost:8101`.

### 2. Menjalankan Frontend Application

```bash
cd frontend
npm install
cp .env.example .env.local
npm run dev
```

Frontend berjalan pada alamat `http://localhost:5101` dan berkomunikasi dengan backend pada `http://localhost:8101/api`.

---

## Prasyarat Lingkungan Pengembangan

| Komponen | Versi Acuan |
| :--- | :--- |
| PHP | 8.3 ke atas |
| Composer | 2.x |
| Node.js | 20 LTS atau 22 LTS |
| npm | 10.x ke atas |
| Basis Data | SQLite 3 / MySQL 8.0 |

---

## Lisensi

Repositori ini dikelola untuk portofolio teknis rekayasa perangkat lunak.
