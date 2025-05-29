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

````
### Answer
![image](https://github.com/user-attachments/assets/88d48416-a2c4-4153-bef2-7010687e7acd)

***

### Question 2
### 2. What is count of records in the fresh_segments.interest_metrics for each month_year value sorted in chronological order (earliest to latest) with the null values appearing first?

```` SQL
  
````
### Answer
![image](https://github.com/user-attachments/assets/b9b6f57d-d4ec-4e6e-b7a3-05b7c6ae51d3)

***

### Question 3
### 3. What do you think we should do with these null values in the fresh_segments.interest_metrics

```` SQL

````
### Answer
![image](https://github.com/user-attachments/assets/f44982c6-b69e-4229-a001-1bb090eb3e12)

***

### Question 4
### 4. How many interest_id values exist in the fresh_segments.interest_metrics table but not in the fresh_segments.interest_map table? What about the other way around?

```` SQL

````
### Answer
![image](https://github.com/user-attachments/assets/bf16d052-15a1-4b1f-b46b-2ee3a6fa5bb0)

***

### Question 5
### 5. Summarise the id values in the fresh_segments.interest_map by its total record count in this table
```` SQL

````
### Answer
![image](https://github.com/user-attachments/assets/8e554b86-d59a-4fa0-a445-56348651878a)

***

### Question 6
### 6. What sort of table join should we perform for our analysis and why? Check your logic by checking the rows where interest_id = 21246 in your joined output and include all columns from fresh_segments.interest_metrics and all columns from fresh_segments.interest_map except from the id column
```` SQL
 
````
### Answer
![image](https://github.com/user-attachments/assets/d1f3eb78-e1a4-49a9-a459-d44a617e37f7)

***

### Question 7
### 7. Are there any records in your joined table where the month_year value is before the created_at value from the fresh_segments.interest_map table? Do you think these values are valid and why?
```` SQL

````
### Answer


***

## B. Interest Analysis

### Question 1
### 1. Which interests have been present in all month_year dates in our dataset?

```` SQL

````
### Answer
![image](https://github.com/user-attachments/assets/7bb7746c-5d70-4ef3-aedd-44f8d4a041d8)

***

### Question 2
### 2. Using this same total_months measure - calculate the cumulative percentage of all records starting at 14 months - which total_months value passes the 90% cumulative percentage value?

```` SQL
  
````
### Answer
![image](https://github.com/user-attachments/assets/cc8d04dc-6dc3-4ac2-961d-4937542bd79c)

***

### Question 3
### 3. If we were to remove all interest_id values which are lower than the total_months value we found in the previous question - how many total data points would we be removing?

--- we can see a relationship between the no of pizza's in an order to the time taken to complete the order
```` SQL

````
### Answer

***

### Question 4
### 4. Does this decision make sense to remove these data points from a business perspective? Use an example where there are all 14 months present to a removed interest example for your arguments - think about what it means to have less months present from a segment perspective.

```` SQL

````
### Answer
![image](https://github.com/user-attachments/assets/887bf57c-1494-4b5f-874e-7f612b010d69)

***

### Question 5
### 5. After removing these interests - how many unique interests are there for each month?
```` SQL

````
### Answer
![image](https://github.com/user-attachments/assets/4e0d42ff-c568-42ce-b4f7-10862ed0d52d)

***


## C. Segment Analysis

### Question 1
### 1. Using our filtered dataset by removing the interests with less than 6 months worth of data, which are the top 10 and bottom 10 interests which have the largest composition values in any month_year? Only use the maximum composition value for each interest but you must keep the corresponding month_year

```` SQL

````
### Answer
![image](https://github.com/user-attachments/assets/f166f7f9-03c7-4dad-8d18-6a8700fba11e)

***

### Question 2
### 2. Which 5 interests had the lowest average ranking value?

```` SQL

````
### Answer
![image](https://github.com/user-attachments/assets/6e815ff7-b74a-4ee6-8c9b-d4f988e88975)

***

### Question 3
### 3. Which 5 interests had the largest standard deviation in their percentile_ranking value?

```` SQL

````
### Answer
![image](https://github.com/user-attachments/assets/c98a0b8b-fb46-4b93-9870-b9b07302a872)

***

### Question 4
### 4. For the 5 interests found in the previous question - what was minimum and maximum percentile_ranking values for each interest and its corresponding year_month value? Can you describe what is happening for these 5 interests?

```` SQL

````
### Answer

***

### Question 5
### 5. How would you describe our customers in this segment based off their composition and ranking values? What sort of products or services should we show to these customers and what should we avoid?

```` SQL

````
### Answer

***


## D. Index Analysis

### The index_value is a measure which can be used to reverse calculate the average composition for Fresh Segments’ clients.
### Average composition can be calculated by dividing the composition column by the index_value column rounded to 2 decimal places.

### Question 1
### 1. What is the top 10 interests by the average composition for each month?

```` SQL

````
### Answer
![image](https://github.com/user-attachments/assets/f166f7f9-03c7-4dad-8d18-6a8700fba11e)

***

### Question 2
### 2. For all of these top 10 interests - which interest appears the most often?

```` SQL

````
### Answer
![image](https://github.com/user-attachments/assets/6e815ff7-b74a-4ee6-8c9b-d4f988e88975)

***

### Question 3
### 3. What is the average of the average composition for the top 10 interests for each month?

```` SQL

````
### Answer
![image](https://github.com/user-attachments/assets/c98a0b8b-fb46-4b93-9870-b9b07302a872)

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

