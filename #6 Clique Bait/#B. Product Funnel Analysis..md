# Case Study #6. Clique Bait
## Part B. Product Funnel Analysis
Link to [DB Fiddle](https://www.db-fiddle.com/f/jmnwogTsUE8hGqkZv9H7E8/636) to execute the queries shown below

### `product_funnel` Table
Using a single SQL query - create a new output table which has the following details:
- How many times was each product viewed?
- How many times was each product added to cart?
- How many times was each product added to a cart but not purchased (abandoned)?
- How many times was each product purchased?
```sql
DROP TABLE IF EXISTS product_funnel;

CREATE TEMPORARY TABLE product_funnel AS
WITH products AS (
    SELECT page_id, 
        page_name,
        product_category,
        product_id,
        COUNT(CASE WHEN event_type = '1' THEN visit_id END) AS page_view,
        COUNT(CASE WHEN event_type = '2' THEN visit_id END) AS cart_add
    FROM clique_bait.events
        JOIN clique_bait.page_hierarchy USING(page_id)
    WHERE product_id IS NOT NULL
    GROUP BY page_id, page_name, product_category, product_id
    ORDER BY page_id
  ), cart AS (
    SELECT visit_id,
        page_id, 
        page_name,
        product_category,
        product_id
    FROM clique_bait.events 
        JOIN clique_bait.page_hierarchy USING(page_id)
    WHERE event_type = '2'
        AND product_category IS NOT NULL
  ), purchase AS (
    SELECT visit_id 
    FROM clique_bait.events
    WHERE event_type = '3'
  ), purchase_count AS (
    SELECT page_id, 
        page_name,
        product_category,
        product_id,
        COUNT(visit_id) as purchase
    FROM cart
        JOIN purchase USING(visit_id)
    GROUP BY page_id, page_name, product_category, product_id
   )
SELECT products.page_id, 
    products.page_name,
    products.product_category,
    products.product_id,
    page_view,
    cart_add,
    purchase,
    cart_add - purchase AS abandon
FROM products
    JOIN purchase_count USING(page_id);
    
SELECT * FROM product_funnel;
```
<img width="1239" height="447" alt="image" src="https://github.com/user-attachments/assets/8b4c6071-19d3-4c8b-84b2-4febcdb99401" />

### `category_funnel` Table
Additionally, create another table which further aggregates the data for the above points but this time for each product category instead of individual products.
```sql
DROP TABLE IF EXISTS category_funnel;

CREATE TEMPORARY TABLE category_funnel AS
WITH products AS (
    SELECT product_category,
        COUNT(CASE WHEN event_type = '1' THEN visit_id END) AS page_view,
        COUNT(CASE WHEN event_type = '2' THEN visit_id END) AS cart_add
    FROM clique_bait.events
        JOIN clique_bait.page_hierarchy USING(page_id)
    WHERE product_category IS NOT NULL
    GROUP BY product_category
  ), cart AS (
    SELECT visit_id,
        product_category
    FROM clique_bait.events 
        JOIN clique_bait.page_hierarchy USING(page_id)
    WHERE event_type = '2'
        AND product_category IS NOT NULL
  ), purchase AS (
    SELECT visit_id 
    FROM clique_bait.events
    WHERE event_type = '3'
  ), purchase_count AS (
    SELECT product_category,
        COUNT(visit_id) as purchase
    FROM cart
        JOIN purchase USING(visit_id)
    GROUP BY product_category
   )
SELECT products.product_category,
    page_view,
    cart_add,
    purchase,
    cart_add - purchase AS abandon
FROM products
    JOIN purchase_count USING(product_category);
    
SELECT * FROM category_funnel;
```
<img width="1244" height="188" alt="image" src="https://github.com/user-attachments/assets/9ce50d14-6d86-4762-a18b-eafbb9138350" />

### 1. Which product had the most views, cart adds and purchases?
```sql
SELECT *
FROM product_funnel
ORDER BY page_view DESC LIMIT 1;

SELECT *
FROM product_funnel
ORDER BY cart_add DESC LIMIT 1;

SELECT *
FROM product_funnel
ORDER BY purchase DESC LIMIT 1;
```
<img width="1248" height="93" alt="image" src="https://github.com/user-attachments/assets/2b3d92f5-ac5a-4a91-b8d0-3a4fae6d48fb" />
<img width="1249" height="92" alt="image" src="https://github.com/user-attachments/assets/43c95ce1-5036-44e2-ba74-0eaddb6367b0" />
<img width="1249" height="97" alt="image" src="https://github.com/user-attachments/assets/1f168404-e5d0-42b8-aca0-895fb0ee1fe2" />

### 2. Which product was most likely to be abandoned?
```sql
SELECT *
FROM product_funnel
ORDER BY abandon DESC LIMIT 1;
```
<img width="1244" height="95" alt="image" src="https://github.com/user-attachments/assets/88ccfde3-f933-4e1a-9096-7d8c3977dfd9" />

### 3. Which product had the highest view to purchase percentage?
```sql
SELECT page_name,
    page_view,
    purchase,
    ROUND(100.00 * purchase / page_view, 2) AS percentage
FROM product_funnel
ORDER BY percentage DESC LIMIT 1;
```
<img width="1249" height="95" alt="image" src="https://github.com/user-attachments/assets/641da38f-4f68-4a86-bbcb-61e07b9910db" />

### 4. What is the average conversion rate from view to cart add?
```sql
SELECT ROUND(AVG(100.00 * cart_add / page_view), 2) AS conversion_rate
FROM product_funnel;
```
<img width="1242" height="98" alt="image" src="https://github.com/user-attachments/assets/f8052933-8263-4034-938b-3d8edb387c4a" />

### 5. What is the average conversion rate from cart add to purchase?
```sql
SELECT ROUND(AVG(100.00 * purchase / cart_add), 2) AS conversion_rate
FROM product_funnel;
```
<img width="1239" height="96" alt="image" src="https://github.com/user-attachments/assets/7d819876-0cf4-496e-94c2-e72785769176" />
