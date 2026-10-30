# Spesifikasi Sistem Desain (Design System)
## Platform Undangan Digital Multi-Template "Berkah Bahagia"

**Versi Dokumen:** 1.2.0  
**Status:** Ditetapkan Berdasarkan Audit Referensi Publik & Mockup Dashboard Pilihan  
**Inspirasi Desain:** 
1. **Website Publik Undangan:** [Invitato Sekar & Aby](https://invitato.net/template-sekaraby/?code=DUMMY-P0001) *(Modern Javanese Botanical)*
2. **Dashboard Pelanggan & Admin:** `dashboard-example.webp` *(Soft Pastel Luxury Minimalist)*

---

# BAGIAN A: SISTEM DESAIN DASHBOARD (PELANGGAN & ADMIN)
*Referensi Visual: `dashboard-example.webp`*

## 1. Filosofi Estetika Dashboard

Antarmuka Dashboard dirancang dengan gaya **Soft Pastel Luxury Minimalist**:
- **Nuansa Hangat & Bersih:** Menggunakan latar belakang *warm blush peach* lembut dipadukan dengan kartu-kartu putih melengkung elegan (*super-rounded corners* `rounded-3xl` / `rounded-2xl`).
- **Palet Warna Aksen Pastel:** Perpaduan hijau mint segar (*soft mint/sage*) dan merah muda persik (*soft peach/salmon*) untuk membedakan kategori data, status, dan kartu metrik tanpa menimbulkan kelelahan mata (*visual fatigue*).
- **Tipografi Modern & Terstruktur:** Menggunakan font sans-serif modern (*Instrument Sans* / *Inter* / *Plus Jakarta Sans*) dengan hierarki angka statistik tebal dan label abu-abu yang tenang.
- **Tata Letak 3-Kolom Proporsional:**
  1. **Sidebar Kiri (Navigasi):** Logo brand dengan aksen bintang berkilau, menu navigasi berbentuk pill membulat, dan indikator badge.
  2. **Konten Tengah (Data & Statistik):** Barisan kartu metrik gradien, grafik analitik kehadiran tamu, dan tabel status undangan.
  3. **Panel Kanan (Widget & Kalender):** Kalender mini interaktif, jadwal acara mendatang, dan linimasa aktivitas terkini.

---

## 2. Token Warna Dashboard (Dashboard Color Tokens)

| Token Desain | Kode Warna | Contoh Penggunaan |
|---|---|---|
| `dash-bg-canvas` | `#FFF2EE` / `#FDF5F2` | Latar belakang seluruh halaman dashboard |
| `dash-card-bg` | `#FFFFFF` | Latar belakang kartu kontainer putih bersih |
| `dash-text-dark` | `#1E293B` / `#2D3748` | Judul halaman, nama pengguna, dan angka metrik |
| `dash-text-muted`| `#64748B` / `#94A3B8` | Label sub-teks, penunjuk waktu, dan keterangan |
| `dash-mint-bg` | `#D2F4EA` | Latar kartu stat mint, pill menu aktif, tombol primer |
| `dash-mint-dark` | `#0D9488` / `#115E59` | Teks dan ikon pada elemen bertema mint |
| `dash-peach-bg` | `#FED7D2` | Latar kartu stat peach, badge highlight, tag kategori |
| `dash-peach-dark`| `#E11D48` / `#BE123C` | Teks dan ornamen pada elemen bertema peach |
| `dash-border-soft`| `#FEE2E2` / `#F1F5F9` | Garis batas tipis pembatas tabel dan widget |

---

## 3. Komponen Utama Dashboard

### 3.1 Kartu Statistik Atas (Top Stat Cards)
- 4 kartu horizontal berjejer dengan gradien pastel halus dan ornamen bintang berkilau:
  1. **Saldo Dompet (Wallet Balance):** Gradien Soft Peach, ikon koin/dompet di dalam lingkaran putih, nominal saldo besar (misal: `Rp150.000`).
  2. **Undangan Aktif:** Gradien Soft Mint, ikon undangan, jumlah undangan yang berstatus `published`.
  3. **Total Tamu:** Gradien Soft Mint, ikon tamu/orang, total tamu terdaftar di seluruh undangan.
  4. **RSVP Masuk:** Gradien Soft Peach, ikon checklist kehadiran, persentase konfirmasi kehadiran.

### 3.2 Visualisasi Analitik & Kehadiran (Analytics Overview)
- **Donut Chart / Circular Progress:** Menampilkan ringkasan status tamu (Hadir = Mint, Belum Respon = Peach, Tidak Hadir = Abu-abu).
- **Line/Bar Chart Respon Acara:** Grafik tren pengiriman RSVP dan ucapan doa per hari/bulan.

### 3.3 Tabel Daftar Undangan (Invitation Status Table)
- Tabel modern dengan sudut membulat dan spasi lega:
  - Kolom: Nama Pengantin / Judul, Template yang Digunakan, Tanggal Acara, Sisa Masa Aktif, dan Status Badge.
  - **Status Badges:**
    - `Published`: Pill hijau mint (`bg-[#D2F4EA] text-[#0D9488]`).
    - `Draft`: Pill persik muda (`bg-[#FED7D2] text-[#BE123C]`).
    - `Inactive`: Pill abu-abu tenang (`bg-[#F1F5F9] text-[#64748B]`).
    - `Expired`: Pill merah pudar (`bg-[#FEE2E2] text-[#991B1B]`).
  - Tombol aksi: Edit Konten, Buka Live Preview, Bagikan Link Tamu.

### 3.4 Panel Widget Kanan (Right Widget Panel)
- **Kalender Mini:** Menampilkan penanggalan bulan berjalan. Tanggal hari H pernikahan ditandai lingkaran lembut warna persik/mint.
- **Jadwal Acara Mendatang (Schedules):** Kartu mini daftar acara berikutnya (Akad Nikah, Resepsi) beserta jam dan ruangan/gedung.
- **Aktivitas Terkini (Recent Activity Feed):** Linimasa vertikal notifikasi real-time:
  - "Budi Santoso mengonfirmasi Hadir (2 orang)."
  - "Pengajuan Top-Up Rp100.000 telah disetujui Admin."
  - "Siti Rahma mengirimkan doa di buku tamu."

---

# BAGIAN B: SISTEM DESAIN WEBSITE PUBLIK UNDANGAN
*Referensi Visual: Template Sekar & Aby (Modern Javanese Botanical)*

## 1. Filosofi Estetika Undangan Publik

Template **Sekar & Aby** mengusung nuansa pernikahan adat Jawa klasik yang dipadukan dengan kesegaran modern botanikal:
- **Sentuhan Alami (Earthy & Botanical):** Dominasi warna krem lembut (*warm ivory*), hijau zaitun (*olive green*), dan sentuhan emas antik (*antique gold*).
- **Animasi Organik Halus:** Gerakan mikro-animasi dedaunan melambai, burung terbang melintas perlahan, kupu-kupu mengepakkan sayap, dan kelopak bunga yang bergoyang lembut saat pengguna berinteraksi.
- **Penyajian Mobile-First:** Lebar konten dibatasi maksimal 500px (`max-w-[500px] mx-auto`) dengan bayangan mewah di tengah layar desktop.

---

## 2. Palet Warna Undangan Publik

| Token Desain | Kode Warna | Contoh Penggunaan |
|---|---|---|
| `bg-primary` | `#FFF7ED` / `rgba(255, 247, 237, 1)` | Latar belakang utama (kertas bertekstur warm ivory) |
| `bg-secondary` | `rgba(187, 87, 109, 0.7)` | Aksen warna mawar teduh / terakota lembut |
| `bg-card` | `rgba(255, 247, 237, 0.81)` | Latar belakang kartu konten dengan efek transparansi |
| `text-main` | `#636345` | Teks judul paragraf dan teks isi utama (deep olive earth) |
| `text-secondary`| `#CFAA64` / `#D6A850` | Aksen nama pengantin dan ornamen emas antik |
| `text-alt` | `#8E8F51` | Teks keterangan orang tua, label waktu, dan sub-teks |
| `border-gold` | `#CFAA64` | Garis tepi kartu doa, countdown box, dan input form |
| `btn-gradient` | `linear-gradient(to right, #8E8F51, #98957E)` | Tombol interaktif pembuka undangan & aksi utama |

---

## 3. Tipografi Undangan Publik

1. **Heading Serif (Nama Pengantin & Judul Utama):** `"Hatolie", "BauerBodoni", "Cormorant Upright", serif`
2. **Body Serif (Teks Isi, Doa, & Keterangan):** `"Cambo", "Instrument Sans", serif`
3. **Aksen Script (Sub-Heading & Monogram):** `"Citadel Script", "BickhamScriptThree", cursive`

---

## 4. Animasi & Mikro-Interaksi

```css
@keyframes pulse {
  0% { transform: scale(1); }
  50% { transform: scale(1.05); }
  100% { transform: scale(1); }
}

@keyframes flowerAnimation {
  0% { transform: rotate(0deg); }
  50% { transform: rotate(4deg); }
  100% { transform: rotate(-2deg); }
}

@keyframes flowerAnimationInvert {
  0% { transform: rotate(0deg); }
  50% { transform: rotate(-4deg); }
  100% { transform: rotate(2deg); }
}

@keyframes butterfly {
  0% { transform: translate(0) scale(1); }
  25% { transform: translate(5px, -10px) scale(0.85); }
  50% { transform: translate(-10px, -15px) scale(0.9); }
  75% { transform: translate(5px, -8px) scale(0.95); }
  100% { transform: translate(0) scale(1); }
}

@keyframes flyCross {
  0% { opacity: 0; transform: translate(0); }
  6% { opacity: 1; }
  44% { opacity: 1; }
  50% { opacity: 0; transform: translate(500px, 9vh); }
  100% { opacity: 0; transform: translate(500px, 9vh); }
}
```

---

## 5. Komponen Undangan Publik

1. **Virtual Cover Gate (Layar Pembuka):** Layar 100vh mengunci scroll sebelum tombol "Buka Undangan" ditekan. Dilengkapi animasi burung melintas dan pemutar musik otomatis.
2. **Kartu Profil Mempelai:** Potret dengan *clip-path* oval vertikal berbingkai floral keemasan dan kupu-kupu melayang.
3. **Linimasa Kisah Cinta (Love Story Timeline):** Garis vertikal menghubungkan simpul peristiwa.
4. **Countdown Timer:** 4 kotak penghitung waktu + tombol "Simpan ke Kalender" (Google Calendar).
5. **Kartu Sesi Acara:** Akad & Resepsi lengkap dengan petunjuk dresscode dan tombol langsung ke Google Maps.
6. **Tab Tanda Kasih:** Pilihan transfer bank (salin nomor rekening instan) dan kado fisik (salin alamat).
7. **Dinding Doa (Wishes Wall):** Form ucapan instan dan daftar doa dengan penanda *pinned by admin*.
8. **Floating Controls Dock:** Tombol musik (Play/Pause), Drawer navigasi samping, dan pengubah bahasa (ID/EN).
