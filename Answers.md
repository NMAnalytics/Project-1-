# Case Study #1 - Danny's Diner answers



### 1. What is the total amount each customer spent at the restaurant?

``` sql 
select s.customer_id, sum(price) as total_sales
from sales as s
join menu as m  
On s.product_id = m.product_id
group by customer_id
```

|customer_id|total_sales|
|---|---|
|A|76|
|B|74|
|C|36|

- Customer A spent $76
- Customer B spent $74
- Customer C spent $36


### 2. How many days has each customer visited the restaurant?
 ```sql
 Select customer_id, count(distinct order_date) As days_visted
From sales
Group by customer_id 
```

|customer_id|days_visted|
|---|---|
|A|4|
|B|6|
|C|2|

- Customer A vistied the resuarant 4 times
- Customer B vistied the resuarant 6 times
- Customer C vistied the resuarant 2 times



### 3. What was the first item from the menu purchased by each customer?
```sql SELECT top 4 customer_id,min(order_date) as date,product_name
from sales as s
join menu  as m 
On s.product_id = m.product_id
GROUP BY order_date,customer_id,product_name
having order_date = min(order_date) 
```
|customer_id|date|product_name|
|---|---|---|
|A|2021-01-01|curry|
|A|2021-01-01|sushi|
|B|2021-01-01|curry|
|C|2021-01-01|ramen|

- Customer A purchased curry and sushi 
- Customer B purchased curry 
- Customer C purchased ramen

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
``` sql
Select product_name, count(s.product_id )as total_purchase
From sales as S
Join menu as M 
On s.product_id = m.product_id
Group by s.product_id, product_name
order by total_purchase DESC
```

|product_name|total_purchase|
|---|---|
|ramen|8|
|curry|4|
|sushi|3|

- Ramen is the most purchased item with 8


### 5. Which item was the most popular for each customer?
```sql
Select customer_id, product_name, count(s.product_id) as most_poular
From sales as S
Join menu as M 
On s.product_id = m.product_id
Group by customer_id, product_name
Order by customer_id, most_poular DESC
```

|customer_id|product_name|most_poular|
|---|---|---|
|A|ramen|3|
|A|curry|2|
|A|sushi|1|
|B|sushi|2|
|B|curry|2|
|B|ramen|2|
|C|ramen|3|

- Customer A  and C most ordered dish was ramen
- Customer B most ordered dish was a tie bewteen ramen, sushi and curry

### 6. Which item was purchased first by the customer after they became a member?
```sql
with aftercte  as
(
select s.order_date, m.join_date, s.customer_id,s.product_id,
dense_rank() over (partition by s.customer_id order by order_date) as rank
 FROM sales AS s
 JOIN members AS m
  ON s.customer_id = m.customer_id
  where s.order_date >= m.join_date
  )
  select customer_id,s.order_date, m2.product_name
  from aftercte as s
  join menu  as m2
  on s.product_id = m2.product_id
  where rank = 1 
```
|customer_id|order_date|product_name|
|---|---|---|
|A|2021-01-07|curry|
|B|2021-01-11|sushi|

Customer A first purchase after membership was curry and customer B first purchase was sushi

### 7. Which item was purchased just before the customer became a member?
```sql 
with firstcte  as
(
select s.order_date, m.join_date, s.customer_id,s.product_id,
dense_rank() over (partition by s.customer_id order by order_date desc) as rank
 FROM sales AS s
 JOIN members AS m
  ON s.customer_id = m.customer_id
  where s.order_date < m.join_date
  )
  select customer_id,s.order_date, m2.product_name
  from firstcte as s
  join menu  as m2
  on s.product_id = m2.product_id
  where rank = 1 
  ```
|customer_id|order_date|product_name|
|---|---|---|
|A|2021-01-01|sushi|
|A|2021-01-01|curry|
|B|2021-01-04|sushi|


- Customer A first purchase before membership was sushi and curry. 
- Customer B first purchase was sushi.

### 8. What is the total items and amount spent for each member before they became a member?  
``` sql 
select s.customer_id, 
sum(price) as amount_spent,
count(DISTINCT m2.product_id) as total_items
 FROM sales AS s
 JOIN members AS m
  ON s.customer_id = m.customer_id
  join menu as M2
  on s.product_id = m2.product_id
  where s.order_date < m.join_date 
 group by s.customer_id
 order by s.customer_id
 ```
 |customer_id|amount_spent|total_items|
|---|---|---|
|A|25|2|
|B|40|2|

- Customer A spent $25 and bought 2 items.
- Customer B spent 40 and bought 2 items.

### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
 ```sql
 with points_cte as 
( select * ,
case when product_id = 1 then price * 20 
else price * 10
end as points 
from menu as m 
)
select s.customer_id, sum(p.points) as total_points
from sales as s
join points_cte as p 
on s.product_id = p.product_id
group by s.customer_id 
order by s.customer_id
```
|customer_id|total_points|
|---|---|
|A|860|
|B|940|
|C|360|

- Customer A had 860 points 
- Customer B has 940 points 
- Customer C has 360 points 

### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
``` SQl
WITH join_date_cte AS 
(
 SELECT *, 
  DATEADD(DAY, 6, join_date) AS end_week, 
  EOMONTH('2021-01-31') AS last_date
 FROM members AS m
)
SELECT j.customer_id, s.order_date, j.join_date, 
 j.end_week, j.last_date, m.product_name, m.price,
 SUM(CASE
  WHEN m.product_name = 'sushi' THEN 2 * 10 * m.price
  WHEN s.order_date BETWEEN j.join_date AND j.end_week THEN 2 * 10 * m.price
  ELSE 10 * m.price
  END) AS points
FROM join_date_cte AS j
JOIN sales AS s
 ON j.customer_id = s.customer_id
JOIN menu AS m
 ON s.product_id = m.product_id
WHERE s.order_date < j.last_date
GROUP BY j.customer_id, s.order_date, j.join_date, j.end_week, j.last_date, m.product_name, m.price
```

|customer_id|order_date|join_date|end_week|last_date|product_name|price|points|
|---|---|---|---|---|---|---|---|
|A|2021-01-01|2021-01-07|2021-01-13|2021-01-31|curry|15|150|
|A|2021-01-01|2021-01-07|2021-01-13|2021-01-31|sushi|10|200|
|A|2021-01-07|2021-01-07|2021-01-13|2021-01-31|curry|15|300|
|A|2021-01-10|2021-01-07|2021-01-13|2021-01-31|ramen|12|240|
|A|2021-01-11|2021-01-07|2021-01-13|2021-01-31|ramen|12|480|
|B|2021-01-01|2021-01-09|2021-01-15|2021-01-31|curry|15|150|
|B|2021-01-02|2021-01-09|2021-01-15|2021-01-31|curry|15|150|
|B|2021-01-04|2021-01-09|2021-01-15|2021-01-31|sushi|10|200|
|B|2021-01-11|2021-01-09|2021-01-15|2021-01-31|sushi|10|200|
|B|2021-01-16|2021-01-09|2021-01-15|2021-01-31|ramen|12|120|

- At the end of January customer A has 1370 points.
-  Customer B has 820 points.

