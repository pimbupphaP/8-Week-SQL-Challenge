# Case Study #6. Clique Bait
## Part C. Campaign Analysis
Link to [DB Fiddle](https://www.db-fiddle.com/f/jmnwogTsUE8hGqkZv9H7E8/636) to execute the queries shown below

Generate a table that has 1 single row for every unique visit_id record and has the following columns:
- user_id
- visit_id
- visit_start_time: the earliest event_time for each visit
- page_views: count of page views for each visit
- cart_adds: count of product cart add events for each visit
- purchase: 1/0 flag if a purchase event exists for each visit
- campaign_name: map the visit to a campaign if the visit_start_time falls between the start_date and end_date
- impression: count of ad impressions for each visit
- click: count of ad clicks for each visit
- (Optional column) cart_products: a comma separated text value with products added to the cart sorted by the order they were added to the cart (hint: use the sequence_number)

### `campaign_analysis` Table
```sql
DROP TABLE IF EXISTS campaign_analysis;

CREATE TEMPORARY TABLE campaign_analysis AS
WITH visit_stats AS (
SELECT user_id,
	visit_id,
    page_id,
    event_type,
    sequence_number,
    event_time,
    FIRST_VALUE(event_time) OVER (PARTITION BY visit_id ORDER BY sequence_number) AS visit_start_time,
  	CASE WHEN event_type = 1 THEN 1 ELSE 0 END AS page_views,
    CASE WHEN event_type = 2 THEN 1 ELSE 0 END AS cart_adds,
  	CASE WHEN event_type = 3 THEN 1 ELSE 0 END AS purchase,
    CASE WHEN event_type = 4 THEN 1 ELSE 0 END AS impression,
  	CASE WHEN event_type = 5 THEN 1 ELSE 0 END AS click
FROM clique_bait.events
	JOIN clique_bait.users USING(cookie_id)
GROUP BY visit_id, user_id, page_id, event_type, sequence_number, event_time
), campaign_dates AS (
SELECT visit_id,
    CASE 
    	WHEN event_time BETWEEN '2020-01-01 00:00:00' AND '2020-01-14 00:00:00' THEN 'BOGOF - Fishing For Compliments'
    	WHEN event_time BETWEEN '2020-01-15 00:00:00' AND '2020-01-28 00:00:00' THEN '25% Off - Living The Lux Life'
        WHEN event_time BETWEEN '2020-02-01 00:00:00' AND '2020-03-31 00:00:00' THEN 'Half Off - Treat Your Shellf(ish)'
    ELSE NULL END
    AS campaign_name
FROM clique_bait.events
GROUP BY visit_id, campaign_name
), products AS (
SELECT visit_id,
    STRING_AGG(page_name, ', ' ORDER BY sequence_number ASC) AS cart_products
FROM clique_bait.events
	JOIN clique_bait.page_hierarchy USING(page_id)
WHERE product_id IS NOT NULL
  AND event_type = '2'
GROUP BY visit_id
)
SELECT user_id,
	visit_id,
    visit_start_time,
    SUM(page_views) AS page_views,
    SUM(cart_adds) AS cart_adds,
    SUM(purchase) AS purchase,
    campaign_name,
    SUM(impression) AS impression,
    SUM(click) AS click,
    cart_products
FROM visit_stats
	JOIN campaign_dates USING(visit_id)
    JOIN products USING(visit_id)
GROUP BY user_id, visit_id, visit_start_time, campaign_name, cart_products
ORDER BY user_id, visit_start_time, visit_id;
```
First 20 entries of `campaign_analysis` Table
<img width="1246" height="751" alt="image" src="https://github.com/user-attachments/assets/d0925915-72e7-4fd9-8806-a48a8694f1c3" />
