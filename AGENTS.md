# Panduan Agen AI (Agent Guidelines)
## Platform Undangan Digital Multi-Template "Berkah Bahagia"

Dokumen ini berisi pedoman operasional dan standar rekayasa perangkat lunak untuk setiap agen AI (Antigravity, Claude Code, Cursor, GitHub Copilot) yang bekerja pada repositori ini.

---

## 1. Prasyarat Lingkungan & Verifikasi

Sebelum menjalankan perintah atau mengembangkan fitur baru, pastikan lingkungan pengembang telah memenuhi spesifikasi:

```sh
php -v        # Minimal PHP 8.3.0
composer -V   # Minimal Composer 2.6+
node -v       # Minimal Node.js 18.x
npm -v
```

Jika perintah di atas belum tersedia pada sistem pengguna, pasang prasyarat sesuai sistem operasi:
- **macOS:** `/bin/bash -c "$(curl -fsSL https://php.new/install/mac/8.5)"`
- **Linux:** `/bin/bash -c "$(curl -fsSL https://php.new/install/linux/8.5)"`
- **Windows (PowerShell):** `Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://php.new/install/windows/8.5'))`

---

## 2. Gambaran Proyek & Batasan Arsitektur

Platform ini adalah sistem SaaS undangan pernikahan digital berbasis **Laravel 13** dengan model pembelian satu kali (*one-time purchase*) berbasis saldo dompet (*wallet*).

### Batasan Arsitektur Wajib:
1. **Satu Backend Inti, Banyak Template:**
   - Dilarang membuat tabel baru, model baru, controller baru, atau migration baru hanya untuk menambahkan satu template undangan baru.
   - Semua template visual harus mengonsumsi kontrak data terstandarisasi (`$invitation`, `$couple`, `$events`, `$stories`, `$galleries`, `$gifts`, `$guest`, `$wishes`).
2. **Pemisahan Data dan Presentasi:**
   - Folder template diisolasi di `resources/views/templates/{template_slug}/`.
   - Setiap template wajib memiliki `manifest.json`.
3. **Penyajian Mobile-First:**
   - Setiap template wajib optimal di viewport layar ponsel (360px - 480px).
   - Pada viewport desktop, sajikan dalam frame ponsel sentral (`max-w-[500px] mx-auto shadow-2xl`) atau split layout yang proporsional.
4. **Keamanan Tanpa Kompromi:**
   - Dilarang menggunakan `eval()` atau `Blade::render()` dari teks input pengguna.
   - Dilarang menulis query SQL mentah di dalam view Blade template.
   - Seluruh input ucapan doa dari publik wajib disanitasi dan di-escape untuk mencegah XSS.
   - Seluruh operasi CRUD di dashboard wajib memvalidasi kepemilikan tenant (`user_id`).

---

## 3. Aturan Bisnis Finansial & Masa Aktif (Wajib Diikuti)

1. **Model Pembelian Satu Kali (One-Time Purchase):**
   - Dilarang membuat fitur langganan bulanan/berkala (*subscription*) pada tahap MVP.
   - Setiap undangan dibeli secara terpisah menggunakan saldo dompet (*wallet*).
   - Satu pelanggan dapat memiliki dan mengelola banyak undangan sekaligus.
   - Harga setiap template dapat berbeda dan dikonfigurasi dinamis oleh Admin melalui database (bukan nilai hardcoded).
2. **Transaksi Saldo Atomik & Snapshot Harga:**
   - Pemotongan saldo dompet dan instansiasi undangan wajib dibungkus dalam `DB::transaction()` dengan `lockForUpdate()` pada model `Wallet`.
   - Wajib mencatat mutasi pada `wallet_transactions` dan snapshot harga beli pada `invitations.price_paid`.
   - Pengeditan konten pada undangan yang sudah dibeli **dilarang memotong saldo lagi**.
3. **Penetapan & Hitungan Masa Aktif (1 Tahun):**
   - Masa aktif default adalah 1 tahun (365 hari) terhitung sejak tanggal transaksi pembelian berhasil (`purchased_at` -> `expires_at`).
   - Dilarang menghitung masa aktif dari tanggal undangan pertama kali dipublikasikan.
   - Menonaktifkan undangan (`status = 'inactive'`) **tidak menghentikan / tidak menjeda hitungan masa aktif**.
   - Tidak ada perpanjangan otomatis (*no auto-renewal*).
4. **Penegakan Status Undangan:**
   - Status resmi: `draft`, `published`, `inactive`, `expired`.
   - Status `expired` wajib dievaluasi berdasarkan perbandingan waktu server `now() >= expires_at`, bukan hanya dari input pengguna.
   - Akses publik ke undangan yang kedaluwarsa wajib diblokir.
   - **Retensi Data:** Dilarang menghapus baris data undangan dari database ketika masa aktif habis. Konten tetap disimpan utuh untuk kemungkinan reaktivasi di masa depan.
5. **Batasan Scope MVP:**
   - Fitur perpanjangan (renewal) ditunda dari scope MVP (jangan membuat tombol, keranjang, atau alur transaksi perpanjangan saat ini).
   - Alur top-up menggunakan pengajuan manual oleh pelanggan (upload struk) dan verifikasi persetujuan oleh Admin.

---

## 4. Konvensi Kode Laravel 13

1. **Routing:**
   - Route publik undangan: `/u/{slug}` (ditangani oleh `PublicInvitationController`).
   - Route dashboard pelanggan: `/dashboard/*` (ditangani oleh `Customer/*Controller` dengan middleware `auth`, `verified`).
   - Route platform admin: `/admin/*` (ditangani oleh `Admin/*Controller` dengan middleware `auth`, `role:admin`).
2. **Form Request & Validasi:**
   - Gunakan kelas dedicated `FormRequest` untuk validasi formulir kompleks (misal: `StoreTopupRequest`, `StoreInvitationRequest`, `StoreRsvpRequest`, `StoreWishRequest`).
   - Jangan menulis validasi manual berulang di dalam controller.
3. **Penanganan Berkas Unggahan:**
   - Bukti transfer top-up disimpan di disk privat: `storage/app/private/topups/{uuid}.ext`.
   - Media publik undangan disimpan melalui disk `public`: `storage/app/public/invitations/{uuid}/...`.
   - Beri nama acak dengan `Str::uuid()`.
   - Validasi ketat MIME type (magic bytes) dan ukuran berkas (maksimal 3MB foto, 5MB audio).

---

## 5. Indeks Referensi Dokumentasi

Sebelum memulai implementasi modul apa pun, agen **wajib** merujuk pada dokumen teknis yang relevan di direktori `docs/`:

- [PRD (Kebutuhan Produk)](file:///Users/Rama/berkahbahagia/docs/PRD.md)
- [Arsitektur Sistem](file:///Users/Rama/berkahbahagia/docs/ARCHITECTURE.md)
- [Rancangan Database & ERD](file:///Users/Rama/berkahbahagia/docs/DATABASE.md)
- [Sistem Desain (Sekar & Aby)](file:///Users/Rama/berkahbahagia/docs/DESIGN_SYSTEM.md)
- [Panduan Pembuatan Template](file:///Users/Rama/berkahbahagia/docs/TEMPLATE_GUIDE.md)
- [Kebijakan Keamanan & Privasi](file:///Users/Rama/berkahbahagia/docs/SECURITY.md)
- [Roadmap Implementasi](file:///Users/Rama/berkahbahagia/docs/ROADMAP.md)

---

## 6. Protokol Kerja Agen

1. **Persetujuan Sebelum Mengubah Skema / Koding:**
   Jangan membuat migrasi, mengubah database, atau menulis kode implementasi sebelum perencanaan dan dokumentasi disetujui oleh pemilik proyek.
2. **Persetujuan Dependensi Baru:**
   Jangan menginstal paket composer atau npm baru tanpa menjelaskan alasan teknis dan manfaatnya terlebih dahulu.
3. **Kerahasiaan Kredensial:**
   Jangan pernah mencetak isi kredensial, password database, APP_KEY, atau token rahasia dari `.env` ke dalam laporan atau respons publik.
4. **Verifikasi Tes:**
   Setiap penambahan fitur baru wajib disertai atau diverifikasi dengan menjalankan pengujian otomatis:
   ```sh
   php artisan test
   ```
