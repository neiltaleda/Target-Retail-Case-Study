# Target-Retail-Case-Study
A comprehensive analysis on the sales data of an international organization

Context:

Target is a globally renowned brand and a prominent retailer in the United States. Target makes itself a preferred shopping destination by offering outstanding value, inspiration, innovation and an exceptional guest experience that no other retailer can deliver.
This particular business case focuses on the operations of Target in Brazil and provides insightful information about 100,000 orders placed between 2016 and 2018. The dataset offers a comprehensive view of various dimensions including the order status, price, payment and freight performance, customer location, product attributes, and customer reviews.
By analyzing this extensive dataset, it becomes possible to gain valuable insights into Target's operations in Brazil. The information can shed light on various aspects of the business, such as order processing, pricing strategies, payment and shipping efficiency, customer demographics, product characteristics, and customer satisfaction levels.

The data is available in 8 csv files: <br>
* customers.csv
* sellers.csv
* order_items.csv
* geolocation.csv
* payments.csv
* reviews.csv
* orders.csv
* products.csv

Dataset Schema: <br>
<img width="500" height="500" alt="image" src="https://github.com/user-attachments/assets/2442deb3-329c-4040-a359-29305e56364f" />

Problem Statement:

Assuming you are a data analyst/ scientist at Target, you have been assigned the task of analyzing the given dataset to extract valuable insights and provide actionable recommendations.

<b>What does 'good' look like?</b>

<b>1. Import the dataset and do usual exploratory analysis steps like checking the structure & characteristics of the dataset:</b><br>
1. Data type of all columns in the "customers" table.
```sql
select column_name, data_type
from scaler-dsml-sql-459505.target.INFORMATION_SCHEMA.COLUMNS
where table_name = 'customers';
```
<img width="397" height="193" alt="image" src="https://github.com/user-attachments/assets/1f1a17f5-0890-4974-9c59-9a8f084b28d6" />
<br><br><br><br>

2. Get the time range between which the orders were placed.
```sql
select
  min(order_purchase_timestamp) as min_date,
  max(order_purchase_timestamp) as max_date,
  date_diff(max(order_purchase_timestamp), min(order_purchase_timestamp), day) as number_of_days
from scaler-dsml-sql-459505.target.orders;
```
<img width="777" height="67" alt="image" src="https://github.com/user-attachments/assets/4848cde5-b98a-4f21-8ba0-721894f2be3b" />
<br><br><br><br>
All the orders are placed within a timeframe of 2 years- from 2016 to 2018
Starting in September 2016 going upto October 2018 over a period of 772 days. <br><br><br><br>

3. Count the Cities & States of customers who ordered during the given period.
```sql
select
  count(distinct concat(customer_city, '-', customer_state)) as places
from target.customers c
join target.orders o
on c.customer_id = o.customer_id;
```

<b>2. In-depth Exploration:</b><br>
1. Is there a growing trend in the no. of orders placed over the past years?
2. Can we see some kind of monthly seasonality in terms of the no. of orders being placed?
3. During what time of the day, do the Brazilian customers mostly place their orders? (Dawn, Morning, Afternoon or Night)
