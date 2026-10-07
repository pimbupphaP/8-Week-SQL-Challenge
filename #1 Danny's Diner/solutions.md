### [Link to DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/11413)

1. What is the total amount each customer spent at the restaurant?
```sql
SELECT customer_id, 
	SUM(price) AS total_spent
FROM sales JOIN menu
	ON sales.product_id = menu.product_id
GROUP BY customer_id
ORDER BY customer_id;
```
