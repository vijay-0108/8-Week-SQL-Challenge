# 📌 Week 1 - [Challenge Title]  

![image](https://github.com/user-attachments/assets/b3864f87-fe6c-495e-9ba6-bf43b6de32da)

 
## 📝 Problem Statement  
Danny wants us to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

He plans on using these insights to help him decide whether he should expand the existing customer loyalty program - additionally he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.

Danny has provided you with a sample of his overall customer data due to privacy issues - but he hopes that these examples are enough for you to write fully functioning SQL queries to help him answer his questions!

***

## 📊 Available Datasets  

### Sales Table
- **customer_id, order_date, product_id**  

### Menu Table
- **product_id, product_name, price**  

### Members Table
- **customer_id, join_date**  


***

### 🔗 ER Diagram  
Below is the **Entity Relationship (ER) Diagram** for the dataset:  

![image](https://github.com/user-attachments/assets/541b2464-52dc-4f73-81c1-8a541b3cd344)

***

## 📜 Queries & Solutions  

### 1. What is the total amount each customer spent at the restaurant?

```` SQL
select sales.customer_id ,
       sum(menu.price) as total_amount_spent
from sales 
left join menu on sales.product_id = menu.product_id group by customer_id;
````
### Answer
![image](https://github.com/user-attachments/assets/22c72ee1-dcea-4e85-a0a6-b3cb410bb4ea)

***

---

### ✅ Next Steps  
- Continue refining SQL queries  
- Document key findings  
- Move on to Week 2 🚀  
