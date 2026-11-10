# Case Study #5: Data Mart
## Part A. Data Cleansing Steps
Link to [DB Fiddle](https://www.db-fiddle.com/f/jmnwogTsUE8hGqkZv9H7E8/622) to execute the queries shown below

In a single query, perform the following operations and generate a new table in the `data_mart` schema named `clean_weekly_sales`:
- Convert the `week_date` to a `DATE` format
- Add a `week_number` as the second column for each `week_date` value, for example any value from the 1st of January to 7th of January will be 1, 8th to 14th will be 2 etc
- Add a `month_number` with the calendar month for each `week_date` value as the 3rd column
- Add a `calendar_year` column as the 4th column containing either 2018, 2019 or 2020 values
- Add a new column called `age_band` after the original `segment` column using the following mapping on the number inside the `segment` value
  - 1 Young Adults
  - 2 Middle Aged
  - 3 or 4 Retirees
- Add a new `demographic` column using the following mapping for the first letter in the `segment` values:
  - C Couples
  - F Families
- Ensure all null string values with an "unknown" string value in the original `segment` column as well as the new `age_band` and `demographic` columns
- Generate a new `avg_transaction` column as the `sales` value divided by `transactions` rounded to 2 decimal places for each record

```sql
CREATE TEMPORARY TABLE clean_weekly_sales AS
    SELECT TO_DATE(week_date, 'DD/MM/YY') AS week_date,
        EXTRACT(WEEK FROM TO_DATE(week_date, 'DD/MM/YY')) AS week_number,
        EXTRACT(MONTH FROM TO_DATE(week_date, 'DD/MM/YY')) AS month_number,
        EXTRACT(YEAR FROM TO_DATE(week_date, 'DD/MM/YY')) AS calendar_year,
        region,
        platform,
        CASE
            WHEN segment != 'null' THEN segment
            ELSE 'Unknown' END
            AS segment,
        CASE
            WHEN SUBSTRING(segment, 2) = '1' THEN 'Young Adults'
            WHEN SUBSTRING(segment, 2) = '2' THEN 'Middle Aged'
            WHEN SUBSTRING(segment, 2) IN ('3', '4') THEN 'Retirees'
            ELSE 'Unknown' END
            AS age_band,
        CASE
            WHEN SUBSTRING(segment, 1, 1) = 'C' THEN 'Couples'
            WHEN SUBSTRING(segment, 1, 1) = 'F' THEN 'Families'
            ELSE 'Unknown' END
            AS demographic,
        customer_type,
        transactions, 
        sales,
        ROUND(sales::NUMERIC/transactions, 2) AS avg_transaction
    FROM weekly_sales;
```

| week_date  | week_number | month_number | calendar_year | region        | platform | segment | age_band     | demographic | customer_type | transactions | sales    | avg_transaction |
| ---------- | ----------- | ------------ | ------------- | ------------- | -------- | ------- | ------------ | ----------- | ------------- | ------------ | -------- | --------------- |
| 2020-08-31 | 36          | 8            | 2020          | ASIA          | Retail   | C3      | Retirees     | Couples     | New           | 120631       | 3656163  | 30.31           |
| 2020-08-31 | 36          | 8            | 2020          | ASIA          | Retail   | F1      | Young Adults | Families    | New           | 31574        | 996575   | 31.56           |
| 2020-08-31 | 36          | 8            | 2020          | USA           | Retail   | Unknown | Unknown      | Unknown     | Guest         | 529151       | 16509610 | 31.20           |
| 2020-08-31 | 36          | 8            | 2020          | EUROPE        | Retail   | C1      | Young Adults | Couples     | New           | 4517         | 141942   | 31.42           |
| 2020-08-31 | 36          | 8            | 2020          | AFRICA        | Retail   | C2      | Middle Aged  | Couples     | New           | 58046        | 1758388  | 30.29           |
| 2020-08-31 | 36          | 8            | 2020          | CANADA        | Shopify  | F2      | Middle Aged  | Families    | Existing      | 1336         | 243878   | 182.54          |
| 2020-08-31 | 36          | 8            | 2020          | AFRICA        | Shopify  | F3      | Retirees     | Families    | Existing      | 2514         | 519502   | 206.64          |
| 2020-08-31 | 36          | 8            | 2020          | ASIA          | Shopify  | F1      | Young Adults | Families    | Existing      | 2158         | 371417   | 172.11          |
| 2020-08-31 | 36          | 8            | 2020          | AFRICA        | Shopify  | F2      | Middle Aged  | Families    | New           | 318          | 49557    | 155.84          |
| 2020-08-31 | 36          | 8            | 2020          | AFRICA        | Retail   | C3      | Retirees     | Couples     | New           | 111032       | 3888162  | 35.02           |
| 2020-08-31 | 36          | 8            | 2020          | USA           | Shopify  | F1      | Young Adults | Families    | Existing      | 1398         | 260773   | 186.53          |
| 2020-08-31 | 36          | 8            | 2020          | OCEANIA       | Shopify  | C2      | Middle Aged  | Couples     | Existing      | 4661         | 882690   | 189.38          |
| 2020-08-31 | 36          | 8            | 2020          | SOUTH AMERICA | Retail   | C2      | Middle Aged  | Couples     | Existing      | 1029         | 38762    | 37.67           |
| 2020-08-31 | 36          | 8            | 2020          | SOUTH AMERICA | Shopify  | C4      | Retirees     | Couples     | New           | 6            | 917      | 152.83          |
| 2020-08-31 | 36          | 8            | 2020          | EUROPE        | Shopify  | F3      | Retirees     | Families    | Existing      | 115          | 35215    | 306.22          |
| 2020-08-31 | 36          | 8            | 2020          | OCEANIA       | Retail   | F3      | Retirees     | Families    | Existing      | 551905       | 30371770 | 55.03           |
| 2020-08-31 | 36          | 8            | 2020          | ASIA          | Shopify  | C3      | Retirees     | Couples     | Existing      | 1969         | 374327   | 190.11          |
| 2020-08-31 | 36          | 8            | 2020          | AFRICA        | Retail   | F1      | Young Adults | Families    | Existing      | 97604        | 5185233  | 53.13           |
| 2020-08-31 | 36          | 8            | 2020          | OCEANIA       | Retail   | C2      | Middle Aged  | Couples     | New           | 111219       | 2980673  | 26.80           |
| 2020-08-31 | 36          | 8            | 2020          | USA           | Retail   | F1      | Young Adults | Families    | New           | 11820        | 463738   | 39.23           |



