Sistem Pendukung Keputusan (SPK) Pemilihan Kendaraan Listrik Menggunakan Metode Simple Additive Weighting (SAW)
<p>
  <a href="https://www.python.org/"><img src="https://img.shields.io/badge/Python-3.9%2B-blue.svg" alt="Python"></a>
  <a href="https://jupyter.org/"><img src="https://img.shields.io/badge/Jupyter-Notebook-orange.svg" alt="Jupyter Notebook"></a>
  <a href="#formulasi-matematis-metode-saw--uji-sensitivitas"><img src="https://img.shields.io/badge/Method-SAW%20(MCDM)-brightgreen.svg" alt="MCDM"></a>
  <a href="#informasi-akademik--sitasi"><img src="https://img.shields.io/badge/UMM-Teknik%20Informatika-red.svg" alt="University"></a>
</p>
> Repositori ini berisi implementasi kode dan *pipeline* eksperimen untuk Tugas Akhir/Skripsi berjudul **"Sistem Pendukung Keputusan Pemilihan Kendaraan Listrik Menggunakan Metode Simple Additive Weighting (SAW) Berbasis Multi-Kriteria Spesifikasi Teknis dan Analisis Sensitivitas Pembobotan"**.
---
Latar Belakang
Perkembangan pasar kendaraan listrik (Electric Vehicles / EV) menghadirkan ratusan varian model dengan spesifikasi teknis yang saling bertolak belakang (trade-off)—seperti kapasitas baterai, jarak tempuh, efisiensi konsumsi energi, daya pengisian cepat DC, akselerasi, dan kecepatan maksimum. Kondisi ini memicu information overload yang menyulitkan calon pengguna dalam menentukan pilihan kendaraan secara objektif.
Berbeda dengan pendekatan berbasis opini media sosial yang cenderung subjektif dan rentan noise, penelitian ini menerapkan pendekatan Multi-Criteria Decision Making (MCDM) yang deterministik menggunakan metode Simple Additive Weighting (SAW) berbasis data kuantitatif spesifikasi teknis pabrikan.
---
Tujuan Penelitian
Adapun tujuan utama dari pengembangan sistem dan eksperimen pada repositori ini meliputi:
Pemodelan SPK Multi-Kriteria: Memodelkan pipeline SPK berbasis metode SAW menggunakan 6 kriteria spesifikasi teknis utama pada 478 varian kendaraan listrik.
Komparasi Skenario Pembobotan: Menganalisis perbandingan urutan peringkat rekomendasi antara skenario pembobotan netral (Equal Weighting / `Baseline`) dengan skenario pembobotan berbasis preferensi pengguna (`S1_Range`, `S2_Performance`, dan `S3_Efficiency_Charging`).
Evaluasi Sensitivitas & Kestabilan Peringkat: Mengevaluasi tingkat kestabilan pemeringkatan dan fenomena pergeseran peringkat (rank reversal) menggunakan evaluasi multi-metrik: Spearman's Rank Correlation ($\rho$) secara global dan Weighted Similarity ($WS$) Rank Coefficient pada jajaran rekomendasi teratas (Top-10).
---
Arsitektur Pipeline Sistem
Mengacu pada rancangan arsitektur sistem dalam penelitian, pipeline komputasi dibagi ke dalam lima lapisan (layer) pemrosesan berurutan:
```text
+-------------------------------------------------------------------------+
| 1. Data Ingestion Layer                                                 |
|    Memuat dataset sekunder EV-Database (478 varian EV, 2024-2025)        |
+------------------------------------+------------------------------------+
                                     |
                                     v
+-------------------------------------------------------------------------+
| 2. Data Preprocessing & Normalization Layer                             |
|    Validasi duplikasi, penanganan missing values, dan normalisasi [0,1] |
+------------------------------------+------------------------------------+
                                     |
                                     v
+-------------------------------------------------------------------------+
| 3. MCDM Core Computation Layer                                          |
|    Perhitungan skor preferensi SAW (V_i) pada 4 skenario pembobotan     |
+------------------------------------+------------------------------------+
                                     |
                                     v
+-------------------------------------------------------------------------+
| 4. Evaluation & Sensitivity Layer                                       |
|    Analisis Rank Reversal, Top-10 Overlap, Spearman's Rho & WS Rank     |
+------------------------------------+------------------------------------+
                                     |
                                     v
+-------------------------------------------------------------------------+
| 5. Presentation Layer                                                   |
|    Tabel komparasi Top-10 rekomendasi & visualisasi Heatmap korelasi    |
+-------------------------------------------------------------------------+
```
---
Dataset & Kriteria Evaluasi
1. Sumber Data & Pra-pemrosesan
Penelitian ini menggunakan dataset sekunder terstandarisasi dari EV-Database (edisi 2024–2025) (`electric_vehicles_spec_2025_new.csv`) yang mencakup $m = 478$ alternatif mobil listrik penumpang. Tahapan pra-pemrosesan meliputi:
Pemeriksaan Duplikasi: Validasi integritas baris data (ditemukan $0$ baris duplikat).
Penanganan Missing Values:
Pengisian nilai kosong pada kolom `model` untuk merek firefly menggunakan identitas mereknya.
Imputasi nilai $0$ pada atribut daya pengisian cepat DC (`fast_charging_power_kw_dc`) untuk varian Renault 5 E-Tech 40kWh 95hp yang tidak mendukung pengisian cepat DC.
2. Pemetaan Kriteria Spesifikasi Teknis
Evaluasi dilakukan terhadap $n = 6$ kriteria teknis terukur yang dikelompokkan ke dalam kategori Benefit (semakin besar semakin baik) dan Cost (semakin kecil semakin baik):
Kode	Nama Atribut Dataset	Deskripsi Kriteria	Satuan	Sifat Kriteria
$C_1$	`range_km`	Jarak tempuh maksimal dalam satu kali pengisian	$\text{km}$	Benefit
$C_2$	`top_speed_kmh`	Kecepatan puncak kendaraan	$\text{km/jam}$	Benefit
$C_3$	`battery_capacity_kWh`	Kapasitas baterai berguna (usable battery)	$\text{kWh}$	Benefit
$C_4$	`fast_charging_power_kw_dc`	Daya maksimum pengisian cepat arus searah (DC)	$\text{kW}$	Benefit
$C_5$	`acceleration_0_100_s`	Waktu akselerasi dari $0$ hingga $100\text{ km/jam}$	$\text{detik}$	Cost
$C_6$	`efficiency_wh_per_km`	Tingkat konsumsi energi listrik rata-rata	$\text{Wh/km}$	Cost
---
Formulasi Matematis Metode SAW & Uji Sensitivitas
1. Normalisasi Matriks Keputusan ($R$)
Setiap elemen matriks keputusan $x_{ij}$ (alternatif ke-$i$, kriteria ke-$j$) dinormalisasi ke dalam rentang skala $[0, 1]$ menggunakan persamaan linier SAW:
Untuk Kriteria Benefit ($C_1, C_2, C_3, C_4$):
$$r_{ij} = \frac{x_{ij}}{\max_{i}(x_{ij})}$$
Untuk Kriteria Cost ($C_5, C_6$):
$$r_{ij} = \frac{\min_{i}(x_{ij})}{x_{ij}}$$
2. Perhitungan Nilai Preferensi Akhir ($V_i$)
Nilai preferensi akhir untuk setiap alternatif kendaraan listrik dihitung melalui penjumlahan terbobot:
$$V_i = \sum_{j=1}^{n} w_j \cdot r_{ij}, \quad \text{dengan } \sum_{j=1}^{n} w_j = 1$$
3. Skenario Pengujian Pembobotan ($W$)
Untuk menguji responsivitas algoritma dan sensitivitas peringkat, dilakukan simulasi terhadap empat skenario pembobotan:
`Baseline` (Equal Weighting): Seluruh kriteria diberi bobot setara sebesar $w_j = \frac{1}{6} \approx 0,1667$ sebagai titik acuan netral tanpa bias prioritas.
`S1_Range` (Fokus Daya Jelajah): Memberikan bobot dominan pada jarak tempuh (`range_km` $= 0,60$) untuk merespons kebutuhan perjalanan jarak jauh (range anxiety).
`S2_Performance` (Fokus Performa Dinamik): Memprioritaskan kecepatan maksimum (`top_speed_kmh`) dan akselerasi (`acceleration_0_100_s`).
`S3_Efficiency_Charging` (Fokus Efisiensi & Pengisian Daya): Memprioritaskan kehematan konsumsi daya (`efficiency_wh_per_km`) serta kecepatan pengisian daya DC (`fast_charging_power_kw_dc`).
4. Metrik Evaluasi Kestabilan Peringkat
Spearman's Rank Correlation ($\rho$): Mengukur konsistensi korelasi peringkat secara global di seluruh populasi $N = 478$ alternatif:
$$\rho = 1 - \frac{6 \sum_{i=1}^{N} d_i^2}{N(N^2 - 1)}$$
di mana $d_i$ merupakan selisih peringkat alternatif ke-$i$ antara skenario pengujian dan skenario `Baseline`.
Weighted Similarity ($WS$) Rank Coefficient: Mengukur sensitivitas pergeseran peringkat dengan memberikan bobot penalti eksponensial lebih tinggi pada perubahan posisi di jajaran peringkat atas (Top-10):
$$WS = 1 - \sum_{i=1}^{N} \left( 2^{-R_{xi}} \cdot \frac{|R_{xi} - R_{yi}|}{\max{|1 - R_{xi}|, |N - R_{xi}|}} \right)$$
---
Ringkasan Hasil Eksperimen
1. Rekomendasi Puncak (#1) Setiap Skenario
Hasil komputasi menunjukkan bahwa metode SAW mampu menyesuaikan rekomendasi puncak secara adaptif sesuai profil bobot pengguna:
Skenario Pembobotan	Orientasi Preferensi	Kendaraan Rekomendasi Peringkat #1	Skor Preferensi ($V_i$)
`Baseline`	Netral (Equal Weighting)	Porsche Taycan Turbo GT Weissach	`0.843247`
`S1_Range`	Jarak Tempuh Maksimal	Lucid Air Grand Touring	`0.896906`
`S2_Performance`	Performa & Akselerasi	Porsche Taycan Turbo GT Weissach	`0.931436`
`S3_Efficiency_Charging`	Efisiensi & Fast Charging	Mercedes-Benz CLA 250+	`0.817880`
2. Temuan Analisis Sensitivitas
Korelasi Global terhadap `Baseline`: Seluruh skenario khusus mempertahankan korelasi positif yang kuat terhadap `Baseline`, yakni $\rho = 0,817$ (`S3_Efficiency_Charging`), $\rho = 0,814$ (`S1_Range`), dan $\rho = 0,788$ (`S2_Performance`).
Deteksi Trade-Off Antar-Skenario Ekstrem: Korelasi menurun signifikan saat membandingkan skenario dengan orientasi teknis yang berlawanan, seperti `S1_Range` vs `S2_Performance` ($\rho = 0,409$) dan `S2_Performance` vs `S3_Efficiency_Charging` ($\rho = 0,435$). Hal ini mengonfirmasi secara empiris adanya trade-off fisik antara kendaraan berdaya pacu tinggi dengan efisiensi konsumsi energi maupun jarak tempuh.
---
Struktur Repositori
```text
├── data/
│   └── electric_vehicles_spec_2025_new.csv   # Dataset spesifikasi teknis 478 varian EV
├── SPK_EV_SAW_Sensitivity_Analysis.ipynb # Notebook utama pipeline SAW & uji sensitivitas
└── README.md                                 # Dokumentasi proyek
```
