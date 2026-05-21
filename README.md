# 🚌 Analisis Perilaku, Transaksi dan Distribusi Halte TransJakarta
### Periode April 2023 | Hacktiv8 CODA-RMT-004

---

## 👥 Tim

| Nama | Role |
|------|------|
| Adhi Rizqi A | Data Engineer |
| Dimas Prasetyo | Data Engineer |
| Davita Nabila R | Data Analyst |
| Allyra Himawaty | Data Analyst |

---

## 📌 Deskripsi Project

TransJakarta merupakan salah satu moda transportasi utama di Jakarta yang melayani jutaan penumpang setiap harinya. Analisis terhadap perilaku dan transaksi penumpang menjadi penting untuk memahami pola perjalanan dan kebutuhan layanan transportasi publik.

Project ini mencakup:
- Pemantauan data transaksi harian penumpang
- Analisis waktu perjalanan (tap-in dan tap-out)
- Preferensi rute dan halte
- Identifikasi tren utama, puncak kepadatan, serta kebutuhan peningkatan infrastruktur

---

## 📂 Dataset

- **Sumber:** [Kaggle — Transjakarta Public Transportation Transaction](https://www.kaggle.com/datasets/dikisahkan/transjakarta-transportation-transaction)
- **Periode:** 01 April 2023 – 30 April 2023
- **Ukuran:** 379,000 baris × 22 kolom
- **Konten:** Data Transaksi, Pengguna, Koridor, dan Halte

---

## 🏗️ Arsitektur Pipeline

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
[Validate] Great Expectations
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

## 🗃️ Data Modelling (Star Schema)

| Tabel | Deskripsi |
|-------|-----------|
| `transaction_fact` | Data transaksi utama (TransID, payCardID, halteID tap in/out, corridorID, duration, dll) |
| `user_dimension` | Data pelanggan (payCardID, bank, nama, jenis kelamin, tanggal lahir) |
| `corridor_dimension` | Data koridor (corridorID, corridorName) |
| `halte_dimension` | Data halte (halteID, halteName, halteCity, halteLat, halteLon) |
| `date_dimension` | Data tanggal (date, month, quarter, half_year, year) |

---

## ✅ Data Validation (Great Expectations)

8 ekspektasi diterapkan untuk memastikan kualitas data:

1. Nilai pada kolom `transID` harus unik
2. Kolom `transID` tidak boleh ada nilai null
3. Nilai pada kolom `paycard_id` harus unik
4. Kolom `paycardsex` harus berisi `M` atau `F`
5. Kolom `paycardbirthdate` harus berisi 4 karakter
6. Nilai pada kolom `corridorID` harus unik
7. Kolom `corridorName` tidak boleh ada nilai null
8. Nilai pada kolom `halteID` harus unik

---

## ⚙️ Workflow — Apache Airflow

DAG `FinalProject_RMT-004` dengan jadwal interval **setiap 1 jam**:

```
extract_data  →  transform_data  →  load_data
```

---

## 📊 Key Findings

| Insight | Detail |
|---------|--------|
| 👥 Mayoritas penumpang | Usia **25–40 tahun** (pekerja kantoran) |
| ♀️ Distribusi gender | **53.3% perempuan**, 46.7% laki-laki |
| 📅 Penumpang weekday vs weekend | ~64K/hari vs ~17K/hari (**turun 73.4%**) |
| 🏙️ Halte terbanyak | Jakarta Timur (1,105 halte) |
| 🚏 Halte keberangkatan terpadat | **Penjaringan** (224 transaksi) |
| 🏁 Halte tujuan terpadat | **BKN** (306 transaksi) |
| 💰 Halte pendapatan tertinggi | **St. MRT Fatmawati** (Rp 1,760,000/bulan) |
| ⏱️ Rata-rata durasi perjalanan | **72 menit** |
| 🔁 Pola perjalanan | Mayoritas **intra-kota** (tap in & tap out di kota yang sama) |

---

## 📈 Dashboard

| Dashboard | Link |
|-----------|------|
| 🧑‍🤝‍🧑 Dashboard Penumpang | [Lihat di Tableau](https://public.tableau.com/views/Book2_17418801072280/DashbaordPenumpang?:language=en-GB&publish=yes&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link) |
| 🚏 Dashboard Halte | [Lihat di Tableau](https://public.tableau.com/views/Book2_17418801072280/DashboardHalte?:language=en-GB&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link) |

---

## 🛠️ Tech Stack

![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)
![SQL](https://img.shields.io/badge/SQL-4479A1?style=flat&logo=postgresql&logoColor=white)
![Apache Spark](https://img.shields.io/badge/PySpark-E25A1C?style=flat&logo=apachespark&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-336791?style=flat&logo=postgresql&logoColor=white)
![Apache Airflow](https://img.shields.io/badge/Apache_Airflow-017CEE?style=flat&logo=apacheairflow&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white)
![Great Expectations](https://img.shields.io/badge/Great_Expectations-FF6B35?style=flat)
![Tableau](https://img.shields.io/badge/Tableau-E97627?style=flat&logo=tableau&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-F37626?style=flat&logo=jupyter&logoColor=white)

---

## 📁 Struktur Repository

```
transjakarta-analysis/
├── Airflow/
│   └── dags/
│       └── _pycache_/
│       └── DAG_Final_Project.py
│   └── data/
│       └── JAKARTABARAT/
│       └── JAKARTAPUSAT/
│       └── JAKARTASELATAN/
│       └── JAKARTATIMUR/
│       └── JAKARTAUTARA/
│       └── corridor_dimension/
│       └── date_dimension/
│       └── halte_dimension/
│       └── user_dimension/
│       └── transaction_fact/
│       └── extract_fp.csv/
│       └── dfTransjakarta.csv
│   └── script/
│       └── extract_fp.py
│       └── load_fp.py
│       └── transform_fp.py
│   └── APT airflow.png
├── GE_Project.ipynb
├── README.md
├── data_cleaning.ipynb.ipynb
├── df_clean

```


---

## 🙏 Acknowledgement

Project ini merupakan Final Project dari program **Hacktiv8 CODA-RMT-004**.
Dataset bersumber dari [Kaggle](https://www.kaggle.com/datasets/dikisahkan/transjakarta-transportation-transaction).
