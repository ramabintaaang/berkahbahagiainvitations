# Panduan Pembuatan Template (Template Developer Guide)
## Cara Menambah & Mengembangkan Template Undangan Baru

**Versi Dokumen:** 1.0.0  
**Sasaran:** Pengembang Frontend & Desainer UI/UX Platform Berkah Bahagia  

---

## 1. Konsep Dasar Desain Template Modular

Platform **Berkah Bahagia** dirancang agar template baru dapat dibuat dan dipasang semudah menambahkan satu folder Blade view baru, **tanpa perlu memodifikasi controller backend, tanpa membuat tabel baru, dan tanpa mengubah skema routing**.

Setiap template hanya bertindak sebagai 'lapisan presentasi' (*view presentation layer*) yang mengonsumsi kontrak data seragam dari backend Laravel.

---

## 2. Struktur Berkas Standar Template

Setiap template ditempatkan dalam direktori terpisah di:
`resources/views/templates/{template_slug}/`

```text
resources/views/templates/{template_slug}/
├── manifest.json              # [Wajib] Berkas konfigurasi & metadata template
├── index.blade.php            # [Wajib] Berkas utama template
├── components/                # [Direkomendasikan] Pecahan section terorganisir
│   ├── cover.blade.php        # Gerbang pembuka / amplop virtual
│   ├── couple.blade.php       # Profil kedua mempelai
│   ├── events.blade.php       # Informasi sesi acara & lokasi
│   ├── love-story.blade.php   # Kisah cinta (opsional)
│   ├── countdown.blade.php    # Hitung mundur waktu acara
│   ├── gallery.blade.php      # Foto & video prewedding (opsional)
│   ├── rsvp.blade.php         # Form konfirmasi kehadiran
│   ├── wishes.blade.php       # Form doa & dinding ucapan
│   ├── gift.blade.php         # Nomor rekening & alamat kado (opsional)
│   └── footer.blade.php       # Ucapan terima kasih & hak cipta
└── assets/                    # [Opsional] Gambar dekorasi, CSS khusus, atau audio
```

---

## 3. Berkas `manifest.json`

Berkas manifes wajib ada di setiap folder template untuk mendaftarkan metadata ke sistem:

```json
{
  "slug": "sekaraby",
  "name": "Sekar & Aby (Javanese Modern Flora)",
  "version": "1.0.0",
  "category": "Tradisional Jawa Modern",
  "author": "Berkah Bahagia Design Studio",
  "thumbnail": "templates/sekaraby/assets/thumbnail.webp",
  "supported_features": [
    "cover_gate",
    "background_music",
    "love_story",
    "countdown_timer",
    "multi_events",
    "rsvp_form",
    "wishes_wall",
    "digital_gift",
    "photo_gallery",
    "multi_language"
  ],
  "default_config": {
    "primary_color": "#8e8f51",
    "secondary_color": "#cfaa64",
    "background_color": "#fff7ed",
    "font_heading": "Hatolie",
    "font_body": "Cambo"
  }
}
```

---

## 4. Kontrak Variabel Data (View Contract)

Saat merender `templates.{slug}.index`, controller secara otomatis menyuplai variabel terstandarisasi berikut:

### 4.1 `$invitation`
Objek utama undangan (`App\Models\Invitation`):
- `$invitation->title`: Judul undangan (misal: "The Wedding of Sekar & Aby")
- `$invitation->slug`: Slug URL publik
- `$invitation->cover_photo`: Path foto sampul utama
- `$invitation->music_url`: Path atau URL lagu latar
- `$invitation->quote_title`: Judul kutipan (misal: "Surat Ar-Rum : 21")
- `$invitation->quote_content`: Isi kutipan/doa pembuka

### 4.2 `$couple`
Array berisi data mempelai:
- `$couple['groom']`: Objek mempelai pria (`call_name`, `full_name`, `order_text`, `father_name`, `mother_name`, `instagram`, `photo_url`).
- `$couple['bride']`: Objek mempelai wanita (atribut serupa).

### 4.3 `$events`
Koleksi sesi acara (`Illuminate\Support\Collection` dari `Event`):
- Loop menggunakan `@foreach($events as $event)`:
  - `$event->title`: Judul sesi (Akad Nikah / Resepsi)
  - `$event->event_date`: Objek Carbon tanggal acara
  - `$event->start_time`: Waktu mulai
  - `$event->end_time`: Waktu selesai
  - `$event->location_name`: Nama gedung / tempat
  - `$event->location_address`: Alamat lengkap
  - `$event->google_maps_url`: Tautan Google Maps

### 4.4 `$stories` (Koleksi Kisah Cinta)
- Loop `@foreach($stories as $story)`:
  - `$story->year_date`: Label tahun/waktu
  - `$story->title`: Judul momen
  - `$story->story`: Narasi cerita
  - `$story->image_url`: Foto kenangan (jika ada)

### 4.5 `$galleries` (Koleksi Galeri Media)
- Loop `@foreach($galleries as $gallery)`:
  - `$gallery->media_type`: 'image' atau 'youtube_video'
  - `$gallery->media_url`: URL media
  - `$gallery->caption`: Keterangan gambar

### 4.6 `$gifts` (Koleksi Rekening & Alamat Kado)
- Loop `@foreach($gifts as $gift)`:
  - `$gift->gift_type`: 'bank', 'ewallet', atau 'physical_address'
  - `$gift->provider_name`: Nama bank (BCA, Mandiri)
  - `$gift->account_number`: Nomor rekening
  - `$gift->account_holder`: Nama pemilik
  - `$gift->physical_address`: Alamat tujuan kado fisik

### 4.7 `$guest` (Data Tamu yang Mengakses)
- Bernilai `null` jika diakses secara umum tanpa parameter kode tamu.
- Jika ada tamu (`$guest != null`):
  - `$guest->name`: Nama tamu (misal: "Bapak Budi Santoso & Keluarga")
  - `$guest->code`: Kode unik tamu
  - `$guest->max_seats`: Kuota tamu maksimal
  - `$guest->status`: Status ('pending', 'opened', 'confirmed')

### 4.8 `$wishes` (Koleksi Doa & Ucapan)
- Berisi daftar ucapan yang telah disetujui, diurutkan dengan ucapan yang disematkan (*pinned*) di paling atas.

---

## 5. Aturan Wajib Template Developer (Security & Best Practices)

1. **Dilarang Query Database Mentah:**
   Jangan pernah menggunakan `DB::select()`, `App\Models\User::find()`, atau pemanggilan Eloquent langsung di dalam Blade template. Gunakan hanya variabel yang telah disediakan controller.
2. **Dukungan Graceful Degradation:**
   Pelanggan mungkin tidak mengisi cerita cinta atau galeri video. Gunakan selalu kondisional `@if(count($stories) > 0)` atau `@if(!empty($invitation->music_url))` sebelum merender section terkait.
3. **Penyajian Mobile-First:**
   Wajib memastikan tata letak diuji pada lebar layar ponsel 360px hingga 480px. Pada layar desktop, bungkus seluruh template dalam kontainer sentral (`max-w-[500px] mx-auto min-h-screen relative shadow-2xl`) atau split layout yang elegan.
4. **Keamanan Sanitasi XSS:**
   Gunakan selalu sintaks escape Blade `{{ $variable }}` untuk menampilkan teks dari pengguna. Jangan gunakan `{!! $variable !!}` kecuali untuk field tertentu yang telah disanitasi secara ketat oleh sistem backend.
5. **Gunakan Shared Component untuk Interaksi Kompleks:**
   Untuk formulir RSVP dan kirim ucapan, gunakan komponen bersama yang terhubung ke endpoint API standar platform agar keamanan CSRF dan validasi tetap konsisten di semua template.

---

## 6. Alur Pendaftaran Template Baru

1. Buat folder baru di `resources/views/templates/{slug_baru}`.
2. Tulis berkas `manifest.json` dan struktur view `index.blade.php`.
3. Jalankan perintah artisan sinkronisasi:
   ```bash
   php artisan template:sync
   ```
   *(Perintah ini akan membaca folder template, memvalidasi `manifest.json`, dan mencatatnya ke tabel database `templates` secara otomatis).*
4. Template baru langsung muncul di katalog pilihan pelanggan di Dashboard!
