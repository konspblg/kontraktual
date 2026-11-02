# Dashboard Monitoring Kontrak AI — UPT Probolinggo

Dashboard satu halaman (`index.html`) yang menampilkan data kontrak AI (Anggaran Investasi) UPT Probolinggo secara **live** langsung dari Google Sheets. Tidak perlu proses build/compile — tinggal buka file HTML-nya di browser atau host lewat GitHub Pages.

🔗 Live: `https://konspblg.github.io/kontraktual/`

---

## Cara Kerja Singkat

Dashboard ini **tidak punya database sendiri**. Setiap kali dibuka, dia mengambil data langsung dari **3 sheet** di Google Sheets memakai teknik `gviz` (Google Visualization API):

1. **MONITORING** (gid `799877147`) — data utama: identitas kontrak, tanggal, BAST, jaminan pelaksanaan, progres fisik & bayar bulanan, status, PIC.
2. **DETIL_KONTRAK** (gid `798316676`) — sheet pelengkap, cuma dipakai untuk 2 hal: **Link Kontrak** dan **Link Sebaran Pekerjaan**, dicocokkan ke sheet MONITORING lewat Nomor Kontrak (kolom C di kedua sheet). Kalau sheet ini gagal dimuat, dashboard utama tetap jalan normal — cuma tombol link-nya saja yang tidak muncul.
3. **Sheet rincian pembayaran per termin** (gid `2042709256`, di dokumen ini disebut sheet **PEMBAYARAN**) — sheet pelengkap, dipakai untuk tabel **Progres Pembayaran per Termin** di kartu detail kontrak. Dicocokkan ke sheet MONITORING lewat Nomor Kontrak (kolom C). Kalau gagal dimuat, dashboard utama tetap jalan — cuma tabel termin yang tampil "Belum ada data pembayaran per termin".

Jadi:

- Update data di Google Sheets → refresh halaman dashboard → data ikut update. Tidak perlu upload ulang file apapun.
- Kalau **struktur kolom** sheet berubah (nambah/hapus kolom) → dashboard bisa salah baca data (nyasar ke kolom lain) → perlu update `IDX` di kode (lihat bagian bawah).

---

## Kapan Perlu Ubah Kode?

### ✅ TIDAK perlu ubah kode kalau:
- Menambah baris kontrak baru
- Mengubah isi data (nilai, tanggal, status, dll) di baris yang sudah ada
- Menambah data bulanan (plan/actual fisik & bayar)

Semua itu otomatis kebaca dashboard tanpa sentuh kode sama sekali.

### ⚠️ PERLU ubah kode kalau:
- **Menambah kolom baru di tengah-tengah** struktur sheet yang sudah ada (kolom lama jadi geser)
- **Menghapus kolom**
- **Mengganti nama tab/sheet** (dari "MONITORING" ke nama lain) atau bikin sheet baru dengan gid berbeda
- Mengganti Spreadsheet (bikin file Google Sheets baru dari nol)
- **Mengubah ejaan isi status** di kolom N (Status Masa Pelaksanaan) atau kolom AA (Status Jampel), misalnya `HAMPIR EXPIRED` jadi `SEGERA EXPIRED` — kartu "Perlu Perhatian" mencocokkan teks persis, jadi string di fungsi `renderPerhatianCards` ikut diubah

---

## Struktur Kolom Saat Ini (`IDX`)

Cari bagian ini di `index.html` (pakai Ctrl+F, cari `const IDX`):

```js
const SHEET_ID = '1OLvF-EFqSZZIwZRK4T3b8VyXLM53OW42GoCzbzasgjI';
const GID = '799877147';

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

> **Catatan penting soal beberapa field:**
> - `statusMasaPelaksanaan` (index 13 = kolom **N**) berisi hasil formula: `AMAN`, `PERLU AMANDEMEN`, `HAMPIR EXPIRED`, kadang muncul error `#VALUE!` (otomatis diabaikan dashboard, lihat bagian "Status Masa Pelaksanaan..." di bawah).
> - `statusJampel` (index 26 = kolom **AA**) berisi status jaminan pelaksanaan; yang dihitung sebagai kedaluwarsa adalah teks persis `EXPIRED`.
> - `akhirJampel` (index 25 = kolom **Z**), `akhirKontrak` (index 9 = kolom **J**), dan `akhirAmd` (index 12 = kolom **M**) dipakai sebagai baris tambahan di modal daftar kontrak.
> - `telahTerbayarPct`, `belumTerbayarRp`, `belumTerbayarPct` (index 14–16) **tidak dipakai di fitur manapun** — sisa pemetaan lama. Boleh dibiarkan atau dihapus.

Setiap angka di situ adalah **posisi kolom** (dihitung dari 0, jadi kolom A = 0, B = 1, C = 2, dst). Kalau bingung menghitung, cara paling gampang: kolom Excel/Sheets "Z" = index 25, "AA" = 26, "AB" = 27, dst — atau tanya AI (ChatGPT/Claude) "kolom AF itu index ke berapa kalau dihitung dari 0", dijawab dalam sekejap.

Field `planFisik`, `actualFisik`, `planBayarPct`, `actualBayarPct`, `planBayarRp`, `actualBayarRp` masing-masing adalah **kolom AWAL dari 12 kolom bulanan** (Januari s.d Desember) yang berurutan tanpa jeda. Jadi kalau kolom "PLAN PROGRES FISIK Januari" pindah posisi, cukup update SATU angka itu (`planFisik`), 11 bulan sisanya otomatis ikut karena dihitung `+1, +2, +3...` dari situ.

### Langkah kalau kolom berubah:

1. Buka sheet MONITORING di Google Sheets.
2. Cari kolom yang jadi patokan (misal kolom "STATUS") — lihat huruf kolomnya di header Sheets (misal kolom S).
3. Hitung ulang index: kolom S = huruf ke-19 (A=1, B=2, ..., S=19) → index 0-based = 19 - 1 = **18**.
4. Update angka itu di `IDX` pada kode.
5. Ulangi untuk semua field yang posisinya berubah.
6. Buka dashboard, tekan **F12** → tab **Console** → refresh halaman → cek log `[Dashboard] Contoh kontrak pertama:` — pastikan semua field terisi data yang masuk akal (bukan `undefined` atau nyasar ke kolom lain).

**Tips:** Kalau ragu-ragu menghitung sendiri, paling cepat kirim beberapa baris pertama sheet (screenshot atau copy-paste ke chat) ke AI (Claude/ChatGPT) sambil bilang "ini struktur sheet saya, tolong hitung ulang IDX mapping-nya" — jauh lebih cepat dan minim salah dibanding hitung manual.

---

## Struktur Kolom Sheet Kedua — DETIL_KONTRAK (`DETIL_IDX`)

Cari `const DETIL_IDX` di `index.html`:

```js
const DETIL_KONTRAK_GID = '798316676';
const DETIL_IDX = { noKontrak: 2, linkKontrak: 35, linkSebaran: 36 };
```

- `noKontrak` (kolom C) — dipakai sebagai **kunci pencocokan** ke sheet MONITORING. Harus persis sama isinya dengan kolom C di MONITORING.
- `linkKontrak` (kolom AJ) — URL yang dibuka lewat tombol "🔗 Buka Link Kontrak" di kartu detail.
- `linkSebaran` (kolom AK) — URL yang dibuka lewat tombol "📍 Buka Link Sebaran Pekerjaan".

Beda dari MONITORING, sheet ini cuma punya **1 baris header** (bukan 3) — kalau lihat kode pemanggilannya (`loadSheet(DETIL_KONTRAK_GID, 1)`), angka `1` di situ artinya jumlah baris header. Kalau strukturnya berubah jadi lebih dari 1 baris header, angka itu juga perlu disesuaikan.

Kalau isi kolom link kosong / `-` / `N/A` untuk suatu kontrak, tombolnya otomatis tidak muncul di kartu detail (bukan ditampilkan sebagai link kosong).

---

## Struktur Kolom Sheet Ketiga — PEMBAYARAN (`PEMBAYARAN_IDX`)

Cari `const PEMBAYARAN_IDX` di `index.html`:

```js
const PEMBAYARAN_GID = '2042709256';
const PEMBAYARAN_IDX = {
  noKontrak: 2,
  terminPlan: 8, fisikPlan: 9, bayarPlan: 10, jumlahPlan: 11, bulanPlan: 12,
  terminReal: 13, fisikReal: 14, bayarReal: 15, jumlahReal: 16, bulanReal: 17
};
```

| Kolom sheet | Isi | Field |
|---|---|---|
| C | Nomor Kontrak — **kunci pencocokan** ke MONITORING | `noKontrak` |
| I | Termin (nomor termin diambil dari kolom ini) | `terminPlan` |
| J, K, L, M | Plan: Fisik, Bayar (%), Jumlah (Rp), Bulan | `fisikPlan`, `bayarPlan`, `jumlahPlan`, `bulanPlan` |
| N | Termin (sisi Realisasi, tidak dipakai) | `terminReal` |
| O, P, Q, R | Realisasi: Fisik, Bayar (%), Jumlah (Rp), Bulan | `fisikReal`, `bayarReal`, `jumlahReal`, `bulanReal` |

Kolom S (STATUS) di sheet ini **sengaja tidak dipakai**.

Hal yang perlu diketahui tentang sheet ini:

- Sheet ini punya **3 baris header** (dipanggil lewat `loadSheet(PEMBAYARAN_GID, 3)`).
- **Satu kontrak bisa punya banyak baris** (1 baris = 1 termin) dan jumlah termin tiap kontrak boleh berbeda-beda. Dashboard mengelompokkannya per Nomor Kontrak.
- Baris kosong pemisah antar kontrak dan baris divider tahun (misalnya "2022") otomatis dilewati karena kolom C-nya kosong.
- Kolom **FISIK** kadang berisi teks seperti `RET` (retensi), bukan angka — dibaca apa adanya sebagai teks.
- Kalau kolom di sheet ini bergeser, update angka di `PEMBAYARAN_IDX` dengan cara yang sama seperti `IDX`.

---

## Status Masa Pelaksanaan, Status Jampel, dan Kartu "Perlu Perhatian"

Dua status ini dibaca dari sheet MONITORING:

- **Status Masa Pelaksanaan** — kolom **N** (`statusMasaPelaksanaan`): `AMAN`, `PERLU AMANDEMEN`, `HAMPIR EXPIRED`.
- **Status Jampel** — kolom **AA** (`statusJampel`).

**Kotak filter di bagian bawah dashboard** — ada 4 kotak dropdown: Status PIC, Status Penyedia, Status Masa Pelaksanaan, Status Jampel. Isi dropdown diambil otomatis dari nilai unik di kolom masing-masing. Nilai kosong, `-`, `N/A`, dan error spreadsheet (teks diawali `#`, seperti `#VALUE!`) otomatis dibuang lewat fungsi `isPlaceholder` di `populateFilters`. Pilih salah satu → muncul modal daftar kontrak. Baris tambahan di modal diatur lewat parameter ke-3 (`extraFields`) fungsi `openListOverlay`:

| Kotak | Baris tambahan di modal |
|---|---|
| Status Masa Pelaksanaan | Akhir Kontrak (kolom J) dan Akhir Kontrak Amandemen (kolom M) |
| Status Jampel | Akhir Jaminan Pelaksanaan (kolom Z) |
| PIC / Penyedia | — |

**Kartu "Perlu Perhatian"** (di antara kotak Detail Kontrak dan tabel Rencana vs Realisasi Bayar) — fungsi `renderPerhatianCards`. Ada 3 kartu dengan pencocokan teks **persis, huruf besar semua**:

| Kartu | Kondisi |
|---|---|
| Kontrak: Hampir Expired | `statusMasaPelaksanaan === 'HAMPIR EXPIRED'` |
| Kontrak: Perlu Amandemen | `statusMasaPelaksanaan === 'PERLU AMANDEMEN'` |
| Jampel: Expired | `statusJampel === 'EXPIRED'` |

- Angka lebih dari 0 → kartu **merah transparan** dan bisa diklik (buka daftar kontrak). Angka 0 → kartu **hijau transparan** (tidak bisa diklik).
- Dihitung dari **semua kontrak di semua tahun** — **tidak** ikut dropdown Tahun. Ini disengaja, karena kontrak yang hampir kedaluwarsa tetap perlu diperhatikan apapun tahun kontraknya. Modal yang terbuka dari kartu ini juga tidak difilter tahun (lewat parameter `ignoreYear` di `openStatusListOverlay`).
- Kalau ejaan status di sheet berubah, ubah string pencocokan di `renderPerhatianCards`.

Cek cepat lewat F12 → Console:
```js
CONTRACTS.filter(k => k.statusMasaPelaksanaan === 'HAMPIR EXPIRED').length
CONTRACTS.filter(k => k.statusJampel === 'EXPIRED').length
```

---

## Progres Pembayaran per Termin & Bagian Collapsible di Kartu Detail

Di kartu detail kontrak, **Progres Fisik Bulanan** dan **Progres Pembayaran per Termin** sekarang default **tertutup**. Judulnya bisa diklik (`▼ Lihat` / `▲ Tutup`), keduanya independen dan bisa dibuka bersamaan. Logikanya di fungsi `toggleCollapse`; ID elemen dibuat unik per kontrak dari Nomor Kontrak (karakter non-alfanumerik diganti `_`, variabel `safeId` di `buildKontrakDetailContent`) supaya beberapa kartu detail yang terbuka bersamaan tidak bentrok.

Tabel termin dibuat oleh `buildTerminBayarTable` dengan data dari `TERMIN_BAYAR_MAP` (diisi oleh `loadTerminBayarMap`). Tiap termin tampil 2 baris: **Plan** (biru) dan **Realisasi** (hijau), dengan kolom Fisik, Bayar %, Jumlah (Rp), Bulan.

---

## Card Target Kinerja (progres fisik)

Card "Rencana vs realisasi progres fisik (Target Kinerja)" berisi: dropdown semester, 2 kartu KPI, chart garis, tabel ringkas Jan–Des, dan tombol screenshot. Semua angka adalah **rata-rata** dari kontrak yang kolom Keterangan-nya mengandung "TARGET KINERJA".

- **Dropdown semester**: Semester 1 (Juni) / Semester 2 (Desember). Pilihan awal otomatis mengikuti bulan sekarang (Jan–Jun = Semester 1, Jul–Des = Semester 2). Logika di `renderFisikKpi`.
- **Kartu Rencana Progres Fisik**: rata-rata plan bulan Juni atau Desember sesuai dropdown.
- **Kartu Realisasi Sampai Saat Ini**: rata-rata realisasi bulan ini, **kecuali** Semester 1 dipilih saat bulan sekarang sudah Juli–Desember — maka dipakai realisasi Juni. Baris di bawahnya: `xx.xx% dari rencana progres fisik` (realisasi ÷ rencana). Kalau belum ada data: `–` dan "belum ada data bulan ini".
- **Chart**: tiap titik realisasi diberi label persen realisasi ÷ rencana (dibulatkan). Label di atas titik jika realisasi ≥ rencana, di bawah titik jika lebih rendah. Memakai plugin datalabels yang dipasang per-chart (`plugins:[ChartDataLabels]`).
- **Tabel ringkas** di bawah chart: fungsi `renderFisikMiniTable`.
- **Modal "Lihat Target Kinerja"**: dropdown bulan (default bulan sekarang) di header, kartu Target & Realisasi (`xx.xx% dari target`), dan tabel kontrak — semuanya ikut bulan yang dipilih. Dibangun oleh `buildTargetKinerjaBody`, diganti bulannya oleh `changeTargetKinerjaMonth`, screenshot-nya oleh `screenshotTargetKinerja`.

---

## Status Bayar: Terbayar / VIP / Belum Terbayar

Di tabel "Rencana vs Realisasi Bayar Bulan Ini", kolom Status Bayar punya **3 kemungkinan**, semuanya dibaca dari satu kolom saja: **Actual Progres Bayar (Rp)** di sheet MONITORING (`actualBayarRp`).

Formula di balik kolom itu (dari sheet sumbernya) polanya seperti ini:
```
=IF(PROGRESS_BAYAR!$C..=$C..,PROGRESS_BAYAR!EV..,"")
```
Artinya:
- Kalau kondisi **tidak cocok** → hasilnya string kosong `""` → dashboard baca sebagai **kosong/dash** → status **Belum Terbayar**.
- Kalau kondisi **cocok** dan nilainya **lebih dari Rp1** → status **Terbayar**.
- Kalau kondisi **cocok** tapi nilainya **persis 0** (angka nol asli, ditampilkan "Rp0.00" karena format sel Keuangan) → status **VIP**.

Logikanya ada di fungsi `renderBayarTable` — cari komentar `isEffectivelyZero` di situ kalau perlu diubah ambang batasnya (saat ini toleransi di bawah Rp1 dianggap nol, untuk jaga-jaga sisa desimal kecil dari formula).

Kolom **Realisasi Rp** (di antara Rencana Rp dan Status Bayar) menampilkan nilai mentah `actualBayarRp` untuk bulan yang dipilih — angka inilah yang menentukan Status Bayar di atas. Baris total di bawah tabel menampilkan total Rencana dan total Realisasi, masing-masing sejajar di bawah kolomnya.

⚠️ Kalau nanti Bapak menandai kontrak sebagai VIP tapi dashboard masih salah baca statusnya, buka F12 → Console, ketik:
```js
CONTRACTS.find(k => k.judul.includes("KATA_KUNCI_JUDUL")).actualBayarRp
```
lalu Enter — ini akan menampilkan 12 angka mentah (Jan–Des) yang benar-benar terbaca dashboard dari kolom Actual Rp, jadi gampang dilacak bulan mana yang salah.

---

## Ganti Password Akses

Cari `const ACCESS_PASSWORD` di `index.html`, ganti nilai di antara tanda kutip:

```js
const ACCESS_PASSWORD = 'PLN123'; // case-sensitive
```

⚠️ **Penting:** ini BUKAN sistem keamanan sungguhan. Password ini tertulis polos di kode HTML — siapapun yang buka "View Page Source" di browser bisa melihatnya langsung. Fungsinya cuma menyaring orang iseng yang kebetulan lewat link, bukan melindungi data sensitif dari orang yang benar-benar berniat masuk.

---

## Ganti Spreadsheet Sumber Data

Kalau suatu saat pindah ke Google Sheets yang lain sama sekali:

1. Buka spreadsheet baru, pastikan sharing-nya **"Siapa saja yang memiliki link dapat melihat"** (kalau masih private, dashboard tidak akan bisa ambil data).
2. Ambil **Spreadsheet ID** dari URL: `docs.google.com/spreadsheets/d/`**`INI-ID-NYA`**`/edit...`
3. Ambil **gid** tab yang dipakai dari URL setelah `#gid=` atau `?gid=`.
4. Update dua baris ini di kode:
   ```js
   const SHEET_ID = 'ID_BARU_DI_SINI';
   const GID = 'GID_BARU_DI_SINI';
   ```
5. Hitung ulang `IDX` sesuai struktur kolom sheet baru (lihat bagian di atas).
6. Kalau sheet pelengkap ikut pindah, update juga `DETIL_KONTRAK_GID` (+ `DETIL_IDX`) dan `PEMBAYARAN_GID` (+ `PEMBAYARAN_IDX`).

---

## Cara Update File di GitHub Pages

1. Edit `index.html` (bisa langsung di GitHub lewat tombol pensil ✏️, atau edit lokal lalu upload).
2. Commit perubahan.
3. Tunggu 1-2 menit, GitHub Pages otomatis re-deploy.
4. Buka dashboard, **hard refresh** (Ctrl+Shift+R) supaya browser tidak pakai cache versi lama.

---

## Struktur Fitur (ringkas)

| Fitur | Lokasi di kode (cari via Ctrl+F) |
|---|---|
| Login / password | `ACCESS_PASSWORD` |
| Mapping kolom sheet MONITORING | `const IDX` |
| Mapping kolom sheet DETIL_KONTRAK (link) | `const DETIL_IDX` |
| Peta link kontrak & sebaran pekerjaan | `function loadLinkKontrakMap` |
| Status kontrak (6 kartu, bisa diklik) | `function renderStatus` |
| Modal daftar kontrak per kategori status | `function openStatusListOverlay` |
| Search live "Detail Kontrak" | `function setupKontrakSearch` |
| Isi kartu detail kontrak (dipakai di 2 tempat) | `function buildKontrakDetailContent` |
| Tombol Tautan Terkait (link kontrak/sebaran) | dalam `buildKontrakDetailContent`, cari `linkButtonsHtml` |
| Tabel bulanan Plan vs Actual progres fisik | `function buildMonthlyFisikTable` |
| Tabel rencana/realisasi bayar bulan ini + status VIP | `function renderBayarTable` |
| Chart tren bayar bulanan | `function renderChartsAndKpi` |
| Target Kinerja (progres fisik) | `function openTargetKinerjaOverlay` |
| Peta data termin pembayaran (sheet PEMBAYARAN) | `function loadTerminBayarMap` |
| Tabel Progres Pembayaran per Termin | `function buildTerminBayarTable` |
| Buka/tutup bagian collapsible di kartu detail | `function toggleCollapse` |
| Kartu "Perlu Perhatian" (3 kartu warning) | `function renderPerhatianCards` |
| Modal daftar kontrak untuk kartu status / Perlu Perhatian | `function openStatusListOverlay` (param: `fieldKey`, `ignoreYear`) |
| Modal daftar kontrak untuk kotak PIC/Penyedia/Status Masa Pelaksanaan/Status Jampel | `function openListOverlay` (param: `extraFields`) |
| KPI semester card Target Kinerja | `function renderFisikKpi` |
| Tabel ringkas Jan–Des card Target Kinerja | `function renderFisikMiniTable` |
| Isi modal Target Kinerja (kartu + tabel per bulan) | `function buildTargetKinerjaBody` |
| Ganti bulan di modal Target Kinerja | `function changeTargetKinerjaMonth` |
| Screenshot modal Target Kinerja | `function screenshotTargetKinerja` |
| Hook screenshot untuk chart (resize di luar layar) | `function makeChartScreenshotHooks` |
| Screenshot ke clipboard | `function screenshotElement` |

---

## Troubleshooting Cepat

**Data tidak muncul / semua kosong**
→ Buka F12 → Console, lihat pesan error. Biasanya karena sheet belum di-share publik, atau `SHEET_ID`/`GID` salah.

**Angka aneh (kelewat besar/kecil, atau 0 semua)**
→ Kemungkinan besar kolom di sheet sudah geser tapi `IDX` belum di-update. Lihat bagian "Kapan Perlu Ubah Kode?" di atas.

**Chart/tabel kosong padahal data ada**
→ Kemungkinan kolom persentase/Rupiah di sheet tersimpan sebagai **teks**, bukan angka (biasa terjadi kalau input manual tanpa format cell yang benar). Cara cek: klik selnya di Sheets, kalau rata kiri artinya teks, kalau rata kanan artinya angka. Perbaiki lewat **Data → Pembersihan data → Ubah teks menjadi angka**.

**Tampilan geser-geser ke samping di HP**
→ Biasanya ada elemen baru (tabel/kartu) yang lebarnya dipatok piksel tetap (`min-width`). Ganti ke persentase, atau tanya AI untuk audit ulang bagian mana yang overflow.

**Status Bayar salah (harusnya VIP tapi kebaca Terbayar/Belum Terbayar, atau sebaliknya)**
→ Lihat bagian "Status Bayar: Terbayar / VIP / Belum Terbayar" di atas. Cek dulu apakah kolom Actual Rp di sheet MONITORING sudah benar menampilkan "Rp0.00" (bukan dash atau angka lain) untuk kontrak yang dimaksud.

**Tombol "Tautan Terkait" tidak muncul padahal linknya ada di sheet**
→ Cek Nomor Kontrak di sheet DETIL_KONTRAK harus **persis sama** (termasuk spasi, huruf besar/kecil biasanya tidak masalah tapi karakter lain harus identik) dengan Nomor Kontrak di sheet MONITORING. Cek juga isi kolom AJ/AK bukan `-` atau `N/A`.

**Tabel "Progres Pembayaran per Termin" kosong / "Belum ada data pembayaran per termin" padahal datanya ada**
→ Cek Nomor Kontrak di sheet PEMBAYARAN (kolom C) harus **persis sama** dengan di sheet MONITORING. Cek juga `PEMBAYARAN_GID` benar dan sheet-nya di-share publik. Buka F12 → Console, cari log `[Dashboard] Jumlah kontrak dengan data termin (dari PEMBAYARAN)` — kalau angkanya 0 atau muncul warning gagal memuat, masalahnya di sheet/gid.

**Angka kartu "Perlu Perhatian" tidak sesuai harapan**
→ Kartu ini menghitung semua kontrak semua tahun dan mencocokkan teks persis (`HAMPIR EXPIRED`, `PERLU AMANDEMEN`, `EXPIRED`). Cek di Console dengan perintah di bagian "Status Masa Pelaksanaan, Status Jampel, dan Kartu Perlu Perhatian". Kalau ejaan di sheet berbeda (spasi ganda, huruf kecil), samakan di sheet atau ubah string di `renderPerhatianCards`.

**Dropdown Status Masa Pelaksanaan menampilkan nilai aneh / kosong**
→ Kemungkinan kolom N bergeser dan `statusMasaPelaksanaan` di `IDX` perlu dihitung ulang. Nilai `#VALUE!` memang otomatis disembunyikan; kalau semua nilai `#VALUE!`, cek formula di kolom N.

---

## Riwayat Singkat

Dashboard ini dibangun secara iteratif bersama AI (Claude) berdasarkan kebutuhan monitoring kontrak AI di UPT Probolinggo. Lihat `PROMPT_rebuild_dashboard.md` di repo ini kalau suatu saat perlu membangun ulang dashboard serupa dari nol (lewat AI manapun).
