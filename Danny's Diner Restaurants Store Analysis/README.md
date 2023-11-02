# 🍜 Case Study #1: Danny's Diner 

<img width="555" alt="Screenshot 2023-09-30 175658" src="https://github.com/GauravvThakurr/Case-Studies-SQL/assets/141028751/9aab3ee1-8470-4ec2-a14b-5ee326799165">

***

## 📑 Task
Danny aims to enhance customer experiences and assess the expansion of his loyalty program using three key datasets: sales, menu, and members. To gain insights, he plans to analyze customer visit patterns, total spending, and favorite menu items. Additionally, he needs basic datasets generated for his team's easy data inspection.

***

## 📍Entity Relationship Diagram
<img width="503" alt="image" src="https://github.com/GauravvThakurr/Case-Studies-SQL/assets/141028751/5f85a30f-5495-45bd-a782-0297e8f60683">

***

# Problem and Solutions
We are going to use Postgresql DBMS for this setup, but we can use any of our choice

**1. What is the total amount each customer spent at the restaurant?**

````sql
SELECT
  s.customer_id AS Customers_Id,
  concat('$', SUM(m.price)) AS Total_amt_customer_spend
FROM dannys_diner.sales s
INNER JOIN dannys_diner.menu m
  ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY SUM(m.price) DESC;
````

#### 🔖 Setps
- Performe **INNER JOIN** between `dannys_diner.sales` and `dannys_diner.menu` table and got `s.customer_id AS Customers_Id` and `m.price as Total_amt_customer_spend`
- Performe Sum on `m.price` and Group the aggregated results by `s.customer_id`
- Also add `$` to aggregated values for better readability (optional)



#### 🎯Insights
<img width="359" alt="image" src="https://github.com/GauravvThakurr/Case-Studies-SQL/assets/141028751/8ff9d1f7-e979-4378-bc54-6b0ac23ec116">

- Customer_Id A spents total of $76
- Customer_id B spents total of$74
- Customer_id C spents total of$36

***

**2. How many days has each customer visited the restaurant?**

````sql
SELECT
  customer_id,
  COUNT(DISTINCT order_date) AS Days_visited
FROM dannys_diner.sales
GROUP BY customer_id;
````

#### 🔖 Setps
- To find unique visits use  `COUNT(DISTINCT order_date)` than group by `customer_id` 

#### 🎯Insights
<img width="255" alt="image" src="https://github.com/GauravvThakurr/Case-Studies-SQL/assets/141028751/bb1e6992-1176-4bb7-978d-3b2d5eb6b769">

- Customer_Id A visited 4 times
- Customer_Id B visited 6 times
- Customer_Id c visited 2 times

***

**3. What was the first item from the menu purchased by each customer?**

````sql
WITH firstcte
AS (SELECT
  s.customer_id,
  s.order_date,
  m.product_name,
  DENSE_RANK() OVER (
  PARTITION BY s.customer_id
  ORDER BY s.order_date) AS first_order
FROM dannys_diner.sales s
INNER JOIN dannys_diner.menu m
  ON s.product_id = m.product_id)

SELECT
  customer_id,
  product_name
FROM firstcte
WHERE first_order = 1
GROUP BY customer_id,
         product_name
````

#### 🔖 Setps 
- Create a Common Table Expression (CTE) called `firstcte` Within the CTE, include a new column named `first_order` which calculates the row number using the `DENSE_RANK()` window function. The data is divided into partitions based on `customer_id` and the rows within each partition are ordered by `order_date`
- In outer Query select required columns and then apply where clause to filter `first_order = 1` which will only select first orders
- Group them all

#### 🎯Insights
<img width="289" alt="image" src="https://github.com/GauravvThakurr/Case-Studies-SQL/assets/141028751/dbce16a3-bf80-4fc9-b389-8383772d6afb">

- Customer_id A brought Sushi and curry
- Customer_id B ordered curry
- Customer_id A brought Ramen

Note: A ordered twice on the same day, we have 2 orders as First since we do not have the exact order time we took the first visit date as the first order day

***

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**
````sql
with most_ordered as (
select product_id
from dannys_diner.sales
group by product_id
order by count(*) desc
limit 1
),

ordered_count as 
(
  select customer_id, product_id , count(*) as no_order
from dannys_diner.sales
where product_id in (select product_id from most_ordered )
group by customer_id, product_id

)

select o.customer_id, m.product_name, o.no_order
from ordered_count o
join dannys_diner.menu m on m.product_id = o.product_id
order by o.customer_id
````

#### 🔖 Setps
- First we find most ordered product, than limited it to 1 row
- Using subquery and CTE we found how many times each customer ordered most ordered product
- 
#### 🎯Insights
<img width="351" alt="Screenshot 2023-10-19 124616" src="https://github.com/GauravvThakurr/Case-Studies-SQL/assets/141028751/6e379184-921e-4923-9a01-8e7716e41585">

- Customer_id A placed 3 ramen order
- Customer_id B placed 2 ramen order
- Customer_id c placed 3 ramen order

***

**5.Which item was the most popular for each customer?**

````sql
with rankedorder as (

select customer_id, product_id, count(*) as no_order,
dense_rank() over (partition by customer_id order by count(*) desc) as rn
from dannys_diner.sales
group by customer_id, product_id

)
select r.customer_id, m.product_name,r.no_order from rankedorder r
join dannys_diner.menu m on m.product_id = r.product_id
where r.rn = 1
order by r.no_order desc
````

#### 🔖 Setps
- We found which product was orderd how many times using `dense_rank()` then enclosed in cte named `rankedorder`
- In main query we join `Menu` table for to pull names of product and used `r.rn = 1` to pull only the top order from each customer
  
#### 🎯Insights
<img width="354" alt="Screenshot 2023-10-19 125827" src="https://github.com/GauravvThakurr/Case-Studies-SQL/assets/141028751/7502df9e-b080-4678-a790-96db7eb62a29">

***

**6.Which item was purchased first by the customer after they became a member?**
