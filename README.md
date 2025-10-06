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

## <b>1. Import the dataset and do usual exploratory analysis steps like checking the structure & characteristics of the dataset:</b><br>
### a. Data type of all columns in the "customers" table.
```sql
select column_name, data_type
from scaler-dsml-sql-459505.target.INFORMATION_SCHEMA.COLUMNS
where table_name = 'customers';
```
<img width="397" height="193" alt="image" src="https://github.com/user-attachments/assets/1f1a17f5-0890-4974-9c59-9a8f084b28d6" />
<br><br><br><br>

### b. Get the time range between which the orders were placed.
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

### c. Count the Cities & States of customers who ordered during the given period.
```sql
select
  count(distinct concat(customer_city, '-', customer_state)) as places
from target.customers c
join target.orders o
on c.customer_id = o.customer_id;
```

## <b>2. In-depth Exploration:</b><br>
### a. Is there a growing trend in the no. of orders placed over the past years?
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


### b. Can we see some kind of monthly seasonality in terms of the no. of orders being placed?
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

  
### c. During what time of the day, do the Brazilian customers mostly place their orders? (Dawn, Morning, Afternoon or Night)
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

## <b>3. Evolution of E-commerce orders in the Brazil region:</b><br>
### a. Get the month on month no. of orders placed in each state.
```sql
select
  c.customer_state,
  extract(month from order_purchase_timestamp) as month_number,
  count(o.order_id) as num_of_orders
from target.customers c
left join target.orders o on c.customer_id = o.customer_id
group by c.customer_state, month_number
order by c.customer_state, month_number
```
<img width="397" height="193" alt="image" src="https://github.com/user-attachments/assets/bb57626c-c1e1-4184-99ad-3c7b68817915" />

**Insights:** For each state the max number of orders are placed in summer months- April and May, so we can use these months to maximise our potential and sell all the excess inventory.  
**Recommendations:** The least number of orders are present in winter months, mostly December, so we can use special discounts like the Christmas discount during those periods to boost sales and efficiency.

### b. How are the customers distributed across all the states?
```sql
select
  customer_state,
  count(customer_id) as num_of_customers
from target.customers
group by customer_state
order by num_of_customers
```
<img width="397" height="193" alt="image" src="https://github.com/user-attachments/assets/584b853d-f3a0-4880-b0b5-ce9bbb81cd2a" />


**Insights:** RR has the least number of customers, while SP has the most number of customers, which indicates Target is doing a good amount of sales in SP. <br>
**Recommendations:** Target can look into what is causing that to happen and provide attractive discount offers and prepare targeted marketing campaigns to boost revenue there.

## <b>4. Impact on Economy: Analyze the money movement by e-commerce by looking at order prices, freight and others.</b><br>
### a. Get the % increase in the cost of orders from year 2017 to 2018 (include months between Jan to Aug only). You can use the "payment_value" column in the payments table to get the cost of orders.
```sql
select
  round(100*(sum(case when order_year = 2017 then total_payment end) / sum(case when order_year = 2018 then total_payment end)),3) as percent_increase
from
(
select
  t.order_year,
  sum(t.payment_value) as total_payment
from
(
select
  *,
  extract(year from order_purchase_timestamp) as order_year,
  extract(month from order_purchase_timestamp) as order_month
from target.orders o left join
target.payments p on o.order_id = p.order_id
) t
where t.order_year in (2017, 2018) and t.order_month <= 8
group by t.order_year
) t1
```

<img width="397" height="193" alt="image" src="https://github.com/user-attachments/assets/689b95e3-240f-4b15-9376-ed20934d6f4a" />

**Insights:** There has been an increase by 42.198% for the months of January to August from 2017 to 2018 which is a very good amount and indicates that the company is making a substantial amount of revenue and is improving over time.

### b. Calculate the Total & Average value of order price for each state.
```sql
select
  c.customer_state,
  round(sum(oi.price),2) as per_state_total_price,
  round(avg(oi.price), 2) as per_state_avg_price
from target.customers c
left join target.orders o on o.customer_id = c.customer_id
left join target.order_items oi on oi.order_id = o.order_id
group by c.customer_state
```
<img width="397" height="193" alt="image" src="https://github.com/user-attachments/assets/4c94723c-7b07-448b-98f1-6576e9c9cdf7" />

**Insights:** SP has the maximum per state average price and as seen earlier(in Q3 part 2) it also has the maximum number of customers.

### c. Calculate the Total & Average value of order freight for each state.
```sql
select
  c.customer_state,
  round(sum(oi.freight_value),2) as order_freight_sum,
  round(avg(oi.freight_value),2) as order_freight_avg
from target.customers c
left join target.orders o on c.customer_id = o.customer_id
left join target.order_items oi on oi.order_id = o.order_id
group by c.customer_state;
```
<img width="397" height="193" alt="image" src="https://github.com/user-attachments/assets/d48a2b6e-caeb-4524-92c8-fb0a2cf34dbd" />

**Insights:** Freight value, also known as freight rate or freight charge, refers to the cost associated with transporting goods from one location to another.
SP has the lowest freight sum value while RR has the highest. SP also has the maximum per state average price therefore it yields high profits. <br>
**Recommendations:** Try to reduce the freight value for AC.


## <b>5. Analysis based on sales, freight and delivery time: </b><br>
### a. Find the no. of days taken to deliver each order from the order’s purchase date as delivery time. Also, calculate the difference (in days) between the estimated & actual delivery date of an order. Do this in a single query.

**You can calculate the delivery time and the difference between the estimated & actual delivery date using the given formula:
time_to_deliver = order_delivered_customer_date - order_purchase_timestamp
diff_estimated_delivery = order_delivered_customer_date - order_estimated_delivery_date**
```sql
select
  order_id,
  timestamp_diff(order_delivered_customer_date, order_purchase_timestamp, day) as time_to_deliver,
  timestamp_diff(order_delivered_customer_date, order_estimated_delivery_date,day) as diff_estimated_delivery
from target.orders
order by time_to_deliver desc
```

<img width="520" height="320" alt="image" src="https://github.com/user-attachments/assets/f764d68b-5fe6-4ad1-97cb-6ad46e0d319b" />

**Insights:** The time_to_deliver gives the actual time it took to deliver while the diff_estimated_delivery gives the estimated time it would take.  <br>
**Recommendations:** In most of the cases there is a difference of around 20 days between these two columns, indicating there is a need to give a better estimate to the customer so as to keep expectations realistic and straightforward, thereby keeping transparency.


### b. Find out the top 5 states with the highest & lowest average freight value.
```sql
select
  s.seller_state,
  avg(freight_value) as avg_freight_val
from target.order_items oi
left join target.sellers s
on s.seller_id = oi.seller_id
group by s.seller_state
order by avg_freight_val desc
limit 5
```

<img width="397" height="193" alt="image" src="https://github.com/user-attachments/assets/024c755f-893b-4676-b2a8-6eddd47db4bd" />
**Insights:** This gives the list of the top 5 states with the highest average freight value. It indicates these states are incurring too many expenses in transporting the goods properly. So we must take initiative and formulate a plan to reduce this time period so that we can cut down on expenses. <br>
**Recommendations:** Try having meetings with the transport department to know their schedule and plan accordingly so as to reduce the amount of time and money spent in transporting the goods.

### c. Find out the top 5 states with the highest & lowest average delivery time.
```sql
with final_query as
(
select
  c.customer_state,
  timestamp_diff(o.order_delivered_customer_date, o.order_purchase_timestamp, day) as delivery_time
from
target.customers c left join
target.orders o
on c.customer_id = o.customer_id
)
select
  final_query.customer_state,
  avg(final_query.delivery_time) as avg_delivery_time
from final_query
group by final_query.customer_state
order by avg_delivery_time
limit 5;
```

<img width="397" height="193" alt="image" src="https://github.com/user-attachments/assets/4a3f73f0-3e65-4866-993f-3465c01e2e43" />

**Insights:** Since these states have low average delivery time, they will also have more sales revenue. According to the analysis from Q3 part 2 SP, PR, and MG are also among the top 5 states with the highest number of customers which is a direct correlation here since lesser average delivery time means one can have a higher number of deliveries in a shorter time span leading to more popularity and preference here.

Highest average delivery time

```sql
with final_query as
(
select
  c.customer_state,
  timestamp_diff(o.order_delivered_customer_date, o.order_purchase_timestamp, day) as delivery_time
from
target.customers c left join
target.orders o
on c.customer_id = o.customer_id
)
select
  final_query.customer_state,
  avg(final_query.delivery_time) as avg_delivery_time
from final_query
group by final_query.customer_state
order by avg_delivery_time desc
limit 5;
```

<img width="397" height="193" alt="image" src="https://github.com/user-attachments/assets/ebe32953-3584-4b39-870d-96438f428136" />

**Insights:** Since these states have high average delivery time, they will also have less sales revenue. According to the analysis from Q3 part 2 RR, AP, and AM are also among the top 5 states with the lowest number of customers which is a direct correlation here since higher average delivery time means one can have a lesser number of deliveries in a longer time span leading to less popularity and preference among users. <br>
**Recommendations:** Target must focus on reducing the delivery time for these states so that they can improve sales and gain more customers. 

### d. Find out the top 5 states where the order delivery is really fast as compared to the estimated date of delivery. You can use the difference between the averages of actual & estimated delivery date to figure out how fast the delivery was for each state.
```sql
select
  t.customer_state,
  (t.actual_days - t.estimated_days) as gap
from
(
with dc as
(
select
  c.customer_state,
  date_diff(o.order_delivered_customer_date, o.order_purchase_timestamp, day) as actual_num_of_days,
  date_diff(o.order_delivered_customer_date, order_estimated_delivery_date, day) as estimated_delivery_days
from target.customers c
left join target.orders o
on c.customer_id = o.customer_id
)
select
  dc.customer_state,
  avg(actual_num_of_days) as actual_days,
  avg(estimated_delivery_days) as estimated_days
from dc
group by dc.customer_state
) t
order by (t.actual_days - t.estimated_days) 
limit 5;
```

<img width="397" height="193" alt="image" src="https://github.com/user-attachments/assets/e9e86a26-8844-4f3b-a797-86b5ab9f84d5" />

**Insights:** Lesser the gap, faster the delivery.
These are the states where the order delivery is really fast as compared to the estimated date of delivery. <br>
**Recommendations:** The company should focus on reducing the gap between estimated date of arrival and actual date of arrival to improve customer experience and also keep track of the products being sent.

