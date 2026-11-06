# Case Study #3: Foodie-Fi
## Part B. Data Analysis Questions
Link to [DB Fiddle](https://www.db-fiddle.com/f/2GtQz4wZtuNNu7zXH5HtV4/696) to execute the queries shown below

### 1. How many unique nodes are there on the Data Bank system?
```sql
SELECT COUNT(DISTINCT node_id) AS node_count
FROM customer_nodes;
```
<img width="933" height="71" alt="image" src="https://github.com/user-attachments/assets/fe7564fa-cc7c-4db1-8e3b-4c3c43805820" />

### 2. What is the number of nodes per region?
```sql
SELECT region_name,
    COUNT(node_id) AS node_count
FROM customer_nodes
    JOIN regions USING(region_id)
GROUP BY region_id, region_name
ORDER BY region_id;
```
<img width="932" height="203" alt="image" src="https://github.com/user-attachments/assets/0a45fc29-21ad-468d-9caa-90fb41f80494" />

### 3. How many customers are allocated to each region?
```sql
SELECT region_name,
    COUNT(DISTINCT customer_id) AS customer_count
FROM customer_nodes
    JOIN regions USING(region_id)
GROUP BY region_id, region_name
ORDER BY region_id;
```
<img width="940" height="206" alt="image" src="https://github.com/user-attachments/assets/acc48223-4123-4b27-b86a-dba30c6a13cf" />

### 4. How many days on average are customers reallocated to a different node?
```sql
WITH first_dates AS (
    SELECT customer_id,
        region_id,
        node_id,
        MIN(start_date) AS first_date
    FROM customer_nodes
    GROUP BY customer_id, region_id, node_id
), reallocation AS (
    SELECT customer_id,
        region_id,
        node_id,
        first_date,
        LEAD(first_date) OVER (PARTITION BY customer_id ORDER BY first_date) - first_date AS reallocation_days
    FROM first_dates
)
SELECT ROUND(AVG(reallocation_days)::NUMERIC,2) AS avg_reallocation_days
FROM reallocation;
```
<img width="936" height="73" alt="image" src="https://github.com/user-attachments/assets/9e5b42c5-5e31-41b4-9baf-daa45abcf1d9" />

### 5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?
```sql
WITH first_dates AS (
    SELECT customer_id,
        region_id,
        node_id,
        MIN(start_date) AS first_date
    FROM customer_nodes
    GROUP BY customer_id, region_id, node_id
), reallocation AS (
    SELECT customer_id,
        region_id,
        node_id,
        first_date,
        LEAD(first_date) OVER (PARTITION BY customer_id ORDER BY first_date) - first_date AS reallocation_days
    FROM first_dates
)
SELECT region_name,
    ROUND(PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY reallocation_days)::NUMERIC, 1) AS median,
    ROUND(PERCENTILE_CONT(0.8) WITHIN GROUP (ORDER BY reallocation_days)::NUMERIC, 1) AS percentile_80,
    ROUND(PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY reallocation_days)::NUMERIC, 1) AS percentile_95
FROM reallocation
    JOIN regions USING(region_id)
GROUP BY region_id, region_name
ORDER BY region_id;
```
<img width="932" height="201" alt="image" src="https://github.com/user-attachments/assets/95799864-3f79-42e4-a235-dd72d49c8195" />
