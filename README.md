# Dashboard Monitoring Kontrak AI — UPT Probolinggo

Dashboard satu halaman (`index.html`) yang menampilkan data kontrak AI (Anggaran Investasi) UPT Probolinggo secara **live** langsung dari Google Sheets. Tidak perlu proses build/compile — tinggal buka file HTML-nya di browser atau host lewat GitHub Pages.

🔗 Live: `https://konspblg.github.io/kontraktual/`

---

## Cara Kerja Singkat

Dashboard ini **tidak punya database sendiri**. Setiap kali dibuka, dia mengambil data langsung dari Google Sheets (sheet **MONITORING**) memakai teknik `gviz` (Google Visualization API). Jadi:

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
| Mapping kolom sheet | `const IDX` |
| Status kontrak (6 kartu) | `function renderStatus` |
| Search live "Detail Kontrak" | `function setupKontrakSearch` |
| Isi kartu detail kontrak | `function buildKontrakDetailContent` |
| Tabel rencana/realisasi bayar bulan ini | `function renderBayarTable` |
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

---

## Riwayat Singkat

Dashboard ini dibangun secara iteratif bersama AI (Claude) berdasarkan kebutuhan monitoring kontrak AI di UPT Probolinggo. Lihat `PROMPT_rebuild_dashboard.md` di repo ini kalau suatu saat perlu membangun ulang dashboard serupa dari nol (lewat AI manapun).
