# UTS_DAM_2501010378

# UTS Data Acquisition & Management — Cagar Budaya Jawa Barat

Proyek UTS mata kuliah **Data Acquisition & Management** (Kelas E).
Topik: sektor **budaya** — jumlah cagar budaya per kabupaten/kota di Jawa Barat (2014–2018), digabung dengan data cuaca (API) dan profil wilayah (scraping).

- **Nama:** Anak Agung Radha Noviyanti
- **NIM:** 2501010378
- **Kampus / Prodi:** INSTIKI / Informatika MDA
- **Notebook (Google Colab):** https://colab.research.google.com/drive/1L4gVUpQvB-hzJwsOKat4hDgB1Vw93r8Z

---

## 1. Ringkasan

Tiga sumber data dengan tiga cara akuisisi digabung menjadi satu tabel analisis (**1 baris = 1 kabupaten/kota × 1 tahun**).

| Akuisisi | Sumber | Metode | Berkas mentah |
|---|---|---|---|
| (a) Berkas | Jumlah Benda Cagar Budaya Berdasarkan Jenis — Dinas Pariwisata dan Kebudayaan Jawa Barat (data.go.id / Open Data Jabar) | Unduh CSV (349 baris, 9 kolom, 2014–2018) | `data/raw/disparbud-od_15354_..._v1.csv` |
| (b) API | Open-Meteo Geocoding API + Historical Weather API (suhu dan curah hujan harian) | HTTP GET, retry otomatis, jeda antar-request | `data/raw/openmeteo_*.json` |
| (c) Scraping | Wikipedia Indonesia — Daftar kabupaten dan kota di Jawa Barat | Cek `robots.txt`, lalu ambil tabel HTML statis | `data/raw/wikipedia_id_kabkota_jabar.html`, `data/raw/robots_id_wikipedia.txt` |

**Hasil penggabungan:** `data/processed/cagar_budaya_jabar_merged.csv` (134 baris: 27 kab/kota × 5 tahun, kecuali Kab. Pangandaran yang baru ada sejak 2015) beserta berkas metadata `.metadata.json`.

## 2. Struktur repositori

```
.
├── UTS_DAM_Cagar_Budaya_Jabar.ipynb    # seluruh alur Soal 1–5
├── README.md
├── requirements.txt
├── .env.example                        # contoh konfigurasi (salin ke .env; jangan di-commit)
├── .gitignore
├── data/
│   ├── raw/                            # data mentah, tidak diubah (read-only + hash MD5)
│   └── processed/                      # tabel gabungan + metadata
├── docs/
│   ├── bukti_lisensi_tos.csv           # lisensi & ketentuan penggunaan tiap sumber
│   ├── data_ethics_checklist.csv/.md   # Data Ethics Checklist 9 butir + bukti
│   ├── pii_scan_semua_sheet.csv        # pemindaian PII & quasi-identifier
│   ├── provenance.json                 # asal data, hash, dan riwayat transformasi
│   ├── provenance.csv                  # versi tabel dari provenance
│   ├── robots_check_evidence.json      # bukti kepatuhan robots.txt
│   └── top10_cagar_budaya.png
└── gambar/                             # screenshot bukti lisensi/ketentuan portal
```

## 3. Peta soal → notebook

| Soal | Isi | Bagian notebook | Bukti di repositori |
|---|---|---|---|
| 1 | Baca CSV dengan parameter tepat, identifikasi dataset (jenis, baris/kolom, ukuran, tipe data), estimasi storage 1 tahun | Bagian 1 | notebook |
| 2 | Periksa lisensi dan ketentuan portal, identifikasi PII/quasi-identifier, Data Ethics Checklist 9 butir | Bagian 2, 5, dan sel bukti | `docs/bukti_lisensi_tos.csv`, `docs/pii_scan_semua_sheet.csv`, `docs/data_ethics_checklist.md` |
| 3 | Akuisisi API (status code, retry, normalisasi JSON) menjadi tabel ≥ 30 baris | Bagian 3 | `data/raw/openmeteo_*.json` |
| 4 | Periksa `robots.txt`, scraping halaman statis | Bagian 4 | `docs/robots_check_evidence.json` |
| 5 | Gabungkan ≥ 2 sumber dengan `merge(validate=...)`, bukti tanpa duplikasi, simpan beserta metadata | Bagian 6 | `data/processed/*` |
| Provenance | Catat URL, tanggal akses, lisensi, hash untuk semua sumber | Fungsi `log_provenance()` | `docs/provenance.json` |

## 4. Temuan kualitas data

- **Skema kategori berubah.** 2014–2017 memakai *BERGERAK / TIDAK BERGERAK*, sedangkan 2018 memakai lima kategori (Benda, Bangunan, Struktur, Situs, Kawasan). Total antartahun harus dibaca hati-hati; tabel gabungan memuat kolom `skema_kategori`.
- **Nilai kosong.** 20 sel `jumlah_benda` kosong (semuanya 2018: Ciamis, Kuningan, Bekasi, Depok). Dibiarkan kosong (NaN), tidak diisi nilai karangan.
- **Nilai janggal.** `1.757` (Kota Bandung 2018, Bangunan Cagar Budaya) diduga 1.757 unit (titik ribuan terbaca sebagai desimal). Koreksi hanya dilakukan pada data *processed*, ditandai, dan dapat dimatikan lewat `APPLY_THOUSAND_FIX = False`. Data mentah tidak diubah.
- **Wilayah baru.** Kab. Pangandaran tidak ada di data 2014.
- **Koordinat.** Geocoding Open-Meteo mencocokkan nama tempat, sehingga beberapa kabupaten (mis. Bandung Barat) bisa tidak ditemukan. Koordinat diisi manual lewat `COORD_OVERRIDE` dan ditandai `manual` di kolom `sumber_koordinat`.

## 5. Etika data

- **PII:** tidak ditemukan. Pemindaian mencakup nama kolom dan pola email, telepon, serta NIK pada seluruh tabel (`docs/pii_scan_semua_sheet.csv`).
- **Quasi-identifier:** wilayah (kode/nama kab-kota), tahun, dan jenis. Data berupa agregat per wilayah administratif, bukan individu, sehingga risiko re-identifikasi rendah.
- **Akses sopan:** `robots.txt` diperiksa sebelum scraping; User-Agent memuat kontak; jeda 1,5 detik antar-request; timeout 30 detik; retry dengan exponential backoff.
- **Kredensial:** tidak ada API key di kode. Email kontak dibaca dari `.env` atau Colab Secrets; `.env` ada di `.gitignore`.
- **Integritas:** data mentah dikunci read-only dan dicatat hash MD5-nya di `docs/provenance.json`.

Detail lengkap ada di `docs/data_ethics_checklist.md`.

## 6. Lisensi dan atribusi

| Sumber | Lisensi / ketentuan | Bukti |
|---|---|---|
| Dataset cagar budaya (Disparbud Jabar) | [isi sesuai halaman dataset] | `docs/bukti_lisensi_tos.csv`, `gambar/` |
| Open-Meteo | CC BY 4.0 (atribusi wajib) | `docs/bukti_lisensi_tos.csv` |
| Wikipedia | CC BY-SA 4.0 | `docs/bukti_lisensi_tos.csv` |

Atribusi: dataset cagar budaya © Dinas Pariwisata dan Kebudayaan Provinsi Jawa Barat (melalui data.go.id / Open Data Jabar) · Weather data by [Open-Meteo.com](https://open-meteo.com/) · Teks Wikipedia oleh kontributor Wikipedia.

Kode di repositori ini dibuat untuk keperluan tugas kuliah.

## 7. Cara menjalankan (Google Colab)

1. Buka `UTS_DAM_Cagar_Budaya_Jabar.ipynb` di Google Colab.
2. Di **Secrets** (ikon kunci) tambahkan `CONTACT_EMAIL` berisi email kampus.
3. Isi `DATASET_URL`, `LISENSI_DATASET`, dan `PORTAL_TOS_URL` pada sel konfigurasi.
4. Jalankan `Runtime > Run all`. Saat diminta, unggah berkas CSV dataset.
5. Unduh folder `data/` dan `docs/`, lalu commit ke repositori ini.

Menjalankan lokal:

```bash
pip install -r requirements.txt
cp .env.example .env      # lalu isi CONTACT_EMAIL
jupyter notebook UTS_DAM_Cagar_Budaya_Jabar.ipynb
```

Data mentah yang sudah ada tidak diunduh ulang kecuali `REFRESH = True`.

## 8. Keterbatasan

- Data cagar budaya hanya sampai 2018 dan tidak seragam antartahun (lihat bagian 4).
- Cuaca dihitung dari satu titik koordinat per kab/kota, bukan rata-rata seluruh wilayah.
- Tabel Wikipedia dapat berubah; berkas HTML mentah disimpan agar hasil dapat diulang.
