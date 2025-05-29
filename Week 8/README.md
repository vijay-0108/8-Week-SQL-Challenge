# 📌 Case Study #8 - Fresh Segments

![image](https://github.com/user-attachments/assets/0b0c4fc0-5a69-4e3b-9453-a429995bacf5)

## 📌 Challenge Overview  
This case study is part of the **[8-Week SQL Challenge](https://8weeksqlchallenge.com)** by **Danny Ma**.  
Check out the original challenge details [here](https://8weeksqlchallenge.com/case-study-8/).  
 
## 📝 Introduction  
Danny created Fresh Segments, a digital marketing agency that helps other businesses analyse trends in online ad click behaviour for their unique customer base.

Clients share their customer lists with the Fresh Segments team who then aggregate interest metrics and generate a single dataset worth of metrics for further analysis.

In particular - the composition and rankings for different interests are provided for each client showing the proportion of their customer list who interacted with online assets related to each interest for each month.

Danny has asked for your assistance to analyse aggregated metrics for an example client and provide some high level insights about the customer list and their interests.

***

## 📊 Available Datasets  

### Interest Metrics
- **_month, _year, month_year,	interest_id, composition, index_value, ranking, percentile_ranking**  

### Interest Map
- **id, interest_name, interest_summary, created_at, last_modifie**


***


## 📜 Questions & Solutions  

## A. Data Exploration and Cleansing

### Question 1
### 1. Update the fresh_segments.interest_metrics table by modifying the month_year column to be a date data type with the start of the month

```` SQL
ALTER TABLE fresh_segments.interest_metrics
ADD COLUMN month_year_temp DATE;

UPDATE fresh_segments.interest_metrics
SET month_year_temp = STR_TO_DATE(CONCAT(TRIM(month_year), '-01'), '%m-%Y-%d')
WHERE month_year IS NOT NULL AND month_year <> 'NULL';

ALTER TABLE fresh_segments.interest_metrics
DROP COLUMN month_year;

ALTER TABLE fresh_segments.interest_metrics
CHANGE COLUMN month_year_temp month_year DATE;
````
### Answer
![image](https://github.com/user-attachments/assets/6afeefa9-c43c-4d20-886f-e9f2e7b35684)

***

### Question 2
### 2. What is count of records in the fresh_segments.interest_metrics for each month_year value sorted in chronological order (earliest to latest) with the null values appearing first?

```` SQL
select month_year , count(*) as count_of_records
from metrics
group by month_year
order by month_year asc;
````
### Answer
![image](https://github.com/user-attachments/assets/d6469216-1a1e-4e20-bd3c-5c0cb17eaa0b)

***

### Question 3
### 3. What do you think we should do with these null values in the fresh_segments.interest_metrics

```` SQL

````
### Answer


***

### Question 4
### 4. How many interest_id values exist in the fresh_segments.interest_metrics table but not in the fresh_segments.interest_map table? What about the other way around?

```` SQL
SELECT 
    (SELECT 
            COUNT(DISTINCT m.interest_id)
        FROM
            fresh_segments.metrics m
                LEFT JOIN
            fresh_segments.map im ON m.interest_id = im.id
        WHERE
            im.id IS NULL) AS metrics_only_count,
    (SELECT 
            COUNT(DISTINCT im.id)
        FROM
            fresh_segments.map im
                LEFT JOIN
            fresh_segments.metrics m ON im.id = m.interest_id
        WHERE
            m.interest_id IS NULL) AS map_only_count;
````
### Answer

### There is 1 interest_id in the fresh_segments.interest_metrics table that does not have a corresponding id in the fresh_segments.interest_map table.
### There are 7 id values in the fresh_segments.interest_map table that do not have a corresponding interest_id in the fresh_segments.interest_metrics table.

![image](https://github.com/user-attachments/assets/79596068-a185-497e-85dc-bc5bd9cf7825)

***

### Question 5
### 5. Summarise the id values in the fresh_segments.interest_map by its total record count in this table
```` SQL
select count(id) as id_count from map;

select id , count(*)
from map
group by id;
````
### Answer
![image](https://github.com/user-attachments/assets/9db95c20-c241-481e-8804-d7714ec0ca88)

***

![image](https://github.com/user-attachments/assets/e5637a7b-74d9-4efe-ad8b-68eeb46b9bc5)

***

### Question 6
### 6. What sort of table join should we perform for our analysis and why? Check your logic by checking the rows where interest_id = 21246 in your joined output and include all columns from fresh_segments.interest_metrics and all columns from fresh_segments.interest_map except from the id column
```` SQL
SELECT _month, _year, interest_id, composition, index_value, ranking, percentile_ranking, month_year ,interest_name, interest_summary, created_at, last_modified
FROM METRICS 
JOIN MAP ON METRICS.INTEREST_ID = MAP.ID
WHERE INTEREST_ID = 21246 AND _month >1;
````
### Answer
![image](https://github.com/user-attachments/assets/97559a96-6304-4862-8235-3faba7cf6d62)

***

### Question 7
### 7. Are there any records in your joined table where the month_year value is before the created_at value from the fresh_segments.interest_map table? Do you think these values are valid and why?
```` SQL
select _month , interest_id, composition, index_value, ranking, percentile_ranking, month_year , interest_name, interest_summary, created_at, last_modified
from map inner join metrics on map.id = metrics.interest_id
where month_year<created_at;

select count(*) as record_count
from map inner join metrics on map.id = metrics.interest_id
where month_year<created_at;
````
### Answer
### There are 188 such records where month_year value is before the created_at value from the fresh_segments.interest_map table. These records are valid cause we initially set the day to 1 for every month.
![image](https://github.com/user-attachments/assets/e72eab4c-6cd3-49cd-b04f-f02e8c0936ab)
***
![image](https://github.com/user-attachments/assets/11b9e12a-e4aa-4ab9-b46d-7ce17e0da834)

***

## B. Interest Analysis

### Question 1
### 1. Which interests have been present in all month_year dates in our dataset?

```` SQL
with x as(
select interest_id , count(distinct month_year) as all_months
from metrics 
group by interest_id
having count(distinct month_year)=(select count(distinct month_year) from metrics))
select interest_id , m.interest_name 
from x left join map as m on x.interest_id = m.id;
````
### Answer
### There are 480 records present in all month_year dates
![image](https://github.com/user-attachments/assets/f18571ed-97f0-488a-8bf0-5778ef7aa8b4)

***

### Question 2
### 2. Using this same total_months measure - calculate the cumulative percentage of all records starting at 14 months - which total_months value passes the 90% cumulative percentage value?

```` SQL
with x as(
select interest_id , count(distinct month_year) as all_months
from metrics 
group by interest_id) , y as(
select all_months , count(interest_id) as interest_count
from x
group by all_months) , z as(
select all_months , 
	   interest_count , 
       (sum(interest_count) over(order by all_months desc rows between unbounded preceding and current row)/sum(interest_count)over(rows between unbounded preceding and unbounded following)) *100 as cummilative
from y
group by all_months)
select *
from z
where cummilative >90;
````
### Answer
![image](https://github.com/user-attachments/assets/f9a2e059-0957-4b7b-92d8-738b9dfbedf3)

***

### Question 3
### 3. If we were to remove all interest_id values which are lower than the total_months value we found in the previous question - how many total data points would we be removing?

```` SQL
with x as(
select interest_id , count(distinct month_year) as month_count
from metrics
group by interest_id
having count(distinct month_year)<=6)
select count(distinct interest_id)
from x;
````
### Answer
![image](https://github.com/user-attachments/assets/2c2a69a3-f70f-4214-9ff8-fc97c51f4209)

***

### Question 4
### 4. Does this decision make sense to remove these data points from a business perspective? Use an example where there are all 14 months present to a removed interest example for your arguments - think about what it means to have less months present from a segment perspective.

```` SQL

````
### Answer


***

### Question 5
### 5. After removing these interests - how many unique interests are there for each month?
```` SQL
with v as(
select interest_id , month_year
from metrics
where interest_id not in(with x as(
select interest_id , count(distinct month_year) as month_count
from metrics
group by interest_id
having count(distinct month_year)<=6)
select interest_id
from x))
select month_year , count(interest_id) as remaining_count
from v
group by month_year;
````
### Answer
![image](https://github.com/user-attachments/assets/a97dfe03-2824-4694-8920-21f395afee40)

***

## C. Segment Analysis

### Question 1
### 1. Using our filtered dataset by removing the interests with less than 6 months worth of data, which are the top 10 and bottom 10 interests which have the largest composition values in any month_year? Only use the maximum composition value for each interest but you must keep the corresponding month_year

### TOP 10
```` SQL
with v as(
select interest_id , month_year
from metrics
where interest_id not in
(with x as(
select interest_id , count(distinct month_year) as month_count
from metrics
group by interest_id
having count(distinct month_year)<=6)
select interest_id
from x)) , i as(
select v.interest_id , composition  , v.month_year
from v
left join metrics on v.interest_id = metrics.interest_id and v.month_year = metrics.month_year) , j as(
select * , dense_rank() over(order by composition desc) as high_composition
from i)
select *
from j
having high_composition<=10;
````

### Bottom 10
```` SQL
with v as(
select interest_id , month_year
from metrics
where interest_id not in
(with x as(
select interest_id , count(distinct month_year) as month_count
from metrics
group by interest_id
having count(distinct month_year)<=6)
select interest_id
from x)) , i as(
select v.interest_id , composition  , v.month_year
from v
left join metrics on v.interest_id = metrics.interest_id and v.month_year = metrics.month_year) , j as(
select * , dense_rank() over(order by composition asc) as low_composition
from i)
select *
from j
having high_composition<=10;
````
### Answer
![image](https://github.com/user-attachments/assets/14c1579c-4094-4f9b-8d57-af88405309e3)
***
![image](https://github.com/user-attachments/assets/c43f313a-5d7b-4fff-b49b-1d3271119952)

***

### Question 2
### 2. Which 5 interests had the lowest average ranking value?

```` SQL
with x as(
select interest_id , avg(ranking) as ranking
from metrics 
group by interest_id)
select interest_id , ranking
from x
order by ranking asc
limit 5;
````
### Answer
![image](https://github.com/user-attachments/assets/6b9b9c4c-c141-428c-8e42-12840c3070e3)

***

### Question 3
### 3. Which 5 interests had the largest standard deviation in their percentile_ranking value?

```` SQL
SELECT
    interest_id,
    STDDEV_SAMP(percentile_ranking) AS percentile_ranking_std_dev
FROM
    metrics 
GROUP BY
    interest_id
HAVING
    COUNT(percentile_ranking) >= 1 
ORDER BY
    percentile_ranking_std_dev DESC
LIMIT 5;
````
### Answer
![image](https://github.com/user-attachments/assets/3ce8c399-0d79-4c22-9fce-263bc842cff0)

***

### Question 4
### 4. For the 5 interests found in the previous question - what was minimum and maximum percentile_ranking values for each interest and its corresponding year_month value? Can you describe what is happening for these 5 interests?

```` SQL
with x as(
SELECT
    interest_id,
    STDDEV_SAMP(percentile_ranking) AS percentile_ranking_std_dev
FROM
    metrics 
GROUP BY
    interest_id
HAVING
    COUNT(percentile_ranking) >= 1 
ORDER BY
    percentile_ranking_std_dev DESC
LIMIT 5) , y as(
select x.interest_id , percentile_ranking , month_year
from metrics right join x using(interest_id))
select distinct interest_id , min(percentile_ranking) over(partition by interest_id) as min_percentile , max(percentile_ranking) over(partition by interest_id) as max_percentile
from y;
````
### Answer
![image](https://github.com/user-attachments/assets/b91256a7-7a0a-45ff-bc5d-e0e166f7fcf3)

***

### Question 5
### 5. How would you describe our customers in this segment based off their composition and ranking values? What sort of products or services should we show to these customers and what should we avoid?


### Answer

***


## D. Index Analysis

### The index_value is a measure which can be used to reverse calculate the average composition for Fresh Segments’ clients.
### Average composition can be calculated by dividing the composition column by the index_value column rounded to 2 decimal places.

### Question 1
### 1. What is the top 10 interests by the average composition for each month?

```` SQL
with x as(
select interest_id , avg(composition) as avg_comp, month_year as month_number
from metrics
group by month_year , interest_id) ,
y as(
select interest_id , avg_comp , month_number , dense_rank() over(partition by month_number order by avg_comp desc) as ranking
from x)
select * 
from y 
where ranking<=10 and month_number is not null;
````
### Answer
![image](https://github.com/user-attachments/assets/1deee044-205a-4e3f-be36-29a193e8c432)

***

### Question 2
### 2. For all of these top 10 interests - which interest appears the most often?

```` SQL
WITH x AS (
    -- Calculate average composition for each interest_id per month
    SELECT
        interest_id,
        AVG(composition) AS avg_comp,
        month_year AS month_number
    FROM
        metrics
    GROUP BY
        month_year,
        interest_id
),
y AS (
    -- Rank interests by avg_comp within each month
    SELECT
        interest_id,
        avg_comp,
        month_number,
        DENSE_RANK() OVER (PARTITION BY month_number ORDER BY avg_comp DESC) AS monthly_ranking
    FROM
        x
),
z AS (
    -- Filter for top 10 interests in each month
    SELECT
        interest_id,
        avg_comp,
        month_number,
        monthly_ranking
    FROM
        y
    WHERE
        monthly_ranking <= 10
        AND month_number IS NOT NULL
),
v AS (
    -- Count how many times each interest_id appeared in the monthly top 10
    SELECT
        interest_id,
        COUNT(interest_id) AS monthly_top10_appearances
    FROM
        z
    GROUP BY
        interest_id
),
i AS (
    -- Rank interests based on their total appearances in the monthly top 10
    SELECT
        interest_id,
        monthly_top10_appearances,
        DENSE_RANK() OVER (ORDER BY monthly_top10_appearances DESC) AS overall_top10_rank -- Renamed for clarity
    FROM
        v -- CORRECTED: Should be from 'v'
)
-- Final selection: Top 10 interests by their overall appearances in monthly top 10
SELECT
    interest_id,
    monthly_top10_appearances
FROM
    i
WHERE
    overall_top10_rank <= 10
ORDER BY
    monthly_top10_appearances DESC;
````
### Answer
![image](https://github.com/user-attachments/assets/86065a9a-80df-46fa-8ab5-3228887c4424)

***

### Question 3
### 3. What is the average of the average composition for the top 10 interests for each month?

```` SQL
with x as(
select interest_id , avg(composition) as avg_comp, month_year as month_number
from metrics
group by month_year , interest_id) ,
y as(
select interest_id , avg_comp , month_number , dense_rank() over(partition by month_number order by avg_comp desc) as ranking
from x) , z as(
select * 
from y 
where ranking<=10 and month_number is not null)
select  month_number , avg(avg_comp) as avg_of_avg_comp
from z
group by month_number;
````
### Answer
![image](https://github.com/user-attachments/assets/9bb1c88c-aea0-485c-9021-6d116a279915)

***

### Question 4
### 4. What is the 3 month rolling average of the max average composition value from September 2018 to August 2019 and include the previous top ranking interests in the same output shown below.

```` SQL

````
### Answer

***

### Question 5
### 5. Provide a possible reason why the max average composition might change from month to month? Could it signal something is not quite right with the overall business model for Fresh Segments?

```` SQL

````
### Answer

***

