# UTS Data Acquisition & Management — Cagar Budaya Jawa Barat

Ujian Tengah Semester **Data Acquisition & Management** (Kelas E, Semester Ganjil 2026)
Program Studi Informatika — INSTIKI
Dosen: Dr. I Gede Totok Suryawan, S.Kom., M.T.

| | |
|---|---|
| **Nama** | Anak Agung Radha Noviyanti |
| **NIM** | 2501010378 |
| **Notebook (Colab)** | https://colab.research.google.com/drive/1L4gVUpQvB-hzJwsOKat4hDgB1Vw93r8Z |
| **Repositori** | https://github.com/gungradha/UTS_DAM_2501010378 |

---

## 1. Topik

**Sebaran cagar budaya di kabupaten/kota Jawa Barat (2014–2018) dalam konteks iklim dan profil wilayah.**

Sektor yang dipilih adalah **budaya**. Pertanyaan yang ingin dijawab dengan data gabungan: apakah jumlah cagar budaya per kabupaten/kota berhubungan dengan karakteristik wilayahnya, yaitu luas, jumlah penduduk, suhu, dan curah hujan?

---

## 2. Tiga Sumber Data

| # | Cara akuisisi | Sumber | Isi | Lisensi | Berkas mentah |
|---|---|---|---|---|---|
| 1 | **Unduh berkas dataset terbuka** | [data.go.id — Jumlah Benda Cagar Budaya Berdasarkan Jenis di Jawa Barat](https://data.go.id/dataset/dataset/jumlah-benda-cagar-budaya-berdasarkan-jenis-di-jawa-barat) (Disparbud Jabar) | 349 baris × 9 kolom, 27 kab/kota, 2014–2018 | Lihat `docs/bukti_lisensi_tos.csv` | `data/raw/disparbud-od_15354_jumlah_benda_cagar_budaya_berdasarkan_jenis_v1.csv` |
| 2 | **API** | [Open-Meteo](https://open-meteo.com/) (Geocoding API + Historical Weather API), tanpa API key | Koordinat tiap kab/kota dan cuaca harian 2014–2018 (suhu rata-rata, curah hujan) | CC BY 4.0 (atribusi Open-Meteo.com) | `data/raw/openmeteo_geocoding_jabar.json`, `data/raw/openmeteo_archive_jabar_2014_2018.json` |
| 3 | **Web scraping halaman statis** | [Wikipedia ID — Daftar kabupaten dan kota di Jawa Barat](https://id.wikipedia.org/wiki/Daftar_kabupaten_dan_kota_di_Jawa_Barat) | Tabel 27 kab/kota: luas, penduduk, jumlah kecamatan | CC BY-SA 4.0 | `data/raw/wikipedia_id_kabkota_jabar.html`, `data/raw/robots_id_wikipedia.txt` |

Tidak ada data dari Booking.com, Agoda, Traveloka, tiket.com, Airbnb, Tripadvisor, Google Maps, maupun media sosial.

### Provenance dan integritas (MD5)

Semua berkas mentah disimpan **apa adanya** di `data/raw/` dan dikunci read-only. Setiap berkas dicatat di `docs/provenance.csv` dan `docs/provenance.json` (URL, tanggal akses UTC, lisensi, ukuran, MD5).

| Berkas | Ukuran (byte) | MD5 |
|---|---:|---|
| `disparbud-od_15354_..._v1.csv` | 24.679 | `aef68f27189cb8ebfabda832ddb4b99a` |
| `openmeteo_geocoding_jabar.json` | 48.838 | `0a2bad96926910c2a78e885618bf869b` |
| `openmeteo_archive_jabar_2014_2018.json` | 976.724 | `2e5000ab46a6b6d848f8fbf0cf30c54a` |
| `robots_id_wikipedia.txt` | 15.801 | `8dd94bf3d01cbb25e0af4e64c0485433` |
| `wikipedia_id_kabkota_jabar.html` | 153.419 | `2c450e795acb66d63f74ab420d4153e6` |

Sel verifikasi akhir di notebook menghitung ulang MD5 dan memastikan semuanya masih cocok.

---

## 3. Struktur Repositori

```
UTS_DAM_2501010378/
├── uts_dam_budaya_2501010378.ipynb   # notebook utama (semua sel sudah dijalankan)
├── README.md
├── requirements.txt
├── .env.example                      # contoh konfigurasi (tanpa nilai rahasia)
├── .gitignore                        # .env tidak ikut ter-commit
├── data/
│   ├── raw/                          # data mentah, tidak diubah
│   │   ├── disparbud-od_15354_jumlah_benda_cagar_budaya_berdasarkan_jenis_v1.csv
│   │   ├── openmeteo_geocoding_jabar.json
│   │   ├── openmeteo_archive_jabar_2014_2018.json
│   │   ├── robots_id_wikipedia.txt
│   │   └── wikipedia_id_kabkota_jabar.html
│   └── processed/
│       ├── cagar_budaya_jabar_merged.csv
│       └── cagar_budaya_jabar_merged.metadata.json
└── docs/
    ├── provenance.csv / provenance.json
    ├── data_ethics_checklist.csv / .md
    ├── bukti_lisensi_tos.csv
    ├── pii_scan_semua_sheet.csv
    ├── robots_check_evidence.json
    └── top10_cagar_budaya.png
```

---

## 4. Ringkasan Pengerjaan per Soal

### Soal 1 — Pembacaan berkas, identifikasi data, estimasi storage

Berkas dibaca dengan `pd.read_csv(sep=",", encoding="utf-8", skipinitialspace=True, dtype={...}, na_values=["", "-", "NA", "N/A", "null", "NULL"])`.

| Aspek | Hasil |
|---|---|
| Jenis data | **Terstruktur** (CSV tabular, skema kolom tetap) |
| Jumlah baris × kolom | 349 × 9 |
| Satuan | UNIT |
| Wilayah | Jawa Barat, 27 kabupaten/kota |
| Waktu | 2014–2018 (tahunan) |
| Ukuran berkas | 24.679 byte (24,10 KB) |

**Estimasi storage 1 tahun**

Asumsi: tiap bulan menyimpan satu salinan arsip berkas dengan ukuran sama, ditambah cadangan 30% untuk backup, metadata, dan pertumbuhan data.

- Berkas CSV saja: 24.679 B × 12 = 296.148 B, lalu × 1,3 = 384.992 B, yaitu **±376 KB (0,367 MB)**
- Seluruh `data/raw` (3 sumber): 1.190,9 KB × 12 × 1,3 = **±18,14 MB**

**Temuan kualitas data**

- Kategori `jenis_benda` **berubah di 2018**: 2014–2017 hanya *Bergerak* dan *Tidak Bergerak*, sedangkan 2018 memakai 5 kategori menurut UU Cagar Budaya. Total antartahun tidak sepenuhnya sebanding, sehingga kolom `skema_kategori` ditambahkan.
- 20 nilai kosong, semuanya di 2018. Dibiarkan `NaN`.
- Satu nilai non-bulat (`1.757`, Kota Bandung 2018) diduga salah baca pemisah ribuan. Dikoreksi menjadi 1757 **hanya di data processed** dan diberi flag.
- Kabupaten Pangandaran tidak ada di 2014 (baru terbentuk).

### Soal 2 — Etika data

- **Lisensi dan ketentuan portal:** dicatat per sumber di `docs/bukti_lisensi_tos.csv` (URL, tanggal akses, kutipan ketentuan, nama berkas screenshot).
- **PII:** dipindai berdasarkan nama kolom dan pola nilai (email, telepon, NIK 16 digit). Hasilnya **tidak ada PII** pada 9 kolom (`docs/pii_scan_semua_sheet.csv`).
- **Quasi-identifier:** `kode_kabupaten_kota`, `nama_kabupaten_kota`, `jenis_benda`, `tahun`. Data berupa agregat per wilayah administratif (bukan individu), 349 kombinasi unik dari 349 baris, sehingga risiko re-identifikasi rendah.
- **Data Ethics Checklist 9 butir:** `docs/data_ethics_checklist.md` (juga `.csv`), lengkap dengan bukti tiap butir.

### Soal 3 — Akuisisi lewat API (Open-Meteo)

Portal data.go.id tidak dipakai sebagai API, jadi dipakai API publik yang relevan: **Open-Meteo**.

1. **Geocoding**: nama 27 kab/kota diubah menjadi koordinat, divalidasi dengan kotak batas Jawa Barat. Kab/kota tanpa hasil geocoding yang valid diberi koordinat manual (`COORD_OVERRIDE`).
2. **Historical Weather**: suhu rata-rata dan curah hujan harian 2014–2018 untuk 21 titik koordinat unik.
3. **Normalisasi JSON → tabel**: **49.302 baris** harian (syarat minimal 30), lalu diagregasi menjadi **135 baris** tahunan (suhu rata-rata, total curah hujan, jumlah hari hujan).

Penanganan request: `Retry(total=3, backoff_factor=2)` untuk status 429/500/502/503/504, `timeout=30`, jeda 1,5 detik antar-request, error 4xx dilaporkan jelas, dan respons divalidasi (harus memuat `daily`). Tidak ada API key.

### Soal 4 — Web scraping etis

- Target: halaman statis Wikipedia ID (daftar kab/kota Jawa Barat).
- `robots.txt` diperiksa dengan `RobotFileParser`: `can_fetch = True`, tanpa crawl-delay. Bukti ada di `docs/robots_check_evidence.json` dan `data/raw/robots_id_wikipedia.txt`.
- Hanya **1 request** ke halaman target, dengan User-Agent berisi email, timeout, dan jeda.
- Tabel diambil dengan `pd.read_html` (cadangan: BeautifulSoup), lalu dibersihkan: angka format Indonesia (`1.740,843` → `1740.843`) dan nama wilayah dicocokkan ke kode kab/kota. Hasilnya **27 dari 27** kab/kota cocok.

### Soal 5 — Penggabungan data

Grain tabel akhir: **1 baris = 1 kabupaten/kota × 1 tahun**.

| Merge | Kiri | Kanan | Kunci | how | `validate` |
|---|---|---|---|---|---|
| 1 | Berkas CSV (diagregasi) | Open-Meteo tahunan | `kode_kabupaten_kota`, `tahun` | left | `one_to_one` |
| 2 | Hasil merge 1 | Wikipedia ID | `kode_kabupaten_kota` | left | `many_to_one` |

**Bukti tidak ada duplikasi**

| Pemeriksaan | Hasil |
|---|---:|
| Baris sebelum merge | 134 |
| Baris setelah merge 1 | 134 |
| Baris setelah merge 2 | 134 |
| Duplikat kunci (kode, tahun) | 0 |
| Baris duplikat penuh | 0 |
| Baris tanpa pasangan API / scraping | 0 / 0 |

Keluaran: `data/processed/cagar_budaya_jabar_merged.csv` (134 baris × 16 kolom) beserta `cagar_budaya_jabar_merged.metadata.json` (skema, parameter merge, catatan kualitas, MD5 keluaran, versi lingkungan).

---

## 5. Cara Menjalankan Ulang

**Google Colab**

1. Buka notebook dari tautan Colab di atas.
2. (Opsional) Tambahkan Colab Secret `CONTACT_EMAIL` berisi email kampus untuk User-Agent.
3. Jalankan semua sel (*Runtime → Run all*). Saat diminta, unggah berkas CSV dari data.go.id.

**Lokal**

```bash
git clone https://github.com/gungradha/UTS_DAM_2501010378.git
cd UTS_DAM_2501010378
pip install -r requirements.txt
cp .env.example .env        # lalu isi CONTACT_EMAIL dengan email kampus
jupyter notebook
```

Secara default notebook memakai cache di `data/raw/` dan tidak mengirim ulang request. Ubah `REFRESH = True` di sel konfigurasi untuk mengambil ulang dari sumber.

---

## 6. Keamanan dan Etika

- Tidak ada API key atau token di notebook. Konfigurasi dibaca dari `.env` atau Colab Secrets (`get_secret()`), dan `.env` masuk `.gitignore`.
- Setiap request memakai **timeout**, **User-Agent berisi email kampus**, dan **jeda** antar-request.
- Data mentah tidak dimodifikasi. Perbaikan hanya dilakukan di `data/processed/` dan ditandai dengan flag.

## 7. Keterbatasan

- Kategori cagar budaya berubah di 2018, jadi perbandingan total 2014–2017 dengan 2018 perlu hati-hati.
- Cuaca Open-Meteo adalah data reanalisis per titik koordinat (hasil geocoding), bukan pengukuran stasiun di tiap kab/kota. Beberapa kab/kota berbagi titik koordinat yang sama.
- Jumlah penduduk dari Wikipedia adalah angka **2025**, sedangkan data cagar budaya 2014–2018, sehingga dipakai sebagai profil wilayah, bukan data sezaman.

## 8. Atribusi

- Dataset cagar budaya: Dinas Pariwisata dan Kebudayaan Provinsi Jawa Barat melalui data.go.id.
- Data cuaca: [Open-Meteo.com](https://open-meteo.com/), CC BY 4.0.
- Tabel wilayah: [Wikipedia ID](https://id.wikipedia.org/), CC BY-SA 4.0. Karena share-alike, turunan data dari Wikipedia di repositori ini sebaiknya mengikuti lisensi yang sama.
