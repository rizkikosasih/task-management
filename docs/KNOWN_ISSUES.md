# Known Issues & Technical Mitigations

Dokumen pencatatan kendala teknis dan mitigasi pada tingkat sistem orchestrator guna mencegah investigasi berulang.

---

## 1. Kegagalan Callback OAuth Tanpa Email Publik

- **Kendala:** Penyedia identitas pihak ketiga (terutama akun GitHub dengan konfigurasi privasi tinggi) dapat mengembalikan payload profil tanpa atribut email publik.
- **Dampak:** Backend gagal melakukan penautan akun otomatis dan melempar eksepsi validasi.
- **Mitigasi Terpasang:** Pengendali `SocialAuthController` memvalidasi keberadaan email. Jika email bernilai nihil, permintaan berbasis peramban diarahkan ke `${FRONTEND_URL}/login?error=oauth_email_missing&provider={provider}` (HTTP 302), sedangkan permintaan klien API mengembalikan respon JSON HTTP 422. Antarmuka login frontend menampilkan banner peringatan untuk memandu pengguna melakukan registrasi manual.

---

## 2. Pelanggaran Transisi Status Selesai (Done Status Reversion)

- **Kendala:** Upaya pengubahan status tugas yang telah berada pada status `done` kembali ke status sebelumnya (`todo` atau `in_progress`).
- **Dampak:** Penolakan transaksi dari aturan bisnis backend dengan respon galat HTTP 422.
- **Mitigasi Terpasang:** Frontend menonaktifkan kontrol drag-and-drop dan aksi dropdown untuk kartu tugas yang telah berstatus `done`, serta membatalkan mutasi optimistik pada TanStack Query jika backend mengembalikan galat penolakan.

---

## 3. Direktori Runtime Storage Laravel yang Terabaikan pada Kloning Git

- **Kendala:** Repositori hasil kloning Git tidak menyertakan folder `storage/framework/{cache,sessions,views}` dan `storage/logs` karena pola pada `.gitignore`.
- **Dampak:** Kegagalan fatal penulisan berkas sesi, kompilasi Blade dokumentasi Scramble, atau penulisan log saat pengujian live.
- **Mitigasi Terpasang:** Setiap lingkungan kerja baru wajib memastikan keberadaan subdirektori tersebut beserta berkas penjaga `.gitignore` internal.

---

## 4. Latensi Jaringan & Toleransi Ketersediaan Service Eksternal (Redis & OAuth API)

- **Kendala:** Gangguan konektivitas ke server Redis lokal atau latensi tinggi saat menghubungi endpoint verifikasi Google Tokeninfo dan GitHub API.
- **Dampak:** Potensi galat fatal `ConnectionRefusedException` pada pembatasan frekuensi atau pemblokiran thread proses backend.
- **Mitigasi Terpasang:** Penyedia `AppServiceProvider` mengonfigurasi mekanisme fallback otomatis ke driver `database` cache apabila instance Redis luring, serta `OAuthTokenValidatorService` membatasi batas waktu koneksi HTTP maksimal 5 detik dengan transformasi ke `BusinessException` berstatus HTTP 422/502.

