# Case Study #1: Danny's Diner 
Link to [DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/11417) to execute the queries shown below

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

6. Which item was purchased first by the customer after they became a member?
```sql
WITH member_join AS (
    SELECT members.customer_id,
        product_id,
        ROW_NUMBER() OVER(PARTITION BY members.customer_id ORDER BY order_date) AS rank
    FROM members JOIN sales
        ON members.customer_id = sales.customer_id
        AND order_date > join_date
  )
SELECT customer_id, product_name
FROM member_join JOIN menu
    ON member_join.product_id = menu.product_id
WHERE rank = 1
GROUP BY customer_id, product_name;
```

7. Which item was purchased just before the customer became a member?
```sql
WITH before_join AS (
    SELECT members.customer_id,
        product_id,
        DENSE_RANK() OVER(PARTITION BY members.customer_id ORDER BY order_date DESC) AS rank
    FROM members JOIN sales
        ON members.customer_id = sales.customer_id
        AND order_date < join_date
  )
SELECT customer_id, product_name
FROM before_join JOIN menu
    ON before_join.product_id = menu.product_id
WHERE rank = 1
GROUP BY customer_id, product_name;
```

8. What is the total items and amount spent for each member before they became a member?
```sql
SELECT sales.customer_id,
    COUNT(sales.product_id) AS item_count,
    SUM(price) AS total_spent
FROM sales 
    JOIN menu
        ON sales.product_id = menu.product_id
    JOIN members
        ON sales.customer_id = members.customer_id
        AND order_date < join_date
GROUP BY sales.customer_id
ORDER BY sales.customer_id ASC;
```

9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
```sql
WITH points AS (
    SELECT product_id,
        product_name,
        CASE
            WHEN product_name = 'sushi' THEN price*2*10
            ELSE price*10 END 
            AS points_calc
    FROM menu
  )
SELECT customer_id,
    SUM(points_calc) AS customer_points
FROM points JOIN sales
    ON points.product_id = sales.product_id
GROUP BY customer_id
ORDER BY customer_id ASC;
```

10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
```sql
WITH member_week AS (
    SELECT customer_id,
        join_date,
        (join_date + INTERVAL '6 days') :: date AS first_week
    FROM members
  )
SELECT sales.customer_id,
    SUM(CASE
        WHEN product_name = 'sushi' THEN price*2*10
        WHEN order_date BETWEEN join_date AND first_week THEN price*2*10
        ELSE price*10 END)
        AS points_calc
FROM sales 
    JOIN member_week
        ON sales.customer_id = member_week.customer_id
        AND order_date <= '2021-01-31'
        AND order_date >= join_date
    JOIN menu
        ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
ORDER BY sales.customer_id;
```

Bonus 1. Join all the things
```sql
SELECT sales.customer_id,
    order_date,
    product_name,
    price,
    CASE
        WHEN order_date < join_date THEN 'N'
        ELSE 'Y' END
        AS member
FROM sales 
    LEFT JOIN members 
        ON sales.customer_id = members.customer_id
    JOIN menu 
        ON sales.product_id = menu.product_id
ORDER BY customer_id, order_date;
```

Bonus 2. Rank all the things
```sql
SELECT sales.customer_id,
    sales.order_date,
    product_name,
    price,
    CASE
        WHEN sales.order_date >= join_date THEN 'Y'
        ELSE 'N' END
        AS member,
    CASE
        WHEN sales.order_date < join_date THEN NULL
        ELSE dense_rank() OVER (PARTITION BY sales.customer_id ORDER BY order_date) END
        AS ranking
FROM sales 
    LEFT JOIN members 
        ON sales.customer_id = members.customer_id
    JOIN menu 
        ON sales.product_id = menu.product_id
ORDER BY sales.customer_id, sales.order_date;
```
