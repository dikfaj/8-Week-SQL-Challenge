
# Case Study #1: Danny's Diner



## Question and Solution



1. What is the total amount each customer spent at the restaurant?

```sql
SELECT
  customer_id,
  SUM(price) as total_spent
FROM sales s
JOIN menu m on s.product_id = m.product_id 
GROUP BY customer_id
ORDER BY customer_id;
```
### Langkah-Langkah
- Gabungkan tabel `sales` dan `menu` menggunakan `JOIN`
- Totalkan `price` dengan fungsi `sum`
- Group hasil agregat berdasarkan `customer_id`

### Jawaban
![1](https://github.com/dikfaj/8-Week-SQL-Challenge/assets/39393133/d712c770-3559-48bf-8bec-0af0c2e88bfb)

- Customer A menghabiskan $76
- Customer B menghabiskan $74
- Customer C menghabiskan $36

# 

2. How many days has each customer visited the restaurant?
```sql
SELECT
  customer_id,
  COUNT(DISTINCT order_date) AS visit_count
FROM sales
GROUP BY customer_id 
ORDER BY customer_id;
```
### Langkah-Langkah
- Gunakan `COUNT(DISTINCT order_date)` untuk mencari nilai unik dari `order_date`
- Gunakan `GROUP BY` dan `ORDER BY`
### Jawaban
![2](https://github.com/dikfaj/8-Week-SQL-Challenge/assets/39393133/1264aa67-05a6-4ad9-832f-380e1fd3e105)
- Customer A mengunjungi restoran sebanyak 4 kali
- Customer B mengunjungi restoran sebanyak 6 kali
- Customer C mengunjungi restoran sebanyak 2 kali

# 
3. What was the first item from the menu purchased by each customer?
```sql
WITH first_item AS (
SELECT
  customer_id,
  order_date,
  product_name,
  DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY order_date) AS rank
FROM sales s
JOIN menu m ON s.product_id = m.product_id
)

SELECT
  customer_id,
  product_name
FROM first_item
WHERE rank = 1
GROUP BY 1,2;
```
### Langkah-Langkah
- Gunakan Common Table Expression (CTE) dengan nama `first_item`. Dalam CTE tersebut, buat sebuah kolom baru `rank` dengan window function `DENSE_RANK()` untuk menentukan row number. `PARTITION BY` membagi data berdasarkan `customer_id` dan `ORDER BY` untuk mengurutkan data berdasarkan `order_date`
- Selanjutnya filter menggunakan `WHERE RANK = 1` untuk menentukan item pertama dari masing-masing customer_id

### Jawaban
![3](https://github.com/dikfaj/8-Week-SQL-Challenge/assets/39393133/19033e2f-4fcc-4028-aa9e-601793833ba1)

- Customer A membeli curry dan sushi sebagai item pembelian pertamanya
- Customer B membeli curry
- Customer C membeli ramen

# 

4. What is the most purchased item on the menu and how many times was it purchased by all customers?

```sql
SELECT
  m.product_name,
  COUNT(product_name) AS total_purchased
FROM sales s
JOIN menu m ON S.product_id = m.product_id
GROUP BY product_name
LIMIT 1;
```
## Langkah-Langkah
- Gabungkan tabel `sales` dan `menu` menggunakan `JOIN` berdasarkan `product_id`
- Hitung `product_name` dengan `COUNT`
- `LIMIT 1` untuk memfilter nilai tertinggi

## Jawaban
![4](https://github.com/dikfaj/8-Week-SQL-Challenge/assets/39393133/0b487a07-c980-43d8-857f-25bc02fb41d2)
- Ramen merupakan produk dengan penjualan terbanyak yaitu 8

# 

5. Which item was the most popular for each customer?
```sql
WITH most_popular AS (
SELECT
  s.customer_id,
  m.product_name,
  COUNT(m.product_id) AS order_count,
  DENSE_RANK() OVER (PARTITION BY s.customer_id ORDER BY COUNT(s.customer_id) DESC) AS rank
FROM sales s 
JOIN menu m ON s.product_id = m.product_id
GROUP BY 1,2
ORDER BY s.customer_id
)

SELECT
  customer_id,
  product_name,
  order_count
FROM most_popular
WHERE rank = 1
```

## Langkah-Langkah
- Gunakan Common Table Expression (CTE) dengan nama `most_popular`. Dalam CTE tersebut, buat sebuah kolom baru `rank` dengan window function `DENSE_RANK()` untuk menentukan row number. `PARTITION BY` membagi data berdasarkan `customer_id` dan `ORDER BY` untuk mengurutkan data berdasarkan `customer_id` secara `DESC`

## Jawaban
![5](https://github.com/dikfaj/8-Week-SQL-Challenge/assets/39393133/329cc92d-97e5-4928-8e70-59b20c51fb7d)
- Customer A membeli ramen sebanyak 3 kali
- Customer B membeli sushi, curry, dan ramen masing-masing sebanyak 3 kali
- Customer C membeli ramen sebanyak 3 kali

# 
6. Which item was purchased first by the customer after they became a member?
```sql
WITH became_member AS (
SELECT
  s.customer_id,
  s.product_id,
  ROW_NUMBER() OVER (PARTITION BY s.customer_id ORDER BY m.join_date) AS rank
FROM sales s 
JOIN members m ON s.customer_id = m.customer_id AND s.order_date > m.join_date 
)

SELECT
  customer_id,
  m.product_name 
FROM became_member b
JOIN menu m ON b.product_id = m.product_id
WHERE rank = 1
ORDER BY customer_id;
```

## Langkah-Langkah
- `JOIN` tabel `sales` dan `member` berdasarkan `customer_id` dan `order_date` > `join_date` berfungsi untuk menampilkan data dimana customer melakukan pembelian setelah menjadi member
- Gunakan Common Table Expression (CTE) dengan nama `became_member`. Dalam CTE tersebut, buat sebuah kolom baru `rank` dengan window function `ROW_NUMBER()` untuk menentukan row number. `PARTITION BY` membagi data berdasarkan `customer_id` dan `ORDER BY` untuk mengurutkan data berdasarkan `join_date`

## Jawaban
![6](https://github.com/dikfaj/8-Week-SQL-Challenge/assets/39393133/9d3fdbb0-12b6-41cb-a250-a53e36fb5718)
- Hanya ada dua customer_id yang melalukan pembelian setelah menjadi member yaitu customer_id A dan B
- Customer A membeli ramen dan Customer B member sushi

# 
7. Which item was purchased just before the customer became a member?
```sql
WITH before_member AS (
SELECT
  s.customer_id,
  s.product_id,
  DENSE_RANK() OVER (PARTITION BY s.customer_id ORDER BY s.order_date DESC) AS rank
FROM sales s 
JOIN members m ON s.customer_id = m.customer_id AND s.order_date < m.join_date 
)

SELECT
  b.customer_id,
  product_name
FROM before_member b
JOIN menu m ON b.product_id = m.product_id 
WHERE rank = 1
GROUP BY 1,2;
```
## Langkah-Langkah
- `JOIN` tabel `sales` dan `member` berdasarkan `customer_id` dan `order_date` < `join_date` berfungsi untuk menampilkan data dimana customer melakukan pembelian sebelum menjadi member
- Gunakan Common Table Expression (CTE) dengan nama `before_member`. Dalam CTE tersebut, buat sebuah kolom baru `rank` dengan window function `DENSE_RANK()` untuk menentukan row number. `PARTITION BY` membagi data berdasarkan `customer_id` dan `ORDER BY` untuk mengurutkan data berdasarkan `order_date` secara `DESC`

## Jawaban
![7](https://github.com/dikfaj/8-Week-SQL-Challenge/assets/39393133/dbaa86bb-e92a-47b3-ac3b-cc38d7baf10a)
- Hanya ada dua customer yang melakukan pembelian sebelum menjadi member yaitu customer A dan B
- Tepat sebelum menjadi member, Customer A membeli curry dan sushi sedangkan Customer B membeli sushi saja

