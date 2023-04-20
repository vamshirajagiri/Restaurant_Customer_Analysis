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
The **sales table** captures all customer_id level purchases with an corresponding _order_date_ and _product_id_ information for when and what menu items were ordered.
![demo_data](https://user-images.githubusercontent.com/108252662/233407002-a9a4452f-5944-41e7-ac92-ab59332f872f.png)

## Table 2: menu
The **menu table** maps the _product_id_ to the actual _product_name_ and _price_ of each menu item.
![demo_data2](https://user-images.githubusercontent.com/108252662/233407615-f40d8713-8ae2-4084-b605-3b30cfc72101.png)

## Table 3: members
The final **members table** captures the _join_date_ when a _customer_id_ joined the beta version of the  loyalty program.
![demo_data3](https://user-images.githubusercontent.com/108252662/233408291-4ea1a04a-e4f5-4627-897e-1d6837336bce.png)
