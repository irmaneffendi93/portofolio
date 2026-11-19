# Studi Kasus: Pemulihan Website WordPress dari Error 502 Bad Gateway

> Kasus nyata (bukan simulasi lab). Dokumentasi ditulis ulang setelah perbaikan selesai; bukti yang tersedia adalah screenshot kondisi sebelum dan sesudah.

## Ringkasan

Website **bmdlaboratory.com** (WordPress di belakang Cloudflare) menampilkan error **502 Bad Gateway**. Penelusuran mempersempit masalah ke origin server, lalu ke sebuah tabel database MyISAM yang berstatus "in use" (crash). Tabel diperbaiki dengan *Repair table* dan situs kembali normal.

| | |
|---|---|
| **Jenis masalah** | Website down (HTTP 502) |
| **Waktu kejadian** | 2026-11-19, 03:48 UTC |
| **Pemulihan** | Situs normal kembali (diverifikasi 2026-11-20) |
| **Dampak data** | Tidak ada data yang dihapus |

## Lingkungan

| Komponen | Detail |
|---|---|
| CMS | WordPress (plugin: iThemes Security, Formidable, e2pdf) |
| Web server | nginx 1.26.3 |
| PHP | 8.4.27 |
| Database | MariaDB 12.1.1, tabel engine MyISAM |
| CDN / proxy | Cloudflare (node Singapore) |
| Alat | Browser, phpMyAdmin |

## Gejala

Halaman error menampilkan "Bad gateway, Error code 502". Diagnostik Cloudflare pada halaman itu menunjukkan:

- **Browser:** Working
- **Cloudflare:** Working
- **Host (origin server):** Error

![Error 502 sebelum perbaikan](images/01-sebelum-error-502.jpeg)

## Analisis

1. **Isolasi lapisan.** Dari diagnostik Cloudflare, browser dan Cloudflare normal, sehingga masalah dipersempit ke origin server.
2. **Identifikasi stack.** Dari halaman Home phpMyAdmin diketahui server memakai nginx, PHP, dan MariaDB. Pada konfigurasi ini, 502 berarti nginx tidak menerima respons valid dari proses PHP.
3. **Pemeriksaan database.** Pada daftar tabel di phpMyAdmin ditemukan tabel MyISAM **`[ISI_NAMA_TABEL]`** dengan status **"in use"**, yang menandakan tabel crash atau tidak tertutup dengan benar.
4. **Dugaan akar masalah.** Query PHP ke tabel yang crash macet, sehingga PHP tidak merespons dan nginx mengembalikan 502. *Catatan: ini kesimpulan dari gejala dan kondisi tabel; belum dikonfirmasi lewat log server.*

## Perbaikan

Mengikuti panduan perbaikan WordPress yang tersedia, langkah di phpMyAdmin:

1. Buka database situs dan cari tabel dengan status **"in use"**.
2. Centang tabel tersebut.
3. Pada menu **With selected**, pilih **Repair table**.
4. Muat ulang situs untuk memverifikasi.

Perbaikan ini memulihkan tabel tanpa menghapus datanya.

## Hasil

Status "in use" hilang dan situs kembali dapat diakses dengan normal, termasuk halaman beranda dan slider.

![Situs normal setelah perbaikan](images/02-sesudah-situs-normal.jpg)

## Pencegahan dan Rekomendasi

- Backup database berkala (export `.sql`) dan simpan di lokasi terpisah.
- Pertimbangkan konversi tabel dari MyISAM ke InnoDB, yang lebih tahan terhadap crash.
- Pantau uptime situs dan penggunaan resource server.
- Periksa kompatibilitas plugin dengan PHP 8.4 sebelum update.
- Batasi akses phpMyAdmin dan hindari memakai akun root untuk operasional harian.

## Yang Saya Pelajari

- Cara membaca halaman diagnostik Cloudflare untuk menentukan lapisan yang bermasalah.
- Alur request nginx, PHP, dan database, serta arti error 502 pada stack tersebut.
- Pemeliharaan tabel MyISAM di phpMyAdmin (status "in use", Repair table).
- Pentingnya backup sebelum mengubah apa pun pada database produksi.

## Referensi

- Panduan Repair table melalui phpMyAdmin (langkah: cari tabel "in use", With selected, Repair table).

