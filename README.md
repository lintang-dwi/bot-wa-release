# WhatsApp Blast - Release Repository & Developer Guide

Repositori ini digunakan untuk mengelola rilis aplikasi **WhatsApp Blast**, menghasilkan installer setup, membuat berkas **Delta Update** (patch), menandatangani berkas biner secara digital dengan **Ed25519**, serta mempublikasikan pembaruan ke GitHub Releases dan GitHub Pages.

---

## 📁 Struktur Dokumentasi (Folder `docs/`)
Seluruh dokumentasi teknis dan panduan pengembangan aplikasi telah disatukan ke dalam folder [`docs/`](file:///d:/Project%20Production/Whatsapp%20Blast/bot-wa-release/docs/):
- **[`Convert.md`](file:///d:/Project%20Production/Whatsapp%20Blast/bot-wa-release/docs/Convert.md)**: Analisis mendalam fitur utama, alur kerja (manajemen sesi, antrean SQLite, dll.), serta rencana migrasi dari library JavaScript ke Go Whatsmeow.
- **[`Whatsmeow peningkatan.md`](file:///d:/Project%20Production/Whatsapp%20Blast/bot-wa-release/docs/Whatsmeow%20peningkatan.md)**: Panduan detail penanganan event WhatsApp, manajemen koneksi chat/media, sinkronisasi kontak/grup, dan penanganan timeout/reconnect menggunakan library Go Whatsmeow.
- **[`whatsmeow-guide.md`](file:///d:/Project%20Production/Whatsapp%20Blast/bot-wa-release/docs/whatsmeow-guide.md)**: Panduan lengkap instalasi, integrasi event handler, pengiriman pesan teks/media, dan pengelolaan sesi Whatsmeow.
- **[`whatsmeow_features.md`](file:///d:/Project%20Production/Whatsapp%20Blast/bot-wa-release/docs/whatsmeow_features.md)**: Rincian fitur-fitur Whatsmeow seperti multi-device login, sinkronisasi riwayat obrolan offline, dan pengiriman media terenkripsi secara native.
- **[`github_publish_guide.md`](file:///d:/Project%20Production/Whatsapp%20Blast/bot-wa-release/docs/github_publish_guide.md)**: Panduan langkah-demi-langkah setup token rilis GitHub, pembuatan rilis tag baru, dan alur publikasi manual/otomatis.

---

## 🛠️ Persiapan Sebelum Melakukan Rilis
Sebelum menjalankan perintah rilis, pastikan lingkungan pengembangan Anda sudah siap:

1. **GitHub Token**:
   Set environment variable `GITHUB_TOKEN` di terminal Anda agar release-tool dapat membuat rilis otomatis di GitHub dan mengunggah aset:
   ```powershell
   $env:GITHUB_TOKEN="your_personal_access_token_here"
   ```
2. **Kunci Ed25519**:
   Kunci privat untuk tanda tangan digital berada di `keys/private.key` (terbaca dari `config.yaml`). Jangan pernah membagikan berkas kunci privat ini ke publik! Kunci publik terkait telah di-embed ke dalam biner klien `bot-wa`.

---

## 🚀 Alur Rilis Update & Installer Baru

Proses rilis, kompilasi, obfuscation (garble), pembuatan installer NSIS, pembuatan delta patch, dan publikasi ke GitHub dilakukan sepenuhnya secara otomatis melalui berkas **`release-tool`**.

### Langkah-Langkah Rilis:

1. Buka berkas **[`config.yaml`](file:///d:/Project%20Production/Whatsapp%20Blast/bot-wa-release/config.yaml)**.
2. Atur nomor versi baru pada field `version` (misalnya `"1.0.6"`).
3. Atur catatan rilis pada field `description` (misalnya `"Rilis versi 1.0.6 - Perbaikan bug koneksi"`).
4. Pastikan `build.output` menunjuk ke `"dist/whatsapp-blast-payload.dat"` dan `build.obfuscated: true`.
5. Jalankan perintah kompilasi dan rilis di folder `bot-wa-release`:
   ```powershell
   go run ./release-tool
   ```

### Apa yang Terjadi di Balik Layar saat Perintah Dijalankan?
1. **Auto-Sync Versi**: Versi di `config.yaml` disinkronkan secara otomatis ke `wails.json` dan `app_update.go` (konstanta `AppVersion`) di repositori `bot-wa`.
2. **Build Obfuscated**: Proyek utama `bot-wa` dikompilasi menggunakan `wails build -clean -nsis -obfuscated` dengan enkripsi string/literal via Garble.
3. **Penyusunan Output Dist**:
   - Biner portable hasil kompilasi diganti namanya menjadi `whatsapp-blast-payload-v<versi>.dat` (tidak menggunakan ekstensi `.exe` agar user tidak mengunduh payload langsung).
   - Installer setup NSIS dipindahkan ke `dist/whatsapp-blast-payload-v<versi>-setup.exe`.
4. **Pembuatan Delta Patch (bsdiff)**:
   - Tool membaca versi rilis sebelumnya dari `docs/version.json` (misalnya `v1.0.5`).
   - Mengunduh payload `.dat` lama dari rilis GitHub v1.0.5 secara otomatis ke folder lokal.
   - Membandingkan biner lama v1.0.5 dengan biner baru v1.0.6 secara byte-to-byte menggunakan `bsdiff.Bytes` di memori RAM.
   - Menghasilkan berkas **`whatsapp-blast-1.0.5_to_1.0.6.patch`** (biasanya berukuran sangat kecil, hanya ~1-2 MB).
5. **Penandatanganan Keamanan (Ed25519)**:
   - Menghasilkan checksum SHA256 dari installer setup, biner payload `.dat`, dan berkas `.patch`.
   - Menandatangani berkas installer setup, payload `.dat`, dan berkas `.patch` menggunakan kunci privat Ed25519.
   - Menulis file metadata `dist/version.json` dan menyalinnya ke `docs/version.json`.
6. **Upload & Publish GitHub**:
   - Membuat rilis baru di GitHub sesuai versi target, lalu mengunggah aset rilis (`-setup.exe`, `.dat` payload, dan `.patch`).
   - Melakukan `git commit` dan `git push` otomatis untuk berkas `docs/version.json` agar terpublikasi di GitHub Pages untuk dibaca oleh fitur auto-updater klien.

---

## 📥 Mekanisme Update di Sisi Pengguna (Aplikasi Klien)

Aplikasi klien yang telah terpasang memiliki logika pembaruan cerdas sebagai berikut:
1. **Check Updates**: Klien membaca berkas `version.json` yang di-host di GitHub Pages.
2. **Delta Update (Utama)**:
   - Jika versi lama klien terdeteksi dalam blok `delta` (misal dari v1.0.5 ke v1.0.6), aplikasi hanya mengunduh berkas patch `.patch` yang kecil (~1.5 MB).
   - Memverifikasi tanda tangan Ed25519 berkas patch tersebut.
   - Membaca biner berjalan klien ke RAM, lalu menerapkan patch di memori untuk menghasilkan biner baru.
   - Memverifikasi biner baru hasil patch dengan data `portable` di metadata.
   - Melakukan penggantian file biner berjalan dengan aman (dengan skema backup/rollback jika gagal), lalu me-restart aplikasi secara otomatis.
3. **Full Installer Fallback (Cadangan)**:
   - Jika delta update gagal atau tidak tersedia untuk versi lama klien, aplikasi akan mengunduh installer setup (`whatsapp-blast-payload-v<versi>-setup.exe`, ~12 MB).
   - Menjalankan installer setup secara normal di sistem operasi pengguna untuk memperbarui seluruh berkas aplikasi, kemudian me-restart aplikasi.
