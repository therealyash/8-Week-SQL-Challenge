## Case Study #1 - Danny's Diner Solution

![1](https://github.com/therealyash/8-Week-SQL-Challenge/assets/114616305/f0d152ef-2485-4e00-b79a-b5caf395c36c)


### Each of the following case study questions can be answered using a single SQL statement:


1. What is the total amount each customer spent 
at the restaurant?
```
SELECT customer_id AS customer, SUM(price) AS total
FROM sales s 
JOIN menu m 
ON s.product_id = m.product_id
GROUP BY 1
```
2. How many days has each customer visited the restaurant?
```
SELECT customer_id AS customer, COUNT(DISTINCT order_date) AS days_visited
FROM sales
GROUP BY 1
```
3. What was the first item from the menu purchased by each customer?
```
SELECT s.customer_id,MIN(s.order_date) AS first_purchase_date,
  m.product_name AS first_item_purchased
FROM sales s
JOIN menu m 
ON s.product_id = m.product_id
GROUP BY s.customer_id;
```
4. What is the most purchased item on the menu and how many times was it purchased by all customers?
```
SELECT m.product_name AS product, COUNT(s.product_id) AS frequency
FROM sales s 
JOIN menu m 
ON s.product_id = m.product_id
GROUP BY 1
ORDER BY 2 DESC
LIMIT 1
```
5. Which item was the most popular for each customer?
```
WITH order_info AS
(SELECT s.customer_id,m.product_name,COUNT(product_name) AS order_count,
RANK() OVER(PARTITION BY customer_id ORDER BY order_count DESC ) AS rank
FROM sales s
JOIN menu m 
ON s.product_id = m.product_id
GROUP BY 1,2)

SELECT customer_id, product_name, order_count
FROM order_info
WHERE rank = 1
```
6. Which item was purchased first by the customer after they became a member?
```
WITH order_info AS
(
  SELECT m.customer_id, s.product_id, s.order_date AS o_date,m.join_date AS j_date,
  DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY s.order_date ASC) AS order_rank
  FROM sales s 
  JOIN members m 
  ON s.customer_id = m.customer_id
  WHERE order_date >= join_date
)

SELECT customer_id, product_name,j_date, o_date   
FROM order_info o
JOIN menu m
ON o.product_id = m.product_id
WHERE order_rank = 1 
ORDER BY 1
```
7. Which item was purchased just before the customer became a member?
```
WITH order_info AS
(
  SELECT s.customer_id, s.product_id, m.join_date, s.order_date,
  DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY order_date DESC) AS order_rank
  FROM sales s 
  JOIN members m 
  ON s.customer_id = m.customer_id
  WHERE order_date < join_date
)

SELECT customer_id, m.product_name, order_date, join_date
FROM order_info o 
JOIN menu m 
ON m.product_id = o.product_id
WHERE order_rank = 1
ORDER BY 1
```
8. What is the total items and amount spent for each member before they became a member?
```
SELECT s.customer_id,
COUNT(*) freq_order,
SUM(price) total
FROM sales s
JOIN menu m 
ON s.product_id = m.product_id
JOIN members me 
ON me.customer_id = s.customer_id
WHERE order_date < join_date
GROUP BY 1
```
9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
```
WITH points_table AS
(
SELECT *, 
CASE 
    WHEN product_name = 'sushi' THEN price*20
    ELSE price*10
END AS points
FROM menu
)

SELECT customer_id, SUM(p.points) total_points
FROM points_table p 
JOIN sales s 
ON s.product_id = p.product_id
GROUP BY customer_id
```
10. In the first week after a customer joins the
    program (including their join date) they earn 2x points on 
    all items, not just sushi - how many points do 
    customer A and B have at the end of January?
```
WITH order_info AS
(SELECT s.customer_id, s.product_id, 
join_date, order_date, price
FROM members m 
JOIN sales s 
ON m.customer_id = s.customer_id
JOIN menu me 
ON me.product_id = s.product_id
WHERE order_date >= join_date
  AND order_date <= '2021-01-31' )

SELECT customer_id,
SUM( 
  CASE 
    WHEN order_date BETWEEN join_date AND DATE_ADD(join_date,INTERVAL 6 DAY) 
      THEN price*20
    WHEN order_date NOT BETWEEN join_date AND DATE_ADD(join_date,INTERVAL 6 DAY) 
      AND product_id = 1
        THEN price*20
    WHEN order_date NOT BETWEEN join_date AND DATE_ADD(join_date,INTERVAL 6 DAY) 
      AND product_id != 1
        THEN price*10
END) AS total_points
FROM order_info
GROUP BY 1
```
