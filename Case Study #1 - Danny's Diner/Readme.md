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

**Answers**
| customer_id | visited |
| ----------- | ------- |
| A           | 4       |
| B           | 6       |
| C           | 2       |

**Insight**
- Customer A datang sebanyak 4x.
- Customer B paling sering datang yaitu 6x.
- Customer C datang sebanyak 2x.
