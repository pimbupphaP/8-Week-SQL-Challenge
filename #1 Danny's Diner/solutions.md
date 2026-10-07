# Case Study #1: Danny's Diner 
Link to [DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/11414) to execute queries shown below

1. What is the total amount each customer spent at the restaurant?
```sql
SELECT customer_id, 
	SUM(price) AS total_spent
FROM sales JOIN menu
	ON sales.product_id = menu.product_id
GROUP BY customer_id
ORDER BY customer_id;
```

2. How many days has each customer visited the restaurant?
```sql
SELECT customer_id,
	COUNT(DISTINCT order_date) AS visits
FROM sales
GROUP BY customer_id
ORDER BY customer_id;
```

3. What was the first item from the menu purchased by each customer?
```sql
WITH rank_order AS (
  SELECT customer_id,
  	order_date,
  	product_name,
  	dense_rank() OVER (PARTITION BY customer_id ORDER BY order_date) AS rank
  FROM sales JOIN menu
  	ON sales.product_id = menu.product_id
  )
SELECT customer_id, product_name
FROM rank_order
WHERE rank = 1
GROUP BY customer_id, product_name;
```

4. What is the most purchased item on the menu and how many times was it purchased by all customers?
```sql
SELECT product_name,
	COUNT(sales.product_id) AS product_purchases
FROM sales JOIN menu
	ON sales.product_id = menu.product_id
GROUP BY product_name
ORDER BY product_purchases DESC LIMIT 1;
```

5. Which item was the most popular for each customer?
```sql
WITH rank_item AS (
  SELECT customer_id,
  	product_name,
  	COUNT(product_name) AS item_count,
  	dense_rank() OVER (PARTITION BY customer_id ORDER BY COUNT(product_name) DESC) AS rank
  FROM sales JOIN menu
  	ON sales.product_id = menu.product_id
  GROUP BY customer_id, product_name
  )
SELECT customer_id, product_name, item_count
FROM rank_item
WHERE rank = 1;
```
