# PROMPT LENGKAP: Dashboard Monitoring Kontrak AI — UPT Probolinggo

Gunakan prompt ini untuk meminta AI (Claude, ChatGPT, atau lainnya) membangun ulang dashboard ini dari nol. Ini adalah versi final setelah banyak revisi — sudah mencakup semua fitur yang berjalan di produksi.

---

## PROMPT SIAP PAKAI (copy dari sini ke bawah)

Saya butuh dashboard HTML satu halaman (single file, HTML+CSS+JS jadi satu) yang terhubung LANGSUNG (live) ke Google Sheets saya via metode gviz — bukan file statis, harus fetch data setiap kali dibuka.

### 1. Sumber Data

Dashboard fetch **3 sheet berbeda** dari spreadsheet yang sama, secara paralel:

**Sheet utama — MONITORING**
- Spreadsheet ID: `1OLvF-EFqSZZIwZRK4T3b8VyXLM53OW42GoCzbzasgjI`
- Sheet/tab: **MONITORING**, gid: `799877147`
- Sheet punya 3 baris header (group header, header bulan, baris total) sebelum data mulai. Beri tahu ini secara EKSPLISIT ke gviz lewat parameter URL `&headers=3` — JANGAN mengandalkan `.slice(3)` manual di JavaScript, karena gviz punya logika auto-deteksi baris header sendiri yang bisa salah tebak dan menggeser semua data kalau tidak diberi tahu eksplisit.

**Sheet kedua (pelengkap) — DETIL_KONTRAK**
- Sheet/tab: **DETIL_KONTRAK**, gid: `798316676`, di spreadsheet ID yang SAMA dengan di atas.
- Cuma punya **1 baris header** (beda dari MONITORING) — panggil dengan `&headers=1`.
- Fungsinya cuma untuk 2 kolom: **Link Kontrak** dan **Link Sebaran Pekerjaan** — dicocokkan ke sheet MONITORING lewat Nomor Kontrak (kolom C, sama persis di kedua sheet).
- Sifatnya PELENGKAP — kalau gagal fetch, JANGAN sampai bikin seluruh dashboard error. Tangkap error-nya secara terpisah, dan dashboard utama tetap jalan tanpa tombol link.
- Fetch sheet ini PARALEL dengan sheet utama (pakai `Promise.all`), bukan berurutan, supaya tidak menambah waktu loading.

**Sheet ketiga (pelengkap) — rincian pembayaran per termin (disebut sheet PEMBAYARAN)**
- Tab dengan gid `2042709256`, di spreadsheet ID yang SAMA.
- Punya **3 baris header** (judul kolom, baris kosong, label PLAN/REALISASI) — panggil dengan `&headers=3`.
- Strukturnya **per termin**, bukan per bulan: SATU kontrak bisa punya BEBERAPA baris (1 baris = 1 termin), jumlah termin tiap kontrak berbeda-beda. Antar kontrak dipisah baris kosong, dan ada baris divider tahun (mis. "2022") di tengah data — semua baris yang kolom C-nya (Nomor Kontrak) kosong HARUS dilewati.
- Dicocokkan ke sheet MONITORING lewat Nomor Kontrak (kolom C). Data dikelompokkan jadi `Map` (key: Nomor Kontrak yang di-trim, value: array baris termin).
- Sifatnya PELENGKAP seperti DETIL_KONTRAK: fetch paralel, try/catch sendiri, kalau gagal dashboard utama tetap jalan (tabel termin cukup menampilkan "Belum ada data pembayaran per termin").

### 2. Mapping Kolom (index 0-based)

**MONITORING:**
```js
const IDX = {
  unit:0, unitPelaksana:1, noKontrak:2, noPrk:3, tahun:4, judul:5, nilai:6,
  tglAwal:7, efektif:8, akhirKontrak:9, amandemenTerakhir:10, nilaiAmd:11, akhirAmd:12,
  statusMasaPelaksanaan:13, telahTerbayarPct:14, belumTerbayarRp:15, belumTerbayarPct:16,
  status:18, bastI:19, durasiPemeliharaan:20, akhirPemeliharaan:21, bastII:22, bapBastII:23,
  bankGaransi:24, akhirJampel:25, statusJampel:26, nilaiJampel:27, penyedia:28, keterangan:29,
  planFisik:31, actualFisik:43, planBayarPct:55, actualBayarPct:67,
  planBayarRp:79, actualBayarRp:91, pic:103
};
```

**DETIL_KONTRAK:**
```js
const DETIL_IDX = { noKontrak: 2, linkKontrak: 35, linkSebaran: 36 };
```

**PEMBAYARAN (kolom C = kunci; I–M = Plan; N–R = Realisasi; kolom S STATUS sengaja TIDAK dipakai):**
```js
const PEMBAYARAN_IDX = {
  noKontrak: 2,
  terminPlan: 8, fisikPlan: 9, bayarPlan: 10, jumlahPlan: 11, bulanPlan: 12,
  terminReal: 13, fisikReal: 14, bayarReal: 15, jumlahReal: 16, bulanReal: 17
};
```
Nomor termin diambil dari `terminPlan` (kolom I). Kolom FISIK kadang berisi teks seperti `RET` (retensi) — baca apa adanya sebagai TEKS, jangan di-parse jadi angka. Kolom Bayar berupa persentase, Jumlah berupa Rupiah, Bulan berupa teks (mis. "October 2024").

Catatan penting:
- `planFisik`, `actualFisik`, `planBayarPct`, `actualBayarPct`, `planBayarRp`, `actualBayarRp` masing-masing adalah AWAL dari rentang 12 kolom berurutan (Januari–Desember). Baca sebagai array 12 nilai mulai dari index itu.
- Kolom `status` (index 18) berisi salah satu dari: `Selesai`, `Masa Pemeliharaan`, `Konstruksi`, `Non-Kontraktual`.
- Kolom `keterangan` (index 29) kadang berisi teks yang mengandung kata **"TARGET KINERJA"** — ini dipakai sebagai penanda kontrak mana yang masuk hitungan rata-rata di chart Target Kinerja (lihat bagian fitur di bawah).
- Kolom `statusMasaPelaksanaan` (index 13 = kolom N) berisi hasil formula: `AMAN`, `PERLU AMANDEMEN`, `HAMPIR EXPIRED`, dan kadang error spreadsheet `#VALUE!`. Teks yang diawali `#` (error spreadsheet) HARUS diperlakukan sebagai kosong/placeholder.
- Kolom `statusJampel` (index 26 = kolom AA) berisi status jaminan pelaksanaan; nilai yang dianggap kedaluwarsa adalah teks persis `EXPIRED`. Kolom `akhirJampel` (index 25 = kolom Z) = tanggal akhir jaminan pelaksanaan.
- Index 14–16 (`telahTerbayarPct`, `belumTerbayarRp`, `belumTerbayarPct`) tidak dipakai fitur manapun — boleh diabaikan.
- SEMUA angka persentase dan Rupiah di sheet sumbernya **kadang tersimpan sebagai teks** (bukan angka asli), karena user mengetik manual tanpa format cell yang benar. Parser HARUS robust: baca dari `cell.v` (value) atau `cell.f` (formatted string) apapun tipenya, buang karakter non-digit (`Rp`, `%`, spasi), lalu `parseFloat()`. Untuk persentase: kalau nilai numerik ≤ 1.5, asumsikan itu pecahan dan kalikan 100; kalau lebih besar, anggap sudah dalam bentuk persen utuh (misal 71.69 = 71.69%, bukan 7169%).
- Banyak kolom di MONITORING adalah formula dengan pola `=IF(kondisi,nilai,"")` yang merujuk ke sheet lain. Kalau kondisinya tidak cocok, hasilnya string kosong `""` (dibaca sebagai null/dash oleh dashboard). Kalau kondisinya cocok TAPI nilai rujukannya kebetulan 0, hasilnya angka 0 ASLI (beda dari string kosong!) — ditampilkan sebagai "Rp0.00" kalau format selnya Keuangan. Perbedaan "kosong beneran" vs "angka nol asli" ini dipakai sebagai sinyal untuk status VIP (lihat poin F di bagian Fitur).

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
- **Tautan Terkait** (kalau ada di data): tombol "🔗 Buka Link Kontrak" dan "📍 Buka Link Sebaran Pekerjaan", masing-masing `target="_blank"`, diambil dari sheet DETIL_KONTRAK yang dicocokkan lewat Nomor Kontrak. Tombol yang link-nya kosong/`-`/`N/A` JANGAN ditampilkan sama sekali (bukan ditampilkan sebagai link mati).
- **Di bagian paling akhir, dua bagian COLLAPSIBLE** (default TERTUTUP; lihat poin M): (1) "Progres Fisik Bulanan — Plan vs Actual (%)" — 12 kolom bulan (Jan–Des), 2 baris (Plan, Actual), datanya dari `planFisik`/`actualFisik` milik kontrak itu sendiri (bukan cuma yang tag Target Kinerja — SEMUA kontrak punya data ini kalau tersedia); (2) "Progres Pembayaran per Termin — Plan vs Realisasi" (poin M).

**F. Rencana vs Realisasi Bayar Bulan Ini**
- Dropdown pilih bulan (Jan–Des).
- Tabel per kontrak (bukan card ringkasan): Judul Kontrak, Penyedia, Rencana %, Rencana Rp (rata kiri), **Realisasi Rp** (nilai mentah `actualBayarRp` bulan itu, rata kiri, diletakkan di antara Rencana Rp dan Status Bayar), **Status Bayar** — pill dengan 3 kemungkinan, SEMUA ditentukan HANYA dari kolom Actual Rp (jangan campur dengan kolom Actual %, karena keduanya bisa punya formula sumber berbeda dan tidak selalu sinkron):
  - **Terbayar** (hijau) — Actual Rp berisi angka > Rp1
  - **VIP** (ungu) — Actual Rp berisi angka 0 yang ASLI (bukan string kosong) — beda dari "belum ada data". Pakai toleransi kecil (`Math.abs(v) < 1`) untuk jaga-jaga sisa desimal super kecil dari formula spreadsheet.
  - **Belum Terbayar** (oranye) — Actual Rp kosong/null (dash `-`)
- Kontrak yang rencana & realisasi & tanda VIP-nya sama-sama kosong disembunyikan dari tabel; kontrak yang ditandai VIP tetap tampil walau rencananya kosong.
- **Pagination 10 baris per halaman** (tombol Sebelumnya/Berikutnya), TIDAK BOLEH ada scroll — tabel memanjang ke bawah secara alami.
- Baris total di bawah (background navy): Total Rencana Rp + Total Realisasi Rp (dengan label kecil "Rencana"/"Realisasi" masing-masing, simetris), dihitung dari SEMUA kontrak bulan itu (bukan cuma halaman yang tampil). Total Realisasi ditaruh sejajar di bawah kolom Realisasi Rp; sel kolom Status Bayar di baris total dikosongkan.
- Tombol screenshot (ikon 📷 saja, TANPA teks label) di pojok kanan atas section — klik untuk salin gambar tabel ke clipboard (lihat poin I).

**G. Rencana vs Realisasi Bayar (Tren Bulanan)**
- 2 kartu ringkas di atas: "Rencana Bayar Tahunan" dan "Realisasi Sampai Saat Ini" (angka Rupiah PENUH, jangan disingkat "M"/miliar), plus teks kecil di bawah kartu realisasi: persentase realisasi terhadap rencana (misal "59,0% dari rencana bayar tahunan").
- Chart batang, Januari–Desember PENUH (12 bulan, jangan skip label sumbu X).
- Di atas tiap bar, tampilkan ANGKA JUMLAH KONTRAK (bukan Rupiah, bukan diberi teks "kontrak" — angka polos saja) yang punya rencana/realisasi bayar di bulan itu.
- Bulan yang belum ada realisasinya: tampil kosong/null (garis terputus atau bar kosong), JANGAN dipaksa jadi 0.
- Tombol screenshot ikon saja di pojok kanan atas section (capture seluruh section: 2 kartu + chart + legend jadi satu gambar).

**H. Rencana vs Realisasi Progres Fisik (Target Kinerja)**
- Filter kontrak yang kolom `keterangan` mengandung teks "TARGET KINERJA" (case-insensitive, normalisasi spasi berlebih). SEMUA angka di bagian ini adalah RATA-RATA per bulan dari kontrak-kontrak yang ke-filter itu saja (array `planFisikAvg` dan `actualFisikAvg`, masing-masing 12 nilai; `null` kalau bulan itu belum ada data).
- Urutan isi card dari atas ke bawah: judul (di kanan: tombol "Lihat Target Kinerja" + tombol screenshot ikon 📷) → dropdown semester → 2 kartu KPI → chart garis → legend → tabel ringkas.
- **Dropdown semester**: "Semester 1 (Juni)" dan "Semester 2 (Desember)". Pilihan awal OTOMATIS mengikuti bulan sekarang (Jan–Jun = Semester 1, Jul–Des = Semester 2), tetap bisa diganti manual.
- **Kartu "Rencana Progres Fisik"**: rata-rata plan bulan Juni (index bulan 5) atau Desember (index 11) sesuai dropdown.
- **Kartu "Realisasi Sampai Saat Ini"**: rata-rata actual bulan ini. PENGECUALIAN: kalau Semester 1 yang dipilih dan bulan sekarang sudah Juli–Desember, pakai rata-rata actual bulan **Juni**. Baris kecil di bawah angka: "xx.xx% dari rencana progres fisik" = realisasi ÷ rencana × 100 (2 desimal). Kalau actual belum terisi: tampil "–" dan teks "belum ada data bulan ini".
- **Chart garis**, Januari–Desember penuh: Plan (biru solid) vs Actual (hijau putus-putus, `spanGaps:false` supaya bulan tanpa data tidak ditarik garis). Di setiap titik Actual tampil label persen = actual ÷ plan × 100 (dibulatkan, tanpa desimal), pakai chartjs-plugin-datalabels yang dipasang PER-CHART (`plugins:[ChartDataLabels]`; dataset Plan diberi `datalabels:{display:false}`). Label di ATAS titik kalau actual ≥ plan, di BAWAH titik kalau actual < plan (supaya tidak menimpa garis Plan). Bulan tanpa actual tidak diberi label. Beri `layout.padding` atas/bawah secukupnya supaya label tidak terpotong.
- **Tabel ringkas** di bawah legend: font kecil (±11px), bulan sebagai KOLOM (Jan–Des), 2 baris (Rencana berwarna biru, Realisasi berwarna hijau), bulan tanpa data tampil "–", dibungkus container `overflow-x:auto` supaya bisa digeser di HP.
- **Tombol screenshot 📷** (di sebelah tombol "Lihat Target Kinerja") menangkap SELURUH isi card: dropdown, kartu, chart, legend, tabel ringkas.
- **Tombol "Lihat Target Kinerja"** → modal LEBAR:
  - Header: judul "Target Kinerja — [Nama Bulan]" di kiri; di kanan berurutan: dropdown bulan (Januari–Desember, default bulan sekarang), tombol screenshot 📷, tombol ✕. TANPA subjudul/keterangan jumlah kontrak.
  - Dua kartu KPI yang RINGKAS (padding tipis, angka kecil, supaya tabel punya ruang tinggi yang cukup): **Target** = rata-rata plan bulan terpilih (tanpa teks tambahan); **Realisasi** = rata-rata actual bulan terpilih + "xx.xx% dari target" (= realisasi ÷ target × 100, hijau kalau ≥ 100%, oranye kalau kurang). Kalau actual belum terisi: "–" dan "belum ada data bulan ini" (untuk bulan berjalan) atau "belum ada data" (bulan lain).
  - Tabel: Judul Kontrak, Penyedia, Plan %, Actual %, Selisih (hijau kalau actual ≥ plan, oranye kalau kurang), dan kolom **"Detail Bulanan"** dengan tombol accordion per baris — klik untuk expand tabel mini 12-bulan (Plan vs Actual) kontrak itu. Semua nilai ikut BULAN YANG DIPILIH di dropdown (bukan hanya bulan berjalan); mengganti bulan menggantikan judul, kartu, dan tabel sekaligus.
  - Baris total di bawah tabel: RATA-RATA (dipakai di chart) — supaya user bisa verifikasi angka rata-rata cocok dengan isi tabel.
  - Tombol screenshot di modal menangkap SELURUH isi tabel (bukan cuma bagian yang terlihat di layar), plus judul "Target Kinerja — [bulan]" yang di layar normal disembunyikan dan hanya ditampilkan saat capture.

**I. Screenshot ke Clipboard (BUKAN download file)**
- Pakai library `html2canvas`.
- Hasil capture **langsung disalin ke clipboard** (pakai `navigator.clipboard.write` dengan `ClipboardItem`), supaya user tinggal Ctrl+V paste ke WhatsApp/Google Slides/Docs — BUKAN download file otomatis (kalau browser tidak mendukung clipboard image, baru fallback ke download).
- PENTING — resolusi di HP: `windowWidth` milik html2canvas HANYA berlaku untuk konten HTML biasa, TIDAK berlaku untuk elemen `<canvas>` (chart Chart.js). Chart digambar oleh Chart.js dengan resolusi sesuai lebar layar SAAT dibuat — kalau dibuka dari HP sempit, resolusinya kecil sejak awal. Untuk screenshot chart yang tajam dari HP: sebelum capture, PAKSA container chart resize ke lebar besar (mis. 1100px) secara **di luar layar** (`position:fixed; left:-3000px;` supaya tidak kelihatan user), panggil `chartInstance.resize()` supaya Chart.js redraw ulang di resolusi besar, tunggu 2 `requestAnimationFrame`, baru capture, lalu kembalikan ukuran semula.
- Untuk tabel HTML biasa, `windowWidth:1200` pada html2canvas options cukup untuk memaksa render lebar desktop meski dibuka dari HP.

- Fungsi screenshot dibuat GENERIK: menerima id elemen yang di-capture, id tombol, dan objek hook opsional `{prepare, restore}` untuk menyiapkan/memulihkan tampilan sebelum dan sesudah capture. Dipakai di 4 tempat: tabel bayar bulan ini, chart tren bayar, card Target Kinerja, dan modal Target Kinerja. Untuk chart, hook-nya dibuat lewat satu fungsi pembuat (`makeChartScreenshotHooks(containerId, getChartInstance)`) yang menerapkan trik resize di luar layar di atas ke container mana pun. Untuk modal Target Kinerja, hook sementara membuka tabel penuh (menghapus `max-height` dan `overflow` dari container scroll-nya, karena html2canvas hanya menangkap bagian yang terlihat) dan menampilkan judul tersembunyi.

**J. Integrasi Sheet Pelengkap (DETIL_KONTRAK dan PEMBAYARAN)**
- Fetch sheet MONITORING (wajib berhasil) serta DETIL_KONTRAK dan PEMBAYARAN (keduanya opsional/pelengkap) secara PARALEL pakai `Promise.all`, bukan berurutan.
- Bungkus fetch tiap sheet pelengkap (DETIL_KONTRAK dan PEMBAYARAN) dengan try/catch sendiri — kalau gagal, JANGAN throw ke luar (jangan bikin dashboard utama ikut gagal), cukup log warning dan lanjut dengan peta link kosong.
- Hasil fetch DETIL_KONTRAK dan PEMBAYARAN masing-masing disimpan sebagai `Map` (key: Nomor Kontrak yang sudah di-trim, value: `{linkKontrak, linkSebaran}`), supaya pencarian saat render kartu detail kontrak cepat (`O(1)` lookup, bukan looping ulang tiap kali).

**K. Section "Perlu Perhatian" (3 kartu warning)**
- Diletakkan di antara kotak "Detail Kontrak" dan section "Rencana vs Realisasi Bayar Bulan Ini". Judul section "Perlu perhatian" dengan titik penanda berwarna merah.
- 3 kartu dalam grid yang sama dengan kartu Status Kontrak (ukuran seragam, mengikuti lebar layar). Label kartu memakai prefix supaya TIDAK ambigu:
  - "KONTRAK: HAMPIR EXPIRED" → jumlah kontrak dengan `statusMasaPelaksanaan === 'HAMPIR EXPIRED'`
  - "KONTRAK: PERLU AMANDEMEN" → jumlah kontrak dengan `statusMasaPelaksanaan === 'PERLU AMANDEMEN'`
  - "JAMPEL: EXPIRED" → jumlah kontrak dengan `statusJampel === 'EXPIRED'`
  (pencocokan teks PERSIS, huruf besar semua)
- Warna per kartu, ditentukan sendiri-sendiri: angka > 0 → merah transparan (pakai warna pill merah) dan bisa DIKLIK untuk membuka modal daftar kontrak yang cocok; angka 0 → hijau transparan dan tidak bisa diklik.
- Dihitung dari SEMUA kontrak di SEMUA tahun — SENGAJA tidak ikut filter Tahun (kontrak yang hampir kedaluwarsa relevan terlepas tahun kontraknya). Modal daftar yang terbuka juga tidak memfilter tahun.
- Fungsi modal daftar kontrak dibuat generik: menerima nilai filter, judul, nama field yang difilter (`status`, `statusMasaPelaksanaan`, atau `statusJampel`), dan flag `ignoreYear`; pill status di tabelnya mengikuti field yang difilter.

**L. Kotak Status PIC, Penyedia, Status Masa Pelaksanaan, Status Jampel**
- 4 kartu putih berisi satu dropdown masing-masing, di bagian bawah halaman (grid auto-fit, lebar minimal ±220px).
- Isi dropdown = nilai unik field masing-masing dari SEMUA kontrak, diurutkan abjad. Nilai placeholder dibuang: kosong, `-`, `N/A`, dan error spreadsheet (teks diawali `#`, mis. `#VALUE!`).
- Field: `pic` (kolom index 103), `penyedia` (28), `statusMasaPelaksanaan` (kolom N, index 13; nilai AMAN / PERLU AMANDEMEN / HAMPIR EXPIRED), `statusJampel` (kolom AA, index 26).
- Memilih nilai → buka modal daftar kontrak yang cocok (judul modal memuat nilai terpilih), lalu dropdown kembali ke posisi awal ("Pilih ...").
- Baris tambahan di bawah tiap kontrak dalam modal (fungsi daftar menerima array `extraFields` berisi `{label, key}`): untuk Status Masa Pelaksanaan → "Akhir Kontrak" (`akhirKontrak`, kolom J) dan "Akhir Kontrak Amandemen" (`akhirAmd`, kolom M); untuk Status Jampel → "Akhir Jaminan Pelaksanaan" (`akhirJampel`, kolom Z); PIC dan Penyedia tanpa baris tambahan.

**M. Progres Pembayaran per Termin + bagian collapsible di kartu detail kontrak**
- Di kartu detail kontrak (poin E), setelah "Progres Fisik Bulanan", ada bagian "Progres pembayaran per termin — Plan vs Realisasi" dengan data dari sheet PEMBAYARAN, dicari lewat Nomor Kontrak.
- Tabel: kolom Termin (digabung `rowspan=2` per termin), kolom label baris, Fisik, Bayar %, Jumlah (Rp), Bulan. Tiap termin tampil 2 baris: **Plan** (latar biru muda, label biru) dan **Realisasi** (latar hijau muda, label hijau). TANPA kolom status. Nilai kosong tampil "–". Kolom Fisik boleh berisi teks seperti "RET". Kalau kontrak tidak punya data termin, tampil teks "Belum ada data pembayaran per termin."
- Dua bagian — "Progres fisik bulanan" dan "Progres pembayaran per termin" — default TERTUTUP. Judul tiap bagian adalah baris yang bisa diklik (tampilan seperti bilah tipis dengan latar abu muda), dengan indikator "▼ Lihat" saat tertutup dan "▲ Tutup" saat terbuka. Keduanya INDEPENDEN (bisa dibuka salah satu atau bersamaan).
- ID elemen yang di-toggle HARUS unik per kontrak (turunkan dari Nomor Kontrak dengan mengganti karakter non-alfanumerik jadi `_`) karena beberapa kartu detail bisa terbuka bersamaan di dalam satu modal daftar (accordion).

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
- Tambahkan `console.info()` log debug di titik-titik penting (jumlah kontrak terbaca, jumlah kontrak Target Kinerja, jumlah kontrak dengan link dari DETIL_KONTRAK, jumlah kontrak dengan data termin dari PEMBAYARAN, dll) — sangat membantu kalau nanti ada bug data dan perlu di-diagnosis lewat DevTools Console.

---

## Konteks tambahan (opsional, sertakan kalau AI-nya butuh)

Ini bagian dari sistem monitoring kontrak pengadaan di PT PLN (Persero) UPT Probolinggo, unit UIT JBM (Jawa Timur-Bali). Struktur kolom sheet BISA BERUBAH sewaktu-waktu kalau user menambah kolom baru — kalau itu terjadi, minta user kirim ulang beberapa baris pertama sheet (atau screenshot Console log `[Dashboard] Contoh kontrak pertama`) untuk verifikasi ulang index sebelum mengubah IDX mapping.
