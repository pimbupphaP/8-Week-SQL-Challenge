# Case Study #3: Foodie-Fi
## Part B. Data Analysis Questions
Link to [DB Fiddle](https://www.db-fiddle.com/f/rHJhRrXy5hbVBNJ6F6b9gJ/805) to execute the queries shown below

1. How many customers has Foodie-Fi ever had?
```sql
SELECT COUNT(DISTINCT customer_id) AS customer_count
FROM subscriptions;
```


2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value
```sql

```


3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name
```sql

```


4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
```sql

```


5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
```sql

```


6. What is the number and percentage of customer plans after their initial free trial?
```sql

```



7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?
```sql

```


8. How many customers have upgraded to an annual plan in 2020?
```sql

```


9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?
```sql

```


10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)
```sql

```


11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?
```sql

```
