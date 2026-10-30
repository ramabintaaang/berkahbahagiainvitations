# Panduan Keamanan & Isolasi Data (Security & Data Isolation)
## Kebijakan Otorisasi, Validasi, Transaksi Keuangan, & Proteksi Privasi

**Versi Dokumen:** 1.1.0  
**Status:** Ditetapkan Berdasarkan Keputusan Model Pembelian & Masa Aktif  
**Standar Kepatuhan:** OWASP Top 10, Praktik Keamanan Finansial Laravel 13  

---

## 1. Prinsip Utama Keamanan Platform

Sebagai platform SaaS multi-tenant dengan sistem dompet (*wallet*) dan pembelian berbasis undangan, **Berkah Bahagia** menerapkan standar keamanan ketat:
1. **Pemisahan Penyewa (Tenant Isolation):** Pelanggan A sama sekali tidak dapat melihat, mengubah, memotong saldo, atau memanipulasi undangan Pelanggan B.
2. **Integritas Transaksi Keuangan (Financial Ledger Integrity):** Saldo dompet pelanggan dilindungi dari saldo minus, transaksi ganda (*double spending*), dan *race conditions*. Seluruh mutasi wajib tercatat dalam buku besar (*ledger*) yang tidak dapat diubah (*immutable audit trail*).
3. **Penegakan Masa Aktif & Akses Publik (Lifecycle & Expiration Enforcement):** Undangan yang kedaluwarsa (`now() >= expires_at`) otomatis diblokir dari akses publik tanpa risiko manipulasi parameter dari sisi klien.
4. **Pencegahan Eksekusi Kode Arbitrer (No Arbitrary Code Execution):** Seluruh input teks, bukti transfer, dan konfigurasi template murni diperlakukan sebagai data pasif.

---

## 2. Keamanan Dompet Saldo & Transaksi Pembelian (Wallet Security)

### 2.1 Pencegahan *Race Condition* & *Double Spending*
Ketika pelanggan membeli undangan, saldo dompet dipotong secara atomik:
```php
DB::transaction(function () use ($user, $template) {
    // Kunci baris dompet pengguna untuk mencegah eksekusi paralel
    $wallet = Wallet::where('user_id', $user->id)->lockForUpdate()->firstOrFail();

    if ($wallet->balance < $template->price) {
        throw new InsufficientBalanceException("Saldo tidak mencukupi untuk membeli template ini.");
    }

    $balanceBefore = $wallet->balance;
    $wallet->balance -= $template->price;
    $wallet->save();

    // Catat mutasi buku kas
    WalletTransaction::create([
        'wallet_id' => $wallet->id,
        'type' => 'purchase',
        'amount' => -$template->price,
        'balance_before' => $balanceBefore,
        'balance_after' => $wallet->balance,
        'reference_type' => Invitation::class,
        'reference_id' => $invitation->id,
        'description' => "Pembelian template: {$template->name}",
    ]);

    // Instansiasi undangan dengan status draft & masa aktif 1 tahun
    return Invitation::create([
        'user_id' => $user->id,
        'template_id' => $template->id,
        'title' => 'Draf Undangan Baru',
        'slug' => Str::slug($user->name . '-' . Str::random(6)),
        'price_paid' => $template->price,
        'purchased_at' => now(),
        'expires_at' => now()->addDays($template->active_days ?? 365),
        'status' => 'draft',
    ]);
});
```

### 2.2 Batasan Saldo Negatif (Zero Sub-Zero Protection)
- Kolom `balance` pada tabel `wallets` didefinisikan sebagai `DECIMAL(12, 2) UNSIGNED`. Jika terjadi kesalahan logika pada level aplikasi yang mencoba mengurangi saldo di bawah nol, level database MySQL akan langsung melempar SQL Constraint Violation dan menggugurkan transaksi.

### 2.3 Validasi & Proteksi Berkas Bukti Transfer Top-Up
1. **Validasi File Bukti Struk:**
   - Format yang diterima hanya `image/jpeg`, `image/png`, dan `image/webp`.
   - Ukuran berkas maksimal 3MB.
   - Pengecekan MIME tipe menggunakan Magic Bytes via `Illuminate\Http\UploadedFile::getMimeType()`.
2. **Penyimpanan Terisolasi:**
   - Berkas bukti transfer disimpan di disk privat (`storage/app/private/topups/{uuid}.ext`), bukan di public disk.
   - Akses file bukti transfer hanya dapat dibuka melalui rute berotentikasi oleh Admin yang berwenang dan pelanggan pemilik transaksi bersangkutan.

---

## 3. Penegakan Masa Aktif & Kontrol Status Undangan

### 3.1 Blokir Otomatis Rute Publik (`/u/{slug}`)
Pemeriksaan waktu kedaluwarsa dilakukan langsung pada level server tanpa bergantung pada flag status input pengguna:
```php
public function show(string $slug)
{
    $invitation = Invitation::where('slug', $slug)->firstOrFail();

    // 1. Evaluasi masa aktif kedaluwarsa
    if ($invitation->expires_at && now()->greaterThanOrEqualTo($invitation->expires_at)) {
        return response()->view('public.expired', compact('invitation'), 410);
    }

    // 2. Evaluasi status publikasi
    if ($invitation->status !== 'published') {
        abort(404, 'Undangan belum dipublikasikan atau dinonaktifkan.');
    }

    // 3. Render template jika aktif & masa berlaku sah
    return $this->templateRenderer->render($invitation);
}
```

### 3.2 Pencegahan Publikasi Ulang Undangan Kedaluwarsa
- Pada controller dashboard pelanggan, aksi `publish` divalidasi dengan aturan ketat:
  Jika `now() >= invitation->expires_at`, sistem menolak permintaan publikasi dan mengembalikan pesan error bahwa masa aktif telah habis.
- Menonaktifkan undangan (`inactive`) **tidak mengubah nilai `expires_at`**, sehingga pelanggan tidak dapat memperpanjang masa aktif dengan cara menonaktifkan dan mengaktifkan kembali undangan.

---

## 4. Isolasi Data Multi-Tenant & Otorisasi Pengguna

### 4.1 Pemisahan Peran (Role-Based Access Control)
- **Role Admin (`role = 'admin'`):** Mengakses `/admin/*`, verifikasi pengajuan top-up, pengaturan katalog harga template, dan audit sistem.
- **Role Customer (`role = 'customer'`):** Hanya mengakses `/dashboard/*` untuk dompet miliknya dan undangan yang dibuat oleh akunnya sendiri.

### 4.2 Penerapan Laravel Policies
Setiap operasi dilindungi oleh Policy khusus:
- **`InvitationPolicy`:**
  - `view(User $user, Invitation $invitation)`: `$user->id === $invitation->user_id || $user->isAdmin()`
  - `update(User $user, Invitation $invitation)`: `$user->id === $invitation->user_id && $invitation->expires_at > now()`
  - `delete(User $user, Invitation $invitation)`: `$user->id === $invitation->user_id`
- **`TopupRequestPolicy`:**
  - Pelanggan hanya dapat melihat riwayat pengajuan top-up miliknya sendiri.

---

## 5. Perlindungan Route Publik & Proteksi Draft

1. **Proteksi Undangan Draft & Inactive:**
   - Publik yang mengakses undangan berstatus `draft` atau `inactive` menerima respons `HTTP 404 Not Found`.
   - Pemilik yang sah dapat melihat preview hanya melalui rute berotentikasi di dashboard: `/dashboard/invitations/{id}/preview`.
2. **Kerahasiaan Daftar Tamu:**
   - Parameter `?code={guest_code}` hanya mengembalikan nama tamu spesifik yang bersangkutan.
   - Endpoint publik dilarang mengekspos koleksi seluruh nomor telepon atau daftar tamu lain kepada publik.

---

## 6. Pencegahan Serangan Injeksi & XSS (Cross-Site Scripting)

1. **Dinding Doa & Ucapan (Public Wishes Wall):**
   - Kolom ucapan disanitasi ketat menggunakan `strip_tags()` pada backend form request (`StoreWishRequest`).
   - Seluruh teks ucapan dirender menggunakan sintaks escape Blade `{{ $wish->message }}`. Dilarang merender dengan `{!! $wish->message !!}`.
2. **Honeypot & Rate Limiting:**
   - Endpoint publik `POST /u/{slug}/rsvp` dan `POST /u/{slug}/wishes` dilindungi rate limit `throttle:6,1` (maksimal 6 pengiriman per menit per IP).
   - Form menyertakan input tersembunyi (*honeypot*) untuk memblokir bot pengirim spam otomatis.
