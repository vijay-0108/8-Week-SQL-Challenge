# 📌 Case Study #3 - Foodie-Fi

![image](https://github.com/user-attachments/assets/2c822ced-ad9a-4647-b7fd-965261ef2147)

## 📌 Challenge Overview  
This case study is part of the **[8-Week SQL Challenge](https://8weeksqlchallenge.com)** by **Danny Ma**.  
Check out the original challenge details [here](https://8weeksqlchallenge.com/case-study-3/).  
 
## 📝 Introduction  
Subscription based businesses are super popular and Danny realised that there was a large gap in the market - he wanted to create a new streaming service that only had food related content - something like Netflix but with only cooking shows!

Danny finds a few smart friends to launch his new startup Foodie-Fi in 2020 and started selling monthly and annual subscriptions, giving their customers unlimited on-demand access to exclusive food videos from around the world!

Danny created Foodie-Fi with a data driven mindset and wanted to ensure all future investment decisions and new features were decided using data. This case study focuses on using subscription style digital data to answer important business questions.

***

## 📊 Available Datasets  

### Plans Table
- **plan_id, plan_name, price**  

### Subscriptions Table
- **customer_id, plan_id, start_date**  

***

## 🔗 ER Diagram  
Below is the **Entity Relationship (ER) Diagram** for the dataset:  

![image](https://github.com/user-attachments/assets/e44e9092-1512-42fc-bec6-e594d1ffee3b)

***

## 📜 Questions & Solutions  

## B. Data Analysis Questions

### Question 1
### 1. How many customers has Foodie-Fi ever had?

```` SQL
select count(distinct customer_id) as customers_count
from subscriptions;
````
### Answer
![image](https://github.com/user-attachments/assets/f8a49763-6db9-4b41-bc88-a81e862681d5)


***

### Question 2
### 2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value?

```` SQL
select count(customer_id) as distribution,
       month(start_date) as month
from plans
right join subscriptions using(plan_id)
where plan_name='trial'
group by month(start_date);;
````
### Answer
![image](https://github.com/user-attachments/assets/c3c8925f-3464-43c5-bb8b-8021beaebed8)

***

### Question 3
### 3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name?

```` SQL
select count(customer_id) as event_count , plan_name
from subscriptions
left join plans using(plan_id)
where year(start_date)>2020
group by plan_name;
````
### Answer
![image](https://github.com/user-attachments/assets/a32e42b8-2dff-4c7a-a95e-5e31e7ad0234)

***

### Question 4
### 4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place??

```` SQL
SELECT 
    (COUNT(DISTINCT CASE WHEN plan_name = 'churn' THEN customer_id END) * 1.0) / 
    COUNT(DISTINCT customer_id)*100 AS churn_rate
FROM subscriptions
LEFT JOIN plans USING (plan_id);
````
### Answer
![image](https://github.com/user-attachments/assets/06fc7f37-4636-46cc-80b2-c8d924ee9dda)

***

### Question 5
### 5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number??
```` SQL
with x as(
select customer_id  , plan_name , start_date
from subscriptions
left join plans using(plan_id)
order by customer_id , start_date),
y as(
select customer_id ,  plan_name  ,
case when plan_name = 'trial' then lead(plan_name) over(partition by customer_id) else null end as next_subscription
from x)
select count(*) as churn_after_trail
from y
where plan_name='trial' and next_subscription='churn';
````
### Answer
![image](https://github.com/user-attachments/assets/1b16b7ee-23a5-48ae-9e19-c30dd6473ea8)

***

### Question 6
### 6. What is the number and percentage of customer plans after their initial free trial??
```` SQL
with x as
(select customer_id , plan_id , case when plan_id='0' then lead(plan_id) over(partition by customer_id) else 'no plan after inital trial' end as next_plan
from subscriptions)
select count(next_plan) as next_plan_count , plan_name
from x left join plans on x.next_plan=plans.plan_id
where next_plan<>'no plan after inital trial'
group by plan_name;  
````
### Answer
![image](https://github.com/user-attachments/assets/62ffc53e-901d-49b1-bfc4-4f8006710c4a)

***

### Question 7
### 7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?
```` SQL
with x as(
select distinct customer_id , last_value(plan_id) over(partition by customer_id rows between unbounded preceding and unbounded following) as last_val_id
from subscriptions
where start_date<='2020-12-31'), 
y as(
select count(distinct customer_id) as customer_count, last_val_id , count(distinct customer_id)/(select count(distinct customer_id) from x) * 100 as percentage
from x
group by last_val_id)
select customer_count , percentage , plan_name
from y 
left join plans on y.last_val_id = plans.plan_id;
````
### Answer
![image](https://github.com/user-attachments/assets/a76e58c0-2cf0-4168-b10f-953f5441ef05)

***

### Question 8
### 8. How many customers have upgraded to an annual plan in 2020?
```` SQL
WITH x AS (
    SELECT customer_id, plan_id, plan_name, start_date
    FROM subscriptions
    LEFT JOIN plans USING(plan_id)
    WHERE YEAR(start_date) = '2020'
    ORDER BY customer_id, plan_id
), y AS (
    SELECT *, 
           CASE 
               WHEN plan_id = 3 
               AND (LAG(plan_id) OVER(PARTITION BY customer_id) IN (1, 2, 0)) 
               THEN 1 
               ELSE 0 
           END AS upgraded_to_annual
    FROM x
)
SELECT SUM(upgraded_to_annual) AS customers_upgraded_to_annual
FROM y;
````
### Answer
![image](https://github.com/user-attachments/assets/f8b291e8-3c4e-4caf-881d-75515215f454)

***

### Question 9
### 9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?
```` SQL
with x as(
select customer_id , plans.plan_id , plan_name , start_date 
from subscriptions
left join plans using(plan_id)
order by customer_id , plan_id), y as (
select * , case when plan_name='pro annual' then first_value(start_date) over(partition by customer_id  order by customer_id , plan_id) else 'not enrolled' end as enrolled_plan_date
from x)
select  round(avg(datediff(start_date , enrolled_plan_date)),2) as annual_plan_average_time
from y
where start_date is not null and enrolled_plan_date is not null;
````
### Answer
![image](https://github.com/user-attachments/assets/20ccdff1-71da-4841-8a82-a889d27632c9)

***

### Question 10
### 10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)?

```` SQL
with x as(
select customer_id , plans.plan_id , plan_name , start_date 
from subscriptions
left join plans using(plan_id)
order by customer_id , plan_id)
, y as (
select * , 
case when plan_name='pro annual' then first_value(start_date) over(partition by customer_id  order by customer_id , plan_id) else 'not enrolled' end as enrolled_plan_date
from x)
,z as(
select customer_id , datediff(start_date , enrolled_plan_date) as average_enroll_date 
from y 
where plan_name='pro annual')
, a as(
select customer_id , average_enroll_date,
case when average_enroll_date between 0 and 30 then '0-30'
	 when average_enroll_date between 31 and 60 then '31-60'
     when average_enroll_date between 61 and 90 then '61-90'
     when average_enroll_date between 91 and 120 then '91-120'
     when average_enroll_date between 121 and 150 then '121-150'
     when average_enroll_date between 151 and 180 then '151-180'
     else 'more than 180' end as time_frame
from z)
select count(average_enroll_date) as time_frame_count, time_frame
from a
group by time_frame;
````
### Answer
![image](https://github.com/user-attachments/assets/9257f317-617a-4835-9a76-bf6c9bd873ae)

***

### Question 11
### 11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?
```` SQL
with x as(
select customer_id , plans.plan_id , plan_name , start_date 
from subscriptions
left join plans using(plan_id)
where year(start_date)='2020'
order by customer_id , plan_id) 
, y as(
select customer_id , plan_name , 
case when plan_name = 'pro monthly' and lead(plan_name) over(partition by customer_id  order by plan_id) = 'basic monthly' then 1 else 0 end as downgrade
from x)
select sum(downgrade) as downgraded_customers
from y;
````
### Answer
![image](https://github.com/user-attachments/assets/a7cadce9-984c-461f-892e-49d5eec83554)

---

### ✅ Next Steps  
- Continue refining SQL queries   
- Move on to [Week 4](https://github.com/vijay-0108/8-Week-SQL-Challenge/blob/main/Week%204/README.md) 🚀  

