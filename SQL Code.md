# Northwind Traders Data Analysis

### Getting to Know the Data
Familiarize yourself with the Northwind company by writing the following queries. Consider saving them as views to help with the rest of the project.
- Combine orders and customers tables to get more detailed information about each order.
- Combine order_details, products, and orders tables to get detailed order information, including the product name and quantity.
- Combine employees and orders tables to see who is responsible for each order.

```sql
select o.order_id, o.customer_id, o.order_date, c.company_name, c.contact_name, c.country, c.city
from orders o
join customers c
on o.customer_id = c.customer_id;

select od.order_id, od.product_id, p.product_name, od.quantity
from order_details od
join products p
on od.product_id = p.product_id;

select o.order_id, o.employee_id, e.first_name, e.last_name
from orders o
join employees e
on o.employee_id = e.employee_id;
```


### Ranking Employee Sales Performance
As the lead Data Analyst at Northwind Traders, you've been tasked with comprehensively reviewing the company's sales performance from an employee perspective. The objective is twofold:
- First, the management team wants to recognize and reward top-performing employees, fostering a culture of excellence within the organization.
- Second, they want to identify employees who might be struggling so that they can offer the necessary training or resources to help them improve.
The management team is keen on encouraging healthy competition and rewarding stellar performers. They've asked you to rank employees based on their total sales amount.

```sql
with employee_sales as(
select o.employee_id, sum(od.unit_price * (1 - od.discount) * od.quantity) as employee_sales_revenue
from order_details od
join orders o
on od.order_id = o.order_id
group by o.employee_id)

select employee_sales.employee_id, employee_sales.employee_sales_revenue,
e.first_name || ' ' || e.last_name as name,
dense_rank() over (order by employee_sales.employee_sales_revenue desc)
from employee_sales
join employees as e
on employee_sales.employee_id = e.employee_id;
```

![image](https://github.com/user-attachments/assets/943da2d1-ad7a-4ff9-b58e-2c36fb57d7da)


### Running Total of Monthly Sales
Having completed the employee performance ranking, you've provided the management team with valuable insights into individual employee contributions. They're now keen on gaining a more macro-level perspective, specifically around the company's overall sales performance over time. They're looking to visualize the progress of the sales and identify trends that might shape the company's future strategies.
Your first task in this new analysis is to visualize the company's sales progress over time on a monthly basis. This will involve aggregating the sales data at a monthly level and calculating a running total of sales by month. This visual will provide the management team with a clear depiction of sales trends and help identify periods of high or low sales activity.

```sql
with monthly_sales as(
select date_trunc('month', o.order_date)::date as month,
sum(od.unit_price * (1 - od.discount) * od.quantity) as monthly_revenue
from order_details od
join orders o
on od.order_id = o.order_id
group by date_trunc('month', o.order_date))

select month, monthly_revenue,
sum(monthly_revenue) over (order by month range between unbounded preceding and current row) as running_total
from monthly_sales; 
```

![image](https://github.com/user-attachments/assets/6f421129-b3f3-4272-b1be-6391d094a54a)


### Month-Over-Month Sales Growth
After you've presented the running sales total by month, the management team is interested in further dissecting these figures. They would like to analyze the month-over-month sales growth rate. Understanding the rate at which sales are increasing or decreasing from month to month will help the management team identify significant trends.

For this task, you'll need to calculate the percentage change in sales from one month to the next using the results from the previous screen. 

```sql
with monthly_sales as(
select date_trunc('month', o.order_date)::date as month,
sum(od.unit_price * (1 - od.discount) * od.quantity) as monthly_revenue
from order_details od
join orders o
on od.order_id = o.order_id
group by date_trunc('month', o.order_date))

select month, monthly_revenue,
(monthly_revenue - lag(monthly_revenue) over (order by month)) / lag(monthly_revenue) over (order by month) * 100 as change_monthly_revenue
from monthly_sales;
```

![image](https://github.com/user-attachments/assets/6d124252-770c-4651-b04c-5bec2a31ba0a)


### Identifying High-Value Customers
Upon completing the sales growth and trend analysis, you've provided the management team valuable insights into the company's sales performance over time. Now, they're interested in a different, equally important, aspect of the business: the customers.

They want to identify high-value customers to whom they can offer targeted promotions and special offers, which could drive increased sales, improve customer retention, and attract new customers.

To do this, they've asked you to identify customers with above-average order values. These customers might be businesses buying in bulk or individuals purchasing high-end products.

```sql
with customer_orders as (
    select o.customer_id as customer_id, o.order_id as order_id,
sum(od.unit_price * (1 - od.discount) * od.quantity) as order_value
from order_details od
join orders o
on od.order_id = o.order_id
group by o.customer_id, o.order_id)

select customer_id,
order_id, order_value,
case
when order_value > (select avg(order_value) from customer_orders) then 'Above Average'
else 'Average/Below Average'
end as average_comparison
from customer_orders
order by order_id
limit 10;
```

![image](https://github.com/user-attachments/assets/f1e75eac-9195-4eae-96ce-0868f4e1a6f4)



###  Percentage of Sales for Each Category
You've been asked to provide the management team with an understanding of sales composition across different product categories. By knowing the percentage of total sales for each product category, they can gain insights into which categories drive most of the company's sales.

This understanding will help guide decisions about inventory (e.g., which categories should be stocked more heavily) and marketing strategies (e.g., which categories should be promoted more aggressively).

```sql
with category_sales as (select p.category_id, 
sum(od.unit_price * (1 - od.discount) * od.quantity) as category_sales,
sum(od.unit_price * (1 - od.discount) * od.quantity) / (select sum(od.unit_price * (1 - od.discount) * od.quantity) from order_details od) * 100 as percentage_category_sales
from order_details od
join products p
on od.product_id = p.product_id
group by p.category_id
order by percentage_category_sales)

select c.category_name, cs.category_id, cs.category_sales, cs.percentage_category_sales
from category_sales cs
join categories c
on cs.category_id = c.category_id
order by percentage_category_sales;
```

![image](https://github.com/user-attachments/assets/f0b857e9-679e-4c60-9afa-6d8988c8fcc2)



### Top Products Per Category
With the knowledge of sales by category, the next step is to drill down further into each group. The management team wants to know the top three items sold in each product category. This will allow them to identify star performers and ensure that these products are kept in stock and marketed prominently.

```sql
with product_sales as(
select p.product_id, p.category_id, p.product_name,
sum(od.unit_price * (1 - od.discount) * od.quantity) as product_sales
from order_details od
join products p
on od.product_id = p.product_id
group by p.category_id, p.product_id),

ordered_product_sales as(
select product_id, product_name, category_id, product_sales, 
row_number() over (partition by category_id order by product_sales desc) as row_number
from product_sales ps)

select product_id, product_name, category_id, product_sales, row_number
from ordered_product_sales ops
where row_number <= 3
limit 9;
```

![image](https://github.com/user-attachments/assets/e5dc8aea-40ae-4f57-90c2-e376ee3a257b)
