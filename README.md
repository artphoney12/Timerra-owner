# Timerra Owner Update Repository

Repository ini dipakai sebagai sumber update untuk Timerra Server, Timerra Kasir, dan Timerra App.

File utama:

- `timerra-update.json` - manifest update yang dibaca aplikasi.
- `Timerra_Local_Portable.rar` - paket portable terbaru.

## Rekomendasi Penempatan File

Gunakan GitHub Releases untuk file RAR, karena ukuran paket portable biasanya besar.

Contoh URL RAR:

```text
https://github.com/artphoney12/Timerra-owner/releases/latest/download/Timerra_Local_Portable.rar
```

Manifest JSON boleh ditaruh di root repository agar mudah dibaca aplikasi:

```text
https://raw.githubusercontent.com/artphoney12/Timerra-owner/main/timerra-update.json
```

## Alur Release Update

1. Build Timerra portable terbaru dari project lokal.
2. Buat RAR dari folder:

```text
Timerra_Local_Portable
```

Nama RAR harus:

```text
Timerra_Local_Portable.rar
```

3. Simpan password RAR di environment variable lokal terlebih dulu. Jangan commit password ini ke GitHub.

```powershell
$env:TIMERRA_RAR_PASSWORD = "ISI_PASSWORD_RAHASIA_DI_SINI"
```

4. Buat RAR dengan kompresi maksimal, password, dan nama file terenkripsi:

```powershell
& "C:\Program Files\WinRAR\rar.exe" a -r -ep1 -m5 "-hp$env:TIMERRA_RAR_PASSWORD" `
  "Timerra_Local_Portable.rar" `
  "Timerra_Local_Portable\*"
```

5. Hitung SHA256 RAR:

```powershell
Get-FileHash .\Timerra_Local_Portable.rar -Algorithm SHA256
```

6. Upload `Timerra_Local_Portable.rar` ke GitHub Releases.
7. Edit `timerra-update.json`:
   - `latestVersion`
   - `publishedAt`
   - `package.url`
   - `package.sha256`
   - `package.sizeBytes`
   - `notes`
8. Commit `timerra-update.json` ke branch `main`.
9. Aplikasi Timerra akan membaca manifest dan menawarkan update bila versi GitHub lebih baru.

## Cara Kerja Self Updater

Timerra tidak menimpa exe yang sedang berjalan secara langsung. Aplikasi akan menjalankan helper:

```text
Timerra Updater.exe
```

Alurnya:

1. Aplikasi cek `timerra-update.json`.
2. Aplikasi download `Timerra_Local_Portable.rar` ke folder temporary.
3. Aplikasi cek SHA256.
4. Aplikasi menjalankan `Timerra Updater.exe`.
5. Aplikasi utama ditutup.
6. Updater extract RAR memakai `unrar.exe`.
7. Updater copy file target sesuai jenis aplikasi:
   - Server
   - Kasir
   - App
8. Updater menghapus file temporary.
9. Updater membuka kembali aplikasi yang selesai diupdate.

## File Yang Boleh Ditimpa

### Timerra Server

Boleh ditimpa:

- `Timerra Server.exe`
- `Timerra Server Core.exe` jika nanti dipisah
- `Timerra Updater.exe`
- `web\operator\*`
- `db\migrations\*`
- `tools\*`
- `third_party\*`

Tidak boleh ditimpa:

- `config\server.json`
- `database\data\*`
- `backups\*`
- `logs\*`
- `exports\*`
- `config\rclone\*`

### Timerra Kasir

Boleh ditimpa:

- `Timerra Kasir.exe`
- `Timerra Updater.exe`

Tidak boleh ditimpa:

- `config\kasir.json`
- `logs\*`

### Timerra App

Boleh ditimpa:

- `Timerra App.exe`
- `Timerra Updater.exe`

Tidak boleh ditimpa:

- `config\app.json`
- `logs\*`

## Akses Menu Update

- Timerra Server: menu `Update Timerra` langsung tersedia.
- Timerra Kasir: menu `Update Timerra` hanya muncul jika login sebagai admin.
- Timerra App: menu `Update Timerra` hanya muncul setelah unlock admin memakai password admin yang sama dari server.

## Catatan Keamanan

- Selalu isi `package.sha256` dengan hash RAR final.
- Jangan update jika hash RAR tidak cocok.
- Password RAR tidak ditaruh di `timerra-update.json` atau `README.md`.
- Jangan timpa folder database, config, backup, log, export, dan rclone.
- Jika update gagal, jalankan ulang paket portable lama atau restore dari backup release.
