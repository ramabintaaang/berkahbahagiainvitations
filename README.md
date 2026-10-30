# Berkah Bahagia — Platform Undangan Digital Multi-Template

<p align="center">
  <strong>Platform SaaS Undangan Pernikahan Digital Berbasis Laravel 13 dengan Arsitektur Multi-Template Dinamis</strong>
</p>

---

## 1. Tentang Proyek

**Berkah Bahagia** adalah platform modern yang dirancang untuk memungkinkan calon pengantin membuat, mengelola, dan membagikan undangan pernikahan digital yang elegan secara mandiri.

Berbeda dari website undangan pernikahan statis sekali pakai, platform ini dibangun dengan prinsip arsitektur dan model bisnis yang terukur:
- **Model Pembelian Satu Kali (One-Time Purchase):** Setiap undangan dibeli secara terpisah menggunakan saldo dompet (*wallet balance*) pelanggan. Tidak ada skema langganan bulanan berkala (*no subscription*).
- **Masa Aktif 1 Tahun:** Setiap undangan memiliki masa aktif default selama 1 tahun (365 hari) sejak transaksi pembelian berhasil, bukan dari saat dipublikasikan.
- **Satu Backend Laravel 13:** Pusat logika bisnis terpadu untuk autentikasi, dompet saldo, manajemen multi-undangan, data pengantin, jadwal acara, galeri media, daftar tamu, RSVP, dan ucapan doa.
- **Katalog Template Dinamis:** Desain visual undangan dipisahkan sepenuhnya dari struktur penyimpanan data. Template baru dapat didaftarkan kapan saja dengan harga dinamis yang ditentukan oleh Admin.
- **Referensi Desain Awal:** Mengadopsi standar estetika tinggi dari template pernikahan adat Jawa modern [Invitato Sekar & Aby](https://invitato.net/template-sekaraby/?code=DUMMY-P0001) dengan ornamen botanikal, animasi mikro organik, dan pengalaman pengguna *mobile-first*.

---

## 2. Fitur Utama Platform (Scope MVP)

### A. Admin Platform (Super Admin)
- Dashboard metrik operasional & keuangan platform.
- Manajemen pengajuan top-up saldo pelanggan (verifikasi bukti struk transfer & persetujuan).
- Manajemen katalog template: penetapan harga individual dan masa aktif default (365 hari).
- Manajemen pengguna (Customer & Admin) dan kontrol status akun.
- Audit log mutasi dompet dan riwayat transaksi pembelian undangan.
- Moderasi undangan publik dan buku tamu.

### B. Dashboard Pelanggan (Customer Self-Service)
- Registrasi mandiri, login, dan manajemen profil.
- Dompet Saldo (*Wallet*): Cek saldo aktif, pengajuan top-up manual dengan bukti transfer, dan riwayat mutasi.
- Pembelian template undangan menggunakan saldo secara instan dan atomik.
- Pengelolaan banyak undangan (*multi-invitations*) dalam satu akun.
- Wizard pengisian data terstruktur (Mempelai, Multi-Acara, Kisah Cinta, Galeri Foto, Tanda Kasih Digital).
- Kontrol status publikasi undangan (*Draft*, *Published*, *Inactive*, *Expired*).
- Tampilan sisa masa aktif dan tanggal kedaluwarsa undangan yang transparan.
- Pratinjau instan (*live preview*) dalam frame ponsel.
- Manajemen tamu: Pembuatan kode personalisasi, generator teks undangan siap kirim ke WhatsApp.
- Pemantauan konfirmasi RSVP dan dinding ucapan doa secara real-time.

### C. Website Undangan Publik (`/u/{slug}`)
- Resolusi slug URL publik yang aman dan ramah pengguna.
- Penjaga masa aktif (*Expiration Guard*): otomatis memblokir akses publik jika masa aktif telah kedaluwarsa (`now() >= expires_at`).
- Layar pembuka virtual (*Cover Envelope Gate*) dengan musik latar otomatis.
- Tampilan responsif *mobile-first* (dibingkai sentral 500px yang elegan pada desktop).
- Informasi profil mempelai berbingkai potret mewah.
- Linimasa kisah cinta (*interactive love story*).
- Hitung mundur acara (*countdown timer*) + tombol "Simpan ke Kalender" (Google Calendar).
- Rincian sesi acara + tombol navigasi langsung ke Google Maps.
- Formulir konfirmasi kehadiran (RSVP) interaktif secara instan (AJAX).
- Dinding doa dan ucapan tamu dengan fitur penyematan ucapan favorit.
- Amplop digital dengan fitur salin nomor rekening bank dan salin alamat kirim kado sekali klik.

---

## 3. Tumpukan Teknologi (Tech Stack)

- **Backend Framework:** Laravel 13 (PHP 8.3+)
- **Database:** MySQL 8.0+
- **Frontend Template Engine:** Laravel Blade Component Architecture
- **Styling:** Tailwind CSS v4 + Vanilla CSS untuk animasi mikro
- **Asset Bundler:** Vite 8
- **Testing:** PHPUnit 12

---

## 4. Indeks Dokumentasi Teknis

Seluruh dokumentasi teknis dan panduan arsitektur platform tersedia secara lengkap dalam folder [`docs/`](file:///Users/Rama/berkahbahagia/docs):

1. [**Product Requirements Document (PRD)**](file:///Users/Rama/berkahbahagia/docs/PRD.md) — Visi produk, model pembelian satu kali, siklus masa aktif 1 tahun, persona pengguna, dan batasan ruang lingkup MVP.
2. [**System Architecture**](file:///Users/Rama/berkahbahagia/docs/ARCHITECTURE.md) — Diagram arsitektur sistem, alur transaksi dompet & pembelian, alur verifikasi top-up, dan alur kerja Expiration Guard.
3. [**Database Schema & ERD**](file:///Users/Rama/berkahbahagia/docs/DATABASE.md) — Rancangan 15 tabel database (termasuk wallets, topup_requests, wallet_transactions, templates.price, invitations.expires_at), relasi entitas, dan indeks performa.
4. [**Design System**](file:///Users/Rama/berkahbahagia/docs/DESIGN_SYSTEM.md) — Token warna, tipografi, keyframe animasi CSS, dan spesifikasi komponen UI Sekar & Aby.
5. [**Template Developer Guide**](file:///Users/Rama/berkahbahagia/docs/TEMPLATE_GUIDE.md) — Panduan pengembang untuk membuat dan mendaftarkan template undangan baru secara modular.
6. [**Security & Data Isolation**](file:///Users/Rama/berkahbahagia/docs/SECURITY.md) — Kebijakan keamanan finansial dompet, isolasi multi-tenant, proteksi data tamu, validasi upload, dan pemblokiran akses expired.
7. [**Implementation Roadmap**](file:///Users/Rama/berkahbahagia/docs/ROADMAP.md) — Rencana kerja bertahap dari Fase 0 (audit) hingga Fase 8 (peluncuran MVP).

---

## 5. Panduan Instalasi Cepat (Quick Start)

### Prasyarat
- PHP >= 8.3 (dengan ekstensi `pdo`, `mbstring`, `openssl`, `tokenizer`, `xml`, `ctype`, `json`, `bcmath`, `fileinfo`)
- Composer >= 2.6
- Node.js >= 18.x & NPM
- MySQL >= 8.0

### Langkah Instalasi
1. Kloning repositori dan masuk ke direktori proyek:
   ```bash
   cd berkahbahagia
   ```

2. Pasang dependensi PHP dan Node:
   ```bash
   composer install
   npm install
   ```

3. Siapkan berkas konfigurasi lingkungan:
   ```bash
   cp .env.example .env
   php artisan key:generate
   ```

4. Sesuaikan kredensial koneksi database pada berkas `.env`.

5. Jalankan migrasi dan seeder awal:
   ```bash
   php artisan migrate --seed
   ```

6. Tautkan storage publik:
   ```bash
   php artisan storage:link
   ```

7. Jalankan server lokal:
   ```bash
   composer run dev
   # atau jalankan secara terpisah:
   # php artisan serve
   # npm run dev
   ```

---

## 6. Lisensi

Platform ini dikembangkan di bawah lisensi internal tertutup untuk proyek **Berkah Bahagia**.
