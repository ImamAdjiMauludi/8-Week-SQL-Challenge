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
- Menggunakan CTE untuk membuat tabel yang berisi **COUNT** total pesanan dan **DENSE_RANK** untuk mengetahui pesanan yang populer oleh tiap customer.
- Lalu filter menggunakan `rank_pesanan = 1` untuk menampilkan rank 1 pesanan populer tiap customer.

**Result**
| customer_id | product_name | pesanan | rank_pesanan |
| ----------- | ------------ | ------- | ------------ |
| C           | ramen        | 3       | 1            |
| A           | ramen        | 3       | 1            |
| B           | curry        | 2       | 1            |
| B           | sushi        | 2       | 1            |
| B           | ramen        | 2       | 1            |

**Insight**
- Customer C memesan ramen sebanyak 3x, menempati posisi pertama pada menu yang sering dipesan.
- Customer A memesan ramen sebanyak 3x juga, menempati posisi pertama pada menu yang sering dipesan.
- Sementara itu, customer B memesan semua menu (sushi, curry, ramen) dengan jumlah yang sama rata yaitu 2x.

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
- Membuat tabel CTE dengan nama `data_join` untuk mengetahui status customer pada saat order dibuat, apakah sudah sudah menjadi member atau belum.
- Untuk mengetaui hal tersebut, digunakan klausa **CASE WHEN** yang akan mengembalikan status 'Sudah Join' jika tanggal order_date >= join_date, dan mengembalikan nilai 'Belum Join' jika selain kondisi tersebut.
- Lalu membungkus CTE `data_join` dengan CTE baru yaitu `cari_rank`. Dengan klausa **DENSE_RANK** untuk mengetahui rank urutan pesanan berdasarkan kolom 'join_date'. Lalu memfilter kolom `status_join = 'Sudah Join'`

**Result**
| customer_id | order_date | join_date  | status_join | product_name | pesanan_setelah_member |
| ----------- | ---------- | ---------- | ----------- | ------------ | ---------------------- |
| A           | 2021-01-07 | 2021-01-07 | Sudah Join  | curry        | 1                      |
| B           | 2021-01-11 | 2021-01-09 | Sudah Join  | sushi        | 1                      |

**Insight**
- Pesanan yang dipesan pertama kali oleh customer A setelah menjadi member adalah Curry.
- Pesanan yang dipesan pertama kali oleh customer B setelah menjadi member adalah Sushi.

## Questions - 7
**7. Which item was purchased just before the customer became a member?**/**7. Item apa yang dibeli sebelum customer menjadi member?**

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
- Soal ini bisa diselesaikan dengan kueri yang hampir sama pada soal nomor 6, namun kueri yang dibuat pada nomor 6 kurang efektif. Mari buat kueri baru.
- Di sini alih-alih menggunakan CTE, yang digunakan adalah subquery, alhasil lebih ringkas.
- Hanya perlu memfilter kondisi `s.order_date < mem.join_date`, lalu pada queri utama difilter kembali dengan `rank_pesanan = 1` agar menampilkan pesanan terakhir 
 persis sebelum bergabung member.

**Result**
| customer_id | order_date | join_date  | status_join | product_name | rank_pesanan |
| ----------- | ---------- | ---------- | ----------- | ------------ | ------------ |
| A           | 2021-01-01 | 2021-01-07 | Belum Join  | sushi        | 1            |
| A           | 2021-01-01 | 2021-01-07 | Belum Join  | curry        | 1            |
| B           | 2021-01-04 | 2021-01-09 | Belum Join  | sushi        | 1            |

**Insight**
- Customer A memesan sushi dan curry, sebelum akhirnya bergabung member.
- Customer B memesan sushi dan curry, sebelum akhirnya bergabung member.

## Questions - 8
**8. What is the total items and amount spent for each member before they became a member?**/**8. Berapa total item dan jumlah yang dibelanjakan setiap customer sebelum menjadi member?**

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
    SUM(total_spending) AS total_belanja
FROM
	spending_sebelum_join
GROUP BY
	customer_id,
    total_product;
~~~~
**STEP**
- Membuat CTE `spending_sebelum_join` yang berfungsi menghitung total spending.
- Filter dengan `s.order_date < mem.join_date` untuk mengetahui spending sebelum menjadi member.
- Kueri selanjutnya menghitung total spending dengan **SUM**, lalu **GROUP BY** kolom selain kolom `total_belanja`.

**Result**
| customer_id | total_product | total_belanja |
| ----------- | ------------- | ------------- |
| A           | 2             | 25            |
| B           | 3             | 40            |
| C           | 3             | 36            |

**Insight**
- Customer A memesan 2 produk dengan total belanja $25.
- Customer B memesan 3 produk dengan total belanja $40.
- Customer C memesan 3 produk dengan total belanja $36.

## Questions - 9
**9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**/**9. Jika setiap $1 yang dibelanjakan setara dengan 10 poin dan sushi memiliki pengali poin 2x - berapa banyak poin yang akan dimiliki setiap pelanggan?**

**Query**
~~~~sql
WITH count_point AS (
SELECT
    s.customer_id,
    m.product_name,
    m.price,
    CASE
    	WHEN product_name = 'sushi' THEN m.price*10*2
    	ELSE m.price*10
    END AS point
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
)
SELECT
	customer_id,
    SUM(point) AS total_point
FROM
	count_point
GROUP BY
	1
ORDER BY
	2 DESC;
~~~~
**STEP**
- Membuat tabel CTE yang menggunakan **CASE WHEN**: 1. Ketika `product_name = 'sushi' THEN m.price*10*2` karena khusus pada produk sushi ada 2x points multiplier, selain itu hanya mengembalikan nilai dari `m.price*10`

**Result**
| customer_id | total_point |
| ----------- | ----------- |
| B           | 940         |
| A           | 860         |
| C           | 360         |

**Insight**
- Customer B memiliki point paling banyak yaitu 940 poin.
- Customer A memiliki point sebanyak yaitu 860 poin.
- Customer C memiliki point sebanyak yaitu 360 poin.

## Questions - 10
**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**/**10. Pada minggu pertama setelah customer bergabung dengan program (termasuk tanggal bergabungnya), mereka memperoleh 2x poin untuk semua item, tidak hanya sushi - berapa banyak poin yang dimiliki pelanggan A dan B pada akhir Januari?**

**Query**
~~~~sql
WITH final_point AS (
SELECT
    s.customer_id,
    m.product_name,
    m.price,
    s.order_date,
    mem.join_date,
    s.order_date - mem.join_date AS hari_setelah_jadi_member,
    CASE
    	WHEN s.order_date - mem.join_date <= 7 
        AND s.order_date - mem.join_date >= 0 
        THEN m.price*10*2
    	ELSE m.price*10
    END AS point
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
)
SELECT
	customer_id,
    SUM(point) AS total_point_january
FROM
	final_point
WHERE
	order_date	< '2021-01-31'
    AND
	order_date >= join_date
GROUP BY
	customer_id
ORDER BY
	1;
~~~~
**STEP**
- Pada CTE `final_point`, berisi kueri untuk menghitung order date dengan klausa **CASE WHEN** ketika `s.order_date - mem.join_date <= 7` (7 disini adalah hari) dan `s.order_date - mem.join_date >= 0` (0 hari, hari ketika menjadi member) maka akan menghitung point yang dikali 2x, selain itu akan menjadi point biasa.
- Namun perlu diperhatikan bahwa catatan transaksi ini tidak hanya dalam bulan Januari, tetapi ada 1 transaksi juga oleh customer B pada bulan Febuari.
- Maka perlu difilter menggunakan `order_date	< '2021-01-31'`, serta `order_date >= join_date` untuk hanya menampilkan ketika customer sudah menjadi member.

**Result**
| customer_id | total_point_january |
| ----------- | ------------------- |
| A           | 1020                |
| B           | 440                 |

**Insight**
- Pada bulan Januari, customer A memiliki point paling banyak yaitu 1020.
- Lalu customer B memiliki point 440.
