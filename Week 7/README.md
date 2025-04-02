# 📌 Case Study #4 - Data Bank

![image](https://github.com/user-attachments/assets/3c1c7b4f-f72d-4e6a-ac6e-a9f4a58d6eed)

## 📌 Challenge Overview  
This case study is part of the **[8-Week SQL Challenge](https://8weeksqlchallenge.com)** by **Danny Ma**.  
Check out the original challenge details [here](https://8weeksqlchallenge.com/case-study-7/).  
 
## 📝 Introduction 
Balanced Tree Clothing Company prides themselves on providing an optimised range of clothing and lifestyle wear for the modern adventurer!

Danny, the CEO of this trendy fashion company has asked you to assist the team’s merchandising teams analyse their sales performance and generate a basic financial report to share with the wider business.

***

## 📊 Available Datasets  

### Product Details
- **product_id, price, product_name, category_id, segment_id, style_id, category_name, segment_name, style_name**  

### Product Hierarchy
- **id, parent_id, level_text, level_name**

### Product Prices
- **id, product_id, price**

### Product Prices
- **prod_id, qty, price, discount, member, txn_id, start_txn_time**


***

## 📜 Questions & Solutions  

## A. High Level Sales Analysis

### Question 1
### 1. What was the total quantity sold for all products?

```` SQL
select sum(qty) as total_quantity_sold
from sales;
````
### Answer
![image](https://github.com/user-attachments/assets/ca51c30c-d861-4268-8328-d5a6b8eb578e)

***

### Question 2
### 2. What is the total generated revenue for all products before discounts?

```` SQL
select sum(qty*price) as total_trans_price_before_discount
from sales;
````
### Answer
![image](https://github.com/user-attachments/assets/abac3c9c-c678-483d-a994-041b6aaa7714)

***

### Question 3
### 3. What was the total discount amount for all products?

```` SQL
select  SUM((qty * price) - (((qty * price) / 100) * (100 - discount))) as total_discount
from sales;
````
### Answer
![image](https://github.com/user-attachments/assets/893d62f6-3fdf-4ed9-b89f-9a23494d5026)

***

## B. Transaction Analysis

### Question 1
### 1. How many unique transactions were there?

```` SQL
select count(distinct txn_id) as distinct_transactions
from sales;
````
### Answer
![image](https://github.com/user-attachments/assets/33b83de7-c0fe-44b7-bd90-153bc81c90a9)

***

### Question 2
### 2. What is the average unique products purchased in each transaction?

```` SQL
with x as(
select txn_id , count(distinct prod_id) as unique_prod_trans
from sales
group by txn_id)
select avg(unique_prod_trans) as avg_unique_prod_trans
from x;
````
### Answer
![image](https://github.com/user-attachments/assets/d7d6e905-ad03-4f9c-aaea-922042739ddb)

***

### Question 3
### 3. What are the 25th, 50th and 75th percentile values for the revenue per transaction?

```` SQL
````
### Answer


***

### Question 4
### 4. What is the average discount value per transaction?
```` SQL
with x as(
SELECT txn_id , SUM((qty * price) - (((qty * price) / 100) * (100 - discount)) ) AS discount_value 
FROM sales 
GROUP BY txn_id)
select avg(discount_value) as avg_discount_value
from x;
````
### Answer
![image](https://github.com/user-attachments/assets/a51684ff-7d06-438f-b9d9-f28762be7815)

***

### Question 5
### What is the percentage split of all transactions for members vs non-members?
```` SQL
with x as(
select count(case when member='t' then 1 end) as member_count,
	   count(case when member='f' then 1 end) as non_member_count
from sales)
select (member_count/(member_count+non_member_count)) * 100 as member_percent , (non_member_count/(member_count+non_member_count)) * 100 as non_member_percent 
from x;
````
### Answer
![image](https://github.com/user-attachments/assets/58534a7b-e065-44a5-8bc6-4ad1a6d1f78b)

***

### Question 6
### What is the average revenue for member transactions and non-member transactions?

```` SQL
select member , avg((((qty * price) / 100) * (100 - discount))) as revenue
from sales
group by member;
````

### Answer

![image](https://github.com/user-attachments/assets/c053f967-fd3b-4b34-9e83-85c70261d939)

***

## C. Product Analysis

### Question 1
### 1. What are the top 3 products by total revenue before discount?

```` SQL
with x as(
select prod_id , sum(price*qty) as total_revenue
from sales
group by prod_id)
select prod_id , total_revenue
from x
order by total_revenue desc
limit 3;
````
### Answer
![image](https://github.com/user-attachments/assets/df46b8ef-d071-48ac-bee9-2794efe5dec6)

***

### Question 2
### 2. What is the total quantity, revenue and discount for each segment?

```` SQL
with x as(
select prod_id , segment_name ,  qty ,  sales.price , discount , member , txn_id , start_txn_time 
from sales 
left join product_details on sales.prod_id = product_details.product_id)
select segment_name , sum(qty) as total_qty , sum((((qty * price) / 100) * (100 - discount))) as total_revenue , SUM((qty * price) - (((qty * price) / 100) * (100 - discount))) as total_discount
from x
group by segment_name;
````
### Answer
![image](https://github.com/user-attachments/assets/25c47b9a-9fcc-4f9c-b25f-2a44828e807d)

***

### Question 3
### 3. What is the top selling product for each segment?

```` SQL
-- in terms of revenue 
with x as(
select prod_id , product_name , segment_name ,  qty ,  sales.price , discount , member , txn_id , start_txn_time 
from sales 
left join product_details on sales.prod_id = product_details.product_id),
y as(
select segment_name , product_name ,  sum((((qty * price) / 100) * (100 - discount))) as total_revenue 
from x
group by 2 , 1),
z as(
select segment_name , product_name , total_revenue , dense_rank() over(partition by segment_name order by total_revenue desc) as ranking
from y)
select * from z where ranking = 1;

-- in terms of quantity
with x as(
select prod_id , product_name , segment_name ,  qty ,  sales.price , discount , member , txn_id , start_txn_time 
from sales 
left join product_details on sales.prod_id = product_details.product_id),
y as(
select segment_name , product_name ,  sum(qty) as total_qty 
from x
group by 2 , 1),
z as(
select segment_name , product_name , total_qty , dense_rank() over(partition by segment_name order by total_qty desc) as ranking
from y)
select * from z where ranking = 1;
````
### Answer
## In Terms Of Revenue
![image](https://github.com/user-attachments/assets/df0d7abf-8972-4681-886f-10788101efd6)
## In Terms Of Category
![image](https://github.com/user-attachments/assets/2a74575c-0392-4ef7-890f-d342c2fabde1)

***

### Question 4
### 4. What is the total quantity, revenue and discount for each category?

```` SQL
with x as(
select prod_id , category_name ,  qty ,  sales.price , discount , member , txn_id , start_txn_time 
from sales 
left join product_details on sales.prod_id = product_details.product_id)
select category_name, sum(qty) as total_qty , sum((((qty * price) / 100) * (100 - discount))) as total_revenue , SUM((qty * price) - (((qty * price) / 100) * (100 - discount))) as total_discount
from x 
group by category_name;
````
### Answer
![image](https://github.com/user-attachments/assets/77a59ab8-77d0-4b8b-a874-ebba4e896445)

***

### Question 5
### 5. What is the top selling product for each category?

```` SQL
-- in terms of revenue
with x as(
select prod_id , product_name , category_name ,  qty ,  sales.price , discount , member , txn_id , start_txn_time 
from sales 
left join product_details on sales.prod_id = product_details.product_id),
y as(
select category_name , product_name ,  sum((((qty * price) / 100) * (100 - discount))) as total_revenue 
from x
group by 2 , 1),
z as(
select category_name , product_name , total_revenue , dense_rank() over(partition by category_name order by total_revenue desc) as ranking
from y)
select * from z where ranking = 1;

-- in terms of quantity
with x as(
select prod_id , product_name , category_name ,  qty ,  sales.price , discount , member , txn_id , start_txn_time 
from sales 
left join product_details on sales.prod_id = product_details.product_id),
y as(
select category_name , product_name ,  sum(qty) as total_qty 
from x
group by 2 , 1),
z as(
select category_name , product_name , total_qty , dense_rank() over(partition by category_name order by total_qty desc) as ranking
from y)
select * from z where ranking = 1;
````
### Answer
## In Terms Of Revenue
![image](https://github.com/user-attachments/assets/5e3fffd3-6244-4848-9a6b-8c95a5e3df8e)
## In Terms Of Category
![image](https://github.com/user-attachments/assets/311df3fa-41fd-4a9a-9579-3add2a59a45f)

***

### Question 6
### 6. What is the percentage split of revenue by product for each segment?

```` SQL
SELECT 
    segment_name, 
    product_name, 
    (SUM(((qty * sales.price) / 100) * (100 - discount)) / SUM(SUM(((qty * sales.price) / 100) * (100 - discount))) OVER ()) * 100 AS revenue_percentage
FROM sales
LEFT JOIN product_details ON sales.prod_id = product_details.product_id
GROUP BY segment_name, product_name;
````
### Answer
![image](https://github.com/user-attachments/assets/18a97296-ebf1-48e2-a3d5-c7a955f21235)

***

### Question 7
### 7. What is the percentage split of revenue by segment for each category?

```` SQL
SELECT 
    segment_name, 
    category_name, 
    (SUM(((qty * sales.price) / 100) * (100 - discount)) / SUM(SUM(((qty * sales.price) / 100) * (100 - discount))) OVER ()) * 100 AS revenue_percentage
FROM sales
LEFT JOIN product_details ON sales.prod_id = product_details.product_id
GROUP BY 2 , 1;
````
### Answer
![image](https://github.com/user-attachments/assets/25470cab-ae02-4e13-8547-1044959d9c07)

***

### Question 8
### 8. What is the percentage split of total revenue by category?

```` SQL
SELECT 
    category_name, 
    (SUM(((qty * sales.price) / 100) * (100 - discount)) / SUM(SUM(((qty * sales.price) / 100) * (100 - discount))) OVER ()) * 100 AS revenue_percentage
FROM sales
LEFT JOIN product_details ON sales.prod_id = product_details.product_id
GROUP BY  1;

````
### Answer
![image](https://github.com/user-attachments/assets/b3da989e-0ca7-499f-9d33-6970a59afdbf)

***

### Question 9
### 9. What is the total transaction “penetration” for each product? (hint: penetration = number of transactions where at least 1 quantity of a product was purchased divided by total number of transactions)

```` SQL
select prod_id , count(case when qty>=1 then txn_id end)/(select count(distinct txn_id) from sales)*100 as penetration
from sales
group by prod_id;
````
### Answer
![image](https://github.com/user-attachments/assets/b7f8322b-d4c2-445f-b399-332677a0b979)

***

### Question 10
### 10. What is the most common combination of at least 1 quantity of any 3 products in a 1 single transaction?

```` SQL

````
### Answer

***

### ✅ Next Steps  
- Continue refining SQL queries   
- Move on to [Week 5](https://github.com/vijay-0108/8-Week-SQL-Challenge/blob/main/Week%205/README.md) 🚀  

