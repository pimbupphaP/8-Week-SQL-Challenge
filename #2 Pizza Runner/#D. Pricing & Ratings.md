# Case Study #2: Pizza Runner
## Part D. Pricing & Ratings
Link to [DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/4344) to execute the queries shown below

### 1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?
```sql
SELECT SUM(CASE
    WHEN customer_orders_temp.pizza_id = '1' THEN 12
        ELSE 10 END)
        AS total_revenue
FROM customer_orders_temp
    JOIN runner_orders_temp USING(order_id)
WHERE cancellation = '';
```

### 2. What if there was an additional $1 charge for any pizza extras?
```sql
SELECT SUM(CASE
        WHEN customer_orders_temp.pizza_id = '1' THEN 12
        ELSE 10 END)
       + 
    SUM(CASE
        WHEN extras IS NULL THEN 0
        ELSE array_length(string_to_array(extras, ','), 1) END)
    AS total_revenue
FROM customer_orders_temp
    JOIN runner_orders_temp USING(order_id)
WHERE cancellation = '';
```

### 3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.
```sql
DROP TABLE IF EXISTS runner_ratings;
CREATE TABLE runner_ratings (
    "order_id" INTEGER,
    "runner_id" INTEGER,
    "runner_rating" INTEGER NOT NULL CHECK (runner_rating BETWEEN 1 and 5)
);

INSERT INTO runner_ratings
    ("order_id", "runner_id", "runner_rating")
    VALUES
    ('1', '1', '3'),
    ('2', '1', '2'),
    ('3', '1', '4'),
    ('4', '2', '2'),
    ('5', '3', '4'),
    ('7', '2', '5'),
    ('8', '2', '1'),
    ('10', '1', '3');
  
SELECT * FROM runner_ratings;
```

### 4. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries? customer_id, order_id, runner_id, rating, order_time, pickup_time, Time between order and pickup, Delivery duration, Average speed, Total number of pizzas
```sql
SELECT customer_id,
    customer_orders_temp.order_id,
    runner_orders_temp.runner_id,
    runner_rating,
    order_time,
    pickup_time,
    ROUND((EXTRACT(EPOCH FROM (pickup_time::timestamp - order_time::TIMESTAMP)) / 60)::NUMERIC, 2) AS time_between_order_and_pickup,
    duration,
    ROUND(AVG(distance)/AVG(duration::NUMERIC / 60),2) AS avg_speed,
    COUNT(pizza_id) AS pizza_count
FROM customer_orders_temp
    JOIN runner_orders_temp USING(order_id)
    JOIN runner_ratings USING(order_id)
GROUP BY customer_orders_temp.order_id, customer_id, runner_orders_temp.runner_id, runner_rating, order_time, pickup_time, duration
ORDER BY customer_orders_temp.order_id;
```

### 5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?
```sql
WITH pizza_revenue AS (
SELECT SUM(CASE
        WHEN customer_orders_temp.pizza_id = '1' THEN 12
        ELSE 10 END)
        AS total_revenue
FROM customer_orders_temp
    JOIN runner_orders_temp USING(order_id)
WHERE cancellation = ''
)
SELECT 
    (SELECT total_revenue FROM pizza_revenue) - (SELECT SUM(distance*0.3) FROM runner_orders_temp) AS leftover_money
```
