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
dense_rank() over (order by employee_sales.employee_sales_revenue)
from employee_sales
join employees as e
on employee_sales.employee_id = e.employee_id;
```
Steve, Michael and Anne are the top 3 employees with the most sales. Nancy, Janet and Margaret have the least sales.


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

![image](https://github.com/user-attachments/assets/38d67324-6c94-4c96-ba9b-a1bfd986e176)









