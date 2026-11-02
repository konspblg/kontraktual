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
- **Tautan Terkait** (kalau tersedia) — tombol **🔗 Buka Link Kontrak** dan **📍 Buka Link Sebaran Pekerjaan**, langsung membuka dokumen terkait di tab baru. Kalau tombolnya tidak muncul, berarti link untuk kontrak itu belum diisi di sumber data.
- **Progres Fisik Bulanan** dan **Progres Pembayaran per Termin** — dua bagian ini awalnya **tertutup**. Klik judulnya (tertulis **▼ Lihat**) untuk membuka, klik lagi (**▲ Tutup**) untuk menutup. Keduanya bisa dibuka bersamaan.
  - *Progres Fisik Bulanan*: perbandingan rencana (Plan) vs realisasi (Actual) progres fisik per bulan, Januari sampai Desember. Tabelnya bisa digeser ke samping.
  - *Progres Pembayaran per Termin*: rincian pembayaran tiap termin. Setiap termin punya 2 baris — **Plan** (biru) dan **Realisasi** (hijau) — berisi Fisik, Bayar (%), Jumlah (Rp), dan Bulan. Jumlah termin tiap kontrak bisa berbeda-beda. Kolom Fisik kadang berisi tulisan **RET** (retensi). Kalau kontrak belum punya data, tertulis "Belum ada data pembayaran per termin".

### D. Perlu Perhatian

Tiga kartu peringatan yang diletakkan di antara kotak **Detail Kontrak** dan tabel **Rencana vs Realisasi Bayar Bulan Ini**:

| Kartu | Artinya |
|---|---|
| **Kontrak: Hampir Expired** | Jumlah kontrak yang Status Masa Pelaksanaannya *Hampir Expired* |
| **Kontrak: Perlu Amandemen** | Jumlah kontrak yang Status Masa Pelaksanaannya *Perlu Amandemen* |
| **Jampel: Expired** | Jumlah kontrak yang jaminan pelaksanaannya (Jampel) sudah *Expired* |

- 🔴 **Kartu merah** = ada kontrak yang perlu ditindaklanjuti (angka lebih dari 0). **Klik kartunya** untuk melihat daftar kontraknya, lalu klik **"▼ Detail"** di baris yang diinginkan untuk rincian lengkap.
- 🟢 **Kartu hijau** = angka 0, tidak ada kontrak bermasalah.
- Angka dihitung dari **semua kontrak di semua tahun** — tidak terpengaruh pilihan dropdown Tahun di atas, supaya kontrak lama yang hampir kedaluwarsa tidak terlewat.

### E. Rencana vs Realisasi Bayar Bulan Ini

1. Pilih bulan dari dropdown.
2. Tabel menampilkan semua kontrak yang punya rencana/realisasi pembayaran di bulan itu — judul kontrak, penyedia, persentase rencana, nilai rencana (Rupiah), nilai realisasi (Rupiah) di bulan itu, dan status pembayaran:
   - 🟢 **Terbayar** — sudah ada realisasi pembayaran di bulan itu
   - 🟣 **VIP** — kontrak yang secara khusus ditandai prioritas/VIP di sumber data, meski realisasi pembayarannya masih nol
   - 🟠 **Belum Terbayar** — belum ada realisasi maupun tanda apapun
3. Kalau datanya lebih dari 10 baris, otomatis terbagi jadi beberapa halaman — pakai tombol **Sebelumnya** / **Berikutnya** di bawah tabel untuk pindah halaman.
4. Baris paling bawah (warna gelap) menampilkan **total keseluruhan** Rencana dan Realisasi (bukan cuma halaman yang sedang dilihat).

**📷 Tombol screenshot** (ikon kamera di pojok kanan atas tabel): klik untuk menyalin gambar tabel langsung ke clipboard perangkat. Setelah itu tinggal buka WhatsApp, Google Slides, atau Word, lalu tekan **Ctrl+V** (di HP: tekan lama lalu pilih "Tempel") — gambar tabel langsung tertempel, tidak perlu simpan file dulu.

### F. Rencana vs Realisasi Bayar (Tren Bulanan)

- 2 kartu ringkas: total rencana bayar setahun, dan total yang sudah terealisasi sampai saat ini (lengkap dengan persentase capaiannya).
- Grafik batang biru (Rencana) vs hijau (Realisasi) per bulan, Januari–Desember. Angka di atas tiap batang menunjukkan **jumlah kontrak** yang punya rencana/realisasi di bulan itu.
- Ada tombol screenshot juga di pojok kanan atas — sama cara pakainya seperti di atas.

### G. Rencana vs Realisasi Progres Fisik (Target Kinerja)

Card ini hanya menghitung kontrak yang secara khusus ditandai sebagai "Target Kinerja" oleh tim (via kolom Keterangan di sheet sumber). Semua angkanya adalah **rata-rata** dari kontrak-kontrak tersebut.

- **Dropdown Semester** — pilih *Semester 1 (Juni)* atau *Semester 2 (Desember)*. Pilihan awalnya otomatis menyesuaikan bulan sekarang: Januari–Juni = Semester 1, Juli–Desember = Semester 2. Tetap bisa diganti manual.
- **Kartu Rencana Progres Fisik** — rata-rata rencana (Plan) pada bulan Juni atau Desember, sesuai semester yang dipilih.
- **Kartu Realisasi Sampai Saat Ini** — rata-rata realisasi bulan ini, dengan keterangan *"xx.xx% dari rencana progres fisik"* di bawahnya. Kalau Semester 1 dipilih tetapi bulan sekarang sudah masuk Juli–Desember, yang ditampilkan adalah realisasi bulan **Juni**. Kalau data bulan ini belum diisi, tampil "–" dan tulisan "belum ada data bulan ini".
- **Grafik garis** — rencana (biru) vs realisasi (hijau putus-putus), Januari–Desember. Di setiap titik realisasi ada **angka persen** = realisasi dibagi rencana. Angka di atas titik berarti realisasi sudah mencapai atau melewati rencana; angka di bawah titik berarti masih di bawah rencana.
- **Tabel kecil di bawah grafik** — nilai Rencana dan Realisasi tiap bulan (Jan–Des). Bulan yang belum ada realisasinya tampil "–". Di HP, tabel bisa digeser ke samping.
- **📷 Tombol screenshot** di pojok kanan atas — menyalin gambar seluruh card (dropdown, kartu, grafik, dan tabel) ke clipboard, cara pakainya sama seperti di bagian lain.
- **Tombol "Lihat Target Kinerja"** — membuka jendela rincian:
  - Di pojok kanan atas ada **dropdown bulan** (Januari–Desember, awalnya bulan sekarang), tombol **📷 screenshot**, dan tombol ✕ untuk menutup.
  - Dua kartu ringkas: **Target** (rata-rata rencana bulan terpilih) dan **Realisasi** (rata-rata realisasi bulan terpilih, dengan keterangan *"xx.xx% dari target"* — hijau kalau sudah 100% atau lebih, oranye kalau di bawahnya).
  - Tabel kontrak: Plan %, Actual %, dan Selisih tiap kontrak untuk bulan yang dipilih. Mengganti bulan di dropdown otomatis mengubah judul, kartu, dan tabel.
  - Di tiap baris ada tombol **"▼ Detail"** untuk melihat rincian bulanan (Januari–Desember) kontrak itu.
  - Baris "RATA-RATA (dipakai di chart)" di bagian bawah tabel menunjukkan angka rata-rata persis seperti yang dipakai di grafik — berguna untuk mengecek.
  - Tombol screenshot di jendela ini menyalin **seluruh isi tabel** (bukan hanya bagian yang terlihat di layar) beserta judul dan bulannya.

### H. Kotak Status PIC, Penyedia, Masa Pelaksanaan, dan Jampel

Empat kotak di bagian bawah, masing-masing berisi dropdown. Pilih salah satu pilihan untuk melihat daftar kontrak yang cocok (jendela daftar akan terbuka; klik **"▼ Detail"** di baris mana saja untuk rincian lengkap).

| Kotak | Pilihan | Info tambahan di daftar |
|---|---|---|
| **Status PIC** | Nama PIC | — |
| **Status Penyedia** | Nama penyedia (vendor) | — |
| **Status Masa Pelaksanaan** | Aman, Perlu Amandemen, Hampir Expired | Akhir Kontrak dan Akhir Kontrak Amandemen |
| **Status Jampel** | Status jaminan pelaksanaan | Akhir Jaminan Pelaksanaan |

Pilihan yang kosong atau berisi error dari sumber data tidak ditampilkan. Kotak-kotak ini menampilkan semua tahun (tidak terpengaruh dropdown Tahun).

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
