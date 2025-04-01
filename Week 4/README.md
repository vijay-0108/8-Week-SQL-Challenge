# 📌 Case Study #4 - Data Bank

![image](https://github.com/user-attachments/assets/4d303f9a-cfaf-405c-9b7b-216575aa92ff)

## 📌 Challenge Overview  
This case study is part of the **[8-Week SQL Challenge](https://8weeksqlchallenge.com)** by **Danny Ma**.  
Check out the original challenge details [here](https://8weeksqlchallenge.com/case-study-4/).  
 
## 📝 Introduction 
There is a new innovation in the financial industry called Neo-Banks: new aged digital only banks without physical branches.

Danny thought that there should be some sort of intersection between these new age banks, cryptocurrency and the data world…so he decides to launch a new initiative - Data Bank!

Data Bank runs just like any other digital bank - but it isn’t only for banking activities, they also have the world’s most secure distributed data storage platform!

Customers are allocated cloud data storage limits which are directly linked to how much money they have in their accounts. There are a few interesting caveats that go with this business model, and this is where the Data Bank team need your help!

The management team at Data Bank want to increase their total customer base - but also need some help tracking just how much data storage their customers will need.

This case study is all about calculating metrics, growth and helping the business analyse their data in a smart way to better forecast and plan for their future developments!

***

## 📌 Table of Contents

- [Introduction](#introduction)  
- [ER Diagram](#er-diagram)  
- [Available Datasets](#available-datasets)  
- [Questions and Solutions](#questions-and-solutions)  
- [Question 1](#Question-1)
- [Question 2](#Question-2)
- [Question 3](#Question-3)
- [Question 4](#Question-4)
- [Question 5](#Question-5)
- [Question 6](#Question-6)
- [Question 7](#Question-7)
- [Question 8](#Question-8)
- [Question 9](#Question-9)
- [Question 10](#Question-10)
***

## 📊 Available Datasets  

### Regions Table
- **region_id, region_name**  

### Customer Nodes
- **customer_id, region_id, node_id, start_date, end_date**  

### Customer Transactions
- **customer_id, txn_date, txn_type, txn_amount**  


***

### 🔗 ER Diagram  
Below is the **Entity Relationship (ER) Diagram** for the dataset:  

![image](https://github.com/user-attachments/assets/13582b09-3440-4ea8-9444-ef0475054808)

***

## 📜 Questions & Solutions  

## A. Customer Nodes Exploration

### Question 1
### 1. How many unique nodes are there on the Data Bank system?

```` SQL
select count(distinct node_id) as unique_node_count
from customer_nodes;
````
### Answer
![image](https://github.com/user-attachments/assets/1af5f61c-e3c9-4678-a555-cd119350d22a)

***

### Question 2
### 2. What is the number of nodes per region?

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
### 3. How many customers are allocated to each region?

```` SQL
select region_id ,
       count(distinct customer_id) as customer_count_per_region
from customer_nodes
group by region_id;
````
### Answer
![image](https://github.com/user-attachments/assets/c92ffca1-d804-43e2-bb01-1ef5b32c2da1)

***

### Question 4
### 4. How many days on average are customers reallocated to a different node?
```` SQL
select node_id , avg(datediff(end_date , start_date)) as timediff
from customer_nodes
where end_date<>'9999-12-31'
group by node_id;
````
### Answer
![image](https://github.com/user-attachments/assets/00b78eca-babc-4dec-a173-7074a0e1937e)

***

## B. Customer Transactions

### Question 1
### 1. What is the unique count and total amount for each transaction type?

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
### 2. What is the average total historical deposit counts and amounts for all customers?

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
### 3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?

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
### 4. What is the closing balance for each customer at the end of the month?
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
### 5. What is the percentage of customers who increase their closing balance by more than 5%?
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

## C. Data Allocation Challenge

### All the three questions have been solved using the given data

### Question 1
### 1. running customer balance column that includes the impact each transaction

```` SQL
with x as(
select * , 
case when txn_type<>'deposit' then -1*txn_amount
else txn_amount end as initial
from customer_transactions)
select * ,sum(initial) over(partition by customer_id order by txn_date rows between unbounded preceding and current row) as running_transaction
from x;
````
### Answer
![image](https://github.com/user-attachments/assets/ea33f8d4-e399-4a20-9475-9f83262aa017)

***

### Question 3
### 3. minimum, average and maximum values of the running balance for each customer

```` SQL
with x as(
select * , 
case when txn_type<>'deposit' then -1*txn_amount
else txn_amount end as initial
from customer_transactions), y as(
select customer_id , txn_date , txn_type , txn_amount ,sum(initial) over(partition by customer_id order by txn_date rows between unbounded preceding and current row) as running_transaction
from x)
select distinct customer_id , min(running_transaction) over(partition by customer_id order by txn_date rows between unbounded preceding and unbounded following) as min_running_sum,
max(running_transaction) over(partition by customer_id order by txn_date rows between unbounded preceding and unbounded following)as max_running_sum,
avg(running_transaction) over(partition by customer_id order by txn_date rows between unbounded preceding and unbounded following)as avg_running_sum
from y;
````
### Answer
![image](https://github.com/user-attachments/assets/91483218-9729-41b3-b677-dcfd6415c213)

***

### ✅ Next Steps  
- Continue refining SQL queries   
- Move on to Week 5 🚀  
