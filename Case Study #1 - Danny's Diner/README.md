
# Case Study #1: Danny's Diner



## Question and Solution



1. What is the total amount each customer spent at the restaurant?

```sql
SELECT
  customer_id,
  sum(price) as total_spent
FROM sales s
JOIN menu m on s.product_id = m.product_id 
GROUP BY customer_id
ORDER BY customer_id;
```
### Step

- Gabungkan tabel `sales` dan `menu` menggunakan `JOIN`
- Totalkan `price` dengan fungsi `sum`
- Group hasil agregat berdasarkan `customer_id`

### Answer
![1](https://github.com/dikfaj/8-Week-SQL-Challenge/assets/39393133/d712c770-3559-48bf-8bec-0af0c2e88bfb)

- Customer A menghabiskan $76
- Customer B menghabiskan $74
- Customer C menghabiskan $36
