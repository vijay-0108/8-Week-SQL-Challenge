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

## B. Runner and Customer Experience

### Question 1
### 1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)?

```` SQL
select 
	case when registration_date between '2021-01-01' and '2021-01-07' then 'week1'
		 when registration_date between '2021-01-08' and '2021-01-14' then 'week2'
         when registration_date between '2021-01-15' and '2021-01-21' then 'week3'
         end as registration_week,
	count(runner_id) as registered_runner_count
from runners
group by registration_week;
````
### Answer
![image](https://github.com/user-attachments/assets/7bb7746c-5d70-4ef3-aedd-44f8d4a041d8)

***

### Question 2
### 2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

```` SQL
with initial_table as(
	select 
		order_id , 
        customer_id , 
        runner_id , 
        order_time , 
        pickup_time
	from customer_orders_temp
	left join runner_orders using(order_id))
select 
	runner_id,
        minute(sec_to_time(avg(time_to_sec(timediff(pickup_time,order_time))))) as average_time_per_runnerinminutes
	from initial_table
group by runner_id;  
````
### Answer
![image](https://github.com/user-attachments/assets/cc8d04dc-6dc3-4ac2-961d-4937542bd79c)

***

### Question 3
### 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?

--- we can see a relationship between the no of pizza's in an order to the time taken to complete the order
```` SQL

````
### Answer

***

### Question 4
### 4. What was the average distance travelled for each customer?

```` SQL
select  round(avg(r.distance),2) as distance , c.customer_id 
from runner_orders as r
left join customer_orders_temp as c using(order_id)
where distance !='null'
group by c.customer_id;
````
### Answer
![image](https://github.com/user-attachments/assets/887bf57c-1494-4b5f-874e-7f612b010d69)

***

### Question 5
### 5. What was the difference between the longest and shortest delivery times for all orders?
```` SQL
select max(duration) - min(duration) as difference
from runner_orders
where duration>=1;
````
### Answer
![image](https://github.com/user-attachments/assets/4e0d42ff-c568-42ce-b4f7-10862ed0d52d)

***

### Question 6
### 6. What was the average speed for each runner for each delivery and do you notice any trend for these values?
```` SQL
select order_id , runner_id , avg(round(distance/(duration/60) , 2)) as speed_in_kmph
from runner_orders
group by runner_id , order_id
order by runner_id , order_id;
````
### Answer
![image](https://github.com/user-attachments/assets/b4194cd6-6df7-49c9-a725-75a18af164a6)

***

### Question 7
### 7. What is the successful delivery percentage for each runner?
```` SQL
with x as(
select runner_id  , case when cancellation ='cancelled' or  cancellation ='cancelled'then 0  else 1 end as success
from runner_orders
order by runner_id)
select runner_id , sum(success)/count(success)*100 as succesfull_delivery_percentage
from x 
group by runner_id;
````
### Answer
![image](https://github.com/user-attachments/assets/18af37a5-e9ce-4367-aad6-43dd89ab5ed1)

***

## C. Ingredient Optimisation

### Question 1
### 1. What are the standard ingredients for each pizza?

```` SQL
select * from pizza_recipes;
````
### Answer
![image](https://github.com/user-attachments/assets/f166f7f9-03c7-4dad-8d18-6a8700fba11e)

***

### Question 2
### 2. What was the most commonly added extra?

```` SQL
WITH RECURSIVE numbers AS (
  SELECT 1 AS n
  UNION ALL
  SELECT n + 1 FROM numbers WHERE n < 10
), y as(
SELECT 
  c.order_id,
  SUBSTRING_INDEX(SUBSTRING_INDEX(c.extras, ',', n), ',', -1) AS extra
FROM customer_orders_temp AS c
LEFT JOIN LATERAL (
  SELECT n FROM numbers
  WHERE n <= LENGTH(c.extras) - LENGTH(REPLACE(c.extras, ',', '')) + 1
) AS x ON TRUE)
select extra , count(extra) as topping_count , topping_name
from y left join pizza_toppings on y.extra = pizza_toppings.topping_id
where extra>0
group by extra , topping_name
order by topping_count desc
limit 1;
````
### Answer
![image](https://github.com/user-attachments/assets/6e815ff7-b74a-4ee6-8c9b-d4f988e88975)

***

### Question 3
### 3. What was the most common exclusion?

```` SQL
WITH RECURSIVE numbers AS (
  SELECT 1 AS n
  UNION ALL
  SELECT n + 1 FROM numbers WHERE n < 10
), y as(
SELECT 
  c.order_id,
  SUBSTRING_INDEX(SUBSTRING_INDEX(c.exclusions, ',', n), ',', -1) AS exclusion
FROM customer_orders_temp AS c
LEFT JOIN LATERAL (
  SELECT n FROM numbers
  WHERE n <= LENGTH(c.exclusions) - LENGTH(REPLACE(c.exclusions, ',', '')) + 1
) AS x ON TRUE)
select exclusion , count(exclusion) as topping_count , topping_name
from y left join pizza_toppings on y.exclusion = pizza_toppings.topping_id
where exclusion>0
group by exclusion , topping_name
order by topping_count desc
limit 1;
````
### Answer
![image](https://github.com/user-attachments/assets/c98a0b8b-fb46-4b93-9870-b9b07302a872)

***

### Question 4
### 4. Generate an order item for each record in the customers_orders table in the format of one of the following:
### Meat Lovers
### Meat Lovers - Exclude Beef
### Meat Lovers - Extra Bacon
### Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers

```` SQL

````
### Answer

***

### Question 5
### 5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
### For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"

```` SQL

````
### Answer

***

### Question 6
### 6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?
```` SQL

````
### Answer

***
