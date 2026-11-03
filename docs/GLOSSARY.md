# Glossary & Domain Dictionary

Kamus istilah domain bisnis dan teknis pada Task Management System.

---

## Istilah Domain Bisnis

- **Admin (Project Owner):** Pengguna dengan otoritas penuh membuat proyek, mengelola detail proyek, mendelegasikan tugas kepada anggota, dan menghapus tugas.
- **Member (Assignee):** Pengguna pelaksana yang ditugaskan pada tugas spesifik di bawah proyek. Berhak memperbarui status tugas miliknya dan berpartisipasi dalam diskusi komentar.
- **Tugas (Task):** Unit pekerjaan mandiri di bawah proyek dengan alur status terdefinisi: `todo`, `in_progress`, dan `done`.
- **Komentar (Comment):** Entitas diskusi tekstual di bawah entitas tugas yang hanya dapat dihapus oleh pembuat komentar tersebut.

---

## Istilah Arsitektur Teknis

- **ApiResponse Envelope:** Format pembungkus standar respon JSON dari backend yang memisahkan status, pesan, data payload, dan metadata.
- **Idempotency Key:** Kunci acak (UUIDv4) yang dikirimkan via header HTTP `X-Idempotency-Key` untuk menjamin operasi pembuatan data hanya dieksekusi satu kali meskipun terjadi transmisi berulang.
- **Pessimistic Locking:** Mekanisme penguncian baris basis data saat transaksi berlangsung (`SELECT ... FOR UPDATE`) untuk mencegah modifikasi konkuren simultan.
- **Optimistic Mutation:** Pembaruan status antarmuka pengguna secara langsung sebelum server mengonfirmasi respon, dengan kemampuan pemulihan (*rollback*) jika server mengembalikan penolakan.

