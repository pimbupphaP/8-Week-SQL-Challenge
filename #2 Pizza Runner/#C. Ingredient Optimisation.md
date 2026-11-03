# Case Study #2: Pizza Runner
## Part C. Ingredient Optimisation
Link to [DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/4344) to execute the queries shown below

### Creating temporary tables to extract extra and exclusions toppings 
```sql
CREATE TEMPORARY TABLE extras_added AS
    SELECT order_id,
        customer_id,
        pizza_id,
        REGEXP_SPLIT_TO_TABLE(extras, ',')::INTEGER AS extras_id
    FROM customer_orders_temp
    WHERE extras != ''
    GROUP BY order_id, extras, customer_id, pizza_id;
    
CREATE TEMPORARY TABLE exclusions_added AS
    SELECT order_id, 
        customer_id,
        pizza_id,
        REGEXP_SPLIT_TO_TABLE(exclusions, ',')::INTEGER AS exclusions_id
    FROM customer_orders_temp
    WHERE exclusions != ''
    GROUP BY order_id, exclusions, customer_id, pizza_id;
```
<img width="935" height="237" alt="image" src="https://github.com/user-attachments/assets/a42797c4-9647-4df1-84fe-f3c9a8dd8bf2" />
<img width="934" height="206" alt="image" src="https://github.com/user-attachments/assets/4045475f-d59c-48a2-b05a-f3694f25a531" />


### 1. What are the standard ingredients for each pizza?
```sql
WITH standard_toppings AS (
    SELECT pizza_id,
        REGEXP_SPLIT_TO_TABLE(toppings, ',')::INTEGER AS topping_id
    FROM pizza_recipes
)
SELECT pizza_name,
    topping_name
FROM standard_toppings 
    JOIN pizza_names ON pizza_names.pizza_id = standard_toppings.pizza_id
    JOIN pizza_toppings ON pizza_toppings.topping_id = standard_toppings.topping_id;
```
<img width="935" height="503" alt="image" src="https://github.com/user-attachments/assets/8fbd7744-46e2-4f55-8213-4ce5e56a60c4" />

### 2. What was the most commonly added extra?
```sql
SELECT topping_name,
    COUNT(extras_id) AS extras_count
FROM extras_added
    JOIN pizza_toppings ON extras_added.extras_id = pizza_toppings.topping_id
GROUP BY topping_name
ORDER BY extras_count DESC;
```
<img width="936" height="139" alt="image" src="https://github.com/user-attachments/assets/f7eff385-b6a1-4a4e-b456-e53ab0ffd47d" />

### 3. What was the most common exclusion?
```sql
SELECT topping_name,
    COUNT(exclusions_id) AS exclusions_count
FROM exclusions_added
    JOIN pizza_toppings ON exclusions_added.exclusions_id = pizza_toppings.topping_id
GROUP BY topping_name
ORDER BY exclusions_count DESC;
```
<img width="933" height="141" alt="image" src="https://github.com/user-attachments/assets/b95ab403-eb1c-49af-b8e0-f6af2e00e161" />

### 4. Generate an order item for each record in the customers_orders table in the format of one of the following: 
- Meat Lovers
- Meat Lovers - Exclude Beef
- Meat Lovers - Extra Bacon
- Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers
```sql
WITH exclusions_names AS (
    SELECT order_id,
        exclusions_id,
        topping_name
    FROM exclusions_added
        JOIN pizza_toppings ON exclusions_added.exclusions_id = pizza_toppings.topping_id
    GROUP BY order_id, exclusions_id, topping_name
), extras_names AS (
    SELECT order_id,
        extras_id,
        topping_name
    FROM extras_added
        JOIN pizza_toppings ON extras_added.extras_id = pizza_toppings.topping_id
    GROUP BY order_id, extras_id, topping_name
)
SELECT customer_orders_temp.order_id,
    customer_orders_temp.customer_id,
    CASE 
        WHEN exclusions_id IS NULL AND extras_id IS NULL THEN pizza_name
        WHEN exclusions_id IS NOT NULL AND extras_id IS NULL THEN CONCAT(pizza_name, ' - Exclude ', STRING_AGG(exclusions_names.topping_name, ', '))
        WHEN exclusions_id IS NULL AND extras_id IS NOT NULL THEN CONCAT(pizza_name, ' - Extra ', STRING_AGG(extras_names.topping_name, ', '))
        ELSE CONCAT(pizza_name, ' - Exclude ', STRING_AGG(exclusions_names.topping_name, ', '), ' - Extra ', STRING_AGG(extras_names.topping_name, ', ')) 
        END AS pizza_description
FROM customer_orders_temp
    LEFT JOIN exclusions_names ON customer_orders_temp.order_id = exclusions_names.order_id
    LEFT JOIN extras_names ON customer_orders_temp.order_id = extras_names.order_id
    JOIN pizza_names ON customer_orders_temp.pizza_id = pizza_names.pizza_id
GROUP BY customer_orders_temp.order_id, customer_orders_temp.customer_id, exclusions_id, extras_id, pizza_name;
```
<img width="931" height="572" alt="image" src="https://github.com/user-attachments/assets/bc0a29ee-3ab9-40fe-90ec-63214ab127dd" />
