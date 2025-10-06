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
```sql
with final_table as
(
select
  order_id,
  extract(year from order_purchase_timestamp) as order_year
from target.orders
)
select
  final_table.order_year,
  count(final_table.order_id) as num_of_orders
from final_table
group by final_table.order_year
order by final_table.order_year;
```
<img width="397" height="193" alt="image" src="https://github.com/user-attachments/assets/1aeed6bf-f103-4ef2-b3f8-20610348799c" /> <br>
**Insights**: Yes, there is a growing trend in the number of orders placed over the past years as you can see the number of orders increasing each year. This signifies that the company is doing well and has gained more popularity over the years.


2. Can we see some kind of monthly seasonality in terms of the no. of orders being placed?
```sql
select
  extract(year from order_purchase_timestamp) as order_year,
  extract(month from order_purchase_timestamp) as order_month,
  count(order_id) as orders_count
from target.orders
group by order_year, order_month
order by order_year, order_month
limit 10;
```
<img width="491" height="365" alt="image" src="https://github.com/user-attachments/assets/05a2f829-cb2b-420d-b2c2-c87a8cde5749" />

  
3. During what time of the day, do the Brazilian customers mostly place their orders? (Dawn, Morning, Afternoon or Night)
    * 0-6 hrs : Dawn
    * 7-12 hrs : Mornings
    * 13-18 hrs : Afternoon
    * 19-23 hrs : Night
```sql
select
  t.hour_number,
  t.time_of_day,
  count(t.order_id) as count_of_orders
from
(
select
  order_id,
  extract(hour from o.order_purchase_timestamp) as hour_number,
  case
    when extract(hour from o.order_purchase_timestamp) between 0 and 6 then 'Dawn'
    when extract(hour from o.order_purchase_timestamp) between 7 and 12 then 'Mornings'
    when extract(hour from o.order_purchase_timestamp) between 13 and 18 then 'Afternoon'
    else 'Night'
  end as time_of_day
from `target.customers` c
join `target.orders` o
on o.customer_id = c.customer_id
) t
group by t.hour_number, t.time_of_day
order by count_of_orders desc
limit 1
```
<img width="535" height="85" alt="image" src="https://github.com/user-attachments/assets/9bb99a4e-ae8b-4afc-af6c-fb1cf4055701" />

**Insights:** Maximum orders are placed during hour 16, i.e during Afternoon 4 pm onwards. This is probably due to the fact that most corporate offices end at around 4 or 5 pm hence the large number of orders during that time frame.<br>
**Recommendations:** Target can send targeted ads through its app and various other online ecommerce platforms for its products during these hours can help boost sales and revenue generation.

<b>3. Evolution of E-commerce orders in the Brazil region:</b><br>
