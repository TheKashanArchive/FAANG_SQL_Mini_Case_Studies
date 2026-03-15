# Warehouse & E-Commerce SQL Analysis

**Skills Demonstrated:** SQL · Joins · Aggregations · Window Functions · CTEs · Ranking · Data Analysis · Inventory Analysis

## Project Overview

This notebook contains SQL queries used to analyze operational and business data from a simulated warehouse and e-commerce system.

The queries progress from **basic SQL queries** to **advanced analytical queries**, demonstrating practical techniques used in data analysis.

---

# Table of Contents

1. Employee Warehouse Locations  
2. Prime Member Customers  
3. Electronics Products  
4. Orders After 2024  
5. Employees Hired in 2024  
6. Product Category Ratings  
7. Warehouse Operational Summary  
8. High-Value Customers  
9. Employees Above Department Average Salary  
10. Seller Performance Report  
11. Hierarchical Employee Report  
12. Inventory Restocking Risk  
13. Highest Value Order per Customer  
14. Customer Purchase Gap Analysis  

---

# 1. Employee Warehouse Locations

### Question
List all employees with their warehouse locations. Show employee first name, last name, and warehouse location.

### SQL Query
```sql
SELECT 
e.first_name AS first_name,
e.last_name AS last_name,
w.location AS warehouse_location
FROM employees e
INNER JOIN warehouses w
ON e.warehouse_id = w.warehouse_id;
```

### Answer / Result
Returns a table showing each employee’s first name, last name, and the location of the warehouse they are assigned to.

### Insight
This query uses an INNER JOIN to combine data from the employees and warehouses tables.  
The join condition `e.warehouse_id = w.warehouse_id` links each employee to the warehouse they belong to.

---

# 2. Prime Member Customers

### Question
Find all customers who are Prime members. Show first name, last name, email, and country.

### SQL Query
```sql
SELECT first_name, last_name, email, country
FROM customers
WHERE prime_member = 1;
```

### Answer / Result
Returns customers who have a Prime membership along with their contact information.

### Insight
The `WHERE` clause filters rows so only customers with Prime membership appear.

---

# 3. Electronics Products

### Question
List all products in the Electronics category.

### SQL Query
```sql
SELECT 
product_name,
price,
stock_quantity
FROM products
WHERE category = 'Electronics';
```

### Answer / Result
Displays all electronics products with their price and stock quantity.

### Insight
Filtering by category allows analysts to examine inventory or pricing within a specific product segment.

---

# 4. Orders After 2024

### Question
Find all orders placed after 2024-01-01.

### SQL Query
```sql
SELECT 
o.order_id,
CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
o.order_date
FROM orders o
INNER JOIN customers c
ON o.customer_id = c.customer_id
WHERE o.order_date > '2024-01-01';
```

### Answer / Result
Returns all orders placed after January 1, 2024.

### Insight
Joining orders with customers allows the result to include readable customer names instead of just IDs.

---

# 5. Employees Hired in 2024

### Question
Retrieve all warehouse employees hired in 2024.

### SQL Query
```sql
SELECT 
e.first_name,
e.last_name,
e.hire_date,
w.warehouse_name,
w.location
FROM employees e
INNER JOIN warehouses w
ON e.warehouse_id = w.warehouse_id
WHERE strftime('%Y', e.hire_date) = '2024'
ORDER BY e.hire_date DESC;
```

### Answer / Result
Shows employees hired during 2024 along with their assigned warehouse.

### Insight
The `strftime()` function extracts the year from a date field in SQLite, allowing filtering by year.

---

# 6. Product Category Ratings

### Question
Calculate the average rating for each product category.

### SQL Query
```sql
SELECT 
p.category,
COUNT(DISTINCT p.product_id) AS product_count,
ROUND(AVG(r.rating), 2) AS avg_rating
FROM products p
INNER JOIN reviews r
ON p.product_id = r.product_id
GROUP BY p.category
HAVING AVG(r.rating) > 4.0
ORDER BY avg_rating DESC;
```

### Answer / Result
Returns categories with an average rating above 4.0 along with the number of products in each category.

### Insight
The `HAVING` clause filters aggregated results after the grouping operation.

---

# 7. Warehouse Operational Summary

### Question
Summarize warehouse operations including employees and shipments.

### SQL Query
```sql
SELECT 
w.warehouse_name,
w.location,
COUNT(DISTINCT e.employee_id) AS employee_count,
COUNT(DISTINCT s.shipment_id) AS shipment_count,
w.capacity
FROM warehouses w
LEFT JOIN employees e
ON w.warehouse_id = e.warehouse_id
LEFT JOIN shipments s
ON w.warehouse_id = s.warehouse_id
GROUP BY w.warehouse_id, w.warehouse_name, w.location, w.capacity
ORDER BY shipment_count DESC;
```

### Answer / Result
Displays warehouse operational metrics including employees, shipments handled, and capacity.

### Insight
LEFT JOIN ensures warehouses appear even if they have no employees or shipments recorded.

---

# 8. High-Value Customers

### Question
Find customers who have placed more than one order.

### SQL Query
```sql
SELECT 
CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
c.email,
COUNT(o.order_id) AS total_orders,
ROUND(SUM(o.total_amount), 2) AS total_spent
FROM customers c
INNER JOIN orders o
ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.first_name, c.last_name, c.email
HAVING COUNT(o.order_id) > 1
ORDER BY total_spent DESC;
```

### Answer / Result
Returns repeat customers ranked by total spending.

### Insight
Combining `COUNT()` and `SUM()` helps identify loyal customers and their lifetime value.

---

# 9. Employees Above Department Average Salary

### Question
Find employees whose salary is higher than their department’s average salary.

### SQL Query
```sql
WITH department_averages AS (
SELECT 
department_id,
CAST(AVG(salary) AS INTEGER) AS dept_avg_salary
FROM employees
GROUP BY department_id
)

SELECT 
CONCAT(e.first_name,' ',e.last_name) AS employee_name,
d.department_name,
e.salary,
da.dept_avg_salary,
e.salary - da.dept_avg_salary AS salary_difference
FROM employees e
INNER JOIN departments d
ON e.department_id = d.department_id
INNER JOIN department_averages da
ON e.department_id = da.department_id
WHERE e.salary > da.dept_avg_salary
ORDER BY salary_difference DESC;
```

### Answer / Result
Lists employees earning more than their department’s average salary.

### Insight
A CTE simplifies complex calculations by computing department averages first.

---

# 10. Seller Performance Report

### Question
Create a seller performance report including revenue and ranking.

### SQL Query
```sql
SELECT 
s.seller_name,
COUNT(DISTINCT oi.order_item_id) AS products_sold,
ROUND(SUM(oi.price * oi.quantity), 2) AS total_revenue,
ROUND(AVG(r.rating), 2) AS avg_rating,
RANK() OVER (ORDER BY SUM(oi.price * oi.quantity) DESC) AS revenue_rank
FROM sellers s
INNER JOIN products p
ON s.seller_id = p.seller_id
INNER JOIN order_items oi
ON p.product_id = oi.product_id
LEFT JOIN reviews r
ON p.product_id = r.product_id
GROUP BY s.seller_id, s.seller_name
HAVING COUNT(DISTINCT oi.order_item_id) >= 2
ORDER BY total_revenue DESC;
```

### Answer / Result
Displays seller performance metrics including revenue and ranking.

### Insight
The window function `RANK()` ranks sellers without affecting grouped results.

---

# 11. Hierarchical Employee Report

### Question
Show employees with their manager, salary comparison, and department ranking.

### SQL Query
```sql
WITH dept_stats AS (
SELECT 
department_id,
ROUND(AVG(salary),2) AS dept_avg_salary
FROM employees
GROUP BY department_id
)

SELECT 
CONCAT(e.first_name,' ',e.last_name) AS employee_name,
CONCAT(m.first_name,' ',m.last_name) AS manager_name,
d.department_name,
e.salary AS employee_salary,
m.salary AS manager_salary,
COALESCE(m.salary - e.salary,0) AS salary_difference,
RANK() OVER (PARTITION BY e.department_id ORDER BY e.salary DESC) AS dept_salary_rank,
ds.dept_avg_salary
FROM employees e
LEFT JOIN employees m
ON e.manager_id = m.employee_id
INNER JOIN departments d
ON e.department_id = d.department_id
INNER JOIN dept_stats ds
ON e.department_id = ds.department_id
ORDER BY d.department_name, dept_salary_rank;
```

### Answer / Result
Shows the employee hierarchy along with department salary rankings.

### Insight
Self-joins allow employees to be matched with their managers within the same table.

---

# 12. Inventory Restocking Risk

### Question
Identify products that may require restocking.

### SQL Query
```sql
WITH sales_data AS (
SELECT 
oi.product_id,
SUM(oi.quantity) AS total_sold,
SUM(oi.price * oi.quantity) AS revenue
FROM order_items oi
GROUP BY oi.product_id
)

SELECT 
p.product_name,
p.category,
w.location,
p.stock_quantity,
COALESCE(ROUND(sd.total_sold / 8.0, 2), 0) AS avg_daily_sales,
CASE 
WHEN COALESCE(sd.total_sold,0) > 0
THEN ROUND(p.stock_quantity * 1.0 / (sd.total_sold / 8.0),1)
ELSE 999
END AS days_remaining,
COALESCE(ROUND(sd.revenue,2),0) AS potential_revenue_loss,
RANK() OVER (ORDER BY p.stock_quantity ASC, sd.total_sold DESC) AS restock_rank
FROM products p
INNER JOIN warehouses w
ON p.warehouse_id = w.warehouse_id
LEFT JOIN sales_data sd
ON p.product_id = sd.product_id
ORDER BY restock_rank
LIMIT 5;
```

### Answer / Result
Shows the top products most likely to run out of stock.

### Insight
Combining stock levels with historical sales helps estimate future inventory risk.

---

# 13. Highest Value Order per Customer

### Question
Find each customer’s highest value order.

### SQL Query
```sql
WITH customer_order_ranks AS (
SELECT 
o.order_id,
CONCAT(c.first_name,' ',c.last_name) AS customer_name,
o.total_amount,
RANK() OVER (
PARTITION BY o.customer_id
ORDER BY o.total_amount DESC
) AS order_rank
FROM orders o
INNER JOIN customers c
ON o.customer_id = c.customer_id
)

SELECT 
customer_name,
order_id,
total_amount,
order_rank
FROM customer_order_ranks
WHERE order_rank = 1
ORDER BY total_amount DESC;
```

### Answer / Result
Displays each customer’s most expensive order.

### Insight
Window functions make it easy to rank rows within groups.

---

# 14. Customer Purchase Gap Analysis

### Question
Calculate the longest gap between consecutive orders for repeat customers.

### SQL Query
```sql
WITH order_gaps AS (
SELECT 
o.customer_id,
o.order_date,
LAG(o.order_date) OVER (
PARTITION BY o.customer_id
ORDER BY o.order_date
) AS prev_order_date,
julianday(o.order_date) - julianday(
LAG(o.order_date) OVER (
PARTITION BY o.customer_id
ORDER BY o.order_date
)
) AS gap_days
FROM orders o
),
customer_gap_stats AS (
SELECT 
customer_id,
COUNT(*) + 1 AS total_orders,
ROUND(AVG(gap_days),1) AS avg_gap_days,
ROUND(MAX(gap_days),1) AS max_gap_days
FROM order_gaps
WHERE prev_order_date IS NOT NULL
GROUP BY customer_id
)

SELECT 
concat(c.first_name,' ',c.last_name) AS customer_name,
c.email,
cgs.total_orders,
cgs.avg_gap_days,
cgs.max_gap_days
FROM customer_gap_stats cgs
INNER JOIN customers c
ON cgs.customer_id = c.customer_id
WHERE cgs.total_orders >= 3
ORDER BY cgs.max_gap_days DESC;
```

### Answer / Result
Shows customers with at least three orders and their order timing behavior.

### Insight
The `LAG()` function compares each order with the previous one to calculate time gaps between purchases.
