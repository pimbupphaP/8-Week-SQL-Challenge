# Case Study #2: Pizza Runner
## Part A. Pizza Metrics
Link to [DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/4338) to execute the queries shown below

1. How many pizzas were ordered?
```sql
SELECT COUNT(order_id) AS total_pizzas
FROM customer_orders_temp;
```

2. How many unique customer orders were made?
```sql
SELECT COUNT(DISTINCT order_id) AS total_pizzas
FROM customer_orders_temp;
```

3. How many successful orders were delivered by each runner?
```sql
SELECT runner_id, 
    COUNT(order_id) AS order_count
FROM runner_orders_temp
WHERE cancellation = ''
GROUP BY runner_id
ORDER BY runner_id;
```

4. How many of each type of pizza was delivered?
```sql
SELECT pizza_name, 
    COUNT(customer_orders_temp.pizza_id) AS pizza_count
FROM customer_orders_temp 
    JOIN pizza_names ON pizza_names.pizza_id = customer_orders_temp.pizza_id
    JOIN runner_orders_temp ON runner_orders_temp.order_id = customer_orders_temp.order_id
WHERE cancellation = ''
GROUP BY pizza_names.pizza_id, pizza_name
ORDER BY pizza_names.pizza_id;
```

5. How many Vegetarian and Meatlovers were ordered by each customer?
```sql
SELECT customer_id,
    pizza_name,
    COUNT(customer_orders_temp.pizza_id) AS pizza_count
FROM customer_orders_temp
    JOIN pizza_names ON pizza_names.pizza_id = customer_orders_temp.pizza_id
GROUP BY pizza_name, customer_id
ORDER BY customer_id;
```

6. What was the maximum number of pizzas delivered in a single order?
```sql
SELECT order_id,
    COUNT(customer_orders_temp.pizza_id) AS pizza_count
FROM customer_orders_temp
GROUP BY order_id
ORDER BY pizza_count DESC LIMIT 1;
```

7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
```sql
SELECT customer_id,
    COUNT(CASE
        WHEN exclusions != '' OR extras != '' THEN '1' END)
        AS changed_orders,
    COUNT(CASE
        WHEN exclusions = '' AND extras = '' THEN '1' END)
        AS no_changes
FROM customer_orders_temp 
    JOIN runner_orders_temp ON customer_orders_temp.order_id = runner_orders_temp.order_id
WHERE cancellation = ''
GROUP BY customer_id
ORDER BY customer_id;
```

8. How many pizzas were delivered that had both exclusions and extras?
```sql
SELECT COUNT(CASE
        WHEN exclusions != '' AND extras != '' THEN '1' END)
        AS exclusions_extras
FROM customer_orders_temp 
	JOIN runner_orders_temp ON customer_orders_temp.order_id = runner_orders_temp.order_id
WHERE cancellation = '';
```

9. What was the total volume of pizzas ordered for each hour of the day?
```sql
SELECT EXTRACT(HOUR FROM order_time) AS hour_of_day,
    COUNT(order_id) AS order_count
FROM customer_orders_temp
GROUP BY hour_of_day
ORDER BY hour_of_day;
```

10. What was the volume of orders for each day of the week?
```sql
SELECT TO_CHAR(order_time,'Day') AS day_of_week,
    COUNT(order_id) AS order_count
FROM customer_orders_temp
GROUP BY EXTRACT(ISODOW FROM order_time), day_of_week
ORDER BY EXTRACT(ISODOW FROM order_time);
```
