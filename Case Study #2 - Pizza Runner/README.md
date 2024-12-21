# Case Study #2 - Pizza Runner

# Task
_This case study has LOTS of questions - they are broken up by area of focus including:_
- _Pizza Metrics_
- _Runner and Customer Experience_
- _Ingredient Optimisation_
- _Pricing and Ratings_

Studi kasus ini memiliki BANYAK pertanyaan - pertanyaan-pertanyaan tersebut dipecah berdasarkan area fokus, termasuk:
- Metrik Pizza
- Pengalaman Runner dan Pelanggan
- Optimalisasi Bahan
- Harga dan Peringkat

# ERD
![image](https://github.com/user-attachments/assets/3331201e-fd02-49a8-8c9f-1c0b9b02b8d4)

# DATA CLEANING WITH SQL

Terdapat tabel yang perlu dibersihkan sebelum memulai analisis data:

_Before you start writing your SQL queries however - you might want to investigate the data, you may want to do something with some of those null values and data types in the customer_orders and runner_orders tables!_

Namun, sebelum Anda mulai menulis kueri SQL Anda, Anda mungkin ingin menyelidiki datanya, Anda mungkin ingin melakukan sesuatu dengan beberapa nilai null dan tipe data di tabel `customer_orders` dan `runner_orders`!

## Cleaning customer_orders

**Query**
~~~~sql
CREATE TEMPORARY TABLE temp_customer_orders AS 
SELECT * FROM customer_orders co;

SELECT *
FROM temp_customer_orders tco;

UPDATE temp_customer_orders 
SET exclusions = 0
WHERE exclusions = "null" OR exclusions = "" OR exclusions IS NULL;

UPDATE temp_customer_orders 
SET extras = 0
WHERE extras = "" OR extras = "null" OR extras IS NULL;

SELECT *
FROM temp_customer_orders tco;
~~~~
**STEP**
- Membuat tabel sementara bernama `temp_customer_orders` yang berasal dari tabel `customer_orders`, karena kita akan mengubah isi tabel tanpa menyentuh tabel asli.
- **SELECT** semua kolom untuk mengetahui isi kolom, apakah ada nilai NULL atau kosong. Ditemukan nilai kosong dan null (string) pada kolom exclusions dan extras.
- **UPDATE** tabel `temp_customer_orders` dengan merubah isi exclusions dengan 0 jika berisi nilai "null" (string), "" (kosong), dan NULL.
- Melakukan kembali **UPADATE** pada tabel `temp_customer_orders` dengan merubah kolom extras dengan 0 jika berisi nilai "null" (string), "" (kosong), dan NULL.
- Terakhir **SELECT** semua kolom untuk mengetahui isi kolom setelah dibersihkan.


**Result**\
**Tabel `customer_orders` sebelum dibersihkan:**
|order_id|customer_id|pizza_id|exclusions|extras|order_time|
|--------|-----------|--------|----------|------|----------|
|1|101|1|||2020-01-01 18:05:02|
|2|101|1|||2020-01-01 19:00:52|
|3|102|1|||2020-01-02 23:51:23|
|3|102|2|||2020-01-02 23:51:23|
|4|103|1|4||2020-01-04 13:23:46|
|4|103|1|4||2020-01-04 13:23:46|
|4|103|2|4||2020-01-04 13:23:46|
|5|104|1|null|1|2020-01-08 21:00:29|
|6|101|2|null|null|2020-01-08 21:03:13|
|7|105|2|null|1|2020-01-08 21:20:29|
|8|102|1|null|null|2020-01-09 23:54:33|
|9|103|1|4|1, 5|2020-01-10 11:22:59|
|10|104|1|null|null|2020-01-11 18:34:49|
|10|104|1|2, 6|1, 4|2020-01-11 18:34:49|

**Tabel `customer_orders` setelah dibuat tabel sementara dengan nama `temp_customer_orders`:**
|order_id|customer_id|pizza_id|exclusions|extras|order_time|
|--------|-----------|--------|----------|------|----------|
|1|101|1|0|0|2020-01-01 18:05:02|
|2|101|1|0|0|2020-01-01 19:00:52|
|3|102|1|0|0|2020-01-02 23:51:23|
|3|102|2|0|0|2020-01-02 23:51:23|
|4|103|1|4|0|2020-01-04 13:23:46|
|4|103|1|4|0|2020-01-04 13:23:46|
|4|103|2|4|0|2020-01-04 13:23:46|
|5|104|1|0|1|2020-01-08 21:00:29|
|6|101|2|0|0|2020-01-08 21:03:13|
|7|105|2|0|1|2020-01-08 21:20:29|
|8|102|1|0|0|2020-01-09 23:54:33|
|9|103|1|4|1, 5|2020-01-10 11:22:59|
|10|104|1|0|0|2020-01-11 18:34:49|
|10|104|1|2, 6|1, 4|2020-01-11 18:34:49|

**Insight**
- Pada kolom exclusions dan extra diberikan value baru yaitu "0".
- Artinya jika tidak ada exclusions topping dan tidak ada extra topping maka nilainya adalah "0".
