### Prediksi Permintaan Sewa Sepeda

#### Pendahuluan 
Proyek ini menganalisis dan memodelkan permintaan sewa sepeda per jam menggunakan dataset publik dari tahun 2011–2012. Tujuannya adalah memprediksi jumlah total sepeda yang disewa (`cnt`) setiap jam, sehingga operator dapat mengoptimalkan alokasi sumber daya (redistribusi sepeda, penjadwalan staf, dll.).

#### Daftar Isi  
1. [Dataset](#dataset)  
2. [Struktur Notebook](#struktur-notebook)  
3. [Temuan Utama](#temuan-utama)  
4. [Pendekatan Pemodelan](#pendekatan-pemodelan)  
5. [Hasil](#hasil)  
6. [Rekomendasi & Pekerjaan Selanjutnya](#rekomendasi--pekerjaan-selanjutnya)  

#### Dataset  
- **Sumber:** Data sewa sepeda per jam (Capital Bikeshare) tahun 2011–2012.  
- **Fitur:**  
  - `dteday`               : tanggal  
  - `hr`                    : jam dalam hari (0–23)  
  - `season`                : 1=Musim Dingin, 2=Musim Semi, 3=Musim Panas, 4=Musim Gugur  
  - `weathersit`            : 1=Cerah, 2=Berkabut, 3=Hujan/Rintik Ringan, 4=Hujan Lebat/Salju  
  - `temp`, `atemp`         : suhu (dinormalisasi & aktual)  
  - `hum`                   : kelembapan
  - `casual`, `registered`  : tipe pengguna  
  - `cnt`                   : total penyewaan (target)  

#### Struktur Notebook  
1. **Pendahuluan & Pemahaman Bisnis**  
##### **a. Context**

Sistem bike-sharing adalah generasi baru dari penyewaan sepeda tradisional di mana seluruh proses, mulai dari keanggotaan, penyewaan, hingga pengembalian, telah menjadi otomatis. Melalui sistem ini, pengguna dapat dengan mudah menyewa sepeda dari suatu lokasi tertentu dan mengembalikannya di lokasi lain. Saat ini, terdapat lebih dari 500 program bike-sharing di seluruh dunia yang terdiri dari lebih dari 500 ribu sepeda. Saat ini, sistem ini sangat diminati karena perannya yang penting dalam masalah lalu lintas, lingkungan, dan kesehatan.

Selain aplikasi dunia nyata yang menarik dari sistem bike-sharing, karakteristik data yang dihasilkan oleh sistem ini juga menarik untuk penelitian. Berbeda dengan layanan transportasi lain seperti bus atau kereta bawah tanah, durasi perjalanan, posisi keberangkatan, dan kedatangan dicatat secara eksplisit dalam sistem ini. Fitur ini menjadikan sistem bike-sharing sebagai jaringan sensor virtual yang dapat digunakan untuk memantau mobilitas di kota. Oleh karena itu, diharapkan peristiwa-peristiwa penting di kota dapat terdeteksi dengan memantau data ini.
##### **b. Problem Statement**

Sistem bike sharing menghadapi tantangan utama dalam menjaga ketersediaan sepeda yang cukup untuk memenuhi kebutuhan pengguna pada waktu-waktu tertentu. Ketersediaan sepeda ini sangat dipengaruhi oleh beberapa faktor, antara lain permintaan yang fluktuatif (misalnya perubahan jumlah pengguna pada jam sibuk, hari libur, atau musim tertentu), kondisi sepeda yang dapat dipergunakan dengan baik (karena sepeda yang rusak atau sedang dalam perawatan tidak dapat digunakan), serta risiko vandalisme dan pencurian yang dapat mengurangi jumlah sepeda yang siap disewakan. Ketidakmampuan dalam mengantisipasi dan mengelola faktor-faktor tersebut dapat menyebabkan kekurangan sepeda saat permintaan tinggi, menurunkan kepuasan pelanggan, dan mengganggu operasional layanan.

Penelitian ini bertujuan untuk memprediksi jumlah sepeda yang harus disediakan pada waktu tertentu dengan mempertimbangkan faktor-faktor tersebut, sehingga operator dapat menjaga ketersediaan sepeda secara optimal dan meningkatkan kualitas layanan kepada pengguna.
##### **c. Goals**

Berdasarkan permasalahan tersebut, operator layanan bike sharing perlu memiliki alat bantu (tool) yang dapat memprediksi serta membantu mereka dalam **menentukan jumlah sepeda yang ideal untuk disediakan pada waktu-waktu tertentu**. Dengan mempertimbangkan berbagai faktor seperti fluktuasi permintaan, kondisi sepeda yang dapat dipergunakan dengan baik, serta risiko vandalisme dan pencurian, prediksi yang dihasilkan akan semakin akurat dan relevan dengan kebutuhan di lapangan.

Bagi operator, adanya prediction tool yang mampu memberikan rekomendasi jumlah sepeda secara optimal akan membantu menjaga ketersediaan sepeda, meningkatkan kepuasan pengguna, serta mendukung efisiensi operasional. Dengan demikian, layanan bike sharing dapat berjalan lebih lancar, jumlah pengguna dapat meningkat, dan pada akhirnya berdampak positif terhadap pendapatan serta citra perusahaan.

2. **Pemuatan & Inspeksi Data**  
3. **Rekayasa Fitur**  
   - Ekstraksi tanggal/jam (tahun, bulan, hari, jam, kerja/akhir pekan)  
   - Enkoding siklikal untuk jam dan bulan  
   - One-hot encoding untuk fitur kategorikal  
4. **Pipeline Preprocessing**  
   - `CyclicalEncoder` kustom  
   - `ColumnTransformer` untuk menggabungkan semua langkah  
5. **Pemodelan & Perbandingan**  
   - Linear Regression, Random Forest, XGBoost, dll.  
   - Cross-validation untuk RMSE dan R²  
6. **Penyetelan Hiperparameter**  
   - `RandomizedSearchCV` pada kandidat terbaik  
7. **Evaluasi & Diagnostik**  
   - Analisis residual (scatter + histogram)  
   - Feature importance  
8. **Kesimpulan & Langkah Selanjutnya**

#### Temuan Utama  
- **Heteroskedastisitas:** Variansi error meningkat seiring kenaikan `cnt`.  
- **Underestimation pada puncak:** Model sering meremehkan permintaan pada jam sibuk.  
- **Fitur paling berpengaruh:** Jam (siklikal), suhu, hari kerja.

#### Pendekatan Pemodelan  
- **Pipeline:**  
  1. Preprocessing: enkoding siklikal + one-hot + standarisasi  
  2. Estimator: berbagai regresor (Linear Regression, Random Forest, XGBoost, …)  
- **Validasi:**  
  - Split berbasis waktu (mis. latih 2011, tes 2012) agar sesuai forecasting  
  - 5-fold cross-validation untuk perbandingan model  
- **Penyetelan:**  
  - `RandomizedSearchCV` dengan `n_iter` yang sesuai & early stopping untuk `StackingRegressor` dengan base model:
    - `XGBoostRegressor`
    - `RandomForestRegressor`
    - `DecissionTreeRegressor`

#### Hasil  
##### Tabel Hasil Sebelum dan Sesudah Tuning

| Metric | BeforeTuning | AfterTuning |
|--------|-------------:|------------:|
| MAE    |     24.699391|    23.569601|
| MAPE   |      0.318190|     0.300400|
| R²     |      0.947330|     0.951367|
| RMSE   |     41.429563|    39.810475|

#### Rekomendasi & Pekerjaan Selanjutnya  
- **Transformasi target:** Gunakan \(\log(1 + cnt)\) untuk menstabilkan varians.  
- **Validasi time-series:** Terapkan `TimeSeriesSplit` atau split berdasarkan tanggal.  
- **Fitur tambahan:**  
  - Data prakiraan cuaca  
  - Flag libur/proksimitas libur waktu nyata  
- **Model lanjutan:** Quantile Regression, objective Poisson, interpretabilitas dengan SHAP.
