# Case Study #4: Data Bank
## Part B. Customer Transactions
Link to [DB Fiddle](https://www.db-fiddle.com/f/2GtQz4wZtuNNu7zXH5HtV4/698) to execute the queries shown below

### 1. What is the unique count and total amount for each transaction type?
```sql
SELECT txn_type,
    COUNT(txn_type) AS txn_count,
    SUM(txn_amount) AS total_amount
FROM customer_transactions
GROUP BY txn_type;
```
<img width="934" height="140" alt="image" src="https://github.com/user-attachments/assets/abf196a3-d493-465d-a50c-c9a87aa317be" />

### 2. What is the average total historical deposit counts and amounts for all customers?
```sql
WITH deposits AS (
    SELECT customer_id, 
        COUNT(txn_type) AS deposit_count,
        SUM(txn_amount) AS deposit_amount
    FROM customer_transactions
    WHERE txn_type = 'deposit'
    GROUP BY customer_id
  )
SELECT ROUND(AVG(deposit_count), 2) AS avg_count,
    ROUND(AVG(deposit_amount), 2) as avg_amount
FROM deposits;
```
<img width="930" height="72" alt="image" src="https://github.com/user-attachments/assets/5416c4a1-c32e-4991-882f-7958b059005b" />

### 3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?
```sql
WITH txn_count AS (
    SELECT customer_id,
        date_trunc('month', txn_date) AS month,
        SUM(CASE WHEN txn_type = 'deposit' THEN 1 ELSE 0 END) AS deposit_count,
        SUM(CASE WHEN txn_type = 'purchase' THEN 1 ELSE 0 END) AS purchase_count,
        SUM(CASE WHEN txn_type = 'withdrawal' THEN 1 ELSE 0 END) AS withdrawal_count
    FROM customer_transactions
    GROUP BY customer_id, date_trunc('month', txn_date)
  )
SELECT to_char(month, 'Month') AS month,
    COUNT(customer_id) AS customer_count
FROM txn_count
WHERE deposit_count > 1
    AND (purchase_count = 1 OR withdrawal_count = 1)
GROUP BY month;
```
<img width="930" height="170" alt="image" src="https://github.com/user-attachments/assets/a80962c1-7f01-4b03-b159-09cf0b1b5f03" />

### 4. What is the closing balance for each customer at the end of the month?
```sql
WITH RECURSIVE monthly_recur AS (
    SELECT DATE_TRUNC('month', MIN(txn_date))::DATE AS month_start
    FROM customer_transactions
    UNION ALL
    SELECT (month_start + INTERVAL '1 month')::DATE 
    FROM monthly_recur
    WHERE month_start < (
        SELECT DATE_TRUNC('month', MAX(txn_date))::DATE 
        FROM customer_transactions)
), monthly_balance AS (
    SELECT customer_id,
        DATE_TRUNC('month', txn_date)::DATE AS month_start,
        SUM(CASE
            WHEN txn_type IN ('withdrawal', 'purchase') THEN -txn_amount
            WHEN txn_type = 'deposit' THEN txn_amount
            ELSE 0 END)
            AS net_txn
    FROM customer_transactions
    GROUP BY customer_id, DATE_TRUNC('month', txn_date)
), customer_months AS (
    SELECT DISTINCT customer_id,
        month_start,
        (month_start + INTERVAL '1 month - 1 day')::DATE AS month_end
    FROM customer_transactions
        CROSS JOIN monthly_recur
), closing_balance AS (
    SELECT customer_months.customer_id,
        month_end,
        COALESCE(net_txn, 0) AS monthly_net_txn
    FROM customer_months
        LEFT JOIN monthly_balance 
            ON customer_months.customer_id = monthly_balance.customer_id
            AND customer_months.month_start = monthly_balance.month_start
) 
SELECT customer_id,
    month_end,
    monthly_net_txn,
    SUM(monthly_net_txn) OVER (PARTITION BY customer_id ORDER BY month_end ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS balance
FROM closing_balance
ORDER BY customer_id, month_end LIMIT 50;
```
<img width="934" height="701" alt="image" src="https://github.com/user-attachments/assets/522b3d6a-d0b6-4e8b-9bc8-ff3ca5d5846c" />

### 5. What is the percentage of customers who increase their closing balance by more than 5%?
```sql
WITH RECURSIVE monthly_recur AS (
    SELECT DATE_TRUNC('month', MIN(txn_date))::DATE AS month_start
    FROM customer_transactions
    UNION ALL
    SELECT (month_start + INTERVAL '1 month')::DATE 
    FROM monthly_recur
    WHERE month_start < (
        SELECT DATE_TRUNC('month', MAX(txn_date))::DATE 
        FROM customer_transactions)
), monthly_txn AS (
    SELECT customer_id,
        DATE_TRUNC('month', txn_date)::DATE AS month_start,
        SUM(CASE
            WHEN txn_type IN ('withdrawal', 'purchase') THEN -txn_amount
            WHEN txn_type = 'deposit' THEN txn_amount
            ELSE 0 END)
            AS net_txn
    FROM customer_transactions
    GROUP BY customer_id, DATE_TRUNC('month', txn_date)
), customer_months AS (
    SELECT DISTINCT customer_id,
        month_start,
        (month_start + INTERVAL '1 month - 1 day')::DATE AS month_end
    FROM customer_transactions
        CROSS JOIN monthly_recur
), closing_balance AS (
    SELECT customer_months.customer_id,
        month_end,
        COALESCE(net_txn, 0) AS monthly_net_txn
    FROM customer_months
        LEFT JOIN monthly_txn 
            ON customer_months.customer_id = monthly_txn.customer_id
            AND customer_months.month_start = monthly_txn.month_start
), monthly_balance AS (
    SELECT customer_id,
        month_end,
        monthly_net_txn,
        SUM(monthly_net_txn) OVER (PARTITION BY customer_id ORDER BY month_end ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS balance
    FROM closing_balance
), jan_apr_compare AS (
SELECT customer_id,
    month_end,
    CASE 
        WHEN LAG(balance) OVER (PARTITION BY customer_id ORDER BY month_end) > 0 AND balance > LAG(balance) OVER (PARTITION BY customer_id ORDER BY month_end) * 1.05 THEN 1
        ELSE 0 END
        AS compare_balance
    FROM monthly_balance
    WHERE month_end IN ('2020-01-31', '2020-04-30')
    GROUP BY customer_id, month_end, balance
)
SELECT SUM(CASE 
        WHEN compare_balance = 1 THEN 1
        ELSE 0 END)
        AS balance_growth,
    COUNT(DISTINCT customer_id) AS total_customers,
    ROUND(SUM(CASE WHEN compare_balance = 1 THEN 1 ELSE 0 END) / COUNT(DISTINCT customer_id)::NUMERIC * 100.0, 1) AS percentage
FROM jan_apr_compare;
```
<img width="929" height="69" alt="image" src="https://github.com/user-attachments/assets/ee51001c-bbc7-4565-af0b-455f297dc93c" />

