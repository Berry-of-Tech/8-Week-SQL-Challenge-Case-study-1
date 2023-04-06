# 8-Week-SQL-Challenge-Case-study-1-Danny’s Diner
---

![]()

## 1. INTRODUCTION

Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen.

Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.

## 2. PROBLEM STATEMENT

Danny wants to use the data to answer a few simple questions about his customers, especially about their

- visiting patterns,
- how much money they’ve spent, and
- which menu items are their favourite.
Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

He plans on using these insights to help him decide whether he should expand the existing customer loyalty program — additionally he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.

The data set contains the following 3 tables which you may refer to the relationship diagram below to understand the connection.

- `sales`
- `members`
- `menu`

---

## 3. ENTITY RELATIONSHIP DIAGRAM

![]()

---

## 4. CASE STUDY QUESTIONS

Each of the following case study questions can be answered using a single SQL statement:

1. What is the total amount each customer spent at the restaurant?
2. How many days has each customer visited the restaurant?
3. What was the first item from the menu purchased by each customer?
4. What is the most purchased item on the menu and how many times was it purchased by all customers?
5. Which item was the most popular for each customer?
6. Which item was purchased first by the customer after they became a member?
7. Which item was purchased just before the customer became a member?
8. What is the total items and amount spent for each member before they became a member?
9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

---

## BONUS QUESTIONS

**Join All The Things**

The following questions are related creating basic data tables that Danny and his team can use to quickly derive insights without needing to join the underlying tables using SQL.

Recreate the following table output using the available data:

| customer_id | order_date | product_name | price | member |
| ----------- | ---------- | ------------ | ----- | ------ |
| A           | 2021-01-01 | curry        | 15    | N      |
| A           | 2021-01-01 | sushi        | 10    | N      |
| A           | 2021-01-07 | curry        | 15    | Y      |
| A           | 2021-01-10 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| B           | 2021-01-01 | curry        | 15    | N      |
| B           | 2021-01-02 | curry        | 15    | N      |
| B           | 2021-01-04 | sushi        | 10    | N      |
| B           | 2021-01-11 | sushi        | 10    | Y      |
| B           | 2021-01-16 | ramen        | 12    | Y      |
| B           | 2021-02-01 | ramen        | 12    | Y      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-07 | ramen        | 12    | N      |

**Rank All The Things**

Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program.

 | customer_id | order_date | product_name | price | member | ranking |
| ----------- | ---------- | ------------ | ----- | ------ | ------- |
| A           | 2021-01-01 | curry        | 15    | N      |         |
| A           | 2021-01-01 | sushi        | 10    | N      |         |
| A           | 2021-01-07 | curry        | 15    | Y      | 1       |
| A           | 2021-01-10 | ramen        | 12    | Y      | 2       |
| A           | 2021-01-11 | ramen        | 12    | Y      | 3       |
| A           | 2021-01-11 | ramen        | 12    | Y      | 3       |
| B           | 2021-01-01 | curry        | 15    | N      |         |
| B           | 2021-01-02 | curry        | 15    | N      |         |
| B           | 2021-01-04 | sushi        | 10    | N      |         |
| B           | 2021-01-11 | sushi        | 10    | Y      | 1       |
| B           | 2021-01-16 | ramen        | 12    | Y      | 2       |
| B           | 2021-02-01 | ramen        | 12    | Y      | 3       |
| C           | 2021-01-01 | ramen        | 12    | N      |         |
| C           | 2021-01-01 | ramen        | 12    | N      |         |
| C           | 2021-01-07 | ramen        | 12    | N      |         |


## 5. SOLUTIONS TO THE CASE STUDY QUESTIONS AND BONUS QUESTIONS

#### 1. What is the total amount each customer spent at the restaurant?

```sql
SELECT s.customer_id, SUM(m.price) AS Total_amount
FROM sales s
JOIN menu m
ON s.product_id = m.product_id
GROUP BY 1
ORDER BY 2 DESC;
 ```
  
| customer_id | total_amount|
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

---
#### 2. How many days has each customer visited the restaurant?

```sql
SELECT customer_id, COUNT(DISTINCT order_date) AS number_of_days_visited
FROM sales 
GROUP BY 1
ORDER BY 2 DESC;
 ```
  
| customer_id | number_of_days_visited |
| ----------- | ---------------------- |
| B           | 6                      |
| A           | 4                      |
| C           | 2                      |

---

#### 3. What was the first item from the menu purchased by each customer?

To get the first item I get the first items ordered by each customer using the `WITH` statement then callout the 
customer_id, product_name, order_date, rank_order column where rank_order is 1

````sql
WITH First_item AS(
SELECT s.customer_id, m.product_name, m.product_id, s.order_date,
	RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date) AS 
	rank_order
FROM sales s
JOIN menu m
ON s.product_id = m.product_id)

SELECT customer_id, product_name, order_date, rank_order
FROM First_item
WHERE rank_order = 1
GROUP BY customer_id, product_name, order_date, rank_order;
````  

| customer_id | product_name | order_date | rank_order |
| ----------- | ------------ | ---------- | ---------- |
| A           | curry        | 2021-01-01 | 1          |
| A           | sushi        | 2021-01-01 | 1          |
| B           | curry        | 2021-01-01 | 1          |
| C           | ramen        | 2021-01-01 | 1          |


The first purchase for customer A was sushi and curry

The first purchase for customer B was curry

The first purchase for customer C was ramen

---

#### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

This is a straightforward question, using the DISTINCT, COUNT and LIMIT functions will do the trick

````sql
SELECT DISTINCT product_name, COUNT(product_name) AS total_count
FROM menu m
JOIN sales s
ON m.product_id = s.product_id
GROUP BY 1
ORDER BY 2 DESC
LIMIT 1;
  ````
| product_name | total_count|
| ------------ | ---------- |
| ramen        | 8          |

 
The most purchased item on the menu was ramen and it was purchased 8 times in total.

---

#### 5. Which item was the most popular for each customer?

Just like the third question, I created a CTE that provide me the popular items for each customer using the RANK() window function
Then filter out where rank_order equals 1

````sql
WITH popular_item AS (
SELECT s.customer_id, m.product_name, m.product_id, COUNT(m.product_id) AS product_count, 
	RANK() OVER(PARTITION BY s.customer_id ORDER BY COUNT(m.product_id)DESC) AS 
	rank_order
FROM sales s
JOIN menu m
ON s.product_id = m.product_id
GROUP BY s.customer_id, m.product_name, m.product_id)

SELECT customer_id, product_name, product_count, rank_order
FROM popular_item
WHERE rank_order = 1
 ```` 
 
| customer_id | product_name | product_count | rank_order |
| ----------- | ------------ | ------------- | ---------- |
| A           | ramen        | 3             | 1          |
| B           | ramen        | 2             | 1          |
| B           | curry        | 2             | 1          |
| B           | sushi        | 2             | 1          |
| C           | ramen        | 3             | 1          |

 
The most popular item for customer A was ramen and it was purchased 3 times.

The most popular item for customer B was curry, ramen and sushi and they purchased each product 2 times.

The most popular item for customer C was ramen and it was purchased 3 times.

---

#### 6. Which item was purchased first by the customer after they became a member?

I created a CTE that sorts the product according to the order_date using the RANK() window fuction
then filter outs where rank_order equals 1

````sql
WITH CTE_items_after_join_date AS ( 
	SELECT s.customer_id, m.product_name, s.order_date,
		RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date ) AS rank_order
FROM menu AS m
JOIN sales AS s
ON m.product_id = s.product_id
JOIN members AS me
ON s.customer_id = me.customer_id
WHERE me.join_date < s.order_date) 

SELECT customer_id, product_name, rank_order, order_date
FROM CTE_items_after_join_date
WHERE rank_order = 1
  ````

| customer_id | product_name | rank_order | order_date   |
| ----------- | ----------   | ---------- | ------------ |
| A           | ramen        | 1          | 2021-01-10   |
| B           | sushi        | 1          | 2021-01-11   |

---

#### 7. Which item was purchased just before the customer became a member?

This is similar to question 6, I only added the DESC to the CTE in order to get the 
item purchased before the customer became a member 

````sql
WITH CTE_items_before_join_date AS ( 
	SELECT s.customer_id, m.product_name, s.order_date,
		RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date DESC) AS rank_order
FROM menu AS m
JOIN sales AS s
ON m.product_id = s.product_id
JOIN members AS me
ON s.customer_id = me.customer_id
WHERE me.join_date > s.order_date) 

SELECT customer_id, product_name, order_date
FROM CTE_items_before_join_date
WHERE rank_order = 1


  ````

| customer_id | product_name | order_date | 
| ----------- | ----------   | ---------- |
| A           | sushi        | 2021-01-01 |
| A           | curry        | 2021-01-01 |
| B           | sushi        | 2021-01-04 |

---

Customer A purchased sushi and curry just before they became a member while customer B purchased Sushi.

#### 8. What is the total items and amount spent for each member before they became a member?

To find the total amount spent by Customer A and B and Total items bought before they became members.
It is similar to question 7, it involves activities before they became members

````sql
SELECT s.customer_id, COUNT(s.product_id) AS total_items, SUM(m.price) AS total_price
FROM sales AS s
JOIN menu AS m
ON s.product_id = m.product_id
JOIN members AS me
ON s.customer_id = me.customer_id
WHERE me.join_date > s.order_date
GROUP BY 1
ORDER BY 3 DESC
  ````

| customer_id | total_items | total_price |
| ----------- | ----------- | ----------  |
| B           | 3           | 40          |
| A           | 2           | 25          |

---

#### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

Using the CASE and SUM functions does the trick here

````sql
SELECT s.customer_id, SUM(CASE WHEN m.product_id = 1 THEN m.price * 20
		ELSE m.price * 10 END) AS Total_points
FROM menu AS m
JOIN sales AS s
ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY 2 DESC;
  ````

| customer_id | points |
| ----------- | ------ |
| B           | 940    |
| A           | 860    |
| C           | 360    |

---

#### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

I created a CTE that gives first week including the join_date then use the CASE function and also filter out WHERE order_date <= '2021-01-31' AND order_date >= join_date

````sql
WITH first_week AS 
(
   SELECT *, 
      join_date + INTERVAL '6 days' AS valid_date 
   FROM members AS m
)

SELECT f.customer_id, SUM(CASE 
						WHEN s.order_date BETWEEN f.join_date AND valid_date 
						THEN price * 20 
					   WHEN order_date NOT BETWEEN join_date AND valid_date 
						AND product_name = 'sushi' THEN price * 20
					   WHEN order_date NOT BETWEEN join_date AND valid_date 
						AND product_name <> 'sushi' THEN price * 10 END) 
						AS total_points
FROM first_week AS f
JOIN sales AS s
ON s.customer_id = f.customer_id
JOIN menu AS m
ON s.product_id = m.product_id
WHERE order_date <= '2021-01-31' AND order_date >= join_date
GROUP BY 1
ORDER BY 1;

  ````

| customer_id | total_points |
| ----------- | ----------   |
| A           | 1020         |
| B           | 320          |

---

Customer A would have 1020 points at the end of January

Customer B would have 320 points at the end of January

--- 

### Bonus Questions

#### Join All The Things

````sql
SELECT s.customer_id, s.order_date, m.product_name, m.price,
		CASE WHEN me.join_date > s.order_date THEN 'N' 
		WHEN me.join_date <= s.order_date THEN 'Y'
		ELSE 'N' END AS member
FROM sales AS s
LEFT JOIN menu AS m
ON s.product_id = m.product_id
LEFT JOIN members AS me
ON me.customer_id = s.customer_id
ORDER BY 1, 2, 4 DESC;

````

| customer_id | order_date | product_name | price | member |
| ----------- | ---------- | ------------ | ----- | ------ |
| A           | 2021-01-01 | curry        | 15    | N      |
| A           | 2021-01-01 | sushi        | 10    | N      |
| A           | 2021-01-07 | curry        | 15    | Y      |
| A           | 2021-01-10 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| B           | 2021-01-01 | curry        | 15    | N      |
| B           | 2021-01-02 | curry        | 15    | N      |
| B           | 2021-01-04 | sushi        | 10    | N      |
| B           | 2021-01-11 | sushi        | 10    | Y      |
| B           | 2021-01-16 | ramen        | 12    | Y      |
| B           | 2021-02-01 | ramen        | 12    | Y      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-07 | ramen        | 12    | N      |

---

### Rank All The Things

First we need to select all the necessary columns from `sales`, `menu` and `members` tables - we do that using CTE and `WITH` statement.
Next we can rank orders from this table by `customer_id` and `member` columns.

````sql
WITH member_cte AS (
SELECT s.customer_id, s.order_date, m.product_name, m.price,
		CASE WHEN me.join_date > s.order_date THEN 'N' 
		WHEN me.join_date <= s.order_date THEN 'Y'
		ELSE 'N' END AS member
FROM sales AS s
LEFT JOIN menu AS m
ON s.product_id = m.product_id
LEFT JOIN members AS me
ON me.customer_id = s.customer_id
ORDER BY 1, 2, 4 DESC)

SELECT *, CASE WHEN member = 'N' THEN NULL
			ELSE RANK() OVER(PARTITION BY customer_id, member ORDER BY 	order_date) END AS Rank
FROM member_cte;

  ````
 
 | customer_id | order_date | product_name | price | member | ranking |
| ----------- | ---------- | ------------ | ----- | ------ | ------- |
| A           | 2021-01-01 | curry        | 15    | N      |         |
| A           | 2021-01-01 | sushi        | 10    | N      |         |
| A           | 2021-01-07 | curry        | 15    | Y      | 1       |
| A           | 2021-01-10 | ramen        | 12    | Y      | 2       |
| A           | 2021-01-11 | ramen        | 12    | Y      | 3       |
| A           | 2021-01-11 | ramen        | 12    | Y      | 3       |
| B           | 2021-01-01 | curry        | 15    | N      |         |
| B           | 2021-01-02 | curry        | 15    | N      |         |
| B           | 2021-01-04 | sushi        | 10    | N      |         |
| B           | 2021-01-11 | sushi        | 10    | Y      | 1       |
| B           | 2021-01-16 | ramen        | 12    | Y      | 2       |
| B           | 2021-02-01 | ramen        | 12    | Y      | 3       |
| C           | 2021-01-01 | ramen        | 12    | N      |         |
| C           | 2021-01-01 | ramen        | 12    | N      |         |
| C           | 2021-01-07 | ramen        | 12    | N      |         |

---













