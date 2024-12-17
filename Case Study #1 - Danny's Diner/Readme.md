# Case 1 - Danny's Dinner

# Task
_Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers._

Danny ingin menggunakan data untuk menjawab beberapa pertanyaan sederhana tentang pelanggannya, terutama tentang pola kunjungan mereka, berapa banyak uang yang mereka habiskan, dan juga item menu apa yang menjadi favorit mereka. Koneksi yang lebih dalam dengan para pelanggannya akan membantunya memberikan pengalaman yang lebih baik dan lebih personal bagi para pelanggan setianya.

# ERD
![image](https://github.com/user-attachments/assets/d82802fb-12ca-41b4-90a9-9916e16b50c8)

# Case Study Questions

## Questions - 1
**1. What is the total amount each customer spent at the restaurant?**/**1. Berapa jumlah total yang dibelanjakan setiap pelanggan di restoran?**

**Query**
~~~~sql
SELECT
	s.customer_id, SUM(mn.price) AS spending
FROM dannys_diner.sales AS s
FULL OUTER JOIN dannys_diner.menu AS mn ON mn.product_id = s.product_id
GROUP BY s.customer_id
ORDER BY spending DESC;
~~~~
**STEP**
- **FULL JOIN** table `sales` dan `menu` untuk menggabungkan data, **JOIN** menggunakan primary key pada kolom `product_id`.
- **SELECT** kolom `customer_id`, lalu **SUM()** kolom `price` untuk mengitung total `spending`

**Result**
| customer_id | spending |
| ----------- | -------- |
| A           | 76       |
| B           | 74       |
| C           | 36       |

**Insight**
- Customer A belanja terbanyak yaitu 76.
- Customer B belanja sebanyak 74.
- Customer C belanja sebanyak 36.

## Questions - 2
**2. How many days has each customer visited the restaurant?**/**2. Berapa hari setiap pelanggan mengunjungi restoran?**

**Query**
~~~~sql
SELECT
	customer_id, COUNT(DISTINCT(order_date)) AS visited
FROM
	dannys_diner.sales
GROUP BY customer_id
ORDER BY customer_id ASC;
~~~~
**STEP**
- **SELECT** kolom `customer_id`, lalu gunakan **COUNT()** untuk menghitung `order_date` sebagai banyak kedatangan.
- Perlu diperhatikan, kolom `order_date` menampilkan daftar pesanan bukan kedatagan, oleh sebab itu harus menggunakan **DISTINCT** agar tidak menghitung duplikat.

**Result**
| customer_id | visited |
| ----------- | ------- |
| A           | 4       |
| B           | 6       |
| C           | 2       |

**Insight**
- Customer A datang sebanyak 4x.
- Customer B paling sering datang yaitu 6x.
- Customer C datang sebanyak 2x.

## Questions - 3
**3. What was the first item from the menu purchased by each customer?**/**3. Apa item pertama dari menu yang dibeli oleh setiap pelanggan?**

**Query**
~~~~sql
WITH urutan AS 
(
SELECT
	s.customer_id, 
	s.order_date,
    DENSE_RANK() OVER (PARTITION BY s.customer_id ORDER BY s.order_date) as rank,
    m.product_name
FROM dannys_diner.sales AS s
JOIN dannys_diner.menu AS m
ON s.product_id = m.product_id
) 
SELECT DISTINCT  
	customer_id,
	order_date,
	product_name,
	rank
FROM urutan
WHERE rank = 1;
~~~~
**STEP**
- Membuat query CTE untuk mengetahui pembelian pertama menggunakan **DENSE_RANK** karena ada menu yang dipesan secara bersamaan di hari yang sama berdasarkan kolom `order_date`, lalu dikelompokkan **(PARTITION)** dengan kolom `customer_id`, sehingga dapat diketahui pembelian pertama tiap customer.
- Selanjutnya menggunakan klausa **WHERE rank = 1** untuk mengetahui menu pertama yang dibeli oleh customer. Karena ada customer yang membeli 2 menu yang sama pada pesanan pertama kali, maka digunakan klausa **DISTINCT** untuk mengambil nilai yang unik.

**Result**
| customer_id | order_date | product_name | rank |
| ----------- | ---------- | ------------ | ---- |
| A           | 2021-01-01 | curry        | 1    |
| A           | 2021-01-01 | sushi        | 1    |
| B           | 2021-01-01 | curry        | 1    |
| C           | 2021-01-01 | ramen        | 1    |

**Insight**
- Menu pertama yang dipesan sustomer A adalah Curry dan Sushi.
- Menu pertama yang dipesan sustomer B adalah Curry.
- Menu pertama yang dipesan sustomer C adalah Ramen.

## Questions - 4
**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**/**4. Apa item yang paling banyak dibeli pada menu dan berapa kali item tersebut dibeli oleh semua pelanggan?**

**Query**
~~~~sql
SELECT
	m.product_name,
	COUNT(s.order_date) AS pesanan
FROM
	dannys_diner.sales AS s
JOIN
	dannys_diner.menu AS m
ON
	s.product_id = m.product_id
GROUP BY
	m.product_name
ORDER BY
	pesanan DESC
LIMIT 1;
~~~~
**STEP**
- **COUNT** kolom `orderdate` untuk megetahui jumlah pesanan.
- **JOIN** tabel `sales` dengan `menu` pada kolom `product_id` untuk mengetahui nama menu.
- Selanjutnya **GROUP BY** kolom `product_name` untuk mengetahui jumlah pesanan tiap menu.

**Result**
| product_name | pesanan |
| ------------ | ------- |
| ramen        | 8       |

**Insight**
- Menu yang paling banyak dipesan adalah Ramen, yaitu sebanyak 8x.

## Questions - 5
**5. Which item was the most popular for each customer?**/**5. Item mana yang paling populer bagi setiap pelanggan?**

**Query**
~~~~sql
WITH fav_menu AS (
SELECT
	s.customer_id,
	m.product_name,
	COUNT(s.order_date) AS pesanan,
	DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY COUNT(s.order_date) DESC) AS rank_pesanan
FROM
	dannys_diner.sales AS s
JOIN
	dannys_diner.menu AS m
ON
	s.product_id = m.product_id
GROUP BY
	s.customer_id, m.product_name
ORDER BY
	pesanan DESC
)
SELECT
	customer_id, 
	product_name, 
	pesanan,
	rank_pesanan
FROM
	fav_menu
WHERE
	rank_pesanan = 1;
~~~~
**STEP**
- 

**Result**
| customer_id | product_name | pesanan | rank_pesanan |
| ----------- | ------------ | ------- | ------------ |
| C           | ramen        | 3       | 1            |
| A           | ramen        | 3       | 1            |
| B           | curry        | 2       | 1            |
| B           | sushi        | 2       | 1            |
| B           | ramen        | 2       | 1            |

**Insight**
- 

## Questions - 6
**6. Which item was purchased first by the customer after they became a member?**/**6. Item apa yang pertama kali dibeli oleh pelanggan setelah menjadi member?**

**Query**
~~~~sql
WITH cari_rank AS (
WITH data_join AS (
SELECT
	s.customer_id,
	s.order_date,
	mem.join_date,
    CASE
    	WHEN s.order_date >= mem.join_date THEN 'Sudah Join'
        ELSE 'Belum Join'
    END AS status_join,
	m.product_name
FROM
	dannys_diner.sales AS s
JOIN
	dannys_diner.members AS mem
ON
	s.customer_id = mem.customer_id
JOIN
	dannys_diner.menu AS m
ON
	s.product_id = m.product_id
)
SELECT
	*,
	DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY order_date) AS pesanan_setelah_member
FROM
	data_join
WHERE
	status_join = 'Sudah Join'
)
SELECT
	*
FROM
	cari_rank
WHERE
	pesanan_setelah_member = 1;
~~~~
**STEP**
- 

**Result**
| customer_id | order_date | join_date  | status_join | product_name | pesanan_setelah_member |
| ----------- | ---------- | ---------- | ----------- | ------------ | ---------------------- |
| A           | 2021-01-07 | 2021-01-07 | Sudah Join  | curry        | 1                      |
| B           | 2021-01-11 | 2021-01-09 | Sudah Join  | sushi        | 1                      |

**Insight**
-

## Questions - 7
**7. Which item was purchased just before the customer became a member?**/**7. Item apa yang dibeli sebelum pelanggan menjadi member?**

**Query**
~~~~sql
SELECT
	*
FROM (
SELECT
	s.customer_id,
	s.order_date,
	mem.join_date,
    CASE
    	WHEN s.order_date >= mem.join_date THEN 'Sudah Join'
        ELSE 'Belum Join'
    END AS status_join,
	m.product_name,
  	DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date DESC) AS rank_pesanan
FROM
	dannys_diner.sales AS s
JOIN
	dannys_diner.members AS mem
ON
	s.customer_id = mem.customer_id
JOIN
	dannys_diner.menu AS m
ON
	s.product_id = m.product_id
WHERE
  s.order_date < mem.join_date
) AS data_belum_join
WHERE 
	rank_pesanan = 1;
~~~~
**STEP**
- 

**Result**
| customer_id | order_date | join_date  | status_join | product_name | rank_pesanan |
| ----------- | ---------- | ---------- | ----------- | ------------ | ------------ |
| A           | 2021-01-01 | 2021-01-07 | Belum Join  | sushi        | 1            |
| A           | 2021-01-01 | 2021-01-07 | Belum Join  | curry        | 1            |
| B           | 2021-01-04 | 2021-01-09 | Belum Join  | sushi        | 1            |

**Insight**
-

## Questions - 8
**8. What is the total items and amount spent for each member before they became a member?**/**8. Berapa total barang dan jumlah yang dibelanjakan setiap anggota sebelum menjadi anggota?**

**Query**
~~~~sql
WITH spending_sebelum_join AS (
SELECT
	s.customer_id,
	COUNT(
      	CASE
      		WHEN m.product_name = 'sushi' THEN 1
      		WHEN m.product_name = 'curry' THEN 1
      		WHEN m.product_name = 'ramen' THEN 1
	ELSE 1
    	END) AS total_product,
    SUM(m.price) AS total_spending
FROM
	dannys_diner.sales AS s
FULL JOIN
	dannys_diner.members AS mem
ON
	s.customer_id = mem.customer_id
FULL JOIN
	dannys_diner.menu AS m
ON
	s.product_id = m.product_id
WHERE
 	s.order_date < mem.join_date
    OR
    mem.join_date IS NULL
GROUP BY 
	s.customer_id
)
SELECT
	customer_id,
   	total_product,
    SUM(total_spending)
FROM
	spending_sebelum_join
GROUP BY
	customer_id,
    total_product;
~~~~
**STEP**
- 

**Result**
| customer_id | total_product | sum |
| ----------- | ------------- | --- |
| A           | 2             | 25  |
| B           | 3             | 40  |
| C           | 3             | 36  |

**Insight**
-
