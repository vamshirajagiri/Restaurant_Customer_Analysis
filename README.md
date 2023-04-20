# Restaurant_Customer_Analysis
"Restaurant Customer Analysis" is a SQL project that examines 
* customer behavior and preferences in a restaurant setting,
* including visiting patterns :bar_chart:, 
* total spending :heavy_dollar_sign:, and 
* favorite menu items. 
By analyzing these factors, the project helps restaurant owners to develop a **deeper understanding** of their customers and create a more **personalized experience** for loyal customers. The project involves **querying a database** to answer key questions such as the 
+ most popular menu item and 
+ the points earned by customers. 
Overall, this project provides **valuable insights** into customer behavior and can assist restaurant owners and managers in _optimizing their business strategy_.

## 3 key datasets :file_folder:
> There's 3 key datasets for this project,we have to query this 3 data sets.
* **sales**
* **menu**
* **members**

## Entity Relationship Diagram
![ER_Dig](https://user-images.githubusercontent.com/108252662/233406294-29077925-57ed-4121-b68d-ab9fd1459769.png)

## Table 1: sales
The **sales table** captures all _customer_id_ level purchases with an corresponding _order_date_ and _product_id_ information for 
when and what menu items were ordered.
![demo_data](https://user-images.githubusercontent.com/108252662/233407002-a9a4452f-5944-41e7-ac92-ab59332f872f.png)

## Table 2: menu
The **menu table** maps the _product_id_ to the actual _product_name_ and _price_ of each menu item.
![demo_data2](https://user-images.githubusercontent.com/108252662/233407615-f40d8713-8ae2-4084-b605-3b30cfc72101.png)

## Table 3: members
The final **members table** captures the _join_date_ when a _customer_id_ joined the beta version of the  loyalty program.
![demo_data3](https://user-images.githubusercontent.com/108252662/233408291-4ea1a04a-e4f5-4627-897e-1d6837336bce.png)


## These are the Case Study Questions and SQL Queries

> 1. What is the total amount each customer spent at the restaurant?

```
select sales.customer_id as customers,sum(menu.price) as spent_amount
from sales left join menu
on sales.product_id=menu.product_id
group by customers;
```

> 2. How many days has each customer visited the restaurant?

```
select sales.customer_id as customers,count(sales.order_date) days_visited
from sales
group by customers;
```
> 3. What was the first item from the menu purchased by each customer?
```
with products_ordered as (
select sales.customer_id,sales.order_date, menu.product_id,menu.product_name,
row_number() over(partition by sales.customer_id order by sales.order_date ) as roww
from menu join sales
on sales.product_id=menu.product_id
)

select * from products_ordered
where roww =1;
```
> 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

```
select sl.product_id,me.product_name,count(sl.product_id) as Total_ordered_quantity
from sales as sl join menu as me 
on me.product_id=sl.product_id
group by sl.product_id,me.product_name;
```

> 5. Which item was the most popular for each customer?

```
select customer_id,product_id,count(product_id)
from sales
group by customer_id,product_id 
having 
count(product_id) =
 (select max(count) from 
 (select customer_id, count(product_id) as count 
 from sales
 group by customer_id,product_id ) as subquery
 where subquery.customer_id=sales.customer_id) ;
 ```

> 6. Which item was purchased first by the customer after they became a member?

```

WITH member_orders AS (
  SELECT sl.customer_id, order_date, product_name,
  ROW_NUMBER() OVER (PARTITION BY sl.customer_id ORDER BY order_date) AS order_rank
  FROM sales AS sl
  JOIN members AS mem ON mem.customer_id = sl.customer_id
  JOIN menu AS men ON men.product_id = sl.product_id
  WHERE order_date >= mem.join_date
), first_purchase AS (
  SELECT customer_id, order_date, product_name
  FROM member_orders
  WHERE order_rank = 1
)
SELECT fp.customer_id, fp.order_date, fp.product_name
FROM first_purchase AS fp;
```

> 7. Which item was purchased just before the customer became a member?

```

with ordered_before_joining as (
select mem.customer_id,sl.order_date,men.product_name,
rank() over(partition by mem.customer_id order by sl.order_date desc) as order_rank
from sales as sl
join members as mem on mem.customer_id=sl.customer_id
join menu as men on men.product_id=sl.product_id
where order_date< mem.join_date
), last_purchase_before_membership as(
select customer_id,order_date,product_name
from ordered_before_joining
where order_rank =1
)
select * from last_purchase_before_membership;
```
 
> 8. What is the total items and amount spent for each member before they became a member?

```

select sl.customer_id,count(sl.product_id),sum(price)from sales as sl
join members as mem 
on sl.customer_id=mem.customer_id
join menu as men on men.product_id=sl.product_id
where sl.order_date<mem.join_date
group by sl.customer_id
order by sl.customer_id;
```

> 9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

```

with allocating_points as (
select sl.customer_id,product_name,sum(men.price ) as spent_amnt,
case when product_name="sushi" then sum(men.price )*10*2 else sum(men.price )*10 end as points
 from sales as sl
join menu as men
on men.product_id=sl.product_id
group by 1,2
)

select customer_id,sum(points) as points from allocating_points
group by customer_id;
```

> 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items,
> not just sushi - how many points do customer A and B have at the end of January?

```

select sl.customer_id, sum(men.price*2) as poits from sales as sl
join members as mem
on sl.customer_id=mem.customer_id
join menu as men
on sl.product_id=men.product_id
where sl.order_date>=mem.join_date
and datediff(order_date,join_date)<=7
and month(sl.order_date)=1
group by 1
order by 1;
```

> Combinig all the DataSets and checking they're **membership status** for each order.
```

select sl.customer_id,sl.order_date,men.product_name,men.price,
case when mem.customer_id=sl.customer_id and sl.order_date >= mem.join_date then "Y" else "N" end as members
from sales as sl 
left join members as mem
on sl.customer_id=mem.customer_id
left join menu as men
on men.product_id=sl.product_id;
``` 






