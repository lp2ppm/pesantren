# Data Pesantren Muhammadiyah

Aplikasi web untuk mengelola dan menampilkan data pesantren Muhammadiyah se-Indonesia, dikelola oleh **Lembaga Pengembangan Pesantren Pimpinan Pusat Muhammadiyah (LP2 PPM)**.

Pengunjung umum dapat melihat direktori, statistik, dan peta sebaran pesantren. Admin pesantren memperbarui data lembaganya sendiri. Operator LP2 PPM di tingkat pusat, wilayah, dan daerah memeriksa kelengkapan data, memverifikasi, dan memantau pesantren yang datanya sudah lama tidak diperbarui.

Alamat: **https://pesantren.muhammadiyah.or.id**

> Status: aktif dikembangkan, belum diluncurkan resmi. Backend versi 1.14.0.

## Fitur

**Untuk publik**
- Direktori pesantren dengan pencarian dan penyaringan (provinsi, kabupaten/kota, satuan pendidikan, jumlah santri, tahun berdiri).
- Halaman profil tiap pesantren.
- Statistik ("Pesantren dalam angka") dan peta sebaran nasional berdasarkan provinsi.
- Peta titik pondok per provinsi (OpenStreetMap). Provinsi yang dibuka tampil jelas dan provinsi lain diredupkan.

**Untuk admin pesantren**
- Mengisi dan memperbarui profil pesantren sendiri, termasuk mengunggah foto, dokumen aset, dan daftar prestasi.
- Isian dikunci menurut jenisnya (angka, tahun, telepon, email, tautan, koordinat) agar data lebih bersih.
- Draf otomatis tersimpan di browser bila koneksi terputus.

**Untuk operator LP2 PPM**
- Panel daftar pesantren dengan **verifikasi per pondok** dan **verifikasi massal** (memerlukan pernyataan pemeriksaan dan password ulang).
- **Pemantauan data**: kapan tiap pesantren terakhir memperbarui datanya, dikelompokkan menurut usia data, untuk menemukan data yang mungkin sudah usang.
- **Hubungi lewat WhatsApp** langsung ke mudir dan admin pondok, dengan pesan yang menyebut isian yang masih kosong.
- **Titik peta**: membaca koordinat dari tautan Google Maps yang diisi pesantren, termasuk tautan pendek, dan mengisi manual bila tidak terbaca.
- Pengelolaan akun (membuat, mereset, menonaktifkan, dan menghapus akun sesuai kewenangan), tinjauan data hasil migrasi, dan riwayat perubahan.

**Untuk super admin**
- Kelola isian (kolom formulir) dan pengaturan tampilan situs.
- **Token undangan**: pendaftaran admin pondok lewat token sekali pakai berbatas waktu. Calon admin mengisi sendiri, lalu super admin memeriksa dan menyetujui.
- **Mode pemeliharaan** saat tim memperbarui fitur. Akses pengguna lain ditolak di server.

## Peran pengguna

| Peran | Cakupan |
|---|---|
| Publik | Membaca data yang berstatus publik |
| Admin pesantren | Mengubah data pesantrennya sendiri |
| Operator daerah | Satu kabupaten/kota: melihat, memverifikasi, membuat akun admin pesantren |
| Operator wilayah | Satu provinsi: seperti operator daerah, untuk seluruh provinsi |
| Operator pusat | Seluruh Indonesia, termasuk membuat akun operator wilayah/daerah |
| Super admin | Semua hal di atas, ditambah pengaturan situs dan mode pemeliharaan |

Pembatasan cakupan dicek di server (Apps Script), bukan hanya disembunyikan di tampilan.

## Arsitektur

```
Browser (index.html, GitHub Pages)
        │  POST/GET JSON
        ▼
Google Apps Script (Web App)  ──►  Google Sheets (database)
        │                     └──►  Google Drive (foto dan berkas)
```

- **Frontend:** satu berkas `index.html` (HTML, CSS, JavaScript tanpa framework), dilayani lewat GitHub Pages.
- **Backend:** Google Apps Script yang dipasang pada spreadsheet database, `Execute as: Me`, `Who has access: Anyone`. Autentikasi memakai token bertanda tangan HMAC.
- **Database:** Google Sheets. Sheet utama: `pesantren`, `profil_teks`, `dokumen`, `users`, `log_perubahan`, `config`, `undangan`, `sampul`, `meta_kolom`, `ref_opsi`, `ref_provinsi`, `ref_wilayah`, `review_migrasi`, `peta_titik`.
- **Peta:** [Leaflet](https://leafletjs.com) dan ubin OpenStreetMap, dimuat saat halaman peta dibuka.

### Isi repositori

```
index.html     aplikasi (frontend)
batas/         batas provinsi (JSON, per provinsi) untuk peta titik pondok
warna.png      logo
CNAME          nama domain khusus (dibuat otomatis oleh GitHub Pages; jangan dihapus)
README.md      dokumen ini
```

Kode backend (`Code.gs`) dipasang langsung di Apps Script dan tidak disimpan di repositori ini.

## Pemasangan

1. **Database.** Buat Google Sheet kosong, lalu buka *Extensions > Apps Script*.
2. **Backend.** Tempel isi `Code.gs`. Sesuaikan `SUPER_ADMIN_EMAIL` pada bagian `CFG`.
3. **Setup awal.** Jalankan fungsi `setupAwal` dari editor dan setujui izin yang diminta. Catat password awal super admin yang ditampilkan. Password wajib diganti saat login pertama.
4. **Deploy backend.** *Deploy > New deployment > Web app*, `Execute as: Me`, `Who has access: Anyone`. Salin URL `/exec`.
5. **Frontend.** Isi `CONFIG.API_URL` di `index.html` dengan URL tersebut.
6. **Hosting.** Aktifkan GitHub Pages untuk repositori ini (*Settings > Pages*). Pastikan folder `batas/` ikut terunggah sejajar dengan `index.html`.
7. **Domain khusus.** Subdomain disediakan IT Pimpinan Pusat Muhammadiyah (rekam CNAME ke akun GitHub LP2 PPM, hanya DNS). Isi *Custom domain* di *Settings > Pages* dan aktifkan *Enforce HTTPS*. Bila aplikasi tidak dipakai lagi, beri tahu IT pusat agar CNAME dihapus.
8. **Cadangan.** Jalankan `pasangCadangan` untuk cadangan otomatis mingguan.
9. **Peta titik.** Setelah data masuk, buka *Panel > Titik peta* lalu klik **Proses tautan Maps**. Apps Script akan meminta izin koneksi ke layanan eksternal, setujui sekali.
10. **Kecepatan hash password.** Jalankan fungsi `ujiKecepatanHash` dari editor. Idealnya satu hash memakan 300-1500 ms; bila lebih dari 2500 ms turunkan `HASH_ITER` pada `CFG`.

Setiap perubahan `Code.gs` perlu di-deploy sebagai **New version** pada deployment yang sama agar URL tidak berubah. Deployment hanya bisa diperbarui oleh akun yang membuatnya, jadi lakukan dengan akun pemilik backend.

## Keamanan dan privasi

- Data yang disimpan adalah data lembaga (profil, jumlah santri dan guru berupa angka total, lokasi, kontak pengurus). Tidak ada data individu santri.
- Kolom bertanda publik saja yang dikirim server ke pengunjung. Kontak pengurus, data tanah, dan catatan internal hanya tampak setelah login sesuai peran.
- **Password.** Disimpan sebagai hash PBKDF2-HMAC-SHA256 (200.000 putaran) dengan garam unik per akun. Hash dari versi lama otomatis ditingkatkan saat pemiliknya login. Akun operator minimal 12 karakter, admin pondok minimal 10, harus memuat huruf dan angka, dan password yang mudah ditebak ditolak.
- **Login.** Percobaan salah dibatasi per akun. Perangkat yang pernah berhasil masuk tidak ikut terkunci bila orang lain sengaja salah memasukkan password akun itu. Lama respons login sama untuk akun yang ada maupun tidak.
- **Sesi.** Berlaku 8 jam dan disimpan hanya selama tab terbuka (bukan permanen di browser). Tombol *Keluar* mencabut semua sesi akun itu di server, begitu pula ganti dan reset password.
- **Pembatasan peran** dicek di server pada setiap aksi, bukan hanya disembunyikan di tampilan.
- **Browser.** Content-Security-Policy membatasi sambungan data hanya ke Apps Script dan skrip hanya dari halaman sendiri dan CDN Leaflet (dengan SRI). Halaman tidak mau ditampilkan di dalam bingkai situs lain (anti-clickjacking), kecuali situs `muhammadiyah.or.id`.
- **Berkas dan tautan.** Unggahan diperiksa isi aslinya, bukan hanya labelnya, dan dibatasi ukuran dan jumlahnya. Server hanya mau membuka tautan pendek Google Maps yang berasal dari domain Google sungguhan.
- Semua perubahan data tercatat (siapa, kapan, nilai lama dan baru). Tidak ada fitur hapus data pesantren.
- Klik kanan dan pintasan alat pengembang dinonaktifkan bagi pengguna biasa. Ini hanya penghalang ringan, perlindungan sebenarnya ada di server.
- Repositori ini tidak memuat rahasia. Kunci penandatangan dan pengaturan lain disimpan di *Script Properties* Apps Script.

Seluruh perlindungan di sisi server bergantung pada akun Google pemilik backend, jadi aktifkan verifikasi dua langkah di akun itu dan batasi siapa yang memegangnya. Aplikasi belum memiliki verifikasi dua langkah sendiri, dan belum ada audit keamanan oleh pihak independen. Bila menemukan kerentanan, mohon laporkan secara tertutup kepada pengelola, bukan lewat *issue* publik.

## Mode pemeliharaan

Super admin dapat mengaktifkannya di *Panel > Pemeliharaan*. Jika super admin tidak bisa masuk saat mode ini aktif, jalankan fungsi `matikanPemeliharaan` dari editor Apps Script.

## Sumber data dan atribusi

- Peta dasar © [OpenStreetMap](https://www.openstreetmap.org/copyright) contributors.
- Pustaka peta [Leaflet](https://leafletjs.com) (lisensi BSD-2-Clause).
- Batas provinsi pada folder `batas/` bersumber dari [geoBoundaries](https://www.geoboundaries.org) (CC BY 4.0; Runfola dkk., 2020, *PLOS ONE* 15(4): e0231866) dan telah disederhanakan. Provinsi hasil pemekaran Papua memakai [Indonesia GeoJSON & TopoJSON Maps (38 Provinces)](https://github.com/denyherianto/indonesia-geojson-topojson-maps-with-38-provinces) (CC BY 4.0). Garis ini hanya untuk memperjelas tampilan peta dan bukan batas administratif resmi.
- Huruf: Atkinson Hyperlegible (Google Fonts).

## Pengembang

Dikembangkan oleh Munawir Gani ([@daeng923](https://github.com/daeng923)) untuk LP2 PPM.

## Lisensi

Hak cipta © 2026 Lembaga Pengembangan Pesantren Pimpinan Pusat Muhammadiyah (LP2 PPM). Seluruh hak dilindungi.

Kode dan data dalam repositori ini tidak boleh disalin, diubah, atau disebarkan ulang tanpa izin tertulis dari LP2 PPM. Pustaka dan data pihak ketiga tetap tunduk pada lisensinya masing-masing (lihat bagian Sumber data dan atribusi).
