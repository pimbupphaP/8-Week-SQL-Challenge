# Case Study #2: Pizza Runner
## Part A. Pizza Metrics
Link to [DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/4344) to execute the queries shown below

1. How many pizzas were ordered?
```sql
SELECT COUNT(order_id) AS total_pizzas
FROM customer_orders_temp;
```
<img width="938" height="71" alt="image" src="https://github.com/user-attachments/assets/6c8785df-19a9-45d6-9e52-3e7f104325a0" />

2. How many unique customer orders were made?
```sql
SELECT COUNT(DISTINCT order_id) AS total_pizzas
FROM customer_orders_temp;
```
<img width="932" height="72" alt="image" src="https://github.com/user-attachments/assets/5ef66fd3-ea53-4614-877f-933977f6e575" />

3. How many successful orders were delivered by each runner?
```sql
SELECT runner_id, 
    COUNT(order_id) AS order_count
FROM runner_orders_temp
WHERE cancellation = ''
GROUP BY runner_id
ORDER BY runner_id;
```
<img width="936" height="142" alt="image" src="https://github.com/user-attachments/assets/2eccff28-4e4c-4f64-b032-ca6a25315a1a" />

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
<img width="934" height="108" alt="image" src="https://github.com/user-attachments/assets/2c54ed3b-68b8-468f-9368-351371e81b21" />

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
<img width="938" height="303" alt="image" src="https://github.com/user-attachments/assets/e513a4af-f993-4539-ab73-e838ca736211" />

6. What was the maximum number of pizzas delivered in a single order?
```sql
SELECT order_id,
    COUNT(customer_orders_temp.pizza_id) AS pizza_count
FROM customer_orders_temp
GROUP BY order_id
ORDER BY pizza_count DESC LIMIT 1;
```
<img width="934" height="72" alt="image" src="https://github.com/user-attachments/assets/fc7f94d5-5cda-4519-8918-05da150c317f" />

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
<img width="937" height="205" alt="image" src="https://github.com/user-attachments/assets/90d07032-0808-4300-b162-1b99a498bca6" />

8. How many pizzas were delivered that had both exclusions and extras?
```sql
SELECT COUNT(CASE
        WHEN exclusions != '' AND extras != '' THEN '1' END)
        AS exclusions_extras
FROM customer_orders_temp 
	JOIN runner_orders_temp ON customer_orders_temp.order_id = runner_orders_temp.order_id
WHERE cancellation = '';
```
<img width="938" height="72" alt="image" src="https://github.com/user-attachments/assets/047fb87a-3217-4968-8b75-821ac7b9a152" />

9. What was the total volume of pizzas ordered for each hour of the day?
```sql
SELECT EXTRACT(HOUR FROM order_time) AS hour_of_day,
    COUNT(order_id) AS order_count
FROM customer_orders_temp
GROUP BY hour_of_day
ORDER BY hour_of_day;
```
<img width="934" height="241" alt="image" src="https://github.com/user-attachments/assets/ec5e2d44-a8c4-418d-8954-604a5e792941" />

10. What was the volume of orders for each day of the week?
```sql
SELECT TO_CHAR(order_time,'Day') AS day_of_week,
    COUNT(order_id) AS order_count
FROM customer_orders_temp
GROUP BY EXTRACT(ISODOW FROM order_time), day_of_week
ORDER BY EXTRACT(ISODOW FROM order_time);
```
<img width="932" height="175" alt="image" src="https://github.com/user-attachments/assets/a0c37b79-fdf8-4edc-834e-c371bd48ab63" />
