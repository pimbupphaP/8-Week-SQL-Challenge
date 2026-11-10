# Case Study #5. Data Bank
## Part C. Before & After Analysis
Link to [DB Fiddle](https://www.db-fiddle.com/f/jmnwogTsUE8hGqkZv9H7E8/621) to execute the queries shown below

Taking the `week_date` value of `2020-06-15` as the baseline week where the Data Mart sustainable packaging changes came into effect.

We would include all `week_date` values for `2020-06-15` as the start of the period after the change and the previous `week_date` values would be before

### 1. What is the total sales for the 4 weeks before and after 2020-06-15? What is the growth or reduction rate in actual values and percentage of sales?
```sql
WITH before_after AS (
  SELECT SUM(CASE WHEN week_number BETWEEN '21' AND '24' THEN sales END) AS before_sales,
  		SUM(CASE WHEN week_number BETWEEN '25' AND '28' THEN sales END) AS after_sales
  FROM clean_weekly_sales
  WHERE calendar_year = '2020'
)
SELECT TO_CHAR(before_sales, 'fm999,999,999,999') AS before_sales,
	TO_CHAR(after_sales, 'fm999,999,999,999') AS after_sales,
    TO_CHAR(after_sales - before_sales, 'fm999,999,999,999') AS actual_dif,
    ROUND((100.00*(after_sales - before_sales) / before_sales), 2) AS pct_change
FROM before_after;
```
<img width="1244" height="95" alt="image" src="https://github.com/user-attachments/assets/53ec0b89-7430-4042-a935-d66e7fe62840" />

### 2. What about the entire 12 weeks before and after?
```sql
WITH before_after AS (
  SELECT SUM(CASE WHEN week_number BETWEEN '13' AND '24' THEN sales END) AS before_sales,
  		SUM(CASE WHEN week_number BETWEEN '25' AND '36' THEN sales END) AS after_sales
  FROM clean_weekly_sales
  WHERE calendar_year = '2020'
)
SELECT TO_CHAR(before_sales, 'fm999,999,999,999') AS before_sales,
	TO_CHAR(after_sales, 'fm999,999,999,999') AS after_sales,
    TO_CHAR(after_sales - before_sales, 'fm999,999,999,999') AS actual_dif,
    ROUND((100.00*(after_sales - before_sales) / before_sales), 2) AS pct_change
FROM before_after;
```
<img width="1243" height="95" alt="image" src="https://github.com/user-attachments/assets/2f0063a3-f7f5-451c-b57f-c41b704cbc94" />

### 3. How do the sale metrics for these 2 periods before and after compare with the previous years in 2018 and 2019?
```sql
WITH before_after AS (
  SELECT calendar_year, 
  		SUM(CASE WHEN week_number BETWEEN '21' AND '24' THEN sales END) AS before_sales,
  		SUM(CASE WHEN week_number BETWEEN '25' AND '28' THEN sales END) AS after_sales
  FROM clean_weekly_sales
  GROUP BY calendar_year
)
SELECT calendar_year, 
	TO_CHAR(before_sales, 'fm999,999,999,999') AS before_sales,
	TO_CHAR(after_sales, 'fm999,999,999,999') AS after_sales,
    TO_CHAR(after_sales - before_sales, 'fm999,999,999,999') AS actual_dif,
    ROUND((100.00*(after_sales - before_sales) / before_sales), 2) AS pct_change
FROM before_after
ORDER BY calendar_year;
```
<img width="1248" height="182" alt="image" src="https://github.com/user-attachments/assets/cee120cf-f28a-4d31-9f51-dbe50d93718d" />

```sql
WITH before_after AS (
  SELECT calendar_year, 
  		SUM(CASE WHEN week_number BETWEEN '13' AND '24' THEN sales END) AS before_sales,
  		SUM(CASE WHEN week_number BETWEEN '25' AND '36' THEN sales END) AS after_sales
  FROM clean_weekly_sales
  GROUP BY calendar_year
)
SELECT calendar_year, 
	TO_CHAR(before_sales, 'fm999,999,999,999') AS before_sales,
	TO_CHAR(after_sales, 'fm999,999,999,999') AS after_sales,
    TO_CHAR(after_sales - before_sales, 'fm999,999,999,999') AS actual_dif,
    ROUND((100.00*(after_sales - before_sales) / before_sales), 2) AS pct_change
FROM before_after
ORDER BY calendar_year;
```
<img width="1241" height="183" alt="image" src="https://github.com/user-attachments/assets/9d7ac4f6-f730-4cfa-b2f8-dab6b3318621" />
