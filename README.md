# eBay Women's Perfume: Advanced Data Preprocessing & Market Landscape Pipeline 🧪🛍️

![Project Status](https://img.shields.io/badge/Status-Completed-success)
![Tools](https://img.shields.io/badge/Tools-Python%20%7C%20Pandas%20%7C%20Scikit--Learn-blue)
![Domain](https://img.shields.io/badge/Domain-E--Commerce%20%26%20Fragrance-purple)

---

## 📌 Business Overview & Problem Statement

Dalam industri retail kosmetik dan wewangian (*fragrance*) di *marketplace* global seperti eBay, data hasil *web scraping* merupakan aset berharga untuk memetakan pergerakan kompetitor, strategi penetapan harga, dan pola manajemen stok. Namun, data mentah dari *web scraping* seringkali dipenuhi masalah kualitas: format teks tidak seragam, kesalahan ketik (*typo*), nilai yang hilang (*missing values*), hingga distribusi data yang sangat timpang akibat angka penjualan produk *best-seller* yang ekstrem.

Proyek ini membangun **pipeline preprocessing data *end-to-end* yang tangguh** menggunakan Python terhadap dataset *eBay Women's Perfume Transactions* (**1.000 baris, 10 kolom**) — mentransformasi data mentah yang tidak andal menjadi *model-grade dataset* yang siap digunakan oleh tim produk, logistik, pemasaran, maupun pemodelan Machine Learning.

> **"Bagaimana cara membangun pipeline preprocessing yang robust untuk mengubah data e-commerce hasil web scraping yang kotor menjadi dataset berkualitas korporat yang siap dianalisis dan dimodelkan?"**

---

## 🗂️ Repository Structure

```
ebay-perfume-preprocessing-pipeline
│
├── README.md                                   # Dokumentasi utama proyek
├── data/
│   └── ebay_womens_perfume.csv                 # Dataset mentah hasil web scraping
├── notebook/
│   └── Datapreprocessing_Benedictus.ipynb      # Jupyter Notebook cleaning & manipulasi data
├── output/
│   └── clean_perfume_dataset.csv               # Output dataset final siap dimodelkan
└── report/
    └── Mini_Project_Data_Preprocessing.pdf     # Laporan presentasi infografis proyek
```

---

## 💻 Tech Stack & Data Pipeline

Pipeline preprocessing ini dibangun menggunakan ekosistem *data science* Python dengan pendekatan bertahap dan terstruktur.

### 1. Data Cleaning & Conditional Imputation

| Tahap | Metode | Keterangan |
|-------|--------|-----------|
| **Handling Duplicates** | `drop_duplicates(keep='first')` | Mengidentifikasi dan menghapus 1 baris data duplikat mutlak |
| **Text Standardization (Type)** | String matching & dictionary mapping | Menyederhanakan ratusan variasi penulisan tipe parfum menjadi **9 kategori utama** (EDP, EDT, dll.) |
| **Text Standardization (Brand)** | Dictionary mapping | Merapikan *typo* dan variasi penulisan dari **235 → 211 brand unik** |
| **Missing Value — `lastUpdated`** | Mode imputation | Diisi menggunakan tanggal dan jam yang paling sering muncul dalam dataset |
| **Missing Value — `sold`** | Fill dengan `0` | Divalidasi bahwa 16 baris kosong merupakan listing baru yang belum pernah terjual |
| **Missing Value — `available`** | Rule-based conditional imputation | Jika `availableText` mengandung *"out of stock"* → diisi `0`; *"last one"* → diisi `1`; sisanya diisi **Median** |
| **Kolom Redundan** | `drop(columns=[...])` | Menghapus `priceWithCurrency` setelah divalidasi isinya identik dengan kolom `price` |

### 2. Outlier Strategy & Normality Testing

**Keputusan: Natural Outliers Dipertahankan**

Analisis IQR mendeteksi pencilan masif pada kolom `price`, `sold`, dan `available`. Namun, diputuskan untuk **tidak menghapus maupun melakukan capping** karena:
- Angka ekstrem pada `price` mencerminkan kondisi riil produk parfum *luxury* yang memang bernilai tinggi
- Angka ekstrem pada `sold` dan `available` mencerminkan produk *best-seller* dengan permintaan masif di pasar

**Uji Normalitas & Transformasi:**

| Kolom | Metode Terbaik | P-Value | Status |
|-------|---------------|---------|--------|
| `price` | Box-Cox | > 0.05 | ✅ Normal setelah transformasi |
| `sold` | Log Transform | < 0.05 | ⚠️ Tetap tidak normal, skewness diredam |
| `available` | Log Transform | < 0.05 | ⚠️ Tetap tidak normal, skewness diredam |

### 3. Feature Engineering

| Fitur Baru | Sumber | Keterangan |
|------------|--------|-----------|
| **`revenue`** | `price × sold` | Kalkulasi total pendapatan per listing |
| **`shipping_days`** | `ship_date - order_date` | Durasi pengiriman barang |
| **`month` & `day`** | Ekstrak dari `lastUpdated` | Dimensi temporal untuk analisis pola waktu |
| **`city`, `state`, `country`** | Split dari `itemLocation` | Memecah alamat tidak seragam menjadi 3 kolom geografis yang rapi |

**Advanced Scaling:**

| Kolom | Scaler | Alasan |
|-------|--------|--------|
| `price` | StandardScaler | Data sudah berdistribusi normal setelah Box-Cox |
| `sold`, `available` | RobustScaler (Median & IQR) | Data masih memiliki outlier — RobustScaler tidak terdistorsi oleh nilai ekstrem |

### 4. Categorical Encoding Strategy

| Kolom | Metode | Alasan |
|-------|--------|--------|
| `type`, `country` | One-Hot Encoding | Kardinalitas rendah — setiap kategori dibuat kolom biner tersendiri |
| `brand` (211 unik), `state` | Binary Encoding | Kardinalitas tinggi — diringkas ke kode biner agar dataset tidak terlalu lebar |
| `title` (975 unik) | Hash Encoding (16 komponen) | Teks sangat panjang dan bervariasi — dipetakan ke 16 angka untuk menangkap pola kata kunci |

---

## 🔍 Key Insights & Market Findings

### 1. 🌸 Dominasi Pasar Eau de Parfum (EDP)

Produk parfum wanita di eBay didominasi kuat oleh tipe **Eau de Parfum**. Hal ini mencerminkan perilaku konsumen yang memprioritaskan ketahanan aroma jangka panjang saat belanja *online*, meskipun harga EDP relatif lebih mahal dibanding *Eau de Toilette* (EDT). Sebaliknya, tipe *Extrait de Parfum* hanya memiliki **4 listing** — mengindikasikan bahwa pasar eBay tidak didominasi segmen parfum *luxury* kelas atas.

### 2. 🌍 Sentralisasi Logistik Pasar

| Negara Asal | Jumlah Listing | Keunggulan Strategis |
|-------------|---------------|---------------------|
| Amerika Serikat | 921 | Pengiriman lokal — ongkir lebih murah, waktu lebih cepat |
| Hong Kong | 2nd | Pelabuhan bebas pajak — harga produk *luxury* tetap kompetitif secara global |

### 3. 💰 Paradoks Strategi Volume vs Margin

| Brand | Revenue | Volume Penjualan | Harga Median | Strategi |
|-------|---------|-----------------|-------------|---------|
| **Calvin Klein** | $1.2 Juta (#1) | 615 unit | ~$25 | Volume tinggi, harga terjangkau |
| **Burberry** | Top 4 | 95 unit | Tinggi | Margin tebal per unit (*luxury*) |
| **Gloria** | — | Tertinggi | Sangat murah | *Mass-market*, perputaran cepat |

---

## 💡 Strategic Recommendations

Rekomendasi ini dirancang secara taktis untuk menguntungkan semua divisi internal dengan pendekatan *win-win* dan efisiensi biaya tinggi (*low-cost, high-impact*).

**1. Optimalisasi Alokasi Anggaran Produk (Tim Merchandising)**

Bagi modal pembelian ke dua jalur: **70%** untuk brand volume tinggi seperti *Calvin Klein* (fokus tipe EDP), dan **30%** untuk brand *high-margin* seperti *Burberry*. Hindari investasi pada kategori *Extrait de Parfum* yang terbukti sangat lambat bergerak di pasar eBay.

*Win-win:* Tim Merchandising mencapai KPI *inventory turnover*, divisi Finansial mendapatkan arus kas yang sehat dan dapat diprediksi.

**2. Efisiensi Jalur Pasokan via Hub Hong Kong (Tim Logistik)**

Pusatkan rute pengiriman internasional produk *luxury* melalui hub Hong Kong guna memanfaatkan regulasi kawasan bebas pajak secara legal — berpotensi memotong biaya bea cukai masuk hingga **15%**.

*Win-win:* Tim Operasional menekan *landed cost* per unit, Tim Sales mendapatkan ruang untuk memasang harga eceran yang lebih kompetitif.

**3. Sistem Otomatisasi Alert Stok Kritis (Tim E-Commerce & Gudang)**

Manfaatkan *threshold* dari kalkulasi RobustScaler di pipeline ini untuk membangun sistem peringatan dini (*early warning alert*) stok. Brand dengan perputaran sangat cepat seperti *Gloria* wajib diprioritaskan — sistem otomatis mengirim perintah *restock* ke supplier saat stok menyentuh angka median kritis.

*Win-win:* Tim E-Commerce terhindar dari *stockout loss*, Tim Gudang mendapatkan jadwal kedatangan barang yang lebih teratur dan terencana — tanpa perlu membeli lisensi software manajemen inventaris tambahan.

---

## 🖥️ Dashboard Preview



---

## 👤 Author

**Benedictus Irvanda Nugroho**
Data Analytics Portfolio Project · 2026

Focused on transforming raw business data into actionable insights through systematic data cleaning, rigorous exploratory analysis, and business intelligence reporting.

* **LinkedIn:** [irvandanugroho](https://linkedin.com/in/irvandanugroho/)
* **GitHub:** [Irvanda08](https://github.com/Irvanda08)
* **Email:** irvandanugroho08@gmail.com

---

*Proyek ini diselesaikan sebagai bagian dari portofolio profesional Data Analytics (2026).*
