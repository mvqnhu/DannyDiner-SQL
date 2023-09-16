## I. Introduction 🍣 🍛 🍜
- Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen.
- Danny’s Diner is in need of assistance to help the restaurant stay afloat 

## II. Challenges
- The restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.
- Use the data to answer a few simple questions about customers : visiting patterns, how much money they’ve spent; which menu items are their favourite
## III. Dataset
<img width="552" alt="image" src="https://github.com/mvqnhu/DannyDiner-SQL/assets/138433845/07165cbc-da88-4659-81b3-4e97b438bc94">

## IV. Query 
---
1. What was the first item from the menu purchased by each customer?
```clj   
    WITH new AS
    (SELECT customer_id, product_id
    FROM
    (SELECT *, row_number() over(partition by customer_id order by order_date) rank
    FROM dannys_diner.sales) cte
    WHERE rank = 1)
    SELECT customer_id, product_name
    FROM new
    JOIN dannys_diner.menu m
    ON m.product_id = new.product_id;
```
| customer_id | product_name |
| ----------- | ------------ |
| A           | sushi        |
| B           | curry        |
| C           | ramen        |

---
2. What is the most purchased item on the menu and how many times was it purchased by all customers??

```clj
    SELECT product_name, count(s.order_date) as num
    FROM dannys_diner.sales s
    JOIN dannys_diner.menu m
    ON s.product_id = m.product_id
    GROUP BY 1
    ORDER BY num DESC
    LIMIT 1;

| product_name | num |
| ------------ | --- |
| ramen        | 8   |
```
---
3. Which item was the most popular for each customer?

```clj
    WITH cte AS
    (SELECT customer_id,
            m.product_name,
          count(order_date) AS times
      FROM dannys_diner.sales s
      JOIN dannys_diner.menu m
      ON s.product_id = m.product_id
      GROUP BY 1,2
      ORDER BY times DESC)

    SELECT  customer_id,
            product_name AS popular_item
    FROM 
    	(SELECT customer_id, product_name,
    		row_number() over(partition by customer_id order by times) AS rank
    	  FROM cte) ctee
    WHERE rank = 1;
```
| customer_id | popular_item |
| ----------- | ------------ |
| A           | sushi        |
| B           | sushi        |
| C           | ramen        |

---
4. Which item was purchased first by the customer after they became a member?

```clj
    SELECT customer_id, product_name
    FROM
    (SELECT customer_id, m.product_name,
    	 row_number() over(partition by customer_id order by order_date ASC) AS rank
    FROM dannys_diner.sales s
    JOIN dannys_diner.menu m
    ON s.product_id = m.product_id) cte
    WHERE rank = 1;
```
| customer_id | product_name |
| ----------- | ------------ |
| A           | curry        |
| B           | curry        |
| C           | ramen        |
---

