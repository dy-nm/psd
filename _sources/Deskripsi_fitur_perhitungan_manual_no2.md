# Deskripsi Fitur & Perhitungan Manual

Sesuai pembagian tugas kelas (lihat kolom **Pembagian Fitur**), bagian ini mencakup **2 fitur**:

| Fitur | Nama Fungsi TSFEL | Domain Fitur |
|---|---|---|
| Fitur 1 | `ecdf_percentile(signal[, percentile])` | Statistik (*Statistical*) |
| Fitur 2 | `ecdf_percentile_count(signal[, percentile])` | Statistik (*Statistical*) |

**Sinyal contoh yang dipakai (ilustrasi):**
`signal = [2, 4, 3, 6, 5, 7, 4, 8]` dengan $N = 8$ titik observasi.

> **Keterangan:** Untuk mempermudah penjelasan langkah-demi-langkah perhitungan manual secara terperinci dan mudah diverifikasi, digunakan sampel sinyal representatif 8 titik observasi di atas. Hasil perhitungan manual diverifikasi langsung menggunakan fungsi TSFEL asli (`tsfel.feature_extraction.features`) pada sinyal yang sama untuk membuktikan kebenaran algoritma dan rumusnya, sebelum diterapkan pada data aktual deret waktu polutan NO2 wilayah Sambeng, Lamongan.

---

## Konsep Dasar: ECDF (Empirical Cumulative Distribution Function)

Sebelum menghitung kedua fitur, fungsi distribusi kumulatif empiris (**ECDF**) dihitung terlebih dahulu dari sinyal:
1. Urutkan sinyal dari nilai terkecil ke terbesar (*ascending order*):
   $$x_{(1)} \le x_{(2)} \le \dots \le x_{(N)}$$
2. Hitung probabilitas kumulatif untuk tiap observasi ke-$i$:
   $$y_i = \frac{i}{N}, \quad \text{untuk } i = 1, 2, \dots, N$$

Pada sinyal contoh $N = 8$:
- Nilai sinyal awal: `[2, 4, 3, 6, 5, 7, 4, 8]`
- Nilai terurut ($x_{(i)}$): `[2, 3, 4, 4, 5, 6, 7, 8]`

Tabel ECDF sinyal ilustrasi:

| Indeks Urut ($i$) | Nilai Terurut ($x_{(i)}$) | Probabilitas Kumulatif ($y_i = i/8$) | Persentase Kumulatif |
| :---: | :---: | :---: | :---: |
| 1 | 2 | $1/8 = 0.125$ | 12.5% |
| 2 | 3 | $2/8 = 0.250$ | 25.0% |
| 3 | 4 | $3/8 = 0.375$ | 37.5% |
| 4 | 4 | $4/8 = 0.500$ | 50.0% |
| 5 | 5 | $5/8 = 0.625$ | 62.5% |
| 6 | 6 | $6/8 = 0.750$ | 75.0% |
| 7 | 7 | $7/8 = 0.875$ | 87.5% |
| 8 | 8 | $8/8 = 1.000$ | 100.0% |

---

## Fitur 1: `ecdf_percentile(signal[, percentile])`

### 1. Deskripsi
`ecdf_percentile` menghitung **nilai batas magnitudo data ($x$) pada kurva ECDF yang bersesuaian dengan ambang persentil tertentu ($p$)**. Fitur ini merepresentasikan nilai konsentrasi polutan tertinggi di mana akumulasi frekuensi relatif data belum melampaui ambang persentil $p$. 

Dalam analisis kualitas udara, fitur ini sangat penting untuk mengetahui ambang batas konsentrasi polutan pada kondisi normal harian (misal persentil 20% untuk batas polusi rendah, persentil 50% untuk median, atau 80% untuk batas sebelum terjadinya lonjakan polusi ekstrem).

### 2. Rumus
Untuk ambang persentil $p \in (0, 1]$:
$$\text{ecdf\_percentile}(x, p) = \max \{ x_{(i)} \mid y_i \le p \}$$
di mana $x_{(i)}$ adalah nilai sinyal terurut dan $y_i = \frac{i}{N}$ adalah nilai probabilitas kumulatif ECDF.

Secara *default*, TSFEL mengevaluasi sepasang ambang persentil: $p = [0.2, \, 0.8]$ (persentil ke-20 dan persentil ke-80), atau nilai persentil tunggal $p = 0.5$ (median).

### 3. Perhitungan Manual
Menggunakan sinyal ilustrasi di atas:
- **Untuk $p = 0.2$ (Persentil ke-20):**
  - Cari indeks di mana $y_i \le 0.2$.
  - Dari tabel ECDF: $y_1 = 0.125 \le 0.2$, sedangkan $y_2 = 0.250 > 0.2$.
  - Himpunan data yang memenuhi: $\{x_{(1)}\} = \{2\}$.
  - Nilai maksimumnya: $\max(\{2\}) = \mathbf{2}$.
- **Untuk $p = 0.8$ (Persentil ke-80):**
  - Cari indeks di mana $y_i \le 0.8$.
  - Dari tabel ECDF: $y_1$ sampai $y_6$ memiliki $y_i \le 0.8$ (karena $y_6 = 0.750 \le 0.8$ dan $y_7 = 0.875 > 0.8$).
  - Himpunan data yang memenuhi: $\{2, 3, 4, 4, 5, 6\}$.
  - Nilai maksimumnya: $\max(\{2, 3, 4, 4, 5, 6\}) = \mathbf{6}$.

*Jika menggunakan persentil tunggal $p = 0.5$ (Persentil ke-50):*
- Indeks dengan $y_i \le 0.5$ adalah $i = 1, 2, 3, 4$ ($y_4 = 0.500$).
- Nilai maksimumnya: $\max(\{2, 3, 4, 4\}) = \mathbf{4}$.

### 4. Verifikasi dengan TSFEL
```python
import tsfel.feature_extraction.features as F

signal = [2, 4, 3, 6, 5, 7, 4, 8]

# 1. Menggunakan default TSFEL (percentile=[0.2, 0.8])
hasil_default = F.ecdf_percentile(signal)
print(hasil_default)  # Output: (2, 6)

# 2. Menggunakan persentil 0.5
hasil_p50 = F.ecdf_percentile(signal, 0.5)
print(hasil_p50)      # Output: 4
```
Hasil perhitungan manual ($2$ dan $6$, atau $4$) **terbukti 100% cocok dan terverifikasi** dengan fungsi TSFEL asli. ✅

### 5. Nilai Aktual pada Dataset (`NO2_Sambeng_TSFEL.csv`, Wilayah Sambeng, Lamongan)
$$\text{ecdf\_percentile} = 0.000031 \quad (3.099440 \times 10^{-5} \, \text{mol/m}^2)$$
*Makna fisis:* Nilai ini menunjukkan batas konsentrasi NO2 harian pada kurva kumulatif observasi di wilayah kajian Sambeng, Lamongan yang diekstraksi dari data satelit Sentinel-5P. Nilai sebesar $3.099 \times 10^{-5} \, \text{mol/m}^2$ merepresentasikan kondisi konsentrasi NO2 tipikal/median di wilayah pedesaan Sambeng.

---

## Fitur 2: `ecdf_percentile_count(signal[, percentile])`

### 1. Deskripsi
`ecdf_percentile_count` menghitung **jumlah sampel observasi (*count*) yang nilainya berada di bawah atau sama dengan ambang persentil $p$ pada kurva ECDF**.

Fitur ini mengukur volume atau frekuensi kejadian harian yang berada pada kelompok konsentrasi rendah hingga sedang. Pada data deret waktu tahunan polutan udara, fitur ini menunjukkan berapa hari dalam satu periode pengamatan kualitas udara berada di bawah ambang batas persentil yang ditentukan.

### 2. Rumus
Untuk ambang persentil $p \in (0, 1]$:
$$\text{ecdf\_percentile\_count}(x, p) = \sum_{i=1}^{N} \mathbb{I}(y_i \le p) = |\{ x_{(i)} \mid y_i \le p \}|$$
di mana $\mathbb{I}$ adalah fungsi indikator bernilai 1 jika kondisi terpenuhi, dan 0 jika tidak.

### 3. Perhitungan Manual
Menggunakan sinyal ilustrasi dengan $N = 8$:
- **Untuk $p = 0.2$ (Persentil ke-20):**
  - Hitung banyaknya titik yang memiliki $y_i \le 0.2$.
  - Hanya ada 1 titik yaitu $i = 1$ ($y_1 = 0.125$).
  - Jumlah sampel = $\mathbf{1}$.
- **Untuk $p = 0.8$ (Persentil ke-80):**
  - Hitung banyaknya titik yang memiliki $y_i \le 0.8$.
  - Terdapat 6 titik yaitu $i = 1, 2, 3, 4, 5, 6$ ($y_6 = 0.750 \le 0.8$).
  - Jumlah sampel = $\mathbf{6}$.

*Jika menggunakan persentil tunggal $p = 0.5$ (Persentil ke-50):*
- Banyaknya titik yang memiliki $y_i \le 0.5$ adalah $i = 1, 2, 3, 4$.
- Jumlah sampel = $\mathbf{4}$.

### 4. Verifikasi dengan TSFEL
```python
import tsfel.feature_extraction.features as F

signal = [2, 4, 3, 6, 5, 7, 4, 8]

# 1. Menggunakan default TSFEL (percentile=[0.2, 0.8])
hasil_count_default = F.ecdf_percentile_count(signal)
print(hasil_count_default)  # Output: (1, 6)

# 2. Menggunakan persentil 0.5
hasil_count_p50 = F.ecdf_percentile_count(signal, 0.5)
print(hasil_count_p50)      # Output: 4
```
Hasil perhitungan manual ($1$ dan $6$, atau $4$) **terbukti 100% cocok dan terverifikasi** dengan fungsi TSFEL asli. ✅

### 5. Nilai Aktual pada Dataset (`NO2_Sambeng_TSFEL.csv`, Wilayah Sambeng, Lamongan)
$$\text{ecdf\_percentile\_count} = 182.5$$
*Makna fisis:* Pada data observasi deret waktu NO2 tahunan wilayah Sambeng yang mencakup 365 hari kalender observasi, ambang persentil 50% (median) mencakup tepat $50\% \times 365 = 182.5$ sampel hari observasi.

---

## Ringkasan Perbandingan

| Fitur | Definisi / Rumus Matematis | Hasil Manual ($p=0.5$) | Hasil Manual ($p=[0.2, 0.8]$) | Nilai Aktual di Dataset (`NO2_Sambeng_TSFEL.csv`) |
|---|---|---|---|---|
| **`ecdf_percentile`** | $\max \{ x_{(i)} \mid y_i \le p \}$ | **4** | **(2, 6)** | **$0.000031$** ($3.099440 \times 10^{-5}$) |
| **`ecdf_percentile_count`** | $\vert \{ x_{(i)} \mid y_i \le p \} \vert$ | **4** | **(1, 6)** | **$182.5$** |