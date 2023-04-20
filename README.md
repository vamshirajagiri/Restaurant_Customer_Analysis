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

## Table 1: Sales
The **sales table** captures all _customer_id_ level purchases with an corresponding _order_date_ and _product_id_ information for 
when and what menu items were ordered.
![demo_data](https://user-images.githubusercontent.com/108252662/233407002-a9a4452f-5944-41e7-ac92-ab59332f872f.png)

## Table 2: Menu
The **menu table** maps the _product_id_ to the actual _product_name_ and _price_ of each menu item.
![demo_data2](https://user-images.githubusercontent.com/108252662/233407615-f40d8713-8ae2-4084-b605-3b30cfc72101.png)

## Table 3: Members
The final **members table** captures the _join_date_ when a _customer_id_ joined the beta version of the  loyalty program.
![demo_data3](https://user-images.githubusercontent.com/108252662/233408291-4ea1a04a-e4f5-4627-897e-1d6837336bce.png)


## These are the Case Study Questions and SQL Queries

> 1. What is the total amount each customer spent at the restaurant?

```
SELECT sales.customer_id AS customers, SUM(menu.price) AS spent_amount
FROM sales
LEFT JOIN menu
ON sales.product_id = menu.product_id
GROUP BY sales.customer_id;

```

> 2. How many days has each customer visited the restaurant?

```
SELECT sales.customer_id AS customers, COUNT(sales.order_date) AS days_visited
FROM sales
GROUP BY customers;

```
> 3. What was the first item from the menu purchased by each customer?
```
WITH products_ordered AS (
    SELECT 
        sales.customer_id,
        sales.order_date, 
        menu.product_id,
        menu.product_name,
        ROW_NUMBER() OVER (PARTITION BY sales.customer_id ORDER BY sales.order_date) AS roww
    FROM menu 
    JOIN sales ON sales.product_id = menu.product_id
)
SELECT *
FROM products_ordered
WHERE roww = 1;

```
> 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

```
SELECT 
    sl.product_id, 
    me.product_name, 
    COUNT(sl.product_id) AS Total_ordered_quantity
FROM sales AS sl 
JOIN menu AS me 
ON me.product_id = sl.product_id
GROUP BY sl.product_id, me.product_name;

```

> 5. Which item was the most popular for each customer?

```
SELECT 
    customer_id, 
    product_id, 
    COUNT(product_id)
FROM sales
GROUP BY customer_id, product_id 
HAVING 
    COUNT(product_id) = (
        SELECT MAX(count) 
        FROM (
            SELECT 
                customer_id, 
                COUNT(product_id) AS count 
            FROM sales
            GROUP BY customer_id, product_id
        ) AS subquery
        WHERE subquery.customer_id = sales.customer_id
    );

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

WITH ordered_before_joining AS (
    SELECT 
        mem.customer_id,
        sl.order_date,
        men.product_name,
        RANK() OVER (PARTITION BY mem.customer_id ORDER BY sl.order_date DESC) AS order_rank
    FROM sales AS sl
    JOIN members AS mem ON mem.customer_id = sl.customer_id
    JOIN menu AS men ON men.product_id = sl.product_id
    WHERE sl.order_date < mem.join_date
), last_purchase_before_membership AS (
    SELECT 
        customer_id,
        order_date,
        product_name
    FROM ordered_before_joining
    WHERE order_rank = 1
)
SELECT * FROM last_purchase_before_membership;

```
 
> 8. What is the total items and amount spent for each member before they became a member?

```

SELECT 
    sl.customer_id,
    COUNT(sl.product_id) AS total_ordered,
    SUM(men.price) AS total_spent
FROM sales AS sl
JOIN members AS mem 
    ON sl.customer_id = mem.customer_id
JOIN menu AS men 
    ON men.product_id = sl.product_id
WHERE sl.order_date < mem.join_date
GROUP BY sl.customer_id
ORDER BY sl.customer_id;

```

> 9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

```

WITH allocating_points AS (
    SELECT 
        sl.customer_id,
        product_name, 
        SUM(men.price) AS spent_amnt,
        CASE 
            WHEN product_name = "sushi" THEN SUM(men.price) * 10 * 2 
            ELSE SUM(men.price) * 10 
        END AS points
    FROM sales AS sl
    JOIN menu AS men ON men.product_id = sl.product_id
    GROUP BY 1, 2
)
SELECT 
    customer_id, 
    SUM(points) AS points 
FROM allocating_points
GROUP BY customer_id;

```

> 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items,
> not just sushi - how many points do customer A and B have at the end of January?

```

SELECT 
    sl.customer_id, 
    SUM(men.price * 2) AS points 
FROM sales AS sl 
JOIN members AS mem 
ON sl.customer_id = mem.customer_id 
JOIN menu AS men 
ON sl.product_id = men.product_id 
WHERE 
    sl.order_date >= mem.join_date 
    AND DATEDIFF(sl.order_date, mem.join_date) <= 7 
    AND MONTH(sl.order_date) = 1 
GROUP BY 1 
ORDER BY 1;

```

> Combinig all the DataSets and checking they're **membership status** for each order.
```

SELECT 
    sl.customer_id, 
    sl.order_date, 
    men.product_name, 
    men.price,
    CASE 
        WHEN mem.customer_id = sl.customer_id AND sl.order_date >= mem.join_date 
            THEN 'Y' 
            ELSE 'N' 
    END AS members
FROM sales AS sl 
LEFT JOIN members AS mem 
ON sl.customer_id = mem.customer_id
LEFT JOIN menu AS men 
ON men.product_id = sl.product_id;

``` 






