# 🚌 Transjakarta Passenger & Transaction Analysis
### End-to-End Data Engineering & Analytics Pipeline | April 2023

---

## 📌 Project Overview

TransJakarta adalah moda transportasi publik utama Jakarta yang melayani ratusan ribu penumpang setiap harinya. Project ini membangun **pipeline data end-to-end** — mulai dari raw data ingestion, cleaning, validation, hingga loading ke cloud database — untuk menghasilkan insight tentang pola perjalanan penumpang, kepadatan halte, dan performa koridor.

**Business Questions yang dijawab:**
- Kapan jam dan hari tersibuk di sistem TransJakarta?
- Halte mana yang menjadi bottleneck kapasitas?
- Bagaimana profil demografis penumpang dan implikasinya terhadap layanan?
- Koridor mana yang menghasilkan pendapatan tertinggi?

---

## 👨‍💻 My Contribution

Sebagai **Data Engineer** di project ini, saya bertanggung jawab atas seluruh backend pipeline dari raw data hingga analytics-ready database.

### 🧹 Data Cleaning (`data_cleaning.ipynb`)
- Menangani **transaksi tap-in tanpa tap-out** — mengidentifikasi records dengan `tapOutStops` dan `tapOutHours` yang null, kemudian menerapkan strategi null-imputation untuk menjaga integritas transaction count tanpa mendistorsi metrik durasi perjalanan
- Standardisasi tipe kolom, penghapusan duplikat, dan validasi koordinat geografis halte untuk keperluan pemetaan

### ⚙️ ETL Pipeline — Apache Airflow (`DAG_Final_Project.py`)
- Merancang dan mengimplementasikan **3-task DAG**: `extract_data → transform_data → load_data` dengan hourly schedule untuk mensimulasikan near-real-time ingestion pipeline
- Mengkonfigurasi task dependencies dan memastikan idempotent load ke PostgreSQL

### ✅ Data Validation — Great Expectations (`GE_Project.ipynb`)
- Mendefinisikan **11 data quality expectations** yang mencakup uniqueness, nullability, value constraints, dan referential integrity
- Validasi kritis: unique `transID`, valid gender values (`M`/`F`), non-null corridor names, balance ≥ 0, tap-in hours dalam range 0–23, dan referential integrity antara fact table dan dimension tables

### 🗄️ Data Load — PostgreSQL / Neon Tech
- Memuat data ke cloud PostgreSQL menggunakan **Star Schema** (1 fact table + 4 dimension tables)
- Schema dirancang untuk mendukung analytical queries pada dashboard perilaku penumpang dan performa koridor

---

## 🏗️ Pipeline Architecture

```
Kaggle CSV
    │
    ▼
[Extract] PySpark
    │
    ▼
[Transform] Data Cleaning & Modelling
    │
    ▼
[Validate] Great Expectations (11 expectations)
    │
    ▼
[Load] PostgreSQL — Neon Tech (Cloud)
    │
    ▼
[Orchestrate] Apache Airflow (DAG, hourly)
    │
    ▼
[Visualize] Tableau Dashboard
```

---

## 🗃️ Data Modelling — Star Schema

| Tabel | Deskripsi |
|---|---|
| `transaction_fact` | Data transaksi utama (TransID, payCardID, halteID tap in/out, corridorID, duration, dll) |
| `user_dimension` | Data pelanggan (payCardID, bank, nama, jenis kelamin, tanggal lahir) |
| `corridor_dimension` | Data koridor (corridorID, corridorName) |
| `halte_dimension` | Data halte (halteID, halteName, halteCity, halteLat, halteLon) |
| `date_dimension` | Data tanggal (date, month, quarter, half_year, year) |

---

## ✅ Data Validation — Great Expectations

11 ekspektasi diterapkan untuk memastikan kualitas data:

| # | Kolom | Expectation | Alasan |
|---|---|---|---|
| 1 | `transID` | Harus unik | Mencegah duplikasi transaksi |
| 2 | `transID` | Tidak boleh null | Primary key integrity |
| 3 | `payCardID` | Harus unik per user | Mencegah duplikasi user |
| 4 | `payCardSex` | Hanya `M` atau `F` | Validasi domain value |
| 5 | `payCardBirthdate` | Harus 4 karakter (YYYY) | Format konsistensi |
| 6 | `corridorID` | Harus unik | Dimension integrity |
| 7 | `corridorName` | Tidak boleh null | Data completeness |
| 8 | `halteID` | Harus unik | Dimension integrity |
| 9 | `payCardBalance` | Harus ≥ 0 | No negative balance |
| 10 | `tapInHours` | Dalam range 0–23 | Valid hour range |
| 11 | `corridorID` (fact) | Harus ada di corridor_dimension | Referential integrity |

---

## ⚙️ Airflow DAG

DAG `FinalProject_RMT-004` dengan jadwal **hourly** untuk mensimulasikan near-real-time ingestion:

```
extract_data  →  transform_data  →  load_data
```

- **Schedule:** `@hourly`
- **Start date:** 8 Januari 2025
- **Dependency:** Linear, task berikutnya hanya jalan jika task sebelumnya sukses
- **Run history:** 14 total runs — 3 success, 11 failed (lihat Challenges untuk detail)

---

## 📊 Key Findings & Business Recommendations

| Insight | Data | Rekomendasi |
|---|---|---|
| **Penumpang mayoritas usia 25–40 tahun** | 16,293 penumpang di kelompok ini; turun 47% di usia 41–59 dan 91% di usia 60–80 | Fokus promosi & partnership pada segmen pekerja kantoran; prioritaskan keandalan jam commuter 06.00–09.00 & 17.00–20.00 |
| **Weekday vs weekend berbeda signifikan secara statistik** | ~64K/hari (weekday) vs ~17K/hari (weekend) — turun 73.4%; dikonfirmasi Two-Sample T-Test: p=0.0029, T-stat=2.97 | Kurangi frekuensi armada weekend, realokasi ke weekday peak hours |
| **Jakarta Barat: underserved** | Penduduk terbesar ke-2 (2.58 juta) tapi rasio 1 halte per 5.442 penduduk — terendah se-Jakarta | Evaluasi penambahan halte di koridor padat Jakarta Barat; investigasi apakah ada preferensi kendaraan lain |
| **Halte BKN: destinasi tertinggi** | 306 transaksi tap-out | Tambah kapasitas shelter & frekuensi armada koridor menuju BKN di jam 07.00–09.00 |
| **Halte Penjaringan: keberangkatan tertinggi** | 224 transaksi tap-in | Evaluasi kapasitas halte — risiko overcrowding di jam sibuk |
| **St. MRT Fatmawati: pendapatan tertinggi** | Rp 1,760,000/bulan; disusul Kejaksaan Agung Rp 1,221,000 | Jaga konektivitas integrasi MRT-TransJakarta; potensi bundling tiket multi-moda |
| **Durasi distribusi right-skewed** | Rata-rata 72 menit; outlier hingga 179 menit | Investigasi perjalanan >120 menit — kemungkinan bottleneck di koridor tertentu atau waktu tunggu armada |
| **53.3% penumpang perempuan** | Konsisten di semua wilayah | Pastikan ketersediaan bus khusus perempuan (pink bus) di halte dan jam tersibuk |
| **Jakarta Timur: halte terbanyak tapi juga tersepi** | 1,105 halte; 308 halte keberangkatan hanya punya 1 transaksi/bulan | Audit utilisasi halte — pertimbangkan konsolidasi halte sepi atau penambahan feeder route |

---

## 📈 Dashboard

| Dashboard | Link |
|---|---|
| 🧑‍🤝‍🧑 Dashboard Penumpang | [Lihat di Tableau Public](https://public.tableau.com/views/Book2_17418801072280/DashbaordPenumpang) |
| 🚏 Dashboard Halte | [Lihat di Tableau Public](https://public.tableau.com/views/Book2_17418801072280/DashboardHalte) |

> 📸 *Screenshot preview dashboard menyusul — akses langsung via link di atas*

---

## 🧱 Challenges & Lessons Learned

Tantangan teknis yang dihadapi selama pengerjaan project ini:

**Pipeline & Infrastruktur**
- **PySpark → Neon DB (PostgreSQL cloud):** Koneksi dari PySpark ke Neon Tech memerlukan konfigurasi JDBC yang tidak straightforward — driver dan connection string harus disesuaikan secara manual
- **Geopandas di Docker:** Library `geopandas` memerlukan sistem-level dependencies (`libgdal`, `libgeos`) yang tidak tersedia di base Airflow Docker image; solusinya dengan custom `Dockerfile` yang meng-install dependencies tersebut
- **Airflow DAG instability:** Dari 14 total runs, 11 mengalami failure sebelum pipeline stabil — mayoritas disebabkan race condition antara task transform dan load serta timeout koneksi ke Neon DB

**Data Engineering**
- **Geolokasi luar Jakarta:** Saat menggunakan shapefiles untuk mapping halte ke kota, beberapa halte di luar Provinsi Jakarta tidak ter-identifikasi — diatasi dengan fallback rule berbasis koordinat GPS
- **Tap-in tanpa tap-out:** Transaksi incomplete (ada tap-in tapi tidak ada tap-out) memerlukan strategi imputation khusus agar tidak mendistorsi metrik durasi perjalanan
- **Schema evolution mid-project:** Data analyst membutuhkan kolom tambahan (`seq_amount`, `duration`, `halteCity`) yang tidak diprediksi di awal data modelling — pipeline harus direvisi di tengah pengerjaan

---

## 📂 Dataset

- **Sumber:** [Kaggle — Transjakarta Public Transportation Transaction](https://www.kaggle.com/datasets/dikisahkan/transjakarta-transportation-transaction)
- **Periode:** 01 April 2023 – 30 April 2023
- **Ukuran:** 379,000 baris × 22 kolom
- **Konten:** Data Transaksi, Pengguna, Koridor, dan Halte

---

## 🛠️ Tech Stack

[![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)](https://python.org)
[![SQL](https://img.shields.io/badge/SQL-4479A1?style=flat&logo=postgresql&logoColor=white)](https://postgresql.org)
[![Apache Spark](https://img.shields.io/badge/PySpark-E25A1C?style=flat&logo=apachespark&logoColor=white)](https://spark.apache.org)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-336791?style=flat&logo=postgresql&logoColor=white)](https://postgresql.org)
[![Apache Airflow](https://img.shields.io/badge/Apache_Airflow-017CEE?style=flat&logo=apacheairflow&logoColor=white)](https://airflow.apache.org)
[![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white)](https://docker.com)
[![Great Expectations](https://img.shields.io/badge/Great_Expectations-FF6B35?style=flat)](https://greatexpectations.io)
[![Tableau](https://img.shields.io/badge/Tableau-E97627?style=flat&logo=tableau&logoColor=white)](https://tableau.com)
[![Jupyter](https://img.shields.io/badge/Jupyter-F37626?style=flat&logo=jupyter&logoColor=white)](https://jupyter.org)

---

## 📁 Repository Structure

```
transjakarta-data-analysis/
├── Airflow/
│   ├── dags/
│   │   └── DAG_Final_Project.py       # Airflow DAG definition
│   ├── script/
│   │   ├── extract_fp.py              # Extract logic
│   │   ├── transform_fp.py            # Transform logic
│   │   └── load_fp.py                 # Load to PostgreSQL
│   └── data/                          # Partitioned output CSVs per wilayah
├── GE_Project.ipynb                   # Great Expectations validation
├── data_cleaning.ipynb                # Data cleaning & preprocessing
└── README.md
```

---

## ⚙️ Setup & Requirements

```bash
# Clone repository
git clone https://github.com/adhirizqi/transjakarta-data-analysis.git
cd transjakarta-data-analysis

# Install dependencies
pip install -r requirements.txt

# Run Airflow (requires Docker)
docker-compose up airflow-init
docker-compose up
```

> **Note:** Untuk menjalankan pipeline lengkap, diperlukan akses ke PostgreSQL instance (Neon Tech) dan dataset dari Kaggle.

---

## 🙏 Acknowledgement

Project ini dikerjakan sebagai bagian dari **Hacktiv8 CODA Data Analyst Bootcamp** secara tim.
Dataset bersumber dari [Kaggle](https://www.kaggle.com/datasets/dikisahkan/transjakarta-transportation-transaction).
