# Project Overview

This notebook contains SQL queries used to analyze operational and business data from a simulated Amazon e-commerce platform containing employees, warehouses, customers, products, orders, sellers, and reviews.

The queries progress from basic SQL queries to advanced analytical queries, demonstrating practical techniques used in data analysis and business reporting.

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
e.first_name,
e.last_name,
w.location AS warehouse_location
FROM employees e
INNER JOIN warehouses w
ON e.warehouse_id = w.warehouse_id
GROUP BY e.first_name, e.last_name, w.location;
```

### Answer / Result
Returns each employee with their assigned warehouse location.

### Insight
This query uses an INNER JOIN to connect employees with warehouses using warehouse_id. GROUP BY ensures structured analytical output.

---

# 2. Prime Member Customers

### Question
Find all customers who are Prime members. Show first name, last name, email, and country.

### SQL Query
```sql
SELECT 
first_name,
last_name,
email,
country
FROM customers
WHERE prime_member = 1
GROUP BY first_name, last_name, email, country;
```

### Answer / Result
Returns all Prime members in the customer base.

### Insight
The WHERE clause filters customers based on membership status for segmentation analysis.

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
WHERE category = 'Electronics'
GROUP BY product_name, price, stock_quantity;
```

### Answer / Result
Returns all electronics products with pricing and stock details.

### Insight
Category filtering helps isolate product-level performance within a specific segment.

---

# 4. Orders After 2024

### Question
Find all orders placed after 2024-01-01.

### SQL Query
```sql
SELECT 
o.order_id,
c.first_name || ' ' || c.last_name AS customer_name,
o.order_date
FROM orders o
INNER JOIN customers c
ON o.customer_id = c.customer_id
WHERE o.order_date > '2024-01-01'
GROUP BY o.order_id, customer_name, o.order_date;
```

### Answer / Result
Returns all orders placed after January 1, 2024.

### Insight
Joining customers improves readability by replacing IDs with meaningful names.

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
GROUP BY e.first_name, e.last_name, e.hire_date, w.warehouse_name, w.location
ORDER BY e.hire_date DESC;
```

### Answer / Result
Returns employees hired in 2024 along with warehouse details.

### Insight
strftime() allows year-based filtering in SQLite datasets.

---

# 6. Product Category Ratings

### Question
Calculate average rating for each product category.

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
Returns categories with strong average ratings.

### Insight
HAVING filters aggregated results after grouping.

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
Returns warehouse operational metrics.

### Insight
LEFT JOIN ensures warehouses are included even if no activity exists.

---

# 8. High-Value Customers

### Question
Find customers who have placed more than one order.

### SQL Query
```sql
SELECT 
c.first_name || ' ' || c.last_name AS customer_name,
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
Combining COUNT and SUM helps identify customer lifetime value.

---

# 9. Employees Above Department Average Salary

### Question
Find employees earning more than their department average.

### SQL Query
```sql
WITH department_avg AS (
SELECT 
department_id,
AVG(salary) AS avg_salary
FROM employees
GROUP BY department_id
)

SELECT 
e.first_name || ' ' || e.last_name AS employee_name,
e.salary,
d.avg_salary
FROM employees e
INNER JOIN department_avg d
ON e.department_id = d.department_id
WHERE e.salary > d.avg_salary
GROUP BY e.first_name, e.last_name, e.salary, d.avg_salary;
```

### Answer / Result
Returns above-average earners per department.

### Insight
CTEs simplify multi-step aggregation logic.

---

# 10. Seller Performance Report

### Question
Create a seller performance report including revenue and ranking.

### SQL Query
```sql
SELECT 
s.seller_name,
COUNT(oi.order_item_id) AS products_sold,
SUM(oi.price * oi.quantity) AS total_revenue,
RANK() OVER (ORDER BY SUM(oi.price * oi.quantity) DESC) AS revenue_rank
FROM sellers s
INNER JOIN products p
ON s.seller_id = p.seller_id
INNER JOIN order_items oi
ON p.product_id = oi.product_id
GROUP BY s.seller_id, s.seller_name
HAVING COUNT(oi.order_item_id) >= 2
ORDER BY total_revenue DESC;
```

### Answer / Result
Returns seller performance rankings.

### Insight
Window functions enable ranking without breaking aggregation logic.

---

# 11. Hierarchical Employee Report

### Question
Show employees with manager relationships and salary comparisons.

### SQL Query
```sql
SELECT 
e.first_name || ' ' || e.last_name AS employee_name,
m.first_name || ' ' || m.last_name AS manager_name,
e.salary,
RANK() OVER (PARTITION BY e.department_id ORDER BY e.salary DESC) AS dept_rank
FROM employees e
LEFT JOIN employees m
ON e.manager_id = m.employee_id
GROUP BY e.first_name, e.last_name, m.first_name, m.last_name, e.salary, e.department_id;
```

### Answer / Result
Returns employee hierarchy structure.

### Insight
Self joins allow hierarchical relationship mapping within a single table.

---

# 12. Inventory Restocking Risk

### Question
Identify products at risk of stock depletion.

### SQL Query
```sql
WITH sales AS (
SELECT 
product_id,
SUM(quantity) AS total_sold
FROM order_items
GROUP BY product_id
)

SELECT 
p.product_name,
p.stock_quantity,
COALESCE(s.total_sold, 0) AS total_sold,
p.stock_quantity - COALESCE(s.total_sold, 0) AS remaining_stock
FROM products p
LEFT JOIN sales s
ON p.product_id = s.product_id
GROUP BY p.product_name, p.stock_quantity, s.total_sold
ORDER BY remaining_stock ASC;
```

### Answer / Result
Returns products with low remaining stock.

### Insight
Combining inventory and sales helps predict restocking needs.

---

# 13. Highest Value Order per Customer

### Question
Find each customer's highest value order.

### SQL Query
```sql
SELECT 
customer_id,
order_id,
total_amount
FROM (
SELECT 
o.customer_id,
o.order_id,
o.total_amount,
RANK() OVER (PARTITION BY o.customer_id ORDER BY o.total_amount DESC) AS rnk
FROM orders o
)
WHERE rnk = 1
GROUP BY customer_id, order_id, total_amount;
```

### Answer / Result
Returns highest value order per customer.

### Insight
Window functions simplify ranking within groups.

---

# 14. Customer Purchase Gap Analysis

### Question
Analyze time gaps between customer orders.

### SQL Query
```sql
SELECT 
customer_id,
order_date,
LAG(order_date) OVER (PARTITION BY customer_id ORDER BY order_date) AS prev_order_date
FROM orders
GROUP BY customer_id, order_date;
```

### Answer / Result
Returns order timeline per customer.

### Insight
LAG helps analyze customer purchase behavior over time.
