# Case Study #5. Data Bank
## Part B. Data Exploration
Link to [DB Fiddle](https://www.db-fiddle.com/f/jmnwogTsUE8hGqkZv9H7E8/622) to execute the queries shown below

### 1. What day of the week is used for each week_date value?
```sql
SELECT DISTINCT TO_CHAR(week_date, 'Day') AS week_day
FROM clean_weekly_sales;
```
<img width="933" height="71" alt="image" src="https://github.com/user-attachments/assets/7027bc98-df20-4af8-9ee6-43bc10cac96d" />

### 2. What range of week numbers are missing from the dataset?
```sql
WITH RECURSIVE all_weeks AS (
    SELECT 1 AS week
    UNION ALL
    SELECT week + 1 
    FROM all_weeks
    WHERE week + 1 <= 52
)
SELECT DISTINCT week AS week,
    week_number
FROM all_weeks
    LEFT JOIN clean_weekly_sales ON all_weeks.week = clean_weekly_sales.week_number
WHERE week_number IS NULL
ORDER BY week;
```
<img width="436" height="776" alt="image" src="https://github.com/user-attachments/assets/1fcc9cf1-5464-4cb7-bfc5-2d5a8af2c7e5" />

### 3. How many total transactions were there for each year in the dataset?
```sql
SELECT calendar_year, 
    TO_CHAR(SUM(transactions), 'fm999,999,999') AS total_transactions
FROM clean_weekly_sales
GROUP BY calendar_year
ORDER BY calendar_year;
```
<img width="934" height="135" alt="image" src="https://github.com/user-attachments/assets/8ceec283-a64c-4f38-b85b-6ec0abd19302" />

### 4. What is the total sales for each region for each month?
```sql
SELECT region,
    month_number,
    TO_CHAR(SUM(sales), 'fm999,999,999,999') AS total_sales
FROM clean_weekly_sales
GROUP BY region, month_number
ORDER BY region, month_number;
```
Limiting results to first 3 regions
<img width="932" height="734" alt="image" src="https://github.com/user-attachments/assets/7c1d7bf5-5a40-4952-981c-c07cc2f49a91" />

### 5. What is the total count of transactions for each platform
```sql
SELECT platform,
    COUNT(transactions) AS count_transactions
FROM clean_weekly_sales
GROUP BY platform;
```
<img width="936" height="106" alt="image" src="https://github.com/user-attachments/assets/fc4adc80-394e-428b-81c1-d70764bff7bb" />

### 6. What is the percentage of sales for Retail vs Shopify for each month?
```sql
WITH retail_sales AS (
    SELECT month_number, 
        SUM(sales) AS retail_sales
    FROM clean_weekly_sales
    WHERE platform = 'Retail'
    GROUP BY month_number
), shopify_sales AS (
    SELECT month_number, 
        SUM(sales) AS shopify_sales
    FROM clean_weekly_sales
    WHERE platform = 'Shopify'
    GROUP BY month_number
)
SELECT month_number,
    TO_CHAR(retail_sales, 'fm999,999,999,999') AS retail_sales,
    TO_CHAR(shopify_sales, 'fm999,999,999,999') AS shopify_sales,
    TO_CHAR((retail_sales + shopify_sales), 'fm999,999,999,999') AS total_sales,
    ROUND(retail_sales::NUMERIC/(retail_sales + shopify_sales)*100.00, 1) AS retail_percent,
    ROUND(shopify_sales::NUMERIC/(retail_sales + shopify_sales)*100.00, 1) AS shopify_percent
FROM retail_sales
    JOIN shopify_sales USING(month_number)
GROUP BY month_number, retail_sales, shopify_sales;
```
<img width="939" height="269" alt="image" src="https://github.com/user-attachments/assets/7e7fd6b0-1908-41ca-8446-37915d882a8f" />

### 7. What is the percentage of sales by demographic for each year in the dataset?
```sql
WITH couples AS (
    SELECT calendar_year,
        SUM(sales) AS couples_sales
    FROM clean_weekly_sales
    WHERE demographic = 'Couples'
    GROUP BY calendar_year
), families AS (
    SELECT calendar_year,
        SUM(sales) AS families_sales
    FROM clean_weekly_sales
    WHERE demographic = 'Families'
    GROUP BY calendar_year
), unknown AS (
    SELECT calendar_year,
        SUM(sales) AS unknown_sales
    FROM clean_weekly_sales
    WHERE demographic = 'Unknown'
    GROUP BY calendar_year
)
SELECT calendar_year,
    TO_CHAR(couples_sales, 'fm999,999,999,999') AS couples_sales,
    TO_CHAR(families_sales, 'fm999,999,999,999') AS families_sales,
    TO_CHAR(unknown_sales, 'fm999,999,999,999') AS unknown_sales,
    TO_CHAR((couples_sales + families_sales + unknown_sales), 'fm999,999,999,999') AS total_sales,
    ROUND(couples_sales::NUMERIC/(couples_sales + families_sales + unknown_sales)*100.00, 1) AS couples_percent,
    ROUND(families_sales::NUMERIC/(couples_sales + families_sales + unknown_sales)*100.00, 1) AS families_percent,
    ROUND(unknown_sales::NUMERIC/(couples_sales + families_sales + unknown_sales)*100.00, 1) AS unknown_percent
FROM couples
    JOIN families USING(calendar_year)
    JOIN unknown USING(calendar_year)
GROUP BY calendar_year, couples_sales, families_sales, unknown_sales
ORDER BY calendar_year;
```
<img width="930" height="137" alt="image" src="https://github.com/user-attachments/assets/e9325854-af70-4a4c-9020-d1b4d3c71e48" />

### 8. Which age_band and demographic values contribute the most to Retail sales?
```sql
WITH total AS (
    SELECT SUM(sales) AS total_sales
    FROM clean_weekly_sales
    WHERE platform = 'Retail'
)
SELECT age_band,
    demographic,
    TO_CHAR(SUM(sales), 'fm999,999,999,999') AS segment_sales,
    ROUND(SUM(sales)::NUMERIC/(SELECT total_sales FROM total)*100.00, 1) AS contribution
FROM clean_weekly_sales
WHERE platform = 'Retail'
GROUP BY age_band, demographic
ORDER BY contribution DESC;
```
<img width="935" height="270" alt="image" src="https://github.com/user-attachments/assets/e5a6d074-2c13-4466-8fee-fb30f5748b73" />

### 9. Can we use the avg_transaction column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?
```sql
SELECT calendar_year,
    platform,
    ROUND(SUM(sales)/SUM(transactions)) AS avg_transactions
FROM clean_weekly_sales
GROUP BY calendar_year, platform
ORDER BY calendar_year;
```
<img width="934" height="235" alt="image" src="https://github.com/user-attachments/assets/1fa5c744-d333-404a-80bf-4012f5954d32" />

