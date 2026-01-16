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

**2. How many days has each customer visited the restaurant?**

```sql
SELECT
    s.customer_id,
    COUNT(distinct s.order_date) as day
FROM sales s
GROUP BY s.customer_id
ORDER BY s.customer_id;
```

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

SELECT customer_id, product_name FROM item_was_purchase_first WHERE rank = 1;
```

#### Answer:

| customer_id | product_name |
| ----------- | ------------ |
| A           | ramen        |
| B           | sushi        |
