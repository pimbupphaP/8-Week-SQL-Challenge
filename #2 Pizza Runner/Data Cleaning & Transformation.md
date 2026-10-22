# Case Study #2: Pizza Runner
## Data Cleaning & Transformation
Link to [DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/4338) to execute the queries shown below

### customer_orders Table
```sql
CREATE TEMPORARY TABLE customer_orders_temp AS 
    SELECT order_id,
        customer_id,
        pizza_id,
        CASE
            WHEN exclusions IS NULL OR exclusions LIKE 'null' THEN ''
            ELSE exclusions END
            AS exclusions,
        CASE
            WHEN extras IS NULL OR extras LIKE 'null' THEN ''
            ELSE extras END
            AS extras,
        order_time
    FROM customer_orders;
```

### runner_orders Table
```sql
CREATE TEMPORARY TABLE runner_orders_temp AS
    SELECT order_id,
        runner_id,
        CASE
            WHEN pickup_time IS NULL OR pickup_time LIKE 'null' THEN ''
            ELSE pickup_time END
            AS pickup_time,
        CASE
            WHEN distance LIKE 'null' THEN ''
            WHEN distance LIKE '%km' THEN TRIM('km' FROM distance)
            ELSE distance END
            AS distance,
        CASE
            WHEN duration LIKE 'null' THEN ''
            WHEN duration LIKE '%mins' THEN TRIM('mins' FROM duration)
            WHEN duration LIKE '%minutes' THEN TRIM('minutes' FROM duration)
            WHEN duration LIKE '%minute' THEN TRIM('minute' FROM duration)
            ELSE duration END
            AS duration,
        CASE
            WHEN cancellation IS NULL OR cancellation LIKE 'null' THEN ''
            ELSE cancellation END
            AS cancellation
    FROM runner_orders;
```
```sql
ALTER TABLE runner_orders_temp
    ALTER COLUMN pickup_time TYPE TIMESTAMP 
        USING to_timestamp(pickup_time,'YYYY-MM-DD HH24:MI:SS'),
    ALTER COLUMN distance TYPE DOUBLE PRECISION 
        USING (CASE WHEN distance = '' THEN 0.0 ELSE distance::DOUBLE PRECISION END),
    ALTER COLUMN duration TYPE INTEGER 
        USING (CASE WHEN duration = '' THEN 0 ELSE duration::INTEGER END);
```
