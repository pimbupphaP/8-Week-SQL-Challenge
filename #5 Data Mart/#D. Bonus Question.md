# Case Study #5. Data Mart
## Part D. Bonus Question
Link to [DB Fiddle](https://www.db-fiddle.com/f/jmnwogTsUE8hGqkZv9H7E8/622) to execute the queries shown below

Which areas of the business have the highest negative impact in sales metrics performance in 2020 for the 12 week before and after period?
- `region`
- `platform`
- `age_band`
- `demographic`
- `customer_type`

### Region
```sql
WITH before_after AS (
    SELECT region, 
        SUM(CASE WHEN week_number BETWEEN '13' AND '24' THEN sales END) AS before_sales,
        SUM(CASE WHEN week_number BETWEEN '25' AND '36' THEN sales END) AS after_sales
    FROM clean_weekly_sales
    WHERE calendar_year = '2020'
    GROUP BY region
)
SELECT region,
    TO_CHAR(before_sales, 'fm999,999,999,999') AS before_sales,
    TO_CHAR(after_sales, 'fm999,999,999,999') AS after_sales,
    TO_CHAR(after_sales - before_sales, 'fm999,999,999,999') AS actual_dif,
    ROUND((100.00*(after_sales - before_sales) / before_sales), 2) AS pct_change
FROM before_after
ORDER BY pct_change ASC;
```
<img width="1250" height="364" alt="image" src="https://github.com/user-attachments/assets/9966c972-8612-4629-bea5-e339d443faaf" />

### Platform
```sql
WITH before_after AS (
    SELECT platform, 
        SUM(CASE WHEN week_number BETWEEN '13' AND '24' THEN sales END) AS before_sales,
        SUM(CASE WHEN week_number BETWEEN '25' AND '36' THEN sales END) AS after_sales
    FROM clean_weekly_sales
    WHERE calendar_year = '2020'
    GROUP BY platform
)
SELECT platform,
    TO_CHAR(before_sales, 'fm999,999,999,999') AS before_sales,
    TO_CHAR(after_sales, 'fm999,999,999,999') AS after_sales,
    TO_CHAR(after_sales - before_sales, 'fm999,999,999,999') AS actual_dif,
    ROUND((100.00*(after_sales - before_sales) / before_sales), 2) AS pct_change
FROM before_after
ORDER BY pct_change ASC;
```
<img width="1245" height="143" alt="image" src="https://github.com/user-attachments/assets/a31596c3-8e27-453d-938c-51cd42048a8f" />

### Age Band
```sql
WITH before_after AS (
    SELECT age_band, 
        SUM(CASE WHEN week_number BETWEEN '13' AND '24' THEN sales END) AS before_sales,
        SUM(CASE WHEN week_number BETWEEN '25' AND '36' THEN sales END) AS after_sales
    FROM clean_weekly_sales
    WHERE calendar_year = '2020'
    GROUP BY age_band
)
SELECT age_band,
    TO_CHAR(before_sales, 'fm999,999,999,999') AS before_sales,
    TO_CHAR(after_sales, 'fm999,999,999,999') AS after_sales,
    TO_CHAR(after_sales - before_sales, 'fm999,999,999,999') AS actual_dif,
    ROUND((100.00*(after_sales - before_sales) / before_sales), 2) AS pct_change
FROM before_after
ORDER BY pct_change ASC;
```
<img width="1243" height="228" alt="image" src="https://github.com/user-attachments/assets/5374a3cc-2784-43c0-8927-1b22d7c14e88" />

### Demographic
```sql
WITH before_after AS (
    SELECT demographic, 
        SUM(CASE WHEN week_number BETWEEN '13' AND '24' THEN sales END) AS before_sales,
        SUM(CASE WHEN week_number BETWEEN '25' AND '36' THEN sales END) AS after_sales
    FROM clean_weekly_sales
    WHERE calendar_year = '2020'
    GROUP BY demographic
)
SELECT demographic,
    TO_CHAR(before_sales, 'fm999,999,999,999') AS before_sales,
    TO_CHAR(after_sales, 'fm999,999,999,999') AS after_sales,
    TO_CHAR(after_sales - before_sales, 'fm999,999,999,999') AS actual_dif,
    ROUND((100.00*(after_sales - before_sales) / before_sales), 2) AS pct_change
FROM before_after
ORDER BY pct_change ASC;
```
<img width="1244" height="185" alt="image" src="https://github.com/user-attachments/assets/06032e42-8b90-4a75-94b0-8d0b910c8551" />

### Customer Type
```sql
WITH before_after AS (
    SELECT customer_type, 
        SUM(CASE WHEN week_number BETWEEN '13' AND '24' THEN sales END) AS before_sales,
        SUM(CASE WHEN week_number BETWEEN '25' AND '36' THEN sales END) AS after_sales
    FROM clean_weekly_sales
    WHERE calendar_year = '2020'
    GROUP BY customer_type
)
SELECT customer_type,
    TO_CHAR(before_sales, 'fm999,999,999,999') AS before_sales,
    TO_CHAR(after_sales, 'fm999,999,999,999') AS after_sales,
    TO_CHAR(after_sales - before_sales, 'fm999,999,999,999') AS actual_dif,
    ROUND((100.00*(after_sales - before_sales) / before_sales), 2) AS pct_change
FROM before_after
ORDER BY pct_change ASC;
```
<img width="1246" height="187" alt="image" src="https://github.com/user-attachments/assets/9cffd2b4-a483-4541-95c7-54f1b5ea8e4d" />

### All areas of business (ordered by top 10 percentage decline)
```sql
WITH before_after AS (
    SELECT region,
        platform,
        age_band,
        demographic,
        customer_type,
        SUM(CASE WHEN week_number BETWEEN '13' AND '24' THEN sales END) AS before_sales,
        SUM(CASE WHEN week_number BETWEEN '25' AND '36' THEN sales END) AS after_sales
    FROM clean_weekly_sales
    WHERE calendar_year = '2020'
    GROUP BY region, platform, age_band, demographic, customer_type
)
SELECT region,
    platform,
    age_band,
    demographic,
    customer_type,
    TO_CHAR(before_sales, 'fm999,999,999,999') AS before_sales,
    TO_CHAR(after_sales, 'fm999,999,999,999') AS after_sales,
    TO_CHAR(after_sales - before_sales, 'fm999,999,999,999') AS actual_dif,
    ROUND((100.00*(after_sales - before_sales) / before_sales), 2) AS pct_change
FROM before_after
ORDER BY pct_change ASC
LIMIT 10;
```
<img width="1240" height="496" alt="image" src="https://github.com/user-attachments/assets/480d142b-d7b3-4e34-835a-1f2e4dd2f2e0" />
