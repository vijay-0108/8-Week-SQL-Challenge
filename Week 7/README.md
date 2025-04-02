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
select count(distinct node_id) as unique_node_count
from customer_nodes;
````
### Answer
![image](https://github.com/user-attachments/assets/1af5f61c-e3c9-4678-a555-cd119350d22a)

***

### Question 2
### 2. What is the total generated revenue for all products before discounts?

```` SQL
select region_id ,
       count(node_id) as nodes_per_region
from customer_nodes
group by region_id;
````
### Answer
![image](https://github.com/user-attachments/assets/1d7fa11c-ba3e-4ff5-ae6a-be7006d3a943)

***

### Question 3
### 3. What was the total discount amount for all products?

```` SQL
select region_id ,
       count(distinct customer_id) as customer_count_per_region
from customer_nodes
group by region_id;
````
### Answer
![image](https://github.com/user-attachments/assets/c92ffca1-d804-43e2-bb01-1ef5b32c2da1)

***

## B. Transaction Analysis

### Question 1
### 1. How many unique transactions were there?

```` SQL
select txn_type ,
       count(distinct customer_id) as unique_customers,
       sum(txn_amount) as total_amount
from customer_transactions
group by txn_type;
````
### Answer
![image](https://github.com/user-attachments/assets/fda155ef-0434-4709-971a-8f3a92a89d8f)

***

### Question 2
### 2. What is the average unique products purchased in each transaction?

```` SQL
select customer_id , count(txn_type) AS historical_deposit_count , avg(txn_amount) AS historical_deposit_amount
from customer_transactions
where txn_type='deposit'
group by customer_id;
````
### Answer
![image](https://github.com/user-attachments/assets/29ce3dc2-a6f8-4ffa-8135-8d9344ca089b)

***

### Question 3
### 3. What are the 25th, 50th and 75th percentile values for the revenue per transaction?

```` SQL
with x as(
select  month(txn_date) as transaction_month, customer_id ,sum(case when txn_type='deposit' then 1 end) as deposits_count , sum(case when txn_type='purchase' then 1 end) as purchase_count , sum(case when txn_type='withdrawal' then 1 end) as withdrawal_count
from customer_transactions
group by transaction_month , customer_id
having deposits_count>1 and (purchase_count>=1 or withdrawal_count>=1))
select transaction_month , count(customer_id) as customer_count
from x
group by transaction_month;
````
### Answer
![image](https://github.com/user-attachments/assets/98c33ac4-7e6a-47a0-90d5-bc9ae7a36d2d)

***

### Question 4
### 4. What is the average discount value per transaction?
```` SQL
with x as(
select customer_id , 
	   sum(case when txn_type='deposit' then txn_amount else 0 end) as deposit_sum , 
       sum(case when txn_type='purchase' then txn_amount else 0 end) as purchase_sum , 
       sum(case when txn_type='withdrawal' then txn_amount else 0 end) as withdrawal_sum , 
       month(txn_date) as month
from customer_transactions
group by month(txn_date) , customer_id) , y as
(select customer_id , month ,  deposit_sum-purchase_sum-withdrawal_sum as end_balance_for_that_month
from x
order by customer_id , month)
select customer_id , month , sum(end_balance_for_that_month) over(partition by customer_id order by month) as closing
from y;
````
### Answer
![image](https://github.com/user-attachments/assets/43a395f6-bc91-41ed-944a-28c338d5014e)

***

### Question 5
### What is the percentage split of all transactions for members vs non-members?
```` SQL
with x as(
select * , closing*1.05 as sol
from question) , y as(
select * ,
case when month = 1 then 0
else case when closing>lag(sol) over(partition by customer_id order by month) then 1 else 0 end
end as solution
from x)
select count(case when solution=1 then 1 end)/count(*) as increase_percentage
from y;
````
### Answer
![image](https://github.com/user-attachments/assets/e0d99c6d-20a9-462e-bbe0-b9008230e27b)

***

### Question 6
### What is the average revenue for member transactions and non-member transactions?
```` SQL
with x as(
select * , closing*1.05 as sol
from question) , y as(
select * ,
case when month = 1 then 0
else case when closing>lag(sol) over(partition by customer_id order by month) then 1 else 0 end
end as solution
from x)
select count(case when solution=1 then 1 end)/count(*) as increase_percentage
from y;
````

***

## C. Product Analysis

### Question 1
### 1. What are the top 3 products by total revenue before discount?

```` SQL
select txn_type ,
       count(distinct customer_id) as unique_customers,
       sum(txn_amount) as total_amount
from customer_transactions
group by txn_type;
````
### Answer
![image](https://github.com/user-attachments/assets/fda155ef-0434-4709-971a-8f3a92a89d8f)

***

### Question 2
### 2. What is the total quantity, revenue and discount for each segment?

```` SQL
select customer_id , count(txn_type) AS historical_deposit_count , avg(txn_amount) AS historical_deposit_amount
from customer_transactions
where txn_type='deposit'
group by customer_id;
````
### Answer
![image](https://github.com/user-attachments/assets/29ce3dc2-a6f8-4ffa-8135-8d9344ca089b)

***

### Question 3
### 3. What is the top selling product for each segment?

```` SQL
with x as(
select  month(txn_date) as transaction_month, customer_id ,sum(case when txn_type='deposit' then 1 end) as deposits_count , sum(case when txn_type='purchase' then 1 end) as purchase_count , sum(case when txn_type='withdrawal' then 1 end) as withdrawal_count
from customer_transactions
group by transaction_month , customer_id
having deposits_count>1 and (purchase_count>=1 or withdrawal_count>=1))
select transaction_month , count(customer_id) as customer_count
from x
group by transaction_month;
````
### Answer
![image](https://github.com/user-attachments/assets/98c33ac4-7e6a-47a0-90d5-bc9ae7a36d2d)

***

### Question 4
### 4. What is the total quantity, revenue and discount for each category?
```` SQL
with x as(
select customer_id , 
	   sum(case when txn_type='deposit' then txn_amount else 0 end) as deposit_sum , 
       sum(case when txn_type='purchase' then txn_amount else 0 end) as purchase_sum , 
       sum(case when txn_type='withdrawal' then txn_amount else 0 end) as withdrawal_sum , 
       month(txn_date) as month
from customer_transactions
group by month(txn_date) , customer_id) , y as
(select customer_id , month ,  deposit_sum-purchase_sum-withdrawal_sum as end_balance_for_that_month
from x
order by customer_id , month)
select customer_id , month , sum(end_balance_for_that_month) over(partition by customer_id order by month) as closing
from y;
````
### Answer
![image](https://github.com/user-attachments/assets/43a395f6-bc91-41ed-944a-28c338d5014e)

***

### Question 5
### 5. What is the top selling product for each category?
```` SQL
with x as(
select * , closing*1.05 as sol
from question) , y as(
select * ,
case when month = 1 then 0
else case when closing>lag(sol) over(partition by customer_id order by month) then 1 else 0 end
end as solution
from x)
select count(case when solution=1 then 1 end)/count(*) as increase_percentage
from y;
````
### Answer
![image](https://github.com/user-attachments/assets/e0d99c6d-20a9-462e-bbe0-b9008230e27b)

***

### Question 6
### 6. What is the percentage split of revenue by product for each segment?
```` SQL
with x as(
select * , closing*1.05 as sol
from question) , y as(
select * ,
case when month = 1 then 0
else case when closing>lag(sol) over(partition by customer_id order by month) then 1 else 0 end
end as solution
from x)
select count(case when solution=1 then 1 end)/count(*) as increase_percentage
from y;
````

***

### Question 7
### 7. What is the percentage split of revenue by segment for each category?

```` SQL
with x as(
select  month(txn_date) as transaction_month, customer_id ,sum(case when txn_type='deposit' then 1 end) as deposits_count , sum(case when txn_type='purchase' then 1 end) as purchase_count , sum(case when txn_type='withdrawal' then 1 end) as withdrawal_count
from customer_transactions
group by transaction_month , customer_id
having deposits_count>1 and (purchase_count>=1 or withdrawal_count>=1))
select transaction_month , count(customer_id) as customer_count
from x
group by transaction_month;
````
### Answer
![image](https://github.com/user-attachments/assets/98c33ac4-7e6a-47a0-90d5-bc9ae7a36d2d)

***

### Question 8
### 8. What is the percentage split of total revenue by category?
```` SQL
with x as(
select customer_id , 
	   sum(case when txn_type='deposit' then txn_amount else 0 end) as deposit_sum , 
       sum(case when txn_type='purchase' then txn_amount else 0 end) as purchase_sum , 
       sum(case when txn_type='withdrawal' then txn_amount else 0 end) as withdrawal_sum , 
       month(txn_date) as month
from customer_transactions
group by month(txn_date) , customer_id) , y as
(select customer_id , month ,  deposit_sum-purchase_sum-withdrawal_sum as end_balance_for_that_month
from x
order by customer_id , month)
select customer_id , month , sum(end_balance_for_that_month) over(partition by customer_id order by month) as closing
from y;
````
### Answer
![image](https://github.com/user-attachments/assets/43a395f6-bc91-41ed-944a-28c338d5014e)

***

### Question 9
### 9. What is the total transaction “penetration” for each product? (hint: penetration = number of transactions where at least 1 quantity of a product was purchased divided by total number of transactions)
```` SQL
with x as(
select * , closing*1.05 as sol
from question) , y as(
select * ,
case when month = 1 then 0
else case when closing>lag(sol) over(partition by customer_id order by month) then 1 else 0 end
end as solution
from x)
select count(case when solution=1 then 1 end)/count(*) as increase_percentage
from y;
````
### Answer
![image](https://github.com/user-attachments/assets/e0d99c6d-20a9-462e-bbe0-b9008230e27b)

***

### Question 10
### 10. What is the most common combination of at least 1 quantity of any 3 products in a 1 single transaction?
```` SQL
````

***

### ✅ Next Steps  
- Continue refining SQL queries   
- Move on to [Week 5](https://github.com/vijay-0108/8-Week-SQL-Challenge/blob/main/Week%205/README.md) 🚀  

