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
<img width="932" height="140" alt="image" src="https://github.com/user-attachments/assets/d6de4cf6-17ac-4c02-a346-503f24c3dec4" />

2. How many days has each customer visited the restaurant?
```sql
SELECT customer_id,
    COUNT(DISTINCT order_date) AS visits
FROM sales
GROUP BY customer_id
ORDER BY customer_id;
```
<img width="931" height="141" alt="image" src="https://github.com/user-attachments/assets/7365f263-1545-4140-8463-2100c3bfd6a0" />

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
<img width="930" height="170" alt="image" src="https://github.com/user-attachments/assets/35d59d29-13ce-43b7-9a5c-4bd7024eae82" />

4. What is the most purchased item on the menu and how many times was it purchased by all customers?
```sql
SELECT product_name,
    COUNT(sales.product_id) AS product_purchases
FROM sales JOIN menu
    ON sales.product_id = menu.product_id
GROUP BY product_name
ORDER BY product_purchases DESC LIMIT 1;
```
<img width="929" height="71" alt="image" src="https://github.com/user-attachments/assets/393ab2fa-337f-4f47-9ad3-f42571565bf5" />

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
<img width="930" height="205" alt="image" src="https://github.com/user-attachments/assets/a62d58c6-fb83-4a95-842c-30c52b0bb9c4" />

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
<img width="933" height="105" alt="image" src="https://github.com/user-attachments/assets/7a95b6a2-2396-4a05-853c-d59334cfcd8b" />

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
<img width="929" height="137" alt="image" src="https://github.com/user-attachments/assets/95d3f6ce-4df8-4425-b11c-b9fa061ab54d" />

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
<img width="928" height="107" alt="image" src="https://github.com/user-attachments/assets/20718f0a-80ae-4a5d-9b5d-9065f5b0c7dd" />

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
<img width="935" height="140" alt="image" src="https://github.com/user-attachments/assets/8923ba30-8da3-4a64-a6b2-8cc39255f898" />

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
<img width="933" height="106" alt="image" src="https://github.com/user-attachments/assets/8620ac1d-be51-40af-ae44-bdcda8732746" />

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
<img width="933" height="537" alt="image" src="https://github.com/user-attachments/assets/402c13b5-aa1f-4e41-bcb3-15e3762e870d" />

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
<img width="931" height="539" alt="image" src="https://github.com/user-attachments/assets/617d7ca6-d87e-4960-a42e-971ab1a56201" />
