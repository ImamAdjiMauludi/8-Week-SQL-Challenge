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
-- 1. Create temp table for customer_orders
CREATE TABLE temp_customer_orders (
	order_id INT,
	customer_id INT,
	pizza_id VARCHAR,
	exclusions VARCHAR,
	extras VARCHAR,
	order_time TIMESTAMP
);

INSERT INTO temp_customer_orders
SELECT *
FROM customer_orders;

SELECT *
FROM temp_customer_orders tco;

--- Replace null or blank to 0
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
- Artinya jika tidak ada exclusions topping dan tidak ada extra topping maka nilainya adalah "0".\

## Cleaning customer_orders
**Query**
~~~~sql
SELECT *
FROM temp_runner_orders tro ; 

CREATE TEMPORARY TABLE before_temp_runner_orders AS 
SELECT * FROM runner_orders ro;

SELECT *
FROM before_temp_runner_orders btro ;

--- Replace blank time to NULL
UPDATE before_temp_runner_orders 
SET pickup_time = NULL 
WHERE pickup_time = "" OR pickup_time = "null";

--- Delete 'km'
UPDATE before_temp_runner_orders 
SET distance = REPLACE(distance, "km","");

--- Replace null or blank to 0
UPDATE before_temp_runner_orders
SET distance = 0
WHERE distance = "" OR distance = "null" OR distance IS NULL;

UPDATE before_temp_runner_orders 
SET duration = SUBSTRING(duration,1,2);

UPDATE before_temp_runner_orders
SET duration = 0 
WHERE duration = "" OR duration = "null" OR duration IS NULL;

UPDATE before_temp_runner_orders 
SET cancellation = 'Order Successfull' 
WHERE cancellation = "" OR cancellation = "null" OR cancellation IS NULL;

SELECT *
FROM before_temp_runner_orders;

--- Copy data to new temp table

CREATE TABLE temp_runner_orders (
	order_id INTEGER,
	runner_id INTEGER, 
	pickup_time TEXT,
	distance INTEGER,
	duration INTEGER,
	cancellation TEXT
);

INSERT INTO temp_runner_orders
SELECT *
FROM before_temp_runner_orders;

SELECT *
FROM temp_runner_orders tro;
~~~~
**STEP**
- step goes here

**Result**\
**Tabel `runner_orders` sebelum dibersihkan:**
|order_id|runner_id|pickup_time|distance|duration|cancellation|
|--------|---------|-----------|--------|--------|------------|
|1|1|2020-01-01 18:15:34|20km|32 minutes||
|2|1|2020-01-01 19:10:54|20km|27 minutes||
|3|1|2020-01-03 00:12:37|13.4km|20 mins||
|4|2|2020-01-04 13:53:03|23.4|40||
|5|3|2020-01-08 21:10:57|10|15||
|6|3||||Restaurant Cancellation|
|7|2|2020-01-08 21:30:45|25km|25mins||
|8|2|2020-01-10 00:15:02|23.4 km|15 minute||
|9|2||||Customer Cancellation|
|10|1|2020-01-11 18:50:20|10km|10minutes||

**Tabel `runner_orders` setelah dibuat tabel sementara dengan nama `temp_runner_orders`:**
|order_id|runner_id|pickup_time|distance|duration|cancellation|
|--------|---------|-----------|--------|--------|------------|
|1|1|2020-01-01 18:15:34|20|32|Order Successfull|
|2|1|2020-01-01 19:10:54|20|27|Order Successfull|
|3|1|2020-01-03 00:12:37|13.4|20|Order Successfull|
|4|2|2020-01-04 13:53:03|23.4|40|Order Successfull|
|5|3|2020-01-08 21:10:57|10|15|Order Successfull|
|6|3||0|0|Restaurant Cancellation|
|7|2|2020-01-08 21:30:45|25|25|Order Successfull|
|8|2|2020-01-10 00:15:02|23.4|15|Order Successfull|
|9|2||0|0|Customer Cancellation|
|10|1|2020-01-11 18:50:20|10|10|Order Successfull|

**Insight**
- insight goes here

# Case Study Questions - A. Pizza Metrics

## Questions - 1
**1. How many pizzas were ordered?**/**1. Berapa banyak pizza yang dipesan?**

**Query**
~~~~sql
SELECT COUNT(DISTINCT order_id) AS total_order_pizza
FROM temp_customer_orders tco;
~~~~
**STEP**
- steps goes here

**Result**
|total_order_pizza|
|-----------------|
|10|

**Insight**
- Insight goes here

## Questions - 2
**2. How many unique customer orders were made?**/**2. Berapa banyak pesanan unik pelanggan yang dibuat?**

**Query**
~~~~sql
SELECT COUNT(DISTINCT customer_id) AS unique_customer
FROM temp_customer_orders tco;
~~~~
**STEP**
- steps goes here

**Result**
|unique_customer|
|---------------|
|5|

**Insight**
- Insight goes here

## Questions - 3
**3. How many successful orders were delivered by each runner?**/**3. Berapa banyak pesanan yang berhasil diantar oleh setiap pelari?**

**Query**
~~~~sql
SELECT DISTINCT runner_id, COUNT(pickup_time) AS successful_orders 
FROM temp_runner_orders tro
WHERE pickup_time IS NOT 0
GROUP BY 1;
~~~~
**STEP**
- steps goes here

**Result**
|runner_id|successful_orders|
|---------|-----------------|
|1|4|
|2|3|
|3|1|

**Insight**
- Insight goes here

## Questions - 4
**4. How many of each type of pizza was delivered?**/**4. Berapa banyak pizza dari setiap jenis yang diantar?**

**Query**
~~~~sql
SELECT tco.pizza_id, COUNT(tco.pizza_id) AS order_successful 
FROM temp_customer_orders tco
FULL JOIN temp_runner_orders tro 
ON tco.order_id = tro.order_id
WHERE tro.pickup_time IS NOT 0
GROUP BY tco.pizza_id;
~~~~
**STEP**
- steps goes here

**Result**
|pizza_id|order_successful|
|--------|----------------|
|1|10|
|2|4|

**Insight**
- Insight goes here

## Questions - 5
**5. How many Vegetarian and Meatlovers were ordered by each customer?**/**5. Berapa banyak Vegetarian dan Meatlovers yang dipesan oleh setiap pelanggan?**

**Query**
~~~~sql
SELECT 
	tco.customer_id,
	COUNT(CASE WHEN tco.pizza_id = 1 THEN 1 END) AS Meatlovers,
	COUNT(CASE WHEN tco.pizza_id = 2 THEN 1 END) AS Vegetarian
FROM temp_customer_orders tco
FULL JOIN temp_runner_orders tro 
ON tro.order_id = tco.order_id
WHERE tro.pickup_time IS NOT 0
GROUP BY 1;
~~~~
**STEP**
- steps goes here

**Result**
|customer_id|Meatlovers|Vegetarian|
|-----------|----------|----------|
|101|2|1|
|102|2|1|
|103|3|1|
|104|3|0|
|105|0|1|

**Insight**
- Insight goes here

## Questions - 6
**6. What was the maximum number of pizzas delivered in a single order?**/**6. Berapa jumlah maksimum pizza yang diantar dalam satu pesanan?**

**Query**
~~~~sql
SELECT 
	tco.order_id,
	COUNT(tco.pizza_id) AS total_order
FROM temp_customer_orders tco 
FULL JOIN temp_runner_orders tro 
ON tro.order_id = tco.order_id 
WHERE tro.pickup_time IS NOT 0
GROUP BY 1
ORDER BY 2 DESC
LIMIT 1;
~~~~
**STEP**
- steps goes here

**Result**
|order_id|total_order|
|--------|-----------|
|4|3|

**Insight**
- Insight goes here

## Questions - 7
**7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?**/**7. Untuk setiap pelanggan, berapa banyak pizza yang diantar yang memiliki sedikitnya 1 perubahan dan berapa banyak yang tidak memiliki perubahan?**

**Query**
~~~~sql
SELECT 
	tco.customer_id,
	COUNT(CASE WHEN tco.exclusions IS NOT 0 OR tco.extras IS NOT 0 THEN 1 END) AS one_change,
	COUNT(CASE WHEN tco.exclusions = 0 OR tco.extras = 0 THEN 1 END) AS no_change
FROM temp_customer_orders tco
FULL JOIN temp_runner_orders tro
ON tro.order_id = tco.order_id
WHERE tro.pickup_time IS NOT 0
GROUP BY 1;
~~~~
**STEP**
- steps goes here

**Result**
|customer_id|one_change|no_change|
|-----------|----------|---------|
|101|0|3|
|102|0|3|
|103|4|3|
|104|2|2|
|105|1|1|

**Insight**
- Insight goes here

## Questions - 8
**8. How many pizzas were delivered that had both exclusions and extras?**/**8. Berapa banyak pizza yang diantar yang memiliki pengecualian dan tambahan?**

**Query**
~~~~sql
SELECT
	tco.order_id,
	COUNT(CASE WHEN tco.exclusions IS NOT 0 AND tco.extras IS NOT 0 THEN 1 END) AS both_exclusions_extras_delivered
FROM temp_customer_orders tco
FULL JOIN temp_runner_orders tro
ON tro.order_id = tco.order_id
WHERE tro.pickup_time IS NOT 0 
GROUP BY 1
HAVING both_exclusions_extras_delivered IS NOT 0
ORDER BY 2 DESC;
~~~~
**STEP**
- steps goes here

**Result**
|order_id|both_exclusions_extras_delivered|
|--------|--------------------------------|
|10|1|
|9|1|

**Insight**
- Insight goes here

## Questions - 9
**9. What was the total volume of pizzas ordered for each hour of the day?**/**9. Berapa total volume pizza yang dipesan untuk setiap jam dalam sehari?**

**Query**
~~~~sql
SELECT 
	TIME((strftime('%s', tro.pickup_time) + 30 * 60) / 3600 * 3600, 'unixepoch') AS rounded_time,
	COUNT(tco.pizza_id) AS pizza_ordered
FROM temp_runner_orders tro
FULL JOIN temp_customer_orders tco 
ON tro.order_id = tco.order_id
WHERE tro.pickup_time IS NOT NULL
GROUP BY 1
ORDER BY 1;
~~~~
**STEP**
- steps goes here

**Result**
|rounded_time|pizza_ordered|
|------------|-------------|
|00:00:00|3|
|14:00:00|3|
|18:00:00|1|
|19:00:00|3|
|21:00:00|1|
|22:00:00|1|

**Insight**
- Insight goes here

## Questions - 10
**10. What was the volume of orders for each day of the week?**/**10. Berapa volume pesanan untuk setiap hari dalam seminggu?**

**Query**
~~~~sql
SELECT 
	CASE strftime('%w', tro.pickup_time)
	WHEN '0' THEN 'Minggu'
        WHEN '1' THEN 'Senin'
        WHEN '2' THEN 'Selasa'
        WHEN '3' THEN 'Rabu'
        WHEN '4' THEN 'Kamis'
        WHEN '5' THEN 'Jumat'
        WHEN '6' THEN 'Sabtu'
    END AS days,
	COUNT(tco.pizza_id) AS pizza_ordered
FROM temp_runner_orders tro
FULL JOIN temp_customer_orders tco 
ON tro.order_id = tco.order_id
WHERE tro.pickup_time IS NOT NULL
GROUP BY 1
ORDER BY 2 DESC;
~~~~
**STEP**
- steps goes here

**Result**
|days|pizza_ordered|
|----|-------------|
|Sabtu|5|
|Rabu|4|
|Jumat|3|

**Insight**
- Insight goes here

# Case Study Questions - B. Runner and Customer Experience

## Questions - 1
**1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)**

**Query**
~~~~sql
SELECT 
	strftime('%W', registration_date) + 1 AS week_period,
	COUNT(runner_id) AS runner_signed_up
FROM runners r
GROUP BY 1;
~~~~
**STEP**
- steps goes here

**Result**
|week_period|runner_signed_up|
|-----------|----------------|
|1|2|
|2|1|
|3|1|

**Insight**
- Insight goes here

## Questions - 2
**2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?**

**Query**
~~~~sql
SELECT 
	tro.runner_id,
	ROUND(AVG(CAST((strftime("%s", tro.pickup_time) - strftime("%s", tco.order_time)) AS FLOAT)/60),2) AS avg_pickup_duration
FROM temp_runner_orders tro
INNER JOIN temp_customer_orders tco 
ON tro.order_id = tco.order_id
WHERE tro.pickup_time IS NOT NULL
GROUP BY 1;
~~~~
**STEP**
- steps goes here

**Result**
|runner_id|avg_pickup_duration|
|---------|-------------------|
|1|15.68|
|2|23.72|
|3|10.47|

**Insight**
- Insight goes here

## Questions - 3
**3. Is there any relationship between the number of pizzas and how long the order takes to prepare?**

**Query**
~~~~sql
SELECT 
	tco.order_id,
	COUNT(tco.pizza_id),
	ROUND(CAST((strftime("%s", tro.pickup_time) - strftime("%s", tco.order_time)) AS FLOAT)/60,2) AS avg_pickup_duration
FROM temp_runner_orders tro
INNER JOIN temp_customer_orders tco 
ON tro.order_id = tco.order_id
WHERE tro.pickup_time IS NOT NULL
GROUP BY 1
ORDER BY 3 DESC;
~~~~
**STEP**
- steps goes here

**Result**
|order_id|COUNT(tco.pizza_id)|avg_pickup_duration|
|--------|-------------------|-------------------|
|4|3|29.28|
|3|2|21.23|
|8|1|20.48|
|10|2|15.52|
|1|1|10.53|
|5|1|10.47|
|7|1|10.27|
|2|1|10.03|

**Insight**
- Insight goes here

## Questions - 4
**4. What was the average distance travelled for each customer?**

**Query**
~~~~sql
SELECT
	tco.customer_id, 
	ROUND(AVG(tro.distance),2) AS km_avg_distance
FROM temp_runner_orders tro
JOIN temp_customer_orders tco 
ON tro.order_id = tco.order_id
WHERE tro.pickup_time IS NOT NULL
GROUP BY 1;
~~~~
**STEP**
- steps goes here

**Result**
|customer_id|km_avg_distance|
|-----------|---------------|
|101|20.0|
|102|16.73|
|103|23.4|
|104|10.0|
|105|25.0|

**Insight**
- Insight goes here

## Questions - 5
**5. What was the difference between the longest and shortest delivery times for all orders?**

**Query**
~~~~sql
SELECT
	MAX(duration) AS longest_duration,
	MIN(duration) AS shortest_duration
FROM temp_runner_orders tro
WHERE tro.pickup_time IS NOT NULL;
~~~~
**STEP**
- steps goes here

**Result**
|longest_duration|shortest_duration|
|----------------|-----------------|
|40|10|

**Insight**
- Insight goes here

## Questions - 6
**6. What was the average speed for each runner for each delivery and do you notice any trend for these values?**

**Query**
~~~~sql
SELECT
	runner_id,
	order_id,
	distance,
	ROUND(AVG(CAST(distance AS FLOAT)/(CAST(duration AS FLOAT)/60)),2) AS avg_speed_kmph
FROM temp_runner_orders tro
WHERE pickup_time IS NOT NULL
GROUP BY 2
ORDER BY 1;
~~~~
**STEP**
- steps goes here

**Result**
|runner_id|order_id|distance|avg_speed_kmph|
|---------|--------|--------|--------------|
|1|1|20|37.5|
|1|2|20|44.44|
|1|3|13.4|40.2|
|1|10|10|60.0|
|2|4|23.4|35.1|
|2|7|25|60.0|
|2|8|23.4|93.6|
|3|5|10|40.0|

**Insight**
- Insight goes here

## Questions - 7
**7. What is the successful delivery percentage for each runner?**

**Query**
~~~~sql
WITH success_data AS (
SELECT
	runner_id,
	CASE
		WHEN cancellation = "Order Successfull" THEN 1
		ELSE 0
	END AS delivery_success
FROM temp_runner_orders tro
)
SELECT 
	runner_id,
	CAST(SUM(delivery_success) AS FLOAT)/CAST(COUNT(delivery_success) AS FLOAT)*100 AS success_rate_percent
FROM success_data
GROUP BY runner_id;
~~~~
**STEP**
- steps goes here

**Result**
|runner_id|success_rate_percent|
|---------|--------------------|
|1|100.0|
|2|75.0|
|3|50.0|

**Insight**
- Insight goes here
