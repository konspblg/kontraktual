# PROMPT LENGKAP: Dashboard Monitoring Kontrak AI — UPT Probolinggo

Gunakan prompt ini untuk meminta AI (Claude, ChatGPT, atau lainnya) membangun ulang dashboard ini dari nol. Ini adalah versi final setelah banyak revisi — sudah mencakup semua fitur yang berjalan di produksi.

---

## PROMPT SIAP PAKAI (copy dari sini ke bawah)

Saya butuh dashboard HTML satu halaman (single file, HTML+CSS+JS jadi satu) yang terhubung LANGSUNG (live) ke Google Sheets saya via metode gviz — bukan file statis, harus fetch data setiap kali dibuka.

### 1. Sumber Data

- Spreadsheet ID: `1OLvF-EFqSZZIwZRK4T3b8VyXLM53OW42GoCzbzasgjI`
- Sheet/tab: **MONITORING**, gid: `799877147`
- Sheet punya 3 baris header (group header, header bulan, baris total) sebelum data mulai. Beri tahu ini secara EKSPLISIT ke gviz lewat parameter URL `&headers=3` — JANGAN mengandalkan `.slice(3)` manual di JavaScript, karena gviz punya logika auto-deteksi baris header sendiri yang bisa salah tebak dan menggeser semua data kalau tidak diberi tahu eksplisit.

### 2. Mapping Kolom (index 0-based)

```js
const IDX = {
  unit:0, unitPelaksana:1, noKontrak:2, noPrk:3, tahun:4, judul:5, nilai:6,
  tglAwal:7, efektif:8, akhirKontrak:9, amandemenTerakhir:10, nilaiAmd:11, akhirAmd:12,
  telahTerbayarRp:13, telahTerbayarPct:14, belumTerbayarRp:15, belumTerbayarPct:16,
  status:18, bastI:19, durasiPemeliharaan:20, akhirPemeliharaan:21, bastII:22, bapBastII:23,
  bankGaransi:24, akhirJampel:25, statusJampel:26, nilaiJampel:27, penyedia:28, keterangan:29,
  planFisik:31, actualFisik:43, planBayarPct:55, actualBayarPct:67,
  planBayarRp:79, actualBayarRp:91, pic:103
};
```

Catatan penting:
- `planFisik`, `actualFisik`, `planBayarPct`, `actualBayarPct`, `planBayarRp`, `actualBayarRp` masing-masing adalah AWAL dari rentang 12 kolom berurutan (Januari–Desember). Baca sebagai array 12 nilai mulai dari index itu.
- Kolom `status` (index 18) berisi salah satu dari: `Selesai`, `Masa Pemeliharaan`, `Konstruksi`, `Non-Kontraktual`.
- Kolom `keterangan` (index 29) kadang berisi teks yang mengandung kata **"TARGET KINERJA"** — ini dipakai sebagai penanda kontrak mana yang masuk hitungan rata-rata di chart Target Kinerja (lihat bagian fitur di bawah).
- SEMUA angka persentase dan Rupiah di sheet sumbernya **kadang tersimpan sebagai teks** (bukan angka asli), karena user mengetik manual tanpa format cell yang benar. Parser HARUS robust: baca dari `cell.v` (value) atau `cell.f` (formatted string) apapun tipenya, buang karakter non-digit (`Rp`, `%`, spasi), lalu `parseFloat()`. Untuk persentase: kalau nilai numerik ≤ 1.5, asumsikan itu pecahan dan kalikan 100; kalau lebih besar, anggap sudah dalam bentuk persen utuh (misal 71.69 = 71.69%, bukan 7169%).

### 3. Fitur — daftar lengkap

**A. Login gate (client-side sederhana)**
- Modal layar penuh sebelum data di-fetch sama sekali, latar navy gradient, kartu putih di tengah.
- Label input: **"AKSES MASUK"** (bukan "Password").
- Password: `PLN123`, **case-sensitive** (pakai `===` di JS).
- Salah password → pesan error merah + kartu goyang (shake animation) + input dikosongkan lagi.
- Beri disclaimer ke user bahwa ini BUKAN keamanan sungguhan (password ada di source code, bisa dilihat siapa saja yang buka View Source) — cuma penyaring akses casual.

**B. Filter Tahun**
- Dropdown di atas, isi otomatis dari nilai unik kolom `tahun`, urut terbaru dulu.

**C. Status Kontrak (6 kartu, bisa DIKLIK)**
- Kartu: Jumlah Kontrak (N), Selesai, Masa Pemeliharaan, Konstruksi, Non-Kontraktual, Total Nilai Kontrak (SUM kolom `nilai`).
- Semua ikut ter-filter oleh Tahun yang dipilih.
- Setiap kartu **bisa diklik** → buka modal lebar berisi tabel semua kontrak dalam kategori itu (No Kontrak, Judul, Penyedia, Nilai, Status, tombol "Detail").
- Klik tombol "Detail" di baris manapun → BUKAN pindah modal, tapi buka baris accordion tambahan di bawahnya berisi kartu detail kontrak lengkap (lihat poin E).
- Warna kartu beda-beda: teal (jumlah), hijau (selesai), amber (pemeliharaan), biru (konstruksi), ungu (non-kontraktual), kuning (total nilai).

**D. Detail Kontrak — search box live-filter (BUKAN dropdown biasa)**
- Kotak ketik (bukan `<select>`), filter live tanpa perlu tekan Enter.
- Cari berdasarkan: judul kontrak, no kontrak, ATAU nama penyedia sekaligus.
- Hasil pencarian dikelompokkan per tahun (terbaru dulu), kata kunci di-highlight kuning di hasilnya.
- Dibatasi maksimal 40 hasil ditampilkan (kalau lebih, kasih pesan "persempit kata kunci").
- Klik salah satu hasil → buka modal kartu detail kontrak.

**E. Kartu Detail Kontrak — isinya (dipakai di 2 tempat: search box di atas, DAN accordion dari kartu status)**
Kelompokkan jadi beberapa bagian kecil (jangan satu list panjang):
- Identitas: No PRK, Tahun, Nilai Kontrak, Penyedia
- Tanggal dan Amandemen: Tgl Awal, Efektif, Akhir Kontrak, Amandemen Terakhir, Nilai Update Amd., Akhir Kontrak Amd.
- BAST dan Masa Pemeliharaan: Tgl BAST I, Durasi Pemeliharaan, Akhir Masa Pemeliharaan, Tgl BAST II, Tgl BAP BAST II
- Jaminan Pelaksanaan: Tgl Bank Garansi, Akhir Jaminan Pelaksanaan, Status Jampel (pill warna), Nilai Jaminan Pelaksanaan
- Keterangan (kalau ada isinya)
- **Di baris paling akhir**: tabel "Progres Fisik Bulanan — Plan vs Actual (%)" — 12 kolom bulan (Jan–Des), 2 baris (Plan, Actual), datanya dari `planFisik`/`actualFisik` milik kontrak itu sendiri (bukan cuma yang tag Target Kinerja — SEMUA kontrak punya data ini kalau tersedia).

**F. Rencana vs Realisasi Bayar Bulan Ini**
- Dropdown pilih bulan (Jan–Des).
- Tabel per kontrak (bukan card ringkasan): Judul Kontrak, Penyedia, Rencana %, Rencana Rp (rata kiri), **Status Bayar** (pill: hijau "Terbayar" kalau realisasi ada isinya, oranye "Belum Terbayar" kalau kosong — BUKAN kolom Realisasi Rp/% mentah).
- Kontrak yang rencana & realisasinya sama-sama kosong disembunyikan dari tabel.
- **Pagination 10 baris per halaman** (tombol Sebelumnya/Berikutnya), TIDAK BOLEH ada scroll — tabel memanjang ke bawah secara alami.
- Baris total di bawah (background navy): Total Rencana Rp + Total Realisasi Rp (dengan label kecil "Rencana"/"Realisasi" masing-masing, simetris), dihitung dari SEMUA kontrak bulan itu (bukan cuma halaman yang tampil).
- Tombol screenshot (ikon 📷 saja, TANPA teks label) di pojok kanan atas section — klik untuk salin gambar tabel ke clipboard (lihat poin I).

**G. Rencana vs Realisasi Bayar (Tren Bulanan)**
- 2 kartu ringkas di atas: "Rencana Bayar Tahunan" dan "Realisasi Sampai Saat Ini" (angka Rupiah PENUH, jangan disingkat "M"/miliar), plus teks kecil di bawah kartu realisasi: persentase realisasi terhadap rencana (misal "59,0% dari rencana bayar tahunan").
- Chart batang, Januari–Desember PENUH (12 bulan, jangan skip label sumbu X).
- Di atas tiap bar, tampilkan ANGKA JUMLAH KONTRAK (bukan Rupiah, bukan diberi teks "kontrak" — angka polos saja) yang punya rencana/realisasi bayar di bulan itu.
- Bulan yang belum ada realisasinya: tampil kosong/null (garis terputus atau bar kosong), JANGAN dipaksa jadi 0.
- Tombol screenshot ikon saja di pojok kanan atas section (capture seluruh section: 2 kartu + chart + legend jadi satu gambar).

**H. Rencana vs Realisasi Progres Fisik (Target Kinerja)**
- Filter kontrak yang kolom `keterangan` mengandung teks "TARGET KINERJA" (case-insensitive, normalisasi spasi berlebih).
- Chart garis, rata-rata Plan vs Actual per bulan, Januari–Desember penuh, dari kontrak-kontrak yang ke-filter itu saja.
- Tombol "Lihat Target Kinerja" di pojok kanan judul section → buka modal LEBAR berisi tabel: Judul Kontrak, Penyedia, Plan % (bulan berjalan), Actual % (bulan berjalan), Selisih (hijau kalau actual≥plan, oranye kalau kurang), dan kolom **"Detail Bulanan"** dengan tombol accordion per baris — klik untuk expand tabel mini 12-bulan (Plan vs Actual) khusus kontrak itu.
- Baris total di bawah tabel modal: RATA-RATA (dipakai di chart) — supaya user bisa verifikasi angka rata-rata di chart cocok dengan isi tabel.

**I. Screenshot ke Clipboard (BUKAN download file)**
- Pakai library `html2canvas`.
- Hasil capture **langsung disalin ke clipboard** (pakai `navigator.clipboard.write` dengan `ClipboardItem`), supaya user tinggal Ctrl+V paste ke WhatsApp/Google Slides/Docs — BUKAN download file otomatis (kalau browser tidak mendukung clipboard image, baru fallback ke download).
- PENTING — resolusi di HP: `windowWidth` milik html2canvas HANYA berlaku untuk konten HTML biasa, TIDAK berlaku untuk elemen `<canvas>` (chart Chart.js). Chart digambar oleh Chart.js dengan resolusi sesuai lebar layar SAAT dibuat — kalau dibuka dari HP sempit, resolusinya kecil sejak awal. Untuk screenshot chart yang tajam dari HP: sebelum capture, PAKSA container chart resize ke lebar besar (mis. 1100px) secara **di luar layar** (`position:fixed; left:-3000px;` supaya tidak kelihatan user), panggil `chartInstance.resize()` supaya Chart.js redraw ulang di resolusi besar, tunggu 2 `requestAnimationFrame`, baru capture, lalu kembalikan ukuran semula.
- Untuk tabel HTML biasa, `windowWidth:1200` pada html2canvas options cukup untuk memaksa render lebar desktop meski dibuka dari HP.

### 4. Desain / UI-UX

- Palet warna PLN: teal `#00A19A` (primary), navy `#1A3A52` (header/judul), kuning PLN `#FDB913` (aksen), background PUTIH bersih dengan border tegas di setiap sel tabel (jangan flat tanpa border).
- Halaman dibungkus sebagai "kertas mengambang": background body abu kebiruan (`#DCE3EA`), konten dalam kartu putih dengan `border-radius`, shadow, dan jarak dari tepi browser (bukan menempel penuh ke layar).
- Font utama: **Geist**, fallback ke **IBM Plex Sans**, lalu sans-serif sistem.
- Status pill berwarna pastel fungsional: abu (netral), biru (berjalan/konstruksi), amber (perhatian/pemeliharaan), hijau (selesai/terbayar), merah (bermasalah/expired), ungu (non-kontraktual).
- Data bar/progress bar dalam sel kalau relevan.
- Semua modal (overlay) pakai animasi fade + scale-in halus, ada tombol close (✕) yang jelas, dan bisa ditutup dengan klik area luar modal.
- Tabel dengan kolom lebar HARUS pakai persentase (`width:XX%`), JANGAN `min-width` piksel tetap — kalau ada teks panjang tanpa spasi (misal nomor kontrak `075.SPKPB/DAN.01.01/F34040000/2026`), pastikan CSS `overflow-wrap:break-word` aktif di semua `<td>`, supaya teks itu patah rapi mengikuti lebar kolom, BUKAN tumpah/numpuk ke kolom sebelah.

### 5. Wajib — Responsif/Fluid di HP

- Tambahkan `overflow-x:hidden` di `html` dan `body` sebagai jaring pengaman global supaya SELURUH HALAMAN tidak pernah bisa digeser horizontal, apapun penyebabnya.
- JANGAN ada elemen manapun (tabel, grid, kartu) dengan lebar piksel tetap yang bisa melebihi lebar layar HP — semua harus pakai persentase, `minmax()` yang wajar (≤160px), atau `max-width:100%`.
- Padding kontainer utama dikurangi khusus di breakpoint `max-width:600px` (dari ~40px jadi ~16px) supaya ruang konten lebih lega di layar sempit.
- Modal/overlay: `width:100%` dengan `max-width` tertentu (bukan lebar piksel tetap), padding menyesuaikan breakpoint mobile juga.

### 6. Teknis

- HTML satu file (CSS & JS di file yang sama).
- Fetch data pakai gviz JSONP: buat `<script>` tag dinamis dengan `src` mengarah ke
  `https://docs.google.com/spreadsheets/d/{SHEET_ID}/gviz/tq?gid={GID}&headers=3&tqx=out:json;responseHandler:{namaCallback}`
  — BUKAN `fetch()` biasa (supaya tidak kena masalah CORS).
- Sheet harus di-share "Siapa saja yang memiliki link dapat melihat".
- Tampilkan pesan loading saat data belum selesai diambil, dan pesan error jelas kalau gagal fetch.
- Library: Chart.js (chart batang & garis), chartjs-plugin-datalabels (angka di atas bar), html2canvas (screenshot).
- Tambahkan `console.info()` log debug di titik-titik penting (jumlah kontrak terbaca, jumlah kontrak Target Kinerja, dll) — sangat membantu kalau nanti ada bug data dan perlu di-diagnosis lewat DevTools Console.

---

## Konteks tambahan (opsional, sertakan kalau AI-nya butuh)

Ini bagian dari sistem monitoring kontrak pengadaan di PT PLN (Persero) UPT Probolinggo, unit UIT JBM (Jawa Timur-Bali). Struktur kolom sheet BISA BERUBAH sewaktu-waktu kalau user menambah kolom baru — kalau itu terjadi, minta user kirim ulang beberapa baris pertama sheet (atau screenshot Console log `[Dashboard] Contoh kontrak pertama`) untuk verifikasi ulang index sebelum mengubah IDX mapping.
