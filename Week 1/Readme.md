# 📌 Case Study #1 - Danny's Diner

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

### 6. Which item was purchased first by the customer after they became a member?
```` SQL
select 
	customer , 
    first_order_after_member , 
    joined_date , 
    after_member.product_id , 
    menu.product_name 
from(select sales.customer_id as customer, 
     		sales.order_date as first_order_after_member, 
     		members.join_date as joined_date, 
     		sales.product_id,
	 		dense_rank() over(partition by sales.customer_id  order by sales.order_date asc) as min_date
	from sales
	inner join members using(customer_id)
	where order_date>join_date) as after_member
inner join menu on after_member.product_id = menu.product_id
where min_date=1
order by customer;
````
### Answer
![image](https://github.com/user-attachments/assets/cb0b4a2e-0f75-4058-ab6a-2afbcde60348)

***

### 7. Which item was purchased just before the customer became a member?
```` SQL
select 
	customer , 
    last_ordered_date_before_member , 
    joined_date , 
    before_member.product_id , 
    menu.product_name 
from(select sales.customer_id as customer, 
     		sales.order_date as last_ordered_date_before_member, 
     		members.join_date as joined_date, 
     		sales.product_id,
	 		dense_rank() over(partition by sales.customer_id  order by sales.order_date desc) as min_date
	from sales
	inner join members using(customer_id)
	where order_date<join_date) as before_member
inner join menu on before_member.product_id = menu.product_id
where min_date=1
order by customer;
````
### Answer
![image](https://github.com/user-attachments/assets/ac6163d1-b33c-4a19-84b9-1095f9353d4f)

***

### 8. What is the total items and amount spent for each member before they became a member?
```` SQL
select
	customer_id , 
    count(product_id) as itrem_ordered_count ,
    sum(price) as total_ordered_price
from(select 
		sales.customer_id , 
    	sales.order_date , 
    	sales.product_id, 
    	members.join_date
	from sales 
	left join members using(customer_id)
	where order_date<join_date) as question
inner join menu using(product_id)
group by customer_id;
````
### Answer
![image](https://github.com/user-attachments/assets/26214dce-5fd3-457f-ad20-6865e6220618)

***

### 9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
```` SQL
select 
	customer_id,
    sum(multiplier) as total_points
from(select 
		sales.customer_id,
    	sales.product_id,
    	menu.price,
    	case 
    		when product_id ='1' then price*20
        	else price*10
        	end as multiplier
	from sales
	left join menu using(product_id)) as initial_multiplier
 group by customer_id;
````
### Answer
![image](https://github.com/user-attachments/assets/fff44f4f-65bd-4027-ba7b-e26ce19aba56)

***

### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

```` SQL
with initial as
(select 
	customer_id , 
    order_date , 
    product_id , 
    join_date , 
    price *10 as spent,
    date_add(join_date, interval 6 day) as first_week
from sales
left join menu using(product_id)
join members using(customer_id)),
total_points as(
select *,
case when order_date between join_date and first_week then 2 
	 when product_id=1 then 2
     else 1
     end as points
from initial
where order_date<='2021-01-31')
select customer_id , sum(spent*points) as points
from total_points
group by customer_id;
````
### Answer
![image](https://github.com/user-attachments/assets/d62d73f8-e53f-4b4a-9791-9f57f7691580)

***

###  Bonus Question 1.
### The following questions are related creating basic data tables that Danny and his team can use to quickly derive insights without needing to join the underlying tables using SQL. Recreate the following table output using the available data:
```` SQL
with initial as(
select 
	customer_id , 
    order_date , 
    product_name , 
    join_date , 
    price,
    date_add(join_date, interval 7 day) as first_week
from sales
left join menu using(product_id)
left join members using(customer_id))
select 
	customer_id ,
    order_date, 
    product_name , 
    price,
    case when order_date >= join_date then 'y'
    else 'N'
    end as member
from initial;
````
### Answer
![image](https://github.com/user-attachments/assets/a32f99c8-6d61-4d83-9365-38a4b297020d)

---

### ✅ Next Steps  
- Continue refining SQL queries   
- Move on to Week 2 🚀  
