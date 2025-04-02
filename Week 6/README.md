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
![image](https://github.com/user-attachments/assets/58cdc0fc-2d08-4961-bbdf-96768d5c4eff)

***

### Question 2
### 2. What range of week numbers are missing from the dataset?

```` SQL
select distinct(week_number)
from clean_weekly_sales
order by week_number;
````
### Answer
![image](https://github.com/user-attachments/assets/2c71573a-077b-4bcb-b99d-ef064be0fba6)
- week numbers from 1-11 and from 36-52 are missing

***

### Question 3
### 3. How many total transactions were there for each year in the dataset?

```` SQL
select count(transactions) as transaction_count
from clean_weekly_sales
group by year_number;
````
### Answer
![image](https://github.com/user-attachments/assets/0c56430f-7e3e-4d9a-a448-46c9d873b683)

***

### Question 4
### 4. What is the total sales for each region for each month?
```` SQL
select region , month_number , sum(sales) as total_sales
from clean_weekly_sales
group by region , month_number;
````
### Answer
![image](https://github.com/user-attachments/assets/54053215-486d-4249-b863-63ca2312eca6)

***

### Question 5
### 5. What is the total count of transactions for each platform?
```` SQL
select platform , count(transactions) as transactions_count_per_platform
from clean_weekly_sales
group by platform;
````
### Answer
![image](https://github.com/user-attachments/assets/22bcf5b0-deb9-4e0b-8c80-0b3d4ffd4836)

***

### Question 6
### 6. What is the percentage of sales for Retail vs Shopify for each month?
```` SQL
with x as(
select year_number , month_number,
sum(case when platform='retail' then sales end) as retail_sales,
sum(case when platform='shopify' then sales end) as shopify_sales
from clean_weekly_sales
group by year_number , month_number)
select year_number , month_number , round(((retail_sales/(retail_sales+shopify_sales))*100),2) as retail_sales_percent , round(((shopify_sales/(retail_sales+shopify_sales))*100),2) as shopify_sales_percent
from x
order by year_number , month_number;
````
### Answer
![image](https://github.com/user-attachments/assets/02ee6e9c-bffc-4b1a-937b-872194002adc)

***

### Question 7
### 7. What is the percentage of sales by demographic for each year in the dataset?
```` SQL
with x as(
select year_number , 
sum(case when demographic='couples' then sales end) as couples_sales,
sum(case when demographic='families' then sales end) as familes_sales,
sum(case when demographic='unknown' then sales end) as unknown_sales
from clean_weekly_sales
group by year_number)
select year_number , 
round(((couples_sales/(couples_sales+familes_sales+unknown_sales))*100),2) as couples_sales_percent,
round(((familes_sales/(couples_sales+familes_sales+unknown_sales))*100),2) as families_sales_percent,
round(((unknown_sales/(couples_sales+familes_sales+unknown_sales))*100),2) as unknown_sales_percent
from x
group by year_number;
````
### Answer
![image](https://github.com/user-attachments/assets/c26ee171-c258-4e2f-b716-4dc32cb746cb)

***

### Question 8
### 8. Which age_band and demographic values contribute the most to Retail sales?
```` SQL
with x as(
select age_band , demographic , sum(sales) as contribution
from clean_weekly_sales
where platform='retail'
group by age_band , demographic)
select * 
from x 
where contribution in (select max(contribution) from x);
````
### Answer
![image](https://github.com/user-attachments/assets/ffd85ead-3cb0-46bd-b65b-e8782f07c859)

***

### Question 9
### 9. Can we use the avg_transaction column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?
```` SQL
select year_number , platform , avg(avg_transaction) as avg_trans                       -- by using avg_transaction column there is a difference compared to regular method
from clean_weekly_sales
group by year_number , platform;

select year_number , platform , round(sum(sales)/sum(transactions),2) as avg_trans      -- by regular method
from clean_weekly_sales
group by year_number , platform;
````
### Answer
![image](https://github.com/user-attachments/assets/4b2b3ff3-2272-43e2-9097-df3a0d7e6bde) By using AVG_TRANSACTIONS 
![image](https://github.com/user-attachments/assets/8b8ebd15-1826-45a2-b0d1-cf76d31d7ee6) By Regular Method 

***

## C. Before & After Analysis

### This technique is usually used when we inspect an important event and want to inspect the impact before and after a certain point in time.
-- Taking the `week_date` value of `2020-06-15` as the baseline week where the Data Mart sustainable packaging changes came into effect. We would include all `week_date` values for `2020-06-15` as the start of the period after the change and the previous week_date values would be before. Using this analysis approach - answer the following questions:

### Question 1
### 1. What is the total sales for the 4 weeks before and after 2020-06-15? What is the growth or reduction rate in actual values and percentage of sales?
```` SQL
CREATE VIEW new_te AS
WITH after_sales AS (
    SELECT SUM(sales) AS total_after_sales
    FROM (
        SELECT sales, DENSE_RANK() OVER (ORDER BY week_date_new) AS ranking
        FROM clean_weekly_sales
        WHERE week_date_new >= '2020-06-15'
    ) ranked
    WHERE ranking <= 4
), 
before_sales AS (
    SELECT SUM(sales) AS total_before_sales
    FROM (
        SELECT sales, DENSE_RANK() OVER (ORDER BY week_date_new DESC) AS ranking
        FROM clean_weekly_sales
        WHERE week_date_new < '2020-06-15'
    ) ranked
    WHERE ranking <= 4
)
SELECT total_after_sales, total_before_sales
FROM after_sales, before_sales;

SELECT total_after_sales - total_before_sales AS sales_variance,
       ROUND((total_before_sales / (total_before_sales + total_after_sales)) * 100, 2) -
       ROUND((total_after_sales / (total_before_sales + total_after_sales)) * 100, 2) AS percent_variance
FROM new_te;
````
### Answer
![image](https://github.com/user-attachments/assets/1e0d7a97-74ce-493c-be22-79b2be7fb60f)

***

### Question 2
### 2. What about the entire 12 weeks before and after?

```` SQL
CREATE VIEW new_tep_tab AS
WITH after_sales AS (
    SELECT SUM(sales) AS total_after_sales
    FROM (
        SELECT sales, DENSE_RANK() OVER (ORDER BY week_date_new) AS ranking
        FROM clean_weekly_sales
        WHERE week_date_new >= '2020-06-15'
    ) ranked
    WHERE ranking <= 12
), 
before_sales AS (
    SELECT SUM(sales) AS total_before_sales
    FROM (
        SELECT sales, DENSE_RANK() OVER (ORDER BY week_date_new DESC) AS ranking
        FROM clean_weekly_sales
        WHERE week_date_new < '2020-06-15'
    ) ranked
    WHERE ranking <= 12
)
SELECT total_after_sales, total_before_sales
FROM after_sales, before_sales;

SELECT total_after_sales - total_before_sales AS sales_variance,
       ROUND((total_before_sales / (total_before_sales + total_after_sales)) * 100, 2) -
       ROUND((total_after_sales / (total_before_sales + total_after_sales)) * 100, 2) AS percent_variance
FROM new_tep_tab;
````
### Answer
![image](https://github.com/user-attachments/assets/054ffa1e-eaf8-43a8-bdf2-3b6e07822f9d)

***

### ✅ Next Steps  
- Continue refining SQL queries   
- Move on to [Week 7](https://github.com/vijay-0108/8-Week-SQL-Challenge/blob/main/Week%207/README.md) 🚀  


