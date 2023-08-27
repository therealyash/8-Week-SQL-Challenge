# A. Pizza Metrics

### 1. How many pizzas were ordered?
```
SELECT COUNT(pizza_id) AS total_pizza_ordered
FROM customer_orders
```
### 2. How many unique customer orders were made?
```
SELECT 
  COUNT(DISTINCT order_id) AS total_unique_orders
FROM customer_orders
```
### 3. How many successful orders were delivered by each runner?
```
SELECT runner_id, COUNT(runner_id) success_order_by_runner
FROM runner_orders
WHERE pickup_time IS NOT NULL
GROUP BY 1
```
### 4. How many of each type of pizza was delivered?
```
SELECT co.pizza_id, COUNT(co.pizza_id) pizza_order_count 
FROM customer_orders co 
JOIN pizza_names pn 
ON pn.pizza_id = co.pizza_id
JOIN runner_orders ro
ON ro.order_id = co.order_id 
WHERE cancellation IS NULL
GROUP BY 1
```
### 5. How many Vegetarian and Meatlovers were ordered by each customer?
```
SELECT customer_id, pn.pizza_name,  COUNT(co.pizza_id) AS no_of_pizza_ordered
FROM customer_orders co 
JOIN pizza_names pn
ON pn.pizza_id = co.pizza_id
GROUP BY 1,2
```
### 6. What was the maximum number of pizzas delivered in a single order?
```
SELECT co.customer_id, co.order_id, COUNT(co.pizza_id) count
FROM customer_orders co
JOIN runner_orders ro 
ON co.order_id = ro.order_id
WHERE cancellation IS NULL
GROUP BY 1,2
ORDER BY 3 DESC
LIMIT 1
```
### 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
```
-- ***
SELECT customer_id,
       SUM(CASE
               WHEN (exclusions IS NOT NULL
                     OR extras IS NOT NULL) THEN 1
               ELSE 0
           END) AS change_in_pizza,
       SUM(CASE
               WHEN (exclusions IS NULL
                     AND extras IS NULL) THEN 1
               ELSE 0
           END) AS no_change_in_pizza
FROM customer_orders
INNER JOIN runner_orders USING (order_id)
WHERE cancellation IS NULL
GROUP BY customer_id
ORDER BY customer_id;
```
### 8. How many pizzas were delivered that had both exclusions and extras?
```
--***
SELECT customer_id,
       SUM(CASE
               WHEN (exclusions IS NOT NULL
                     AND extras IS NOT NULL) THEN 1
               ELSE 0
           END) AS both_extra_and_exclusion
FROM customer_orders
INNER JOIN runner_orders USING (order_id)
WHERE cancellation IS NULL
GROUP BY 1
ORDER BY 1;
```
### 9. What was the total volume of pizzas ordered for each hour of the day?
```
SELECT hour(order_date_time) AS 'Hour',
       COUNT(order_time) AS 'No. of pizzas ordered',
       ROUND(100*COUNT(order_id) / SUM(COUNT(order_id)) OVER(), 2) AS 'Volume of Pizzas Ordered'
FROM customer_orders
GROUP BY 1
ORDER BY 1;   
```
### 10. What was the volume of orders for each day of the week?
```
SELECT DAYNAME(order_datetime) AS 'Day Of Week',
       count(order_id) AS 'Number of pizzas ordered',
       round(100*count(order_id) /sum(count(order_id)) over(), 2) AS 'Volume of pizzas ordered'
FROM customer_orders
GROUP BY 1
ORDER BY 2 DESC;
```
