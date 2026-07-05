# WhatsApp Blast - Premium Desktop Application

Aplikasi desktop WhatsApp Blast premium yang dirancang untuk kebutuhan pengiriman pesan terjadwal dan pengiriman masal secara mandiri, aman, dan tangguh. Aplikasi ini beroperasi sepenuhnya di sisi lokal komputer Anda (self-hosted) tanpa bergantung pada server pihak ketiga untuk menjaga privasi data Anda 100%.

---

## 🎯 Fitur Utama

- **🚀 Blast Pesan Masal & Terjadwal**: Kirim pesan atau media (Gambar, Dokumen, Video) ke banyak kontak sekaligus secara terjadwal dengan aman.
- **📥 Kontak & Grup Sync**: Impor dan sinkronisasikan data kontak serta grup WhatsApp Anda secara instan dari perangkat Anda.
- **🛡️ Sistem Hardening Antrean (SQLite)**: Jika komputer mati mendadak atau koneksi terputus, sisa antrean pesan tersimpan aman di database lokal SQLite Anda dan akan dilanjutkan secara otomatis saat aplikasi dibuka kembali.
- **📦 System Tray Integration**: Tutup aplikasi ke System Tray di taskbar Windows agar pengiriman pesan background tetap berjalan lancar tanpa mengganggu aktivitas kerja Anda.
- **🔐 Secure Auto-Update**: Sistem pengecekan pembaruan otomatis yang aman dengan verifikasi integritas **SHA256** dan tanda tangan digital **Ed25519** untuk menjamin keamanan aplikasi dari ancaman pembajakan.

---

## 📥 Cara Download dan Instalasi

Untuk mendapatkan versi rilis terbaru dari WhatsApp Blast:

1. Pergi ke tab **[Releases](https://github.com/lintang-dwi/bot-wa/releases)** di bagian kanan halaman ini.
2. Cari rilis terbaru (misalnya `v1.0.4` atau versi terbaru yang berlabel **Latest**).
3. Di bawah bagian **Assets**, klik dan unduh berkas **`whatsapp-blast.exe`**.
4. Setelah terunduh, letakkan berkas `.exe` tersebut di folder pilihan Anda (misalnya Desktop atau Documents).
5. Jalankan aplikasi dengan mengeklik ganda berkas tersebut.

> [!TIP]
> Aplikasi ini tidak memerlukan instalasi (portable). Seluruh data sesi masuk WhatsApp Anda akan disimpan secara aman di folder konfigurasi lokal komputer Anda.

---

## 🛡️ Kebijakan Privasi & Keamanan
Aplikasi ini dibangun menggunakan modul enkripsi WhatsMeow yang terhubung langsung ke server resmi WhatsApp Web. Seluruh database pesan, histori log pengiriman, dan nomor kontak disimpan secara lokal di harddisk Anda dan **tidak pernah dikirimkan ke server eksternal mana pun**.
