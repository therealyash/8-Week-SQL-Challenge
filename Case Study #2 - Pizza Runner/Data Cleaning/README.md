# Customer Orders

The "exclusions" and "extras" columns within the "customer_orders" table require refinement prior to their utilization in queries. Within these columns, there exist vacant spaces as well as null values that necessitate rectification.

```
SELECT order_id,
       customer_id,
       pizza_id,
       CASE
           WHEN exclusions = '' THEN NULL
           WHEN exclusions = 'null' THEN NULL
           ELSE exclusions
       END AS exclusions,
       CASE
           WHEN extras = '' THEN NULL
           WHEN extras = 'null' THEN NULL
           ELSE extras
       END AS extras,
       order_time
FROM customer_orders;
```
# Converting order_time to DateTime

### Try using the STR_TO_DATE function to convert the order_time column to a date column. Use a duplicate to avoid losing the data.

```
UPDATE customer_orders
SET order_date_time = STR_TO_DATE(order_time, '%d-%m-%Y')
```

Unfortunately this didn't work properly for some reason.Then I created 2 columns order_datetime & order_timing which would contain substring of order_time with date and time respectively.

```
// Date column
ALTER TABLE customer_orders
ADD COLUMN order_datetime VARCHAR(25);

// Time column
ALTER TABLE customer_orders
ADD COLUMN order_timing VARCHAR(25);

// Date time column
ALTER TABLE customer_orders
ADD COLUMN order_date_time DATETIME;

// Create a date substring
UPDATE customer_orders 
SET order_timing = SUBSTRING(order_time, 1,10);

// Create a time substring
UPDATE customer_orders 
SET order_timing = SUBSTRING(order_time, 12,16);

// Merge both the columns together
UPDATE customer_orders
SET order_date_time = STR_TO_DATE(CONCAT(order_datetime, ' ', order_timing), '%Y-%m-%d %H:%i:%s');

```


# Pizza Recipe

## Create a new table to hold the expanded toppings
```
CREATE TABLE expanded_toppings (
    pizza_id INT,
    topping INT
);
```

## Insert the expanded data into the new table
```
INSERT INTO expanded_toppings (pizza_id, topping)
SELECT
    pizza_id,
    CAST(SUBSTRING_INDEX(SUBSTRING_INDEX(toppings, ',', n.n), ',', -1) AS INT) AS topping
FROM
    pizza_recipes
JOIN (
    SELECT 1 + units.i + tens.i * 10 AS n
    FROM (
        SELECT 0 AS i UNION SELECT 1 UNION SELECT 2 UNION SELECT 3
    ) units
    JOIN (
        SELECT 0 AS i UNION SELECT 1 UNION SELECT 2 UNION SELECT 3
    ) tens
    LIMIT 100
) n
ON CHAR_LENGTH(toppings)
    -CHAR_LENGTH(REPLACE(toppings, ',', '')) >= n.n - 1;

```

## Query the expanded_toppings table
```
SELECT * FROM expanded_toppings;
```

# Runner Orders

```
UPDATE runner_orders
SET cancellation = NULL
WHERE cancellation = 'Nan'
```