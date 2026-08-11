# Dashboard Monitoring Kontrak AI — UPT Probolinggo

Dashboard satu halaman (`index.html`) yang menampilkan data kontrak AI (Anggaran Investasi) UPT Probolinggo secara **live** langsung dari Google Sheets. Tidak perlu proses build/compile — tinggal buka file HTML-nya di browser atau host lewat GitHub Pages.

🔗 Live: `https://konspblg.github.io/kontraktual/`

---

## Cara Kerja Singkat

Dashboard ini **tidak punya database sendiri**. Setiap kali dibuka, dia mengambil data langsung dari **2 sheet** di Google Sheets memakai teknik `gviz` (Google Visualization API):

1. **MONITORING** (gid `799877147`) — data utama: identitas kontrak, tanggal, BAST, jaminan pelaksanaan, progres fisik & bayar bulanan, status, PIC.
2. **DETIL_KONTRAK** (gid `798316676`) — sheet pelengkap, cuma dipakai untuk 2 hal: **Link Kontrak** dan **Link Sebaran Pekerjaan**, dicocokkan ke sheet MONITORING lewat Nomor Kontrak (kolom C di kedua sheet). Kalau sheet ini gagal dimuat, dashboard utama tetap jalan normal — cuma tombol link-nya saja yang tidak muncul.

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

---

## Struktur Kolom Saat Ini (`IDX`)

Cari bagian ini di `index.html` (pakai Ctrl+F, cari `const IDX`):

```js
const SHEET_ID = '1OLvF-EFqSZZIwZRK4T3b8VyXLM53OW42GoCzbzasgjI';
const GID = '799877147';

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

---

## Riwayat Singkat

Dashboard ini dibangun secara iteratif bersama AI (Claude) berdasarkan kebutuhan monitoring kontrak AI di UPT Probolinggo. Lihat `PROMPT_rebuild_dashboard.md` di repo ini kalau suatu saat perlu membangun ulang dashboard serupa dari nol (lewat AI manapun).
