# 📌 Case Study #6 - Clique Bait

![image](https://github.com/user-attachments/assets/376af17f-6b97-4870-b426-340fcacad471)

## 📌 Challenge Overview  
This case study is part of the **[8-Week SQL Challenge](https://8weeksqlchallenge.com)** by **Danny Ma**.  
Check out the original challenge details [here](https://8weeksqlchallenge.com/case-study-6/).  
 
## 📝 Introduction 
Clique Bait is not like your regular online seafood store - the founder and CEO Danny, was also a part of a digital data analytics team and wanted to expand his knowledge into the seafood industry!

In this case study - you are required to support Danny’s vision and analyse his dataset and come up with creative solutions to calculate funnel fallout rates for the Clique Bait online store.

***

## 📌 Table of Contents

- [Introduction](#📝-introduction)  
- [ER Diagram](#🔗-er-diagram)  
- [Available Datasets](#📊-available-datasets)  
- [Questions and Solutions](#📜-questions-and-solutions)  
  - [A. Data Cleansing Steps](#a-data-cleansing-steps)
    - [Question 1](#question-1)
  - [B. Data Exploration](#b-data-exploration)
    - [Question 1](#question-1-1)
    - [Question 2](#question-2-1)
    - [Question 3](#question-3-1)
    - [Question 4](#question-4-1)
    - [Question 5](#question-5)
    - [Question 6](#question-6)
    - [Question 7](#question-7)
    - [Question 8](#question-8)
    - [Question 9](#question-9)
  - [C. Before & After Analysis](#c-before--after-analysis)
    - [Question 1](#question-1-2)
    - [Question 2](#question-2-2)

***

## 📊 Available Datasets  

### Campaign Identifier 
- **campaign_id, products, campaign_name, start_date, end_date** 

### Event Identifier 
- **event_type, event_name**

### Events
- **visit_id, cookie_id, page_id, event_type, sequence_number, event_time**

### Page Hierarchy
- **page_id, page_name, product_category, product_id**

### Users
- **user_id, cookie_id, start_date**

***

## 📜 Questions & Solutions  

## B. Digital Analysis

### Question 1
### 1. How many users are there?

```` SQL
select count(distinct user_id) as users_count
from users;
````
### Answer
![image](https://github.com/user-attachments/assets/63e6a2da-a1c0-4136-b7cb-72764c95a13b)

***

### Question 2
### 2. How many cookies does each user have on average?

```` SQL
with x as(
select user_id ,
       count(cookie_id) as cookie_count
from users
group by user_id)
select avg(cookie_count) as avg_cookie_count
from x;
````
### Answer
![image](https://github.com/user-attachments/assets/cad53e22-6a25-47df-8b3a-d00141a37a1f)

***

### Question 3
### 3. What is the unique number of visits by all users per month?

```` SQL
select month(event_time) as month ,
       count( distinct visit_id) as visit_count_per_month
from events
group by  month;
````
### Answer
![image](https://github.com/user-attachments/assets/1c03aa68-e1c7-4726-9e9b-10f8d115a2b5)

***

### Question 4
### 4. What is the number of events for each event type?

```` SQL
select event_type ,
       count(event_type) as no_of_event_type
from events
group by event_type;
````
### Answer
![image](https://github.com/user-attachments/assets/13dfd989-f357-4878-b12e-29da0bd33ac0)

***

### Question 5
### 5. What is the percentage of visits which have a purchase event?

```` SQL
with x as(
select visit_id ,
       cookie_id ,
       page_id ,
       page_name ,
       event_type ,
       event_name ,
       sequence_number ,
       event_time
from events left join event_identifier using(event_type)
left join page_hierarchy using(page_id))
select (((select count(case when page_id='13' then visit_id end) as purchase_count from x)/(select count(distinct visit_id) from x))*100) as purchase_percentage;
````
### Answer
![image](https://github.com/user-attachments/assets/8eb98543-24e6-4b84-8b8d-7e8baab47d56)

***

### Question 6
### 6. What is the percentage of visits which view the checkout page but do not have a purchase event?

```` SQL
with x as(
select visit_id ,
       cookie_id ,
       user_id ,
       page_id ,
       page_name ,
       product_category ,
       event_type ,
       event_name ,
       sequence_number ,
       event_time ,
       max(page_id) over(partition by visit_id) as max_page_id
from events left join event_identifier using(event_type)
left join page_hierarchy using(page_id)
left join users using(cookie_id)) , y as(
select visit_id,
       page_name ,
       event_name ,
       max_page_id ,
case when page_name = 'checkout' and max_page_id <> '13' then 1 else 0 end as checkout_no_purchase
from x)
select ((count(case when checkout_no_purchase ='1' then 1 end))/count(case when page_name = 'checkout' then 1 end))*100 as percent
from y;
````
### Answer
![image](https://github.com/user-attachments/assets/32a2cae6-b6a9-40f2-aff0-4342cca959f6)

***

### Question 7
### 7. What are the top 3 pages by number of views?

```` SQL
select page_id ,
       count(page_id) as page_visit_count
from events
group by page_id
order by count(page_id) desc
limit 3;
````
### Answer
![image](https://github.com/user-attachments/assets/e71563a9-f2eb-4d19-9852-1912c15b59c3)

***

### Question 8
### 8. What is the number of views and cart adds for each product category?

```` SQL
with x as(
select visit_id ,
       cookie_id ,
       page_id ,
       page_name ,
       event_type ,
       event_name  ,
       product_category
from events left join event_identifier using(event_type)
left join page_hierarchy using(page_id))
select product_category ,
       count(case when event_name='page view' then 1 end) as view_count ,
       count(case when event_name = 'add to cart' then 1 end) as cart_count
from x
where page_id not in (1 , 2 , 12 , 13)
group by product_category; 
````
### Answer

![image](https://github.com/user-attachments/assets/2dbd080e-80cf-427d-921a-5c720d329f1f)

***

### Question 9
### 9. What are the top 3 products by purchases?

```` SQL
with x as(
select visit_id ,
       cookie_id ,
       page_id ,
       page_name ,
       event_type ,
       event_name ,
       product_category ,
       max(page_id) over(partition by visit_id) as max_page
from events left join event_identifier using(event_type)
left join page_hierarchy using(page_id))
select page_name ,
       count(case when event_name = 'add to cart' then 1 end) as purchase_count
from x
where page_id not in (1 , 2 , 12 ) and max_page='13'
group by page_name
order by PURCHASE_COUNT desc
limit 3;
````
### Answer
![image](https://github.com/user-attachments/assets/9cba90b9-e9bc-40ea-bf91-a80023607a12)

***

## c. Product Funnel Analysis

### Question 1
### Using a single SQL query - create a new output table which has the following details:
-- 1. How many times was each product viewed?
-- 2. How many times was each product added to cart?
-- 3. How many times was each product added to a cart but not purchased (abandoned)?
-- 4. How many times was each product purchased?

```` SQL
with x as(
select visit_id , cookie_id , page_id ,  page_name , product_category , event_type , event_name , sequence_number , event_time , max(page_id) over(partition by visit_id) as max_page_id
from events left join event_identifier using(event_type)
left join page_hierarchy using(page_id)) , 
y as(
select visit_id , cookie_id , page_id ,  page_name , product_category , event_type , event_name , sequence_number , event_time ,
 case when max_page_id='13' then 'items_purchased'
	else 'items_not_purchased'
	end as purchase_category
from x)
select page_name , 
	count(case when event_name = 'page view' then 1 end) as page_view_count , 
    count(case when event_name = 'add to cart' then 1 end) as added_to_cart_count , 
    count(case when event_name = 'add to cart' and purchase_category = 'items_not_purchased' then 1 end) as added_to_cart_not_purchased,
    count(case when event_name = 'add to cart' and purchase_category = 'items_purchased' then page_id  end) as added_to_cart_purchased
from y
where page_name not in ('home page' , 'all products' , 'checkout' , 'confirmation')
group by page_name;
````
### Answer
![image](https://github.com/user-attachments/assets/58cdc0fc-2d08-4961-bbdf-96768d5c4eff)

***

### Additionally, create another table which further aggregates the data for the above points but this time for each product category instead of individual products.

```` SQL
with x as(
select visit_id , cookie_id , page_id ,  page_name , product_category , event_type , event_name , sequence_number , event_time , max(page_id) over(partition by visit_id) as max_page_id
from events left join event_identifier using(event_type)
left join page_hierarchy using(page_id)) , 
y as(
select visit_id , cookie_id , page_id ,  page_name , product_category , event_type , event_name , sequence_number , event_time ,
 case when max_page_id='13' then 'items_purchased'
	else 'items_not_purchased'
	end as purchase_category
from x)
select product_category , 
	count(case when event_name = 'page view' then 1 end) as page_view_count , 
    count(case when event_name = 'add to cart' then 1 end) as added_to_cart_count , 
    count(case when event_name = 'add to cart' and purchase_category = 'items_not_purchased' then 1 end) as added_to_cart_not_purchased,
    count(case when event_name = 'add to cart' and purchase_category = 'items_purchased' then page_id  end) as added_to_cart_purchased
from y
where page_name not in ('home page' , 'all products' , 'checkout' , 'confirmation')
group by product_category;
````
### Answer
![image](https://github.com/user-attachments/assets/7152c739-f44c-4a27-a6a0-bb1e1e6f7b29)

***

### Use your 2 new output tables - answer the following questions:
-- Which product had the most views, cart adds and purchases?
````
-- Most Views - Oyster
-- Most Cart Adds - Lobster
-- Most Purchases - Lobster
````
-- Which product was most likely to be abandoned?
````SQL
-- Based on added to cart not purchsed and its also the least purchased item russian caviar
````
-- Which product had the highest view to purchase percentage?
````SQL
with x as(
select visit_id , cookie_id , page_id ,  page_name , product_category , event_type , event_name , sequence_number , event_time , max(page_id) over(partition by visit_id) as max_page_id
from events left join event_identifier using(event_type)
left join page_hierarchy using(page_id)) , 
y as(
select visit_id , cookie_id , page_id ,  page_name , product_category , event_type , event_name , sequence_number , event_time ,
 case when max_page_id='13' then 'items_purchased'
	else 'items_not_purchased'
	end as purchase_category
from x),
z as(
select page_name , 
	count(case when event_name = 'page view' then 1 end) as page_view_count , 
    count(case when event_name = 'add to cart' then 1 end) as added_to_cart_count , 
    count(case when event_name = 'add to cart' and purchase_category = 'items_not_purchased' then 1 end) as added_to_cart_not_purchased,
    count(case when event_name = 'add to cart' and purchase_category = 'items_purchased' then page_id  end) as added_to_cart_purchased
from y
where page_name not in ('home page' , 'all products' , 'checkout' , 'confirmation')
group by page_name)
select page_name , round(((added_to_cart_purchased/page_view_count) * 100) , 2) as view_to_percent
from z
order by view_to_percent desc
limit 1;
````
### Answer
![image](https://github.com/user-attachments/assets/09f15971-a9fe-48bc-a830-13b65fe9a183)

-- What is the average conversion rate from view to cart add?
````SQL
````
-- What is the average conversion rate from cart add to purchase?
````SQL
````

## D. Campaigns Analysis

### Question 1
### Generate a table that has 1 single row for every unique visit_id record and has the following columns:
- `user_id`
- `visit_id`
- `visit_start_time`: the earliest event_time for each visit
- `page_views`: count of page views for each visit
- `cart_adds`: count of product cart add events for each visit
- `purchase`: 1/0 flag if a purchase event exists for each visit
- `campaign_name`: map the visit to a campaign if the visit_start_time falls between the start_date and end_date
- `impression`: count of ad impressions for each visit
- `click`: count of ad clicks for each visit
- (Optional column) `cart_products`: a comma separated text value with products added to the cart sorted by the order they were added to the cart (hint: use the sequence_number)

### Answer
````SQL
````
### ✅ Next Steps  
- Continue refining SQL queries   
- Move on to [Week 7](https://github.com/vijay-0108/8-Week-SQL-Challenge/blob/main/Week%207/README.md) 🚀  


