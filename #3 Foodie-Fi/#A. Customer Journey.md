# Case Study #3: Foodie-Fi
## Part A. Customer Journey
Link to [DB Fiddle](https://www.db-fiddle.com/f/rHJhRrXy5hbVBNJ6F6b9gJ/815) to execute the queries shown below

Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.
```sql
SELECT customer_id, 
    plan_id,
    plan_name,
    price,
    start_date,
    AGE(start_date, LAG(start_date) OVER (PARTITION BY customer_id ORDER BY start_date)) AS date_dif
FROM plans 
    JOIN subscriptions USING(plan_id)
WHERE customer_id IN (1, 2, 11, 13, 15, 16, 18, 19)
ORDER BY customer_id, start_date;
```
<img width="935" height="703" alt="image" src="https://github.com/user-attachments/assets/6960c860-2720-4527-b5b5-3fb0af9af78b" />
