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
    WITH cte as
    (SELECT s.customer_id, s.product_id, order_date,
    		min(order_date) over(partition by s.customer_id) AS date  	
    FROM dannys_diner.sales s
    JOIN dannys_diner.members m
    ON s.customer_id = m.customer_id
    WHERE join_date <= order_date)

    SELECT customer_id, cte.product_id, product_name
    FROM cte
    JOIN dannys_diner.menu 
    ON cte.product_id = menu.product_id
    WHERE date = order_date;
```
| customer_id | product_id | product_name |
| ----------- | ---------- | ------------ |
| B           | 1          | sushi        |
| A           | 2          | curry        |

---
5. Which item was purchased just before the customer became a member?
```clj
    WITH cte as
    (SELECT s.customer_id, s.product_id, order_date,
    		max(order_date) over(partition by s.customer_id) AS date  	
    FROM dannys_diner.sales s
    JOIN dannys_diner.members m
    ON s.customer_id = m.customer_id
    WHERE join_date > order_date)
    
    SELECT customer_id, cte.product_id, product_name
    FROM cte
    JOIN dannys_diner.menu 
    ON cte.product_id = menu.product_id
    WHERE date = order_date;
```
| customer_id | product_id | product_name |
| ----------- | ---------- | ------------ |
| B           | 1          | sushi        |
| A           | 1          | sushi        |
| A           | 2          | curry        |

---
6. What is the total items and amount spent for each member before they became a member?
```clj
    SELECT s.customer_id,
    		count(s.product_id) AS total,
            sum(price) AS amount_spent
    FROM dannys_diner.sales s
    LEFT JOIN dannys_diner.members me
    ON s.customer_id = me.customer_id
    JOIN dannys_diner.menu m
    ON m.product_id = s.product_id
    WHERE order_date < join_date
    GROUP BY s.customer_id;
```
| customer_id | total | amount_spent |
| ----------- | ----- | ------------ |
| B           | 3     | 40           |
| A           | 2     | 25           |

---
7. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
```clj
    WITH cte as
    (SELECT s.customer_id, product_name,
            sum(price) AS total_spent
    FROM dannys_diner.sales s
    LEFT JOIN dannys_diner.members me
    ON s.customer_id = me.customer_id
    JOIN dannys_diner.menu m
    ON m.product_id = s.product_id
    GROUP BY 1,2)
    
    SELECT customer_id, sum(point) AS total_point
    FROM
    (SELECT customer_id, product_name, total_spent,
    		CASE WHEN product_name = 'sushi' THEN 20*total_spent
            	 ELSE 10*total_spent END AS point
    FROM cte) point
    GROUP BY customer_id
    ORDER BY customer_id;
```
| customer_id | total_point |
| ----------- | ----------- |
| A           | 860         |
| B           | 940         |
| C           | 360         |

---
8. JOIN all the things
 ```clj
    SELECT s.customer_id, 
     		order_date, 
     		product_name,
     		price,
      CASE WHEN order_date >= join_date THEN 'Y' ELSE 'N' END AS member
    FROM dannys_diner.sales s
    LEFT JOIN dannys_diner.members me
    ON s.customer_id = me.customer_id
    JOIN dannys_diner.menu m
    ON m.product_id = s.product_id
    ORDER BY 1,2;
```
| customer_id | date                     | product_name | price | member |
| ----------- | ------------------------ | ------------ | ----- | ------ |
| A           | 2021-01-01T00:00:00.000Z | sushi        | 10    | N      |
| A           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      |
| A           | 2021-01-07T00:00:00.000Z | curry        | 15    | Y      |
| A           | 2021-01-10T00:00:00.000Z | ramen        | 12    | Y      |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      |
| B           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      |
| B           | 2021-01-02T00:00:00.000Z | curry        | 15    | N      |
| B           | 2021-01-04T00:00:00.000Z | sushi        | 10    | N      |
| B           | 2021-01-11T00:00:00.000Z | sushi        | 10    | Y      |
| B           | 2021-01-16T00:00:00.000Z | ramen        | 12    | Y      |
| B           | 2021-02-01T00:00:00.000Z | ramen        | 12    | Y      |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      |
| C           | 2021-01-07T00:00:00.000Z | ramen        | 12    | N      |

---
