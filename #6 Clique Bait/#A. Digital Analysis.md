# Case Study #6. Clique Bait
## Part A. Digital Analysis
Link to [DB Fiddle](https://www.db-fiddle.com/f/jmnwogTsUE8hGqkZv9H7E8/636) to execute the queries shown below

### 1. How many users are there?
```sql
SELECT COUNT(DISTINCT user_id) FROM clique_bait.users;
```
<img width="1233" height="95" alt="image" src="https://github.com/user-attachments/assets/2de79fe0-9afb-4dfe-82fb-8c766eae45ec" />

### 2. How many cookies does each user have on average?
```sql
WITH user_cookies AS (
    SELECT user_id,
        COUNT(cookie_id) AS cookie_count
    FROM clique_bait.users
    GROUP BY user_id
  )
SELECT ROUND(AVG(cookie_count), 2) AS avg_cookies
FROM user_cookies;
```
<img width="1237" height="93" alt="image" src="https://github.com/user-attachments/assets/937fee07-5e01-4a0d-a6c9-eb2fed8ed165" />

### 3. What is the unique number of visits by all users per month?
```sql
SELECT to_char(date_trunc('month', event_time), 'MM-YYYY') AS month,
    COUNT(DISTINCT visit_id) AS visit_count
FROM clique_bait.events
GROUP BY month;
```
<img width="1239" height="269" alt="image" src="https://github.com/user-attachments/assets/37339e29-16c2-4c42-84ab-94a441fb11d6" />

### 4. What is the number of events for each event type?
```sql
SELECT event_name,
    COUNT(event_type) AS event_count
FROM clique_bait.events 
    JOIN clique_bait.event_identifier USING(event_type)
GROUP BY event_type, event_name
ORDER BY event_type;
```
<img width="1237" height="272" alt="image" src="https://github.com/user-attachments/assets/209b86d9-d2cb-4005-814f-ae48262ae3fc" />

### 5. What is the percentage of visits which have a purchase event?
```sql
WITH purchases AS (
    SELECT visit_id AS purchase_visits
    FROM clique_bait.events 
    WHERE event_type = '3'
  )
SELECT COUNT(DISTINCT purchase_visits) AS purchase_count,
    COUNT(DISTINCT visit_id) AS total_visits,
    ROUND(100.00 * COUNT(DISTINCT purchase_visits) / COUNT(DISTINCT visit_id), 2) AS percentage
FROM clique_bait.events 
    LEFT JOIN purchases ON purchases.purchase_visits = clique_bait.events.visit_id;
```
<img width="1241" height="93" alt="image" src="https://github.com/user-attachments/assets/5b09c064-8fe5-4ca2-aeb2-71bdb898c164" />

### 6. What is the percentage of visits which view the checkout page but do not have a purchase event?
```sql
WITH visit_count AS (
    SELECT COUNT(DISTINCT CASE WHEN page_id = '12' AND event_type = '1' THEN visit_id END) AS checkout_visits,
        COUNT(DISTINCT CASE WHEN event_type = '3' THEN visit_id END) AS purchases
    FROM clique_bait.events
  )
SELECT checkout_visits,
    checkout_visits - purchases AS no_purchase,
    ROUND(100 - (100.00 * purchases / checkout_visits), 2) AS percentage
FROM visit_count;
```
<img width="1242" height="93" alt="image" src="https://github.com/user-attachments/assets/789b14f3-c7cf-4888-9a29-734d5fda7954" />

### 7. What are the top 3 pages by number of views?
```sql
SELECT page_name,
    COUNT(visit_id) AS view_count
FROM clique_bait.events
    JOIN clique_bait.page_hierarchy USING(page_id)
WHERE event_type = '1'
GROUP BY page_name
ORDER BY view_count DESC LIMIT 3;
```
<img width="1240" height="184" alt="image" src="https://github.com/user-attachments/assets/495b0044-42ec-4480-b2c4-913be84d072b" />

### 8. What is the number of views and cart adds for each product category?
```sql
SELECT product_category,
    COUNT(CASE WHEN event_type = '1' THEN visit_id END) AS view_count,
    COUNT(CASE WHEN event_type = '2' THEN visit_id END) AS cart_count
FROM clique_bait.events
    JOIN clique_bait.page_hierarchy USING(page_id)
WHERE product_category IS NOT NULL
GROUP BY product_category
ORDER BY view_count DESC, cart_count;
```
<img width="1238" height="183" alt="image" src="https://github.com/user-attachments/assets/8a86116c-9797-4dec-80d9-0565d9236398" />

### 9. What are the top 3 products by purchases?
```sql
WITH cart AS (
    SELECT visit_id,
        page_name
    FROM clique_bait.events 
        JOIN clique_bait.page_hierarchy USING(page_id)
    WHERE event_type = '2'
        AND product_category IS NOT NULL
  ), purchase AS (
    SELECT visit_id
    FROM clique_bait.events
    WHERE event_type = '3'
  )
SELECT page_name,
    COUNT(visit_id) as purchase_count
FROM cart
    JOIN purchase USING(visit_id)
GROUP BY page_name
ORDER BY purchase_count DESC;    
```
<img width="1246" height="448" alt="image" src="https://github.com/user-attachments/assets/57cfa7fc-70b0-40d2-af58-5df74cd1bca7" />
