# Task Management System

Sistem manajemen proyek dan tugas kolaboratif berbasis arsitektur client-server terpisah. Backend menyediakan RESTful API dengan jaminan idempotensi transaksi dan kontrol konkurensi; frontend mengonsumsi API tersebut melalui antarmuka SPA berbasis React.

---

## Struktur Repositori

```
task-management/
├── backend/     → Laravel 11 REST API (Sanctum + Spatie RBAC + Pessimistic Locking)
└── frontend/    → Vite + React 19 SPA (TanStack Query + React Hook Form + Zod)
```

---

## Arsitektur Sistem

```
┌─────────────────────────────────────────────────────────┐
│                    CLIENT BROWSER                       │
│                                                         │
│  ┌──────────────────────────────────────────────────┐   │
│  │          Vite + React 19 SPA (Frontend)          │   │
│  │                                                  │   │
│  │  ┌─────────────┐  ┌──────────────┐              │   │
│  │  │ TanStack    │  │ React Hook   │              │   │
│  │  │ Query v5    │  │ Form + Zod   │              │   │
│  │  └──────┬──────┘  └──────┬───────┘              │   │
│  │         │                │                       │   │
│  │  ┌──────▼────────────────▼───────┐              │   │
│  │  │   Axios HTTP Client           │              │   │
│  │  │   Bearer Token Interceptor    │              │   │
│  │  │   X-Idempotency-Key Generator │              │   │
│  │  └──────────────┬────────────────┘              │   │
│  └─────────────────┼────────────────────────────────┘  │
└────────────────────┼────────────────────────────────────┘
                     │ HTTPS REST
                     ▼
┌────────────────────────────────────────────────────────┐
│              Laravel 11 REST API (Backend)             │
│                                                        │
│  ┌──────────────┐  ┌─────────────┐  ┌──────────────┐  │
│  │ Sanctum Auth │  │ Spatie RBAC │  │ Laravel      │  │
│  │ + OAuth2     │  │ + Policy    │  │ Socialite    │  │
│  │ (Google/GH)  │  │ (ABAC)      │  │              │  │
│  └──────────────┘  └─────────────┘  └──────────────┘  │
│                                                        │
│  ┌──────────────────────────────────────────────────┐  │
│  │  Idempotency Middleware + Pessimistic Locking    │  │
│  └──────────────────────────────────────────────────┘  │
│                                                        │
│  ┌──────────────────────────────────────────────────┐  │
│  │           MySQL / SQLite Database                │  │
│  └──────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────┘
```

---

## Modul Fungsional

| Modul | Deskripsi | Backend | Frontend |
| :--- | :--- | :---: | :---: |
| Autentikasi Lokal | Registrasi, login, logout via Sanctum token | ✓ | ✓ |
| OAuth2 | Login/register via Google & GitHub (Socialite) | ✓ | ✓ |
| Manajemen Proyek | CRUD proyek dengan isolasi kepemilikan Admin | ✓ | ✓ |
| Papan Tugas | Kanban board (todo/in_progress/done) + konkurensi | ✓ | ✓ |
| Komentar | Thread diskusi per tugas dengan ABAC penghapusan | ✓ | ✓ |
| RBAC/ABAC | Kontrol akses berbasis peran dan atribut kepemilikan | ✓ | ✓ |
| Idempotensi | Header X-Idempotency-Key pada POST project/task | ✓ | ✓ |

---

## Kontrol Akses (RBAC + ABAC)

| Aksi | Admin (Pemilik) | Admin (Lain) | Member (Assignee) | Member (Lain) |
| :--- | :---: | :---: | :---: | :---: |
| Buat proyek | ✓ | ✓ | ✗ | ✗ |
| Edit/hapus proyek | ✓ | ✗ | ✗ | ✗ |
| Buat tugas | ✓ | ✗ | ✗ | ✗ |
| Edit detail tugas | ✓ | ✗ | ✗ | ✗ |
| Ubah status tugas | ✓ | ✗ | ✓ | ✗ |
| Hapus tugas | ✓ | ✗ | ✗ | ✗ |
| Buat komentar | ✓ | ✓ | ✓ | ✓* |
| Hapus komentar | Milik sendiri | Milik sendiri | Milik sendiri | Milik sendiri |

*Member lain hanya bisa melihat/mengomentari jika memiliki akses ke tugas terkait.

---

## Backend

**Stack:** Laravel 11 · PHP 8.3 · MySQL · Laravel Sanctum · Spatie Permission · Laravel Socialite

**Fitur Teknis:**
- Autentikasi hibrida: token lokal (Sanctum) + OAuth2 (Google, GitHub) dengan account linking otomatis berbasis email.
- Kontrol akses dua lapis: RBAC via Spatie Permission + ABAC via Laravel Policy (kepemilikan `created_by` / `assigned_to`).
- Idempotensi transaksi via middleware `X-Idempotency-Key` pada endpoint POST project dan task.
- Kontrol konkurensi via pessimistic locking (`lockForUpdate`) pada transisi status tugas.
- Standar respon terpadu `ApiResponse` envelope di seluruh endpoint.
- Dokumentasi OpenAPI 3.0 auto-generated via Scramble.

**Dokumentasi:**
- [PRD Backend](backend/PRD.md)
- [API Standardization](backend/docs/api/STANDARDIZATION.md)
- [Kontrak Auth](backend/docs/api/contracts/auth.md)
- [Kontrak OAuth](backend/docs/api/contracts/oauth.md)
- [Kontrak Projects](backend/docs/api/contracts/projects.md)
- [Kontrak Tasks](backend/docs/api/contracts/tasks.md)
- [Kontrak Comments](backend/docs/api/contracts/comments.md)
- [Decision Log](backend/docs/DECISION_LOG.md)
- [Glossary](backend/docs/GLOSSARY.md)

**Memulai Backend:**

```bash
cd backend
composer install
cp .env.example .env
php artisan key:generate
php artisan migrate
php artisan db:seed
php artisan serve
```

---

## Frontend

**Stack:** Vite · React 19 · TypeScript (strict) · Tailwind CSS · TanStack Query v5 · React Hook Form · Zod · Axios

**Fitur Teknis:**
- SPA murni dengan client-side routing dan route guard berbasis peran (Admin/Member).
- Server state management terisolasi via TanStack Query v5 dengan caching dan auto-invalidation.
- Validasi form client-side via Zod schema yang merefleksikan aturan validasi backend.
- HTTP client terpusat dengan interceptor Bearer Token dan generator `X-Idempotency-Key` (UUIDv4).
- Penanganan galat konkurensi HTTP 409/422 pada transisi status tugas.
- TypeScript strict mode: seluruh payload request/response memiliki definisi interface eksplisit.

**Dokumentasi:**
- [PRD Frontend](frontend/PRD.md)
- [State Management Map](frontend/docs/STATE_MANAGEMENT_MAP.md)
- [Component Tokens](frontend/docs/registry/COMPONENT_TOKENS.md)
- [Module Boundary Map](frontend/docs/MODULE_BOUNDARY_MAP.md)

**Memulai Frontend:**

```bash
cd frontend
npm install
cp .env.example .env.local
npm run dev
```

**Variabel Lingkungan:**

| Variabel | Keterangan | Contoh |
| :--- | :--- | :--- |
| `VITE_API_BASE_URL` | URL base backend API | `http://localhost:8000/api` |
| `VITE_APP_NAME` | Nama aplikasi yang tampil di UI | `Task Management` |

---

## Endpoint API Backend

Base URL: `http://localhost:8000/api`

| Metode | Endpoint | Deskripsi | Auth |
| :--- | :--- | :--- | :---: |
| POST | `/auth/register` | Registrasi akun baru | ✗ |
| POST | `/auth/login` | Login lokal | ✗ |
| POST | `/auth/logout` | Logout & revoke token | ✓ |
| GET | `/auth/me` | Profil pengguna aktif | ✓ |
| GET | `/oauth/{provider}/redirect` | Redirect ke OAuth provider | ✗ |
| GET | `/oauth/{provider}/callback` | Callback OAuth provider | ✗ |
| POST | `/oauth/{provider}/link` | Link akun OAuth tambahan | ✓ |
| GET | `/projects` | Daftar proyek (paginasi + search) | ✓ |
| POST | `/projects` | Buat proyek baru (idempotent) | ✓ Admin |
| GET | `/projects/{id}` | Detail proyek | ✓ |
| PUT | `/projects/{id}` | Edit proyek | ✓ Admin Owner |
| DELETE | `/projects/{id}` | Hapus proyek | ✓ Admin Owner |
| GET | `/tasks` | Daftar tugas (filter + paginasi) | ✓ |
| POST | `/tasks` | Buat tugas baru (idempotent) | ✓ Admin |
| GET | `/tasks/{id}` | Detail tugas | ✓ |
| PUT | `/tasks/{id}` | Edit detail tugas | ✓ Admin Owner |
| PATCH | `/tasks/{id}/status` | Ubah status tugas | ✓ Owner/Assignee |
| DELETE | `/tasks/{id}` | Hapus tugas | ✓ Admin Owner |
| GET | `/tasks/{id}/comments` | Daftar komentar | ✓ |
| POST | `/tasks/{id}/comments` | Tambah komentar | ✓ |
| DELETE | `/comments/{id}` | Hapus komentar | ✓ Pemilik |

---

## Prasyarat Sistem

| Komponen | Versi Minimum |
| :--- | :--- |
| PHP | 8.3 |
| Composer | 2.x |
| Node.js | 20 LTS |
| npm | 10.x |
| MySQL | 8.0 |

---

## Lisensi

Proyek ini dibangun untuk keperluan portofolio teknis. Seluruh hak cipta dimiliki oleh pemilik repositori.

