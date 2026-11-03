# Case Study #3: Foodie-Fi
## Part D. Outside the Box Questions
Link to [DB Fiddle](https://www.db-fiddle.com/f/rHJhRrXy5hbVBNJ6F6b9gJ/815) to execute the queries shown below

### 1. How would you calculate the rate of growth for Foodie-Fi?
- Monthly revenue 
```sql
WITH monthly_revenue AS (
  SELECT to_char(date_trunc('month', payment_date), 'MM-YYYY') AS month,
  SUM(amount) AS revenue
FROM payments
WHERE EXTRACT(YEAR FROM payment_date) = 2020
GROUP BY date_trunc('month', payment_date)
)
SELECT month,
	to_char(revenue, 'FM999G999D99') AS revenue,
    ROUND(100.0 * (revenue - LAG(revenue) OVER (ORDER BY month)) / LAG(revenue) OVER (ORDER BY month), 2) AS percent_change,
    to_char(SUM(revenue) OVER (ORDER BY month), 'fm999G999D99') AS cumulative_revenue
FROM monthly_revenue;
```
<img width="935" height="437" alt="image" src="https://github.com/user-attachments/assets/c02c98d6-2497-470e-8b19-c5ff154681a1" />

- Quarterly revenue 
```sql
WITH quarterly_revenue AS (
  SELECT to_char(date_trunc('quarter', payment_date), '"Q"Q-YYYY') AS quarter,
  	SUM(amount) AS revenue
FROM payments
WHERE EXTRACT(YEAR FROM payment_date) = 2020
GROUP BY date_trunc('quarter', payment_date)
  )
SELECT quarter,
	to_char(revenue, 'fm999G999D99') AS revenue,
    ROUND(100.0 * (revenue - LAG(revenue) OVER (ORDER BY quarter)) / LAG(revenue) OVER (ORDER BY quarter), 2) AS percent_change,
    to_char(SUM(revenue) OVER (ORDER BY quarter), 'fm999G999D99') AS cumulative_revenue
FROM quarterly_revenue;
```
<img width="935" height="172" alt="image" src="https://github.com/user-attachments/assets/6b4db788-79c0-4e1e-be39-59d896a48d46" />

- January 2020 vs December 2020 revenue comparison
```sql
WITH monthly_revenue AS (
  SELECT to_char(date_trunc('month', payment_date), 'MM-YYYY') AS month,
  SUM(amount) AS revenue
FROM payments
WHERE EXTRACT(YEAR FROM payment_date) = 2020
GROUP BY date_trunc('month', payment_date)
)
SELECT month,
	to_char(revenue, 'FM999G999D99') AS revenue,
    ROUND(100.0 * (revenue - LAG(revenue) OVER (ORDER BY month)) / LAG(revenue) OVER (ORDER BY month), 2) AS percent_change
FROM monthly_revenue
WHERE month IN ('01-2020', '12-2020');
```
<img width="937" height="108" alt="image" src="https://github.com/user-attachments/assets/2317f92c-7d3d-497e-99db-8ef73b366c62" />


### 2. What key metrics would you recommend Foodie-Fi management to track over time to assess performance of their overall business?
- Quarterly new customer recruitment (trials)
```sql
WITH quarterly_new AS (
  SELECT to_char(date_trunc('quarter', payment_date), '"Q"Q-YYYY') AS quarter,
  	COUNT(DISTINCT customer_id) AS new_customers
FROM payments
WHERE EXTRACT(YEAR FROM payment_date) = 2020
  AND plan_id = '0'
GROUP BY date_trunc('quarter', payment_date)
  )
SELECT quarter,
	new_customers,
    ROUND(100.0 * (new_customers - LAG(new_customers) OVER (ORDER BY quarter)) / LAG(new_customers) OVER (ORDER BY quarter), 2) AS percent_change,
    SUM(new_customers) OVER (ORDER BY quarter) AS cumulative_customers
FROM quarterly_new;
```
<img width="935" height="173" alt="image" src="https://github.com/user-attachments/assets/013b37bf-0f61-4932-ba31-a683982bce1a" />

- Quarterly customer retention (exclude trials and churn)
```sql
WITH quarterly_retain AS (
  SELECT to_char(date_trunc('quarter', payment_date), '"Q"Q-YYYY') AS quarter,
  	COUNT(DISTINCT customer_id) AS retained_customers
FROM payments
WHERE EXTRACT(YEAR FROM payment_date) = 2020
  AND plan_id != '0' OR plan_id != '4'
GROUP BY date_trunc('quarter', payment_date)
  )
SELECT quarter,
	retained_customers,
    ROUND(100.0 * (retained_customers - LAG(retained_customers) OVER (ORDER BY quarter)) / LAG(retained_customers) OVER (ORDER BY quarter), 2) AS percent_change
FROM quarterly_retain;
```
<img width="935" height="172" alt="image" src="https://github.com/user-attachments/assets/a9506650-a554-4431-8160-8ad6d5592fc0" />

- Quarterly customer churn
```sql
WITH quarterly_churn AS (
  SELECT to_char(date_trunc('quarter', payment_date), '"Q"Q-YYYY') AS quarter,
  	COUNT(DISTINCT customer_id) AS churn_customers,
  	plan_name
FROM payments
WHERE EXTRACT(YEAR FROM payment_date) = 2020
  AND plan_id = '4'
GROUP BY date_trunc('quarter', payment_date), plan_name
  )
SELECT quarter,
	churn_customers,
    ROUND(100.0 * (churn_customers - LAG(churn_customers) OVER (ORDER BY quarter)) / LAG(churn_customers) OVER (ORDER BY quarter), 2) AS percent_change
FROM quarterly_churn;
```
<img width="932" height="170" alt="image" src="https://github.com/user-attachments/assets/61412a0a-cfb4-4486-a897-a295aec90476" />


### 3. What are some key customer journeys or experiences that you would analyse further to improve customer retention?
- Watch time and platform engagement: If a customer's watch time is high and they consume a variety of shows, they are likely to stay with the Foodie-Fi platform. However, when a customer's watch time drops over time and/or they engage with only a few shows, this signals a loss of interest and a risk of churning in upcoming months or a downgrade in plan tier. Knowing the benchmarks for a healthy vs risky behavior will help us anticipate who may be churning soon as well as gauge the performance of various platform activities. For example, is the newest show attracting new customers and increasing the watch times of existing customers? After a popular show was concluded, did platform usage drop or did customers maintain engagement by moving to a different show on the platform?
- Incentives behind plan upgrades or downgrades: Do discounts and promotional campaigns influence upgrades? If limited/short-term discounts (e.g. first 3 months at lower rates) lead to upgrades, how long do customers stay with the upgraded tier or do they eventually downgrade after the offer ends? Which exclusive features of each plan tier do customer value and justifies an upgrade, such as content/show availability, technical features (e.g. unlimited watch times, download for offline viewing, video quality and speed)? Were there recent changes to these exclusive features that may have driven downgrades?


### 4. If the Foodie-Fi team were to create an exit survey shown to customers who wish to cancel their subscription, what questions would you include in the survey?
- What was the primary reason for cancelling your subscription?
  - Price: need affordable prices, value offered does not justify price
    - What would be the ideal price range for you to continue subscribing to our platform?
    - How has your budget or financial priorities shifted recently?
  - Content: loss of interest or utilization, unsatisfying or uninteresting shows
    - What were your favorite shows on our platform?
    - Was there a specific show that led you to initially subscribe to our platform?
    - What kind of shows are you looking for that we don't have?
  - Service and technical issues: platform was difficult to use, too many technical bugs
    - What technical difficulties have you had with using our platform?
    - What could we improve to make your experience on this platform smoother?
- Are you moving to a competitor? If yes...
  - Which alternative did you choose?
  - Which plan or price tier did you subscribe to?
  - What shows or features interested you?
