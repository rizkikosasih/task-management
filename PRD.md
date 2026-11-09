# Product Requirement Document (PRD) - Task Management Orchestrator

## 1. Ringkasan Eksekutif

Task Management System adalah platform manajemen proyek kolaboratif berbasis arsitektur terpisah antara antarmuka Single Page Application (SPA) dan RESTful API. Sistem mengintegrasikan kontrol akses berbasis peran (RBAC) dan atribut kepemilikan (ABAC), kontrol konkurensi berbasis penguncian pesimistis, jaminan idempotensi transaksi, dan komunikasi tim terpadu.

Repositori tingkat root berfungsi sebagai meta-workspace orchestrator yang mengelola integrasi lintas domain antara repositori backend Laravel dan frontend React.

---

## 2. Sasaran Sistem & Kriteria Penerimaan

- **Integrasi Terpadu:** Antarmuka SPA mengonsumsi backend REST API dengan kontrak respons seragam (ApiResponse envelope).
- **Keandalan Transaksi:** Setiap mutasi pembuatan data (proyek, tugas) memancarkan header X-Idempotency-Key (UUIDv4) untuk mencegah duplikasi data akibat transmisi jaringan ganda.
- **Konsistensi Konkurensi:** Perubahan status pada papan kanban divalidasi dengan penguncian pesimistis pada backend, dengan penanganan galat konkurensi HTTP 409 pada klien.
- **Otentikasi Multi-Metode:** Mendukung sesi Bearer Token via Laravel Sanctum dan login pihak ketiga (Google & GitHub) dengan penautan akun berbasis alamat email.

---

## 3. Matriks Hak Akses & Peran

| Hak Akses | Admin (Pemilik) | Admin (Lain) | Member (Assignee) | Member (Lain) |
| :--- | :---: | :---: | :---: | :---: |
| Pembuatan Proyek | Izinkan | Izinkan | Tolak | Tolak |
| Modifikasi / Hapus Proyek | Izinkan | Tolak | Tolak | Tolak |
| Pembuatan Tugas | Izinkan | Tolak | Tolak | Tolak |
| Modifikasi Detail Tugas | Izinkan | Tolak | Tolak | Tolak |
| Transisi Status Tugas | Izinkan | Tolak | Izinkan | Tolak |
| Penghapusan Tugas | Izinkan | Tolak | Tolak | Tolak |
| Diskusi Komentar | Izinkan | Izinkan | Izinkan | Izinkan* |
| Hapus Komentar | Hak Milik | Hak Milik | Hak Milik | Hak Milik |

Catatan: Member lain hanya dapat mengakses komentar jika terdaftar memiliki akses pada tugas terkait.

---

## 4. Arsitektur Komponen Orchestrator

```text
task-management/ (Orchestrator)
├── backend/ (Laravel 12 REST API)
│   ├── Sanctum Authentication
│   ├── Spatie RBAC & Policy ABAC
│   └── Idempotency & Pessimistic Locking
└── frontend/ (Vite + React 19 SPA)
    ├── TanStack Router (Type-Safe Navigation)
    ├── TanStack Query v5 (Server State & Invalidation)
    ├── Zustand (Client Auth Session)
    ├── dnd-kit (Kanban Board Drag-and-Drop)
    └── Axios Client (Token & Idempotency Interceptors)
```

---

## 5. Standar Komunikasi & Respon API

Seluruh respon backend dibungkus dalam struktur amplop baku:

```json
{
  "success": true,
  "message": "Deskripsi respon",
  "data": {},
  "meta": {}
}
```

Format respon galat:

```json
{
  "success": false,
  "message": "Deskripsi galat",
  "errors": {}
}
```

