# Cek NA dan Duplikat
colSums(is.na(data))
sum(duplicated(data))

# Cek anomali nilai pada penjualan (mencari nilai minus)
summary(data$Weekly_Sales)
```

### 3.2 Identifikasi Masalah

Dilakukan pengecekan kualitas data

```r
# Cek NA dan Duplikat
colSums(is.na(data))
sum(duplicated(data))

# Cek anomali nilai pada penjualan (mencari nilai minus)
summary(data$Weekly_Sales)
```

### 3.3 Eksekusi Cleaning

Menggunakan teknik imputasi nol untuk diskon (MarkDown) dan filtering untuk menghapus penjualan negatif.

```r
# 1. Hapus baris duplikat
data <- data[!duplicated(data), ]

# 2. Buang sales minus (retur)
data <- data[data$Weekly_Sales >= 0, ]

# 3. Imputasi NA pada kolom diskon (MarkDown) dengan 0 (Asumsi tidak ada promo)
data$MarkDown1[is.na(data$MarkDown1)] <- 0
data$MarkDown2[is.na(data$MarkDown2)] <- 0
data$MarkDown3[is.na(data$MarkDown3)] <- 0
data$MarkDown4[is.na(data$MarkDown4)] <- 0
data$MarkDown5[is.na(data$MarkDown5)] <- 0
```

### 3.4 Transformasi Tipe Data

Melakukan pengecekan tipe dan skala data, kemudian mengubah format kolom agar sesuai dengan standar analisis.

```r
# Ubah character ke format Date (Penting untuk peramalan Time Series)
data$Date <- as.Date(data$Date, format = "%Y-%m-%d")

# Ubah character ke format Factor
data$Type <- as.factor(data$Type)
```

### 3.5 Data Reduction

Menghapus redundansi kolom yang muncul akibat proses penggabungan data (merge).

```r
# Validasi apakah isi kolom IsHoliday.x dan y sama identik
all(data$IsHoliday.x == data$IsHoliday.y)

# Reduksi (Buang kolom redundan sisa merge)
data$IsHoliday.y <- NULL

# Benerin nama kolom kembali ke awal
names(data)[names(data) == "IsHoliday.x"] <- "IsHoliday"
```

### 3.6 Data Encoding

Tahap ini mengubah data kategorikal menjadi angka agar dapat diolah oleh model matematika seperti Regresi Linear.

```r
# 1. Label Encoding untuk kolom IsHoliday (TRUE/FALSE jadi 1/0)
data$IsHoliday_Num <- as.numeric(data$IsHoliday)

# 2. Dummy Encoding untuk kolom Type (A, B, C) menggunakan model.matrix
dummies <- model.matrix(~Type-1, data=data)

# 3. Menggabungkan kolom Dummy ke dataset utama 
# (Hanya TypeA dan TypeB yang diambil untuk menghindari Dummy Variable Trap)
data <- cbind(data, dummies[, c("TypeA", "TypeB")])

# Menyimpan hasil akhir pembersihan yang sudah siap model
write.csv(data, "walmart_cleaning.csv", row.names = FALSE)
```

### 3.7 Catatan Tambahan Pemodelan
Sebagai catatan untuk tahapan *fitting* model selanjutnya:
* **Encoding:** Transformasi variabel kategorikal `Type` menjadi *dummy* (`TypeA` & `TypeB`) serta konversi biner `IsHoliday` telah tuntas dilakukan pada tahap ini.
* **Scaling:** Proses standarisasi sengaja **tidak dilakukan** pada tahap pre-processing. Hal ini untuk menjaga nilai target (`Weekly_Sales`) tetap dalam satuan asli demi kemudahan interpretasi peramalan *Time Series*. Tim pemodelan disarankan melakukan *scaling* secara mandiri pada variabel independen di *script* masing-masing.
