# 🍜 Case Study #1: Danny's Diner

<img src="https://user-images.githubusercontent.com/81607668/127727503-9d9e7a25-93cb-4f95-8bd0-20b87cb4b459.png" alt="Image" width="500" height="520">

## 📚 Table of Contents

- [Business Task](#business-task)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Question and Solution](#question-and-solution)

Please note that all the information regarding the case study has been sourced from the following link: [here](https://8weeksqlchallenge.com/case-study-1/).

---

## Business Task

Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite.

---

## Entity Relationship Diagram

![image](https://user-images.githubusercontent.com/81607668/127271130-dca9aedd-4ca9-4ed8-b6ec-1e1920dca4a8.png)

---

## Question and Solution

**1. What is the total amount each customer spent at the restaurant?**

```sql
SELECT
    s.customer_id,
    SUM(m.price) AS total_sales
FROM sales s
JOIN menu m ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id;
```

#### Answer:

| customer_id | total_sales |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

**2. How many days has each customer visited the restaurant?**

```sql
SELECT
    s.customer_id,
    COUNT(distinct s.order_date) as day
FROM sales s
GROUP BY s.customer_id
ORDER BY s.customer_id;
```

#### Answer:

| customer_id | visit_count |
| ----------- | ----------- |
| A           | 4           |
| B           | 6           |
| C           | 2           |

**3. What was the first item from the menu purchased by each customer?**

```sql
WITH first_item_was_purchased as (
    SELECT
        s.customer_id,
        me.product_name,
        RANK() OVER(
            PARTITION BY s.customer_id
            ORDER BY s.order_date
        ) as rank
    FROM sales s
    JOIN menu me ON me.product_id = s.product_id
)
SELECT
    customer_id,
    product_name
FROM first_item_was_purchased
WHERE rank = 1
GROUP BY customer_id, product_name
ORDER BY customer_id;
```

#### Answer:

| customer_id | product_name |
| ----------- | ------------ |
| A           | curry        |
| A           | sushi        |
| B           | curry        |
| C           | ramen        |

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**

```sql
SELECT
    m.product_name,
    COUNT(s.product_id) AS most_purchased_item
FROM sales s
JOIN menu m ON s.product_id = m.product_id
GROUP BY m.product_name
ORDER BY most_purchased_item DESC
LIMIT 1;
```

#### Answer:

| most_purchased | product_name |
| -------------- | ------------ |
| 8              | ramen        |

**5. Which item was the most popular for each customer?**

```sql
WITH most_popular_item_of_each_customer as (
    SELECT
        s.customer_id,
        COUNT(m.product_id) AS order_count,
        m.product_name,
        DENSE_RANK() OVER(
            PARTITION BY s.customer_id
            ORDER BY COUNT(s.product_id) DESC
        ) as rank
    FROM sales s
    JOIN menu m ON m.product_id = s.product_id
    GROUP BY s.customer_id
)

SELECT
    customer_id,
    product_name,
    order_count
FROM most_popular_item_of_each_customer
WHERE rank = 1
ORDER BY customer_id;
```

**6. Which item was purchased first by the customer after they became a member?**

```sql
WITH first_purchased_item AS (
SELECT
    m.product_name,
    s.customer_id,
	s.order_date,
    RANK() OVER (
        PARTITION BY s.customer_id
        ORDER BY order_date
    ) as rank
FROM sales s
JOIN members mem ON mem.customer_id = s.customer_id AND s.order_date > mem.join_date
JOIN menu m ON m.product_id = s.product_id
)

SELECT customer_id, product_name FROM first_purchased_item WHERE rank = 1;
```

**7. Which item was purchased just before the customer became a member?**

```sql
WITH ITEMS_WAS_PURCHASED_BEFORE_BECOME_MEMBER AS (
    SELECT s.customer_id, me.product_name, s.order_date, m.join_date,
    DENSE_RANK() OVER (
        PARTITION BY s.customer_id
        ORDER BY s.order_date DESC
    ) AS date_rank
    FROM sales s
    JOIN members m ON m.customer_id = s.customer_id
    JOIN menu me ON me.product_id = s.product_id
    WHERE s.order_date < m.join_date
)

SELECT customer_id, product_name, order_date, join_date FROM ITEMS_WAS_PURCHASED_BEFORE_BECOME_MEMBER WHERE date_rank = 1
ORDER BY customer_id;
```

**8.What is the total items and amount spent for each member before they became a member?**

```sql
SELECT s.customer_id,
    COUNT(me.product_name) AS total_items,
    SUM(me.price) AS total_sales
FROM sales s
JOIN members m ON m.customer_id = s.customer_id AND m.join_date > s.order_date
JOIN menu me ON me.product_id = s.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id;
```

#### Answer:

| customer_id | total_items | total_sales |
| ----------- | ----------- | ----------- |
| A           | 2           | 25          |
| B           | 3           | 40          |

**9.If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**

```sql
WITH cte as (
    SELECT s.customer_id,
        CASE
            WHEN me.product_name  = 'sushi' THEN me.price * 20
            ELSE me.price * 10
        END AS total
    FROM sales s
    JOIN menu me ON s.product_id = me.product_id
)
SELECT customer_id, SUM(total) FROM cte GROUP BY customer_id;
```

#### Answer:

| customer_id | total_points |
| ----------- | ------------ |
| A           | 860          |
| B           | 940          |
| C           | 360          |

**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**

```sql
WITH CALCULATE_POINT AS (
SELECT
    s.customer_id,
    CASE
        WHEN s.order_date <= m.join_date + 6 THEN me.price * 20
            ELSE me.price * 10
	END AS total_point
FROM sales s
JOIN menu me ON s.product_id = me.product_id
LEFT JOIN members m ON s.customer_id = m.customer_id
WHERE s.order_date < '2021-01-31'
)

SELECT customer_id, SUM(total_point) FROM CALCULATE_POINT GROUP BY customer_id;
```
