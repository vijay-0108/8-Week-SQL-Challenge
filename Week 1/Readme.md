# 📌 Week 1 - [Challenge Title]  

![image](https://github.com/user-attachments/assets/b3864f87-fe6c-495e-9ba6-bf43b6de32da)

 
## 📝 Problem Statement  
Danny wants us to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

He plans on using these insights to help him decide whether he should expand the existing customer loyalty program - additionally he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.

Danny has provided you with a sample of his overall customer data due to privacy issues - but he hopes that these examples are enough for you to write fully functioning SQL queries to help him answer his questions!

***

## 📊 Available Datasets  

### Sales Table
- **customer_id, order_date, product_id**  

### Menu Table
- **product_id, product_name, price**  

### Members Table
- **customer_id, join_date**  


***

### 🔗 ER Diagram  
Below is the **Entity Relationship (ER) Diagram** for the dataset:  

![image](https://github.com/user-attachments/assets/541b2464-52dc-4f73-81c1-8a541b3cd344)

***

## 📜 Queries & Solutions  

### 1. What is the total amount each customer spent at the restaurant?

```` SQL
select sales.customer_id ,
       sum(menu.price) as total_amount_spent
from sales 
left join menu on sales.product_id = menu.product_id group by customer_id;
````
### Answer
![image](https://github.com/user-attachments/assets/22c72ee1-dcea-4e85-a0a6-b3cb410bb4ea)

***

### 2. How many days has each customer visited the restaurant?

```` SQL
select customer_id , 
       count(order_date) as no_of_visited_days
from sales  group by customer_id;
````
### Answer
![image](https://github.com/user-attachments/assets/4131b08b-9f7e-4f88-9fd3-aec3643fda80)

***

### 3. What was the first item from the menu purchased by each customer?

```` SQL
select e.customer_id , e.product_id , e.order_date , m.product_name 
from (select 
	  customer_id , product_id , order_date ,
	  dense_rank() over(partition by customer_id order by order_date) as ord_date
	  from sales) as e
left join menu as m 
using(product_id)
where ord_date=1
order by customer_id;
````
### Answer
![image](https://github.com/user-attachments/assets/a8a88263-cfb3-452b-8a1f-32e42c16db24)

***

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

```` SQL
select product_name , count(product_id) as ordered_count 
from sales
left join menu using(product_id)
group by product_name
order by ordered_count desc
limit 1;
````
### Answer
![image](https://github.com/user-attachments/assets/e31c8079-7d27-4a3a-88b0-296a9795f423)

***

### 5. Which item was the most popular for each customer?
```` SQL
select 
  customer_id, 
  product_name as customer_fav_item, 
  favorite_item_count
from (
  select 
    sales.customer_id, 
    product_name, 
    count(sales.product_id) as favorite_item_count,  
    dense_rank() over (partition by sales.customer_id order by count(sales.product_id) desc) as rank_order
  from sales
  left join menu using(product_id)
  group by sales.customer_id, product_name
) as ranked_items
where rank_order = 1;
````
### Answer
![image](https://github.com/user-attachments/assets/8baff77e-67fc-4079-bbf6-6da325e5e8b0)

***
























---

### ✅ Next Steps  
- Continue refining SQL queries  
- Document key findings  
- Move on to Week 2 🚀  
