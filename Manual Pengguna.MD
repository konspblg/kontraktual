# Panduan Penggunaan — Dashboard Monitoring Kontrak AI
### UPT Probolinggo

---

## 1. Cara Masuk

1. Buka `https://konspblg.github.io/kontraktual/`
2. Muncul kotak **AKSES MASUK** — masukkan password yang sudah diberikan oleh admin.
3. Password bersifat **case-sensitive** — huruf besar/kecil harus persis sama.
4. Kalau salah, kotak akan bergetar dan muncul pesan error — coba lagi.

---

## 2. Bagian-bagian Dashboard

### A. Filter Tahun
Di paling atas ada dropdown **Tahun** — pilih tahun untuk memfilter data Status Kontrak. Kartu-kartu di bawahnya otomatis menyesuaikan.

### B. Status Kontrak (6 kartu)

| Kartu | Isi |
|---|---|
| **Jumlah Kontrak (N)** | Total kontrak di tahun terpilih |
| **Selesai** | Kontrak yang statusnya sudah Selesai |
| **Masa Pemeliharaan** | Kontrak dalam masa pemeliharaan |
| **Konstruksi** | Kontrak yang masih berjalan/konstruksi |
| **Non-Kontraktual** | Kontrak berstatus non-kontraktual |
| **Total Nilai Kontrak** | Jumlah nilai seluruh kontrak di tahun itu |

**💡 Semua kartu ini bisa DIKLIK.** Klik salah satu → muncul jendela (modal) berisi daftar lengkap kontrak dalam kategori itu, dengan tombol **"▼ Detail"** di kolom paling kanan tiap baris. Klik tombol itu untuk melihat rincian lengkap kontrak — akan terbuka di bawah baris itu juga (tidak pindah halaman), klik lagi untuk menutupnya.

### C. Detail Kontrak (kotak pencarian)

Ini **bukan dropdown biasa** — ketik langsung kata kunci (judul kontrak, nomor kontrak, atau nama vendor/penyedia), hasil pencarian langsung muncul sambil Bapak mengetik, **tidak perlu tekan Enter**.

Klik salah satu hasil pencarian untuk membuka kartu detail lengkap kontrak tersebut, berisi:
- Identitas kontrak (No PRK, Tahun, Nilai, Penyedia)
- Tanggal dan Amandemen
- BAST dan Masa Pemeliharaan
- Jaminan Pelaksanaan (termasuk status Jampel, apakah masih berlaku atau sudah kedaluwarsa)
- Keterangan (kalau ada)
- **Tabel Progres Fisik Bulanan** — perbandingan rencana (Plan) vs realisasi (Actual) progres fisik per bulan, Januari sampai Desember

### D. Rencana vs Realisasi Bayar Bulan Ini

1. Pilih bulan dari dropdown.
2. Tabel menampilkan semua kontrak yang punya rencana/realisasi pembayaran di bulan itu — judul kontrak, penyedia, persentase rencana, nilai rencana (Rupiah), dan status pembayaran (**Terbayar** / **Belum Terbayar**).
3. Kalau datanya lebih dari 10 baris, otomatis terbagi jadi beberapa halaman — pakai tombol **Sebelumnya** / **Berikutnya** di bawah tabel untuk pindah halaman.
4. Baris paling bawah (warna gelap) menampilkan **total keseluruhan** (bukan cuma halaman yang sedang dilihat).

**📷 Tombol screenshot** (ikon kamera di pojok kanan atas tabel): klik untuk menyalin gambar tabel langsung ke clipboard perangkat. Setelah itu tinggal buka WhatsApp, Google Slides, atau Word, lalu tekan **Ctrl+V** (di HP: tekan lama lalu pilih "Tempel") — gambar tabel langsung tertempel, tidak perlu simpan file dulu.

### E. Rencana vs Realisasi Bayar (Tren Bulanan)

- 2 kartu ringkas: total rencana bayar setahun, dan total yang sudah terealisasi sampai saat ini (lengkap dengan persentase capaiannya).
- Grafik batang biru (Rencana) vs hijau (Realisasi) per bulan, Januari–Desember. Angka di atas tiap batang menunjukkan **jumlah kontrak** yang punya rencana/realisasi di bulan itu.
- Ada tombol screenshot juga di pojok kanan atas — sama cara pakainya seperti di atas.

### F. Rencana vs Realisasi Progres Fisik (Target Kinerja)

- Grafik garis membandingkan rata-rata progres fisik yang direncanakan (biru) vs yang benar-benar tercapai (hijau), per bulan.
- Grafik ini hanya menghitung kontrak yang secara khusus ditandai sebagai "Target Kinerja" oleh tim (via kolom Keterangan di sheet sumber).
- Klik tombol **"Lihat Target Kinerja"** di pojok kanan atas untuk membuka tabel rinciannya — bisa dilihat kontrak mana saja yang masuk hitungan, berikut selisih Plan vs Actual masing-masing.
- Di tiap baris tabel itu ada tombol **"▼ Detail"** untuk melihat rincian bulanan (Januari–Desember) khusus kontrak tersebut.
- Baris "RATA-RATA (dipakai di chart)" di bagian bawah tabel menunjukkan angka persis yang ditampilkan di grafik — berguna untuk mengecek/memverifikasi.

### G. Status PIC & Status Penyedia

Dua kotak di bagian bawah — pilih nama PIC atau nama Penyedia (vendor) dari dropdown untuk melihat daftar kontrak yang ditangani orang/vendor tersebut.

---

## 3. Tips Penggunaan

- **Data selalu real-time** — begitu ada perubahan di Google Sheets sumber (oleh tim admin), tinggal refresh halaman dashboard, datanya langsung ikut update. Tidak perlu instal apapun atau minta file baru.
- **Buka dari HP juga bisa** — tampilan otomatis menyesuaikan layar kecil, dan fitur screenshot pun tetap menghasilkan gambar setajam versi PC.
- Kalau ada data yang terlihat aneh atau tidak sesuai (misalnya angka 0 semua atau kelewat besar), kemungkinan ada perubahan struktur kolom di sheet sumber — laporkan ke admin/tim IT untuk dicek.

---

## 4. Bantuan

Kalau menemukan kendala teknis (dashboard tidak bisa dibuka, data error, dll), hubungi tim yang mengelola dashboard ini dengan menyertakan:
1. Screenshot layar yang bermasalah
2. Screenshot tab **Console** di DevTools browser (tekan F12, klik tab "Console") — ini sangat membantu tim developer melacak masalahnya lebih cepat.
