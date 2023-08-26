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