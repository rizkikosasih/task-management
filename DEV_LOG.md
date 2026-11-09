# Developer Log - November 2026

## 2026-11-04

### Inisiasi Orchestrator & Penyelarasan Multi-Repo

- **Konfigurasi Git Multi-Repo:** Menetapkan repositori root `task-management` sebagai meta-orchestrator yang mengisolasi repositori lokal `backend` dan `frontend` melalui konfigurasi `.gitignore`.
- **Stabilisasi Direktori Runtime Backend:** Merekonstruksi struktur subdirektori `storage/framework/cache/data`, `storage/framework/sessions`, `storage/framework/views`, dan `storage/logs` pada direktori backend beserta berkas `.gitignore` untuk mencegah kegagalan runtime I/O.
- **Registrasi Backlog Modernisasi Backend:** Mendaftarkan butir agenda teknis (Logging JSON, Tagged Cache Redis, Standarisasi Indeks B-Tree Basis Data, dan Health Probes) pada branch `docs/backend-backlog-tasks` di repositori `backend-task-management`.
- **Penyusunan Dokumentasi Sistem:** Menginisiasi master dokumen spesifikasi sistem (`PRD.md`), Master ADR (`docs/DECISION_LOG.md`), katalog kendala (`docs/KNOWN_ISSUES.md`), dan glosarium terminologi (`docs/GLOSSARY.md`).
- **Scaffolding Frontend SPA:** Menginisialisasi basis kode klien berbasis Vite, React 19, TypeScript strict mode, dengan integrasi TanStack Router, TanStack Query, Zustand, dnd-kit, dan Axios.
- **Implementasi Modul Inti Frontend:** Menyelesaikan integrasi modul Otentikasi (lokal & OAuth2), Manajemen Proyek terpaginasi, Papan Tugas Kanban drag-and-drop dengan mutasi optimistik, serta linimasa komentar tugas pada repositori frontend via PR #1.
- **Registrasi Backlog Frontend:** Mendaftarkan 5 butir agenda teknis (desain token semantik dark/light, optimasi mobile-first kanban, setup test runner Vitest & MSW, serta audit aksesibilitas) pada docs/TASKS.md di repositori frontend via PR #2.

### Implementasi Sistem Tema Semantik & Desain Token Frontend

- **Isolasi Branch Kerja:** Membuat branch fitur baru `feat/semantic-theme-tokens` pada repositori `frontend` dari basis cabang integrasi `main`.
- **Desain Token Semantik Tailwind v4:** Mengonfigurasi token warna semantik (`background`, `surface`, `foreground`, `border`, `primary`, `secondary`) pada `@theme` di `src/index.css` dengan rasio kontras WCAG 2.1 AA (minimal 4.5:1 untuk teks normal).
- **Zustand Theme Store:** Membangun `useThemeStore` dengan persistensi `localStorage`, sinkronisasi event storage antar-tab peramban, dan watcher reaktif `prefers-color-scheme`.
- **Mitigasi Defek Runtime Tema:** Memasang skrip pemblokir sinkron di `index.html` untuk mengeliminasi FOUC, penyesuaian CSS `color-scheme` untuk kontrol bawaan peramban dan scrollbar, penanganan distorsi background autofill, serta enkapsulasi eksepsi akses storage.
- **Penyelarasan Komponen Antarmuka:** Memigrasikan seluruh komponen UI dasar (`Button`, `Input`, `Modal`, `Badge`) serta komponen fitur (`TaskCard`, `KanbanColumn`, `ProjectCard`, `TaskDetailModal`, `CommentList`, `CommentForm`, `LoginForm`, `RegisterForm`, `dashboard.tsx`, `projects/index.tsx`, `projects/$projectId.tsx`, `index.tsx`) ke token semantik.
- **Sinkronisasi Dokumentasi:** Menyelaraskan registri desain token pada `docs/registry/COMPONENT_TOKENS.md` dan pemetaan arsitektur state klien pada `docs/STATE_MANAGEMENT_MAP.md`.

### Restrukturisasi Dokumentasi README Frontend

- **Aset Visual & Favicon:** Menyalin aset visual `og-image.png` dan `favicon.ico` dari repositori backend ke `frontend/.github/assets/og-image.png` dan `frontend/public/favicon.ico`.
- **Infrastruktur Code Quality & Formatter:** Mengonfigurasi `.prettierrc` dan `.prettierignore` (dengan perlindungan isolasi berkas markdown `*.md`), konfigurasi aturan linter `.oxlintrc.json`, serta menambahkan skrip `typecheck`, `lint`, `format:check`, `format`, dan `check` pada `frontend/package.json`.
- **Restrukturisasi Dokumentasi README:** Menyusun ulang berkas `frontend/README.md` mencakup deskripsi fungsional sistem, arsitektur tema semantik, peta hierarki direktori, panduan instalasi, dan spesifikasi standar mutu kode.

## 2026-11-09

### Konsolidasi Task Backlog & Penyelarasan Multi-Repo

- **Pemusatan Task Backlog ke Workspace:** Memindahkan seluruh agenda backlog dari `backend/docs/TASKS.md` dan `frontend/docs/TASKS.md` ke `docs/TASKS.md` tingkat workspace dengan struktur hierarki 3 level standar (`Priority` -> `Sub-Poin` -> `Item Pekerjaan`). Berkas backlog pada repositori backend dan frontend dikosongkan dan diarahkan ke berkas root workspace via PR backend #12 dan PR frontend #6.
- **Pendaftaran Agenda CI Frontend & Dokumentasi OpenAPI:** Mendaftarkan item pekerjaan pembuatan pipeline GitHub Actions CI untuk frontend dan standarisasi OpenAPI 3.0 via Swagger UI/Scramble pada `docs/TASKS.md` via PR workspace #3 dan #4.

### Sinkronisasi Kontrak Kritis & Penanganan Defek Antar-Layanan

- **Penyelarasan Header Idempotensi Backend:** Mengubah `IdempotencyMiddleware` agar membaca header `X-Idempotency-Key` dan `Idempotency-Key` secara fleksibel dengan validasi UUIDv4. Menambahkan pengujian integrasi `tests/Feature/IdempotencyTest.php` via PR backend #13.
- **Perbaikan Alur Redirect & Callback OAuth End-to-End:** Menyelaraskan respon endpoint `/oauth/{provider}/redirect` pada backend agar menyertakan properti `redirect_url` dan `url`. Mengubah callback backend agar melakukan pengalihan peramban HTTP 302 ke `${FRONTEND_URL}/oauth/callback?token={token}` untuk permintaan web dan mempertahankan respon JSON untuk permintaan API. Menambahkan mitigasi profil penyedia tanpa email publik melalui pengalihan ke `/login?error=oauth_email_missing` dan respon 422 JSON via PR backend #13.
- **Penegakan Hak Akses Proyek Member:** Memperketat metode `ProjectService::getProjects` dengan filter `whereHas('tasks', assigned_to == user_id)` untuk peran member guna mencegah kebocoran data proyek luar tugas penugasan via PR backend #13.
- **Perbaikan Pemicu CI Backend:** Menambahkan pemicu event `pull_request` pada `.github/workflows/laravel-ci.yml` dan memverifikasi kelulusan pemeriksaan CI via PR backend #14.
- **Penyelarasan Kontrak & Port Lokal Frontend:** Mengonfigurasi port server lokal frontend ke port 5101 (`vite.config.ts`), menyelaraskan konsumsi `redirect_url` pada `src/features/auth/components/LoginForm.tsx`, dan menambahkan ekstraksi serta penayangan banner kesalahan OAuth pada antarmuka login via PR frontend #7.
- **Penyelarasan Dokumentasi Sistem Workspace:** Menyelaraskan spesifikasi amplop respon pada `PRD.md` dengan properti boolean `success`, melengkapi catatan mitigasi pada `docs/KNOWN_ISSUES.md`, merestrukturisasi `README.md` workspace dengan tautan repositori remote tanpa tautan dokumentasi lokal rapuh, dan menandai penyelesaian pekerjaan pada `docs/TASKS.md`.

