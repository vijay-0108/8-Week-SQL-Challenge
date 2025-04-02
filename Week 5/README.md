# 📌 Case Study #5 - Data Mart

![image](https://github.com/user-attachments/assets/60a97a28-cf80-4a12-a19e-cb69ac6c3bb2)

## 📌 Challenge Overview  
This case study is part of the **[8-Week SQL Challenge](https://8weeksqlchallenge.com)** by **Danny Ma**.  
Check out the original challenge details [here](https://8weeksqlchallenge.com/case-study-5/).  
 
## 📝 Introduction 
Data Mart is Danny’s latest venture and after running international operations for his online supermarket that specialises in fresh produce - Danny is asking for your support to analyse his sales performance.

In June 2020 - large scale supply changes were made at Data Mart. All Data Mart products now use sustainable packaging methods in every single step from the farm all the way to the customer.

Danny needs your help to quantify the impact of this change on the sales performance for Data Mart and it’s separate business areas.

The key business question he wants you to help him answer are the following:

What was the quantifiable impact of the changes introduced in June 2020?
Which platform, region, segment and customer types were the most impacted by this change?
What can we do about future introduction of similar sustainability updates to the business to minimise impact on sales?

***

## 📌 Table of Contents

- [Introduction](#introduction)  
- [ER Diagram](#er-diagram)  
- [Available Datasets](#available-datasets)  
- [Questions and Solutions](#questions-and-solutions)  
  - [A. Customer Nodes Exploration](#a-customer-nodes-exploration)
    - [Question 1](#question-1)
    - [Question 2](#question-2)
    - [Question 3](#question-3)
    - [Question 4](#question-4)
  - [B. Customer Transactions](#b-customer-transactions)
    - [Question 1](#question-1-1)
    - [Question 2](#question-2-1)
    - [Question 3](#question-3-1)
    - [Question 4](#question-4-1)
    - [Question 5](#question-5)
  - [C. Data Allocation Challenge](#c-data-allocation-challenge)
    - [Question 1](#question-1-2)
    - [Question 3](#question-3-2)
- [Next Steps](#next-steps)

***

## 📊 Available Datasets  

### Weekly Sales 
- **week_date, region, platform, segment, customer_type, transactions, sales**  

***

### 🔗 ER Diagram  
Below is the **Entity Relationship (ER) Diagram** for the dataset:  

![image](https://github.com/user-attachments/assets/d2dd9537-f0ea-46ae-b9fd-8d3545351642)

***

## 📜 Questions & Solutions  

## A. Data Cleansing Steps

### Question 1
### In a single query, perform the following operations and generate a new table in the `data_mart` schema named `clean_weekly_sales`:
- Convert the `week_date` to a `DATE` format
- Add a `week_number` as the second column for each `week_date` value, for example any value from the 1st of January to 7th of January will be 1, 8th to 14th will be 2 etc
- Add a `month_number` with the calendar month for each `week_date` value as the 3rd column
- Add a `calendar_year` column as the 4th column containing either 2018, 2019 or 2020 values
- Add a new column called `age_band` after the original segment column using the following mapping on the number inside the segment value
  
<img width="166" alt="image" src="https://user-images.githubusercontent.com/81607668/131438667-3b7f3da5-cabc-436d-a352-2022841fc6a2.png">
  
- Add a new `demographic` column using the following mapping for the first letter in the `segment` values:  

| segment | demographic | 
| ------- | ----------- |
| C | Couples |
| F | Families |

- Ensure all `null` string values with an "unknown" string value in the original `segment` column as well as the new `age_band` and `demographic` columns
- Generate a new `avg_transaction` column as the sales value divided by transactions rounded to 2 decimal places for each record

```` SQL
ALTER TABLE data_mart.weekly_sales ADD COLUMN week_date_new DATE;
alter table weekly_sales add column week_number int;
alter table weekly_sales add column month_number int;
alter table weekly_sales add column year_number int;
ALTER TABLE data_mart.weekly_sales ADD COLUMN age_band VARCHAR(255);
ALTER TABLE data_mart.weekly_sales ADD COLUMN demographic VARCHAR(255); 
ALTER TABLE data_mart.weekly_sales ADD COLUMN avg_transaction DECIMAL(10, 2);
alter table weekly_sales add column updated_segment varchar(255);
UPDATE data_mart.weekly_sales 
SET week_date_new = STR_TO_DATE(week_date, '%d/%m/%y'),
    week_number = WEEK(week_date_new),
    month_number = MONTH(week_date_new),
    year_number = YEAR(week_date_new),
    age_band = CASE 
                  WHEN segment LIKE '%1' THEN 'Young Adults'
                  WHEN segment LIKE '%2' THEN 'Middle Aged'
                  WHEN segment LIKE '%3' OR segment LIKE '%4' THEN 'Retirees'
                  ELSE 'Unknown'
               END,
    demographic = CASE 
                    WHEN segment LIKE 'c%' THEN 'Couples'
                    WHEN segment LIKE 'f%' THEN 'Families'
                    ELSE 'Unknown'
                 END,
    avg_transaction = ROUND(sales / transactions, 2),
    updated_segment  = case when segment='null' then 'unknown'
							else segment
                            end;
````
### Answer
![image](https://github.com/user-attachments/assets/37c7b4a6-9c5d-4ff4-ae35-d2d8648a9e7c)

***

## B. Data Exploration

### Question 1
### 1. What day of the week is used for each week_date value?

```` SQL
select dayofweek(week_date_new) as dow
from clean_weekly_sales;
````
### Answer
![image](https://github.com/user-attachments/assets/95028c77-184a-4467-a20a-28308beba5b2)
- Monday has been used

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
-- ![image](https://github.com/user-attachments/assets/4b2b3ff3-2272-43e2-9097-df3a0d7e6bde)
-- By using AVG_TRANSACTIONS 
-- ![image](https://github.com/user-attachments/assets/8b8ebd15-1826-45a2-b0d1-cf76d31d7ee6)
-- By Regular Method 

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
- Move on to [Week 6](https://github.com/vijay-0108/8-Week-SQL-Challenge/blob/main/Week%206/README.md) 🚀  

