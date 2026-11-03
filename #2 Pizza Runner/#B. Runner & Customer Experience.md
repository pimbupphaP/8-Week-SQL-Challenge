# Case Study #2: Pizza Runner
## Part B. Runner and Customer Experience
Link to [DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/4344) to execute the queries shown below

### 1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
```sql
SELECT DATE_PART('week', registration_date) AS registration_week,
    COUNT(runner_id) AS runner_count
FROM runners
GROUP BY DATE_PART('week', registration_date);
```
<img width="935" height="140" alt="image" src="https://github.com/user-attachments/assets/af441036-b929-49d8-928c-7950e6b7c739" />

### 2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
```sql
WITH runner_times AS (
    SELECT runner_id,
        runner_orders_temp.order_id,
        order_time,
        pickup_time,
        EXTRACT(EPOCH FROM (pickup_time::timestamp - order_time::TIMESTAMP)) / 60 AS mins_dif
    FROM customer_orders_temp 
        JOIN runner_orders_temp ON customer_orders_temp.order_id = runner_orders_temp.order_id
    WHERE pickup_time IS NOT NULL
)
SELECT runner_id,
    ROUND(AVG(mins_dif::NUMERIC),2) as avg_mins
FROM runner_times
GROUP BY runner_id
ORDER BY runner_id;
```
<img width="935" height="140" alt="image" src="https://github.com/user-attachments/assets/f248d034-0e90-4d07-bb88-13be5b845c0e" />

### 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?
```sql
WITH prep_time AS (
    SELECT customer_orders_temp.order_id,
        COUNT(customer_orders_temp.order_id) AS order_count,
        order_time,
        pickup_time,
        EXTRACT(EPOCH FROM (pickup_time::timestamp - order_time::TIMESTAMP)) / 60 AS prep_time
    FROM customer_orders_temp 
        JOIN runner_orders_temp ON customer_orders_temp.order_id = runner_orders_temp.order_id
    WHERE pickup_time IS NOT NULL
    GROUP BY customer_orders_temp.order_id, order_time, pickup_time
)
SELECT order_count,
    ROUND(AVG(prep_time::NUMERIC),2) AS prep_mins,
    ROUND(AVG(prep_time::NUMERIC) / order_count,2) AS mins_per_order
FROM prep_time
GROUP BY order_count
ORDER BY order_count;
```
<img width="935" height="138" alt="image" src="https://github.com/user-attachments/assets/934d2e51-938f-4790-88e3-3b1b482a49d3" />

### 4. What was the average distance travelled for each customer?
```sql
SELECT customer_id,
    ROUND(AVG(distance)) AS avg_km
FROM customer_orders_temp 
    JOIN runner_orders_temp ON customer_orders_temp.order_id = runner_orders_temp.order_id
WHERE cancellation = ''
GROUP BY customer_id
ORDER BY customer_id;
```
<img width="935" height="209" alt="image" src="https://github.com/user-attachments/assets/5ea3f3e1-3e81-4b78-a9e6-848993f62c6e" />

### 5. What was the difference between the longest and shortest delivery times for all orders?
```sql
SELECT MAX(duration) - MIN(duration) AS time_dif
FROM runner_orders_temp;
```
<img width="934" height="77" alt="image" src="https://github.com/user-attachments/assets/ef0c8ac9-ed7a-477c-8b30-d4378ce2e286" />

### 6. What was the average speed for each runner for each delivery and do you notice any trend for these values?
```sql
SELECT runner_id,
    order_id,
    ROUND(AVG(distance)) AS avg_distance_km,
    ROUND(AVG(duration::NUMERIC / 60),2) AS avg_duration_hr,
    ROUND(AVG(distance)/AVG(duration::NUMERIC / 60),2) AS km_per_hr
FROM runner_orders_temp
WHERE cancellation = ''
GROUP BY runner_id, order_id;
```
<img width="935" height="302" alt="image" src="https://github.com/user-attachments/assets/81bf4652-b816-404c-92cb-aa9dda3a118a" />

### 7. What is the successful delivery percentage for each runner?
```sql
SELECT runner_id,
    COUNT(order_id) AS total_orders,
    COUNT(pickup_time) AS total_pickups,
    ROUND(COUNT(pickup_time)::NUMERIC / COUNT(order_id)::NUMERIC * 100) AS success_percent
FROM runner_orders_temp
GROUP BY runner_id
ORDER BY runner_id;
```
<img width="935" height="140" alt="image" src="https://github.com/user-attachments/assets/59888adb-ba53-42b5-bb87-88b9f44871f3" />
