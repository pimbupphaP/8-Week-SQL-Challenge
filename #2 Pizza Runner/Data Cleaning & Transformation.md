# Case Study #2: Pizza Runner
## Data Cleaning & Transformation
Link to [DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/4341) to execute the queries shown below

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
<img width="934" height="503" alt="image" src="https://github.com/user-attachments/assets/470542b3-0446-4c48-8d27-0f41f2b25c87" />


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
      USING (CASE WHEN pickup_time = '' THEN NULL ELSE to_timestamp(pickup_time,'YYYY-MM-DD HH24:MI:SS') END),
   ALTER COLUMN distance TYPE NUMERIC
      USING (CASE WHEN distance = '' THEN NULL ELSE distance::NUMERIC END),
   ALTER COLUMN duration TYPE INTEGER 
      USING (CASE WHEN duration = '' THEN NULL ELSE duration::INTEGER END);
```
<img width="934" height="374" alt="image" src="https://github.com/user-attachments/assets/8f5721e2-42f6-4f46-8a32-cd78ad339068" />


