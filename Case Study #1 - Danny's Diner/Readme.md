# Case 1 - Danny's Dinner

# Task
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

# ERD
![image](https://github.com/user-attachments/assets/d82802fb-12ca-41b4-90a9-9916e16b50c8)

# Case Study Questions

## Questions - 1
**1. What is the total amount each customer spent at the restaurant?**

**Query**
~~~~sql
SELECT
	s.customer_id, SUM(mn.price) AS spending
FROM dannys_diner.sales as s
FULL OUTER JOIN dannys_diner.menu AS mn ON mn.product_id = s.product_id
GROUP BY s.customer_id
ORDER BY spending DESC;
~~~~
**STEP**
- test

**Answers**
| customer_id | spending |
| ----------- | -------- |
| A           | 76       |
| B           | 74       |
| C           | 36       |

**Insight**
- 

## Questions - 2
**2. How many days has each customer visited the restaurant?**

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
- test

**Answers**
| customer_id | visited |
| ----------- | ------- |
| A           | 4       |
| B           | 6       |
| C           | 2       |

**Insight**
- 
