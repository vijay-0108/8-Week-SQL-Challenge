# 📌 Case Study #2 - Pizza Runner

![image](https://github.com/user-attachments/assets/d25a0b96-603b-4c01-b6c7-7dfce8596f89)

## 📌 Challenge Overview  
This case study is part of the **[8-Week SQL Challenge](https://8weeksqlchallenge.com)** by **Danny Ma**.  
Check out the original challenge details [here](https://8weeksqlchallenge.com/case-study-2/).  
 
## 📝 Introduction  
Did you know that over 115 million kilograms of pizza is consumed daily worldwide??? (Well according to Wikipedia anyway…)

Danny was scrolling through his Instagram feed when something really caught his eye - “80s Retro Styling and Pizza Is The Future!”

Danny was sold on the idea, but he knew that pizza alone was not going to help him get seed funding to expand his new Pizza Empire - so he had one more genius idea to combine with it - he was going to Uberize it - and so Pizza Runner was launched!

Danny started by recruiting “runners” to deliver fresh pizza from Pizza Runner Headquarters (otherwise known as Danny’s house) and also maxed out his credit card to pay freelance developers to build a mobile app to accept orders from customers.

***

## 📊 Available Datasets  

### Runners
- **runner_id, registration_date**  

### Customer Orders
- **order_id, customer_id, pizza_id, exclusions, extras, order_time**

### Runner Orders
- **order_id, runner_id, pickup_time, distance, duration, cancellation**  

### Pizza Names
- **pizza_id, pizza_name**

### Pizza Recipes
- **pizza_id, toppings**  

### Pizza Toppings
- **topping_id, topping_name**  

***

## 🔗 ER Diagram  
Below is the **Entity Relationship (ER) Diagram** for the dataset:  

![image](https://github.com/user-attachments/assets/36911a41-fe10-42c3-adcd-48657bd9279e)

***

## 📜 Questions & Solutions  

## A. Pizza Metrics

### Question 1
### 1. How many pizzas were ordered?

```` SQL
select count(order_id) as total_ordered_pizzas
from customer_orders_temp;
````
### Answer
![image](https://github.com/user-attachments/assets/88d48416-a2c4-4153-bef2-7010687e7acd)

***

### Question 2
### 2. How many unique customer orders were made?

```` SQL
select count(distinct order_id) as unique_customer_orders
from customer_orders;  
````
### Answer
![image](https://github.com/user-attachments/assets/b9b6f57d-d4ec-4e6e-b7a3-05b7c6ae51d3)

***

### Question 3
### 3. How many successful orders were delivered by each runner?

```` SQL
select 
	runner_id ,
    count(order_id) as successfull_deliveries
from runner_orders
where cancellation='null'
group by runner_id;
````
### Answer
![image](https://github.com/user-attachments/assets/f44982c6-b69e-4229-a001-1bb090eb3e12)

***

### Question 4
### 4. How many of each type of pizza was delivered?

```` SQL
select 
    pizza_id,
    count(pizza_id) as succesfull_delivery
from customer_orders
left join runner_orders using(order_id)
where cancellation='null'
group by pizza_id;
````
### Answer
![image](https://github.com/user-attachments/assets/bf16d052-15a1-4b1f-b46b-2ee3a6fa5bb0)

***

### Question 5
### 5. How many Vegetarian and Meatlovers were ordered by each customer?
```` SQL
select customer_id , pizza_name as pizza_type , count(order_id) as ordered_count
from customer_orders_temp
left join pizza_names using(pizza_id)
group by pizza_name , customer_id
order by customer_id;;
````
### Answer
![image](https://github.com/user-attachments/assets/8e554b86-d59a-4fa0-a445-56348651878a)

***

### Question 6
### 6. What was the maximum number of pizzas delivered in a single order?
```` SQL
select order_id , count(order_id) as no_of_pizza_per_order
from customer_orders_temp
group by order_id;;  
````
### Answer
![image](https://github.com/user-attachments/assets/d1f3eb78-e1a4-49a9-a459-d44a617e37f7)

***

### Question 7
### 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
```` SQL

````
### Answer


***

### Question 8
### 8. How many pizzas were delivered that had both exclusions and extras?
```` SQL

````
### Answer

***

### Question 9
### 9. What was the total volume of pizzas ordered for each hour of the day??
```` SQL
select hour(order_time) as hour , count(order_id)
from customer_orders_temp
group by hour(order_time);
````
### Answer
![image](https://github.com/user-attachments/assets/154bf02b-6eb9-463b-97de-522eed0e9105)

***

### Question 10
### 10. What was the volume of orders for each day of the week?

```` SQL
select dayofweek(order_time) as dayofweek , count(order_id) as ordered_count
from customer_orders_temp
group by dayofweek(order_time);  
````
### Answer
![image](https://github.com/user-attachments/assets/1731d46c-7e03-4adb-a5d4-5c89f867c5bf)

***


