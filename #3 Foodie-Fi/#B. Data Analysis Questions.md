# Case Study #3: Foodie-Fi
## Part B. Data Analysis Questions
Link to [DB Fiddle](https://www.db-fiddle.com/f/rHJhRrXy5hbVBNJ6F6b9gJ/815) to execute the queries shown below

### 1. How many customers has Foodie-Fi ever had?
```sql
SELECT COUNT(DISTINCT customer_id) AS customer_count
FROM subscriptions;
```
<img width="939" height="70" alt="image" src="https://github.com/user-attachments/assets/a697e002-3447-4b2a-9ff6-4d0002b54dce" />

### 2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value
```sql
SELECT DATE_PART('month', start_date) AS trial_month,
    COUNT(*) AS trial_count
FROM subscriptions
WHERE plan_id = 0
GROUP BY trial_month
ORDER BY trial_month;
```
<img width="935" height="442" alt="image" src="https://github.com/user-attachments/assets/3c7a523e-4653-4506-9ba5-ec0710c7cee1" />

### 3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name
```sql
SELECT plan_name,
    COUNT(*) AS sub_count
FROM subscriptions
    JOIN plans USING(plan_id)
WHERE start_date > '2020-12-31'
GROUP BY plan_name
ORDER BY plan_name;
```
<img width="935" height="177" alt="image" src="https://github.com/user-attachments/assets/ecdadee7-7052-415c-b630-2564a9a9c463" />

### 4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
```sql
WITH churn_count AS (
    SELECT customer_id AS churn_customers
    FROM subscriptions
    WHERE plan_id = 4
  )
SELECT COUNT(DISTINCT subscriptions.customer_id) AS total_customers,
    COUNT(DISTINCT churn_customers) AS churn_customers_count,
    ROUND((COUNT(DISTINCT churn_customers)::NUMERIC / COUNT(DISTINCT subscriptions.customer_id)) * 100, 1) AS churn_percent
FROM subscriptions
    LEFT JOIN churn_count ON subscriptions.customer_id = churn_count.churn_customers;
```
<img width="937" height="78" alt="image" src="https://github.com/user-attachments/assets/1f2efd77-0a6a-480b-9c1c-d675243df8c0" />

### 5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
```sql
WITH trial_churn AS (
    SELECT customer_id,
        CASE 
            WHEN LAG(plan_id) OVER (PARTITION BY customer_id ORDER BY plan_id) = '0' AND plan_id = '4' THEN 1 
            ELSE NULL END
            AS churn_after_trial
    FROM subscriptions
), trial_churn_customers AS(
    SELECT customer_id
    FROM trial_churn
    WHERE churn_after_trial = 1
)
SELECT COUNT(DISTINCT subscriptions.customer_id) AS total_customers,
    COUNT(DISTINCT trial_churn_customers.customer_id) AS trial_churn_customers_count,
    ROUND((COUNT(DISTINCT trial_churn_customers.customer_id)::NUMERIC / COUNT(DISTINCT subscriptions.customer_id)) * 100) AS churn_trial_percent
FROM subscriptions
    LEFT JOIN trial_churn_customers USING(customer_id);
```
<img width="938" height="78" alt="image" src="https://github.com/user-attachments/assets/6dbf29a7-2ea1-4772-a597-a203e88257b0" />

### 6. What is the number and percentage of customer plans after their initial free trial?
```sql
WITH after_trial_plans AS (
    SELECT customer_id,
        plan_id,
        LEAD(plan_name) OVER (PARTITION BY customer_id ORDER BY plan_id) AS after_trial
	FROM subscriptions
		JOIN plans USING(plan_id)
)
SELECT after_trial,
    COUNT(DISTINCT customer_id) AS customer_count,
    ROUND(COUNT(DISTINCT customer_id)::NUMERIC / (SELECT COUNT(DISTINCT customer_id) FROM subscriptions) * 100, 1) AS percentage
FROM after_trial_plans
WHERE after_trial IS NOT NULL 
    AND plan_id = '0'
GROUP BY after_trial;
```
<img width="770" height="175" alt="image" src="https://github.com/user-attachments/assets/dbe9ac69-285a-4917-868b-d4af166ffbdc" />

### 7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?
```sql
WITH next_plans AS (
    SELECT customer_id,
        plan_id,
        start_date,
        LEAD(plan_id) OVER (PARTITION BY customer_id ORDER BY plan_id) AS next_plan,
        LEAD(start_date) OVER (PARTITION BY customer_id ORDER BY start_date) AS next_date
    FROM subscriptions
)
    SELECT plan_name,
        COUNT(customer_id) AS customer_count,
        ROUND(COUNT(customer_id)::NUMERIC / (SELECT COUNT(DISTINCT customer_id) FROM subscriptions) * 100, 1) AS percentage
    FROM next_plans
        JOIN plans USING(plan_id)
    WHERE (next_plan IS NULL AND  start_date <= '2020-12-31')
        OR (next_plan IS NOT NULL AND (start_date <= '2020-12-31' AND next_date > '2020-12-31'))
    GROUP BY plan_name, plan_id
    ORDER BY plan_id;    
```
<img width="938" height="208" alt="image" src="https://github.com/user-attachments/assets/8aa2f5e9-9b4c-41f9-a03a-4e7c6545dc92" />

### 8. How many customers have upgraded to an annual plan in 2020?
```sql
SELECT COUNT(customer_id) AS customer_count
FROM subscriptions
WHERE plan_id = 3
    AND EXTRACT(YEAR FROM start_date) = 2020;
```
<img width="936" height="72" alt="image" src="https://github.com/user-attachments/assets/bf78894b-7250-4a5b-a1ce-d244f1d3e284" />


### 9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?
```sql
WITH plan_start AS (
    SELECT customer_id,
        MIN(start_date) AS trial_start,
        LEAD(start_date) OVER (PARTITION BY customer_id ORDER BY start_date) AS annual_start,
        (LEAD(start_date) OVER (PARTITION BY customer_id ORDER BY start_date)) - MIN(start_date) AS days_dif
    FROM subscriptions
    WHERE (plan_id = '0' OR plan_id = '3')
    GROUP BY customer_id, start_date
)
SELECT ROUND(AVG(days_dif),1) AS avg_conversion_days
FROM plan_start
WHERE trial_start IS NOT NULL
    AND annual_start IS NOT NULL;
```
<img width="935" height="76" alt="image" src="https://github.com/user-attachments/assets/c6ca5211-0ff4-464a-8930-0a23a8eade26" />

### 10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)
```sql
WITH RECURSIVE plan_start AS (
    SELECT customer_id,
        MIN(start_date) AS trial_start,
        LEAD(start_date) OVER (PARTITION BY customer_id ORDER BY start_date) AS annual_start,
        (LEAD(start_date) OVER (PARTITION BY customer_id ORDER BY start_date)) - MIN(start_date) AS days_dif
    FROM subscriptions
    WHERE (plan_id = '0' OR plan_id = '3')
    GROUP BY customer_id, start_date
), days_recursion AS (
    SELECT
        0 AS start_period,
        30 AS end_period
  UNION ALL
    SELECT 
        end_period + 1 AS start_period,
        end_period + 30 AS end_period
    FROM days_recursion
    WHERE end_period < 360
)
SELECT start_period,
    end_period,
    COUNT(customer_id) AS customer_count
FROM days_recursion
    LEFT JOIN plan_start ON (days_dif >= days_recursion.start_period) AND (days_dif <= days_recursion.end_period)
GROUP BY start_period, end_period
ORDER BY start_period;
```
<img width="935" height="439" alt="image" src="https://github.com/user-attachments/assets/b4ff5bde-78f8-452b-b52f-e008f6f5c21a" />

### 11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?
```sql
WITH next_plans AS (
    SELECT customer_id,
        plan_id,
         start_date,
        LEAD(plan_id) OVER (PARTITION BY customer_id ORDER BY plan_id) AS next_plan
	FROM subscriptions
)
SELECT COUNT(customer_id) AS customer_count
FROM next_plans
WHERE plan_id = 2 
    AND plan_id = 1
    AND EXTRACT(YEAR FROM start_date) = 2020;
```
<img width="935" height="72" alt="image" src="https://github.com/user-attachments/assets/a6e20044-8481-4983-b2ae-abe083f81e7d" />
