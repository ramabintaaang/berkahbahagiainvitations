# Product Requirements Document (PRD)
## Platform Undangan Digital Multi-Template ("Berkah Bahagia")

**Versi Dokumen:** 1.1.0  
**Status:** Ditetapkan Berdasarkan Keputusan Final Model Pembelian & Masa Aktif  
**Target Rilis:** Tahap MVP (Minimum Viable Product)  
**Basis Teknologi:** Laravel 13, PHP 8.3+, MySQL, Tailwind CSS v4, Vite  

---

## 1. Ringkasan Eksekutif & Visi Produk

Platform **Berkah Bahagia** adalah platform SaaS undangan pernikahan digital modern yang memungkinkan calon pengantin atau penyelenggara acara membuat, mengelola, dan mempublikasikan undangan berbasis web secara mandiri.

Sistem ini menggabungkan fleksibilitas visual dan model bisnis yang teruji:
- **Model Pembelian Satu Kali (One-Time Purchase):** Setiap undangan dibeli secara terpisah menggunakan saldo dompet (*wallet balance*) pelanggan. Tidak ada skema langganan berkala (*no recurring subscription*) pada tahap awal.
- **Masa Aktif Pasti (1 Tahun):** Setiap undangan yang berhasil dibeli memiliki masa aktif default selama 1 tahun (365 hari) yang dihitung sejak transaksi pembelian berhasil, bukan dari tanggal publikasi pertama kali.
- **Satu Backend Inti, Banyak Template:** Desain visual dipisahkan dari logika bisnis dan data mentah. Template baru dapat didaftarkan dengan harga dinamis yang ditentukan oleh Admin.
- **Pengalaman Tamu Mobile-First:** Tampilan mewah berstandar tinggi yang dioptimalkan untuk smartphone dengan animasi mikro, gerbang amplop virtual, integrasi navigasi, dan RSVP instan.

---

## 2. Pengguna Sasaran (User Personas)

### 2.1 Super Admin / Platform Admin
- **Peran:** Pengelola operasional dan keuangan platform.
- **Tujuan:**
  - Mengatur katalog template, menetapkan harga masing-masing template, dan mengatur masa aktif default.
  - Memeriksa pengajuan *top-up* saldo pelanggan, memvalidasi bukti transfer, dan menyetujui penambahan saldo wallet.
  - Memantau audit transaksi pembelian, riwayat masa aktif, dan status kedaluwarsa undangan.
  - Memoderasi akun pengguna, undangan publik, dan buku tamu.

### 2.2 Pelanggan (Customer / Pasangan Pengantin)
- **Peran:** Pengguna terdaftar yang memiliki dompet saldo dan mengelola undangan.
- **Tujuan:**
  - Melakukan pendaftaran dan pengajuan pengisian saldo (*top-up* wallet) melalui transfer manual.
  - Membeli satu atau banyak undangan digital sesuai template pilihan menggunakan saldo yang dimiliki.
  - Mengelola data masing-masing undangan secara mandiri (data mempelai, jadwal acara, cerita cinta, galeri media, dan amplop digital).
  - Mengontrol status publikasi (*draft*, *published*, *inactive*) selama masa aktif masih berlaku.
  - Memantau sisa masa aktif undangan dan tanggal kedaluwarsa (*expires_at*).
  - Mengelola daftar tamu, menyebarkan tautan undangan personal via WhatsApp, serta memantau rekap RSVP dan ucapan doa.

### 2.3 Tamu Undangan (Guest / Publik)
- **Peran:** Penerima tautan undangan pernikahan.
- **Tujuan:** Mengakses undangan yang aktif, membuka amplop virtual, melihat rincian acara, mengonfirmasi kehadiran (RSVP), mengirim ucapan doa, dan menyalin nomor rekening untuk memberikan tanda kasih.
- **Batasan:** Jika undangan berstatus *expired* atau *inactive*, tamu akan melihat halaman pemberitahuan yang sopan dan tidak dapat mengakses konten privat.

---

## 3. Model Bisnis, Dompet (Wallet), & Masa Aktif

### 3.1 Model Pembelian Satu Kali (One-Time Purchase)
1. **Tidak Ada Biaya Langganan Berkala:** Pelanggan tidak dikenakan tagihan bulanan atau tahunan otomatis.
2. **Pembelian Berbasis Undangan (Per-Invitation Purchase):**
   - Setiap undangan dibuat melalui transaksi pembelian template individual.
   - Satu akun pelanggan dapat memiliki dan mengelola banyak undangan sekaligus (misal: satu undangan untuk resepsi keluarga, satu untuk perayaan sahabat, atau untuk saudara).
3. **Harga Template Fleksibel (Configurable Pricing):**
   - Harga setiap template ditentukan secara dinamis oleh Admin melalui dashboard, bukan nilai statis di dalam kode.
   - Contoh: Template *Sekar & Aby* Rp75.000, template dengan fitur animasi 3D/interaktif lebih kompleks Rp120.000.
4. **Snapshot Harga Pembelian:**
   - Saat transaksi pembelian berhasil, sistem mencatat snapshot harga beli (`price_paid`) pada catatan undangan dan riwayat transaksi.
   - Perubahan harga template oleh Admin di kemudian hari tidak akan memengaruhi data transaksi lama.
5. **Konsistensi Transaksi Database:**
   - Pemotongan saldo dompet dan instansiasi undangan wajib dibungkus dalam `DB::transaction()` dengan penguncian pesimistik (*pessimistic locking*) untuk mencegah saldo minus atau pembelian ganda (*double spending*).
   - Pengeditan konten pada undangan yang sudah aktif tidak memotong saldo pelanggan.

### 3.2 Siklus Alur Saldo & Top-Up (MVP Flow)
1. **Pengajuan Top-Up oleh Pelanggan:**
   - Pelanggan memilih nominal pengisian saldo dan mengunggah bukti struk transfer bank pada formulir dashboard.
   - Data tercatat dengan status `pending`.
2. **Verifikasi Manual oleh Admin:**
   - Admin memeriksa mutasi rekening dan memvalidasi keabsahan bukti struk transfer.
   - Jika valid, Admin menyetujui (*Approve*) pengajuan. Saldo dompet pelanggan bertambah secara otomatis dan dicatat dalam buku kas (*wallet transaction ledger*).
   - Jika tidak valid, Admin menolak (*Reject*) disertai alasan penolakan.
3. **Pembelian Template Menggunakan Saldo:**
   - Pelanggan memilih template di katalog dan menekan tombol beli.
   - Jika saldo mencukupi, sistem mendebit saldo dan menginisialisasi undangan baru dengan status `draft`.

### 3.3 Siklus Hidup & Status Undangan (Lifecycle & Validity)
Setiap undangan memiliki empat status resmi yang diatur oleh sistem dan pelanggan:

```text
+-----------+       Beli Berhasil         +-----------+
| (Katalog) | --------------------------> |   draft   |
+-----------+                             +-----------+
                                                |
                               +----------------+----------------+
                               |                                 |
                        Klik "Publikasikan"              Klik "Nonaktifkan"
                               |                                 |
                               v                                 v
                        +-----------+   Klik "Nonaktifkan" +-----------+
                        | published | -------------------> | inactive  |
                        +-----------+ <------------------- +-----------+
                               |         Klik "Aktifkan"
                               | (jika now >= expires_at)
                               v
                        +-----------+
                        |  expired  |
                        +-----------+
```

1. **`draft`:** Undangan baru saja dibeli, sedang diisi datanya oleh pelanggan, dan belum dapat diakses oleh publik.
2. **`published`:** Undangan aktif dan dapat diakses publik melalui URL `/u/{slug}` selama masa aktif masih berlaku (`now() < expires_at`).
3. **`inactive`:** Pelanggan sengaja menonaktifkan sementara tampilan publik (misal: sedang ada revisi mendadak). Akses publik ditutup, namun sisa masa aktif tetap berjalan.
4. **`expired`:** Masa aktif undangan telah berakhir (`now() >= expires_at`).
   - Ditentukan secara ketat berdasarkan perbandingan waktu server dengan kolom `expires_at`, bukan hanya berdasarkan field status yang diubah pengguna.
   - Halaman publik `/u/{slug}` otomatis memblokir akses konten dan menampilkan informasi masa aktif berakhir.
   - Pelanggan tidak dapat mengubah status menjadi `published` kembali tanpa mekanisme yang sah.
   - **Retensi Data:** Data pengantin, acara, galeri, dan ucapan **TIDAK DIHAPUS** dari database ketika masa aktif habis, melainkan tetap diarsipkan agar dapat dipertimbangkan untuk diaktifkan kembali jika fitur perpanjangan dibuka di masa depan.

### 3.4 Perhitungan Masa Aktif
- **Masa Aktif Default:** 1 tahun (365 hari) terhitung sejak tanggal transaksi pembelian berhasil (`purchased_at`).
- **Aturan Ketat:**
  - Masa aktif **TIDAK** dihitung dari saat pertama kali undangan dipublikasikan.
  - Mengubah status menjadi `inactive` **TIDAK** menghentikan atau menjeda (*pause*) hitungan masa aktif.
  - Tidak ada sistem perpanjangan otomatis (*no auto-renewal*).

---

## 4. Batasan Ruang Lingkup MVP (MVP Scope Boundaries)

Untuk memastikan peluncuran yang cepat, stabil, dan minim risiko, ditetapkan batasan tegas:

### Fitur yang Masuk Scope MVP:
1. Registrasi, login, dan manajemen profil pelanggan.
2. Dompet saldo pelanggan (*Customer Wallet*).
3. Pengajuan top-up saldo manual oleh pelanggan dengan upload bukti transfer.
4. Panel Admin untuk verifikasi, persetujuan (*approval*), dan penolakan top-up.
5. Katalog template dengan penetapan harga dan masa aktif oleh Admin.
6. Pembelian undangan menggunakan pemotongan saldo wallet secara atomik.
7. Dashboard pelanggan untuk mengelola banyak undangan secara terpisah.
8. CRUD konten undangan (mempelai, jadwal multi-acara, cerita cinta, galeri media, tanda kasih).
9. Kontrol status publikasi undangan (*draft*, *published*, *inactive*, *expired*).
10. Proteksi akses publik saat masa aktif 1 tahun kedaluwarsa.
11. Template perdana teruji: **Sekar & Aby (Javanese Modern Flora)**.
12. Manajemen tamu sederhana (daftar nama & generator tautan WhatsApp).
13. Formulir RSVP dan dinding ucapan doa publik (AJAX).

### Fitur yang DITUNDA dari Scope MVP:
1. **Fitur Perpanjangan Masa Aktif (Renewal Extension):**
   - Tidak ada tombol perpanjangan, tidak ada keranjang perpanjangan, dan tidak ada transaksi perpanjangan pada versi awal.
   - Struktur database dipersiapkan agar kompatibel untuk perpanjangan masa depan, namun implementasi UI/proses bisnisnya ditunda.
2. **Payment Gateway Otomatis (Midtrans/Xendit):**
   - Menggunakan alur verifikasi top-up manual oleh Admin terlebih dahulu.
3. **Editor Drag-and-Drop Visual Kompleks:**
   - Menggunakan formulir wizard terstruktur yang rapi dan teruji.
4. **Langganan Berkala / Paket Subscription Bulanan:**
   - Ditiadakan sepenuhnya sesuai model pembelian satu kali.

---

## 5. Kebutuhan Fungsional Berdasarkan Peran

### 5.1 Admin Platform (Admin Panel)
1. **Manajemen Saldo & Top-Up:**
   - Meninjau antrean pengajuan top-up pelanggan beserta bukti transfer.
   - Tombol *Approve* (menambahkan saldo ke wallet secara otomatis) dan *Reject* (dengan catatan alasan).
   - Buku besar (*ledger*) riwayat seluruh transaksi saldo.
2. **Katalog & Konfigurasi Template:**
   - Menetapkan harga template individual (misal: Rp75.000 / Rp120.000).
   - Menetapkan masa aktif default (default: 365 hari).
   - Mengatur status ketersediaan template (Aktif / Non-aktif).
3. **Monitoring Undangan:**
   - Melihat daftar seluruh undangan, pemiliknya, tanggal pembelian, dan tanggal kedaluwarsa.
   - Melihat riwayat transaksi pembelian undangan.
4. **Manajemen Pengguna:**
   - Meninjau daftar pengguna terdaftar dan mengelola status akun (aktif/suspend).

### 5.2 Dashboard Pelanggan
1. **Ringkasan Dompet (My Wallet):**
   - Tampilan saldo saat ini secara real-time.
   - Tombol "Ajukan Top-Up Saldo" (nominal + upload bukti transfer).
   - Riwayat mutasi saldo (Top-up disetujui, pembelian undangan).
2. **Manajemen Multi-Undangan (My Invitations):**
   - Daftar seluruh undangan yang dimiliki dengan indikator status jelas (*Draft*, *Published*, *Inactive*, *Expired*).
   - Informasi tanggal pembelian (`purchased_at`), tanggal kedaluwarsa (`expires_at`), dan sisa hari masa aktif.
   - Tombol toggle status publikasi (*Publish* / *Unpublish*).
   - Tombol pengeditan konten undangan dan tombol pratinjau (*Live Preview*).
3. **Pembelian Undangan Baru:**
   - Menjelajahi katalog template beserta harga masing-masing.
   - Konfirmasi pembelian menggunakan saldo dompet.
4. **Editor Konten Undangan:**
   - Form data pengantin (pria & wanita, gelar, orang tua, media sosial, foto profil).
   - Form sesi acara (multi-acara, tanggal, jam, alamat, Google Maps, dresscode).
   - Cerita cinta & galeri foto prewedding.
   - Pengaturan amplop digital (rekening bank & alamat kado fisik).
5. **Buku Tamu & RSVP:**
   - Kelola daftar tamu dan buat tautan personalisasi WhatsApp.
   - Rekap konfirmasi kehadiran tamu (Hadir, Tidak Hadir, Jumlah Kursi).
   - Moderasi ucapan doa dan fitur penyematan (*pin*) ucapan ke atas.

### 5.3 Website Undangan Publik (`/u/{slug}`)
1. **Pemeriksaan Akses & Masa Aktif:**
   - Jika status `draft` atau `inactive`: tampilkan halaman "Undangan belum dapat diakses".
   - Jika `now() >= expires_at`: tampilkan halaman khusus "Masa Aktif Undangan Telah Berakhir".
   - Jika `published` dan `now() < expires_at`: render template lengkap.
2. **Render Interaktif Template:**
   - Gerbang pembuka fullscreen (*Cover Gate*) dengan tombol "Buka Undangan" dan pemutar musik otomatis.
   - Seluruh section konten: Mempelai, Acara, Hitung Mundur + Google Calendar, Peta Google Maps, Love Story, Galeri, Tanda Kasih Digital (salin nomor rekening instan).
   - Form RSVP instan dan form kirim doa/ucapan via AJAX.

---

## 6. Kebutuhan Non-Fungsional (Non-Functional Requirements)

1. **Keamanan Transaksi Dompet:**
   - Menjamin integritas saldo pelanggan dengan transaksi atomik dan pembatasan saldo negatif (`balance >= 0`).
   - Mencegah *double-spending* ketika pelanggan menekan tombol pembelian secara cepat dan berulang.
2. **Proteksi & Validasi File Bukti Transfer:**
   - Bukti transfer dibatasi format `image/jpeg`, `image/png`, `image/webp` dengan ukuran maksimal 3MB.
   - Disimpan di disk privat atau terproteksi yang hanya dapat dilihat oleh Admin dan pengunggah yang sah.
3. **Performa Render Publik:**
   - Pengecekan masa aktif dan resolusi template harus di-cache dan efisien agar tidak membebani server saat ribuan tamu membuka tautan serentak.
4. **Responsivitas Layar:**
   - Desain 100% optimal pada ponsel pintar (360px - 480px) dan dibingkai sentral 500px pada desktop view.
