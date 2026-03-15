## Question:
List all employees with their warehouse locations. Show employee first name, last name, and warehouse location.

SQL Query:

```
SELECT 
e.first_name AS first_name,
e.last_name AS last_name,
w.location AS warehouse_location
FROM employees e
INNER JOIN warehouses w
ON e.warehouse_id = w.warehouse_id;

### Answer / Result:
Returns a table showing each employee’s first name, last name, and the location of the warehouse they are assigned to.

### Insight:
This query uses an INNER JOIN to combine data from the employees and warehouses tables.
The join condition e.warehouse_id = w.warehouse_id links each employee to the warehouse they belong to.
Only employees with a matching warehouse record will appear in the result.



### Question:
Find all customers who are Prime members. Show first name, last name, email, and country.

SQL Query:

```
SELECT first_name, last_name, email, country
FROM customers
WHERE prime_member = 1;

### Answer / Result:
Returns a list of customers who have a Prime membership, displaying their first name, last name, email, and country.

### Insight:
The WHERE clause filters the dataset so that only rows where prime_member = 1 are returned. In many databases, 1 represents true, meaning the customer is a Prime member.



### Question:
List all products in the 'Electronics' category. Show product name, price, and stock quantity.

SQL Query:

```
SELECT 
product_name,
price,
stock_quantity
FROM products
WHERE category = 'Electronics';

### Answer / Result:
Returns a list of all products that belong to the Electronics category, displaying their name, price, and available stock quantity.

### Insight:
The WHERE clause filters the rows so that only products where the category column equals 'Electronics' are included



### Question:
Find all orders placed after 2024-01-01. Show order ID, customer full name (first and last name combined), and order date.

SQL Query:

```
SELECT 
o.order_id,   
CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
o.order_date
FROM orders o
INNER JOIN customers c
ON o.customer_id = c.customer_id
WHERE o.order_date > '2024-01-01';

### Answer / Result:
Returns all orders placed after January 1, 2024, displaying the order ID, the customer's full name, and the order date.

### Insight:
This query combines data from the orders and customers tables using an INNER JOIN on customer_id. 
The CONCAT() function merges first_name and last_name into a single customer_name field. 
The WHERE clause filters the results to include only orders placed after the specified date.



### Question:
Retrieve all warehouse employees hired in 2024. Show employee first name, last name, hire date, warehouse name, and warehouse location.

SQL Query:

```
SELECT 
e.first_name AS first_name,
e.last_name AS last_name,
e.hire_date AS hire_date,
w.warehouse_name AS warehouse_name,
w.location AS warehouse_location
FROM employees e
INNER JOIN warehouses w
ON e.warehouse_id = w.warehouse_id
WHERE strftime('%Y', e.hire_date) = '2024'
ORDER BY e.hire_date DESC;

### Answer / Result:
Returns a list of employees who were hired in 2024, along with their names, hire date, warehouse name, and warehouse location, sorted by most recent hire date first.

### Insight:
The strftime('%Y', e.hire_date) function extracts the year from the hire_date column (commonly used in SQLite). 
This allows filtering employees hired specifically in 2024. 
The INNER JOIN links employees to their warehouses, and ORDER BY e.hire_date DESC ensures the newest hires appear first.



### Question:
Calculate the average rating for each product category. Show category name, number of products, and average rating. Only include categories with an average rating above 4.0.

SQL Query:

```
SELECT 
p.category AS category,
COUNT(DISTINCT p.product_id) AS product_count,
ROUND(AVG(r.rating), 2) AS avg_rating
FROM products p
INNER JOIN reviews r
ON p.product_id = r.product_id
GROUP BY p.category
HAVING AVG(r.rating) > 4.0
ORDER BY avg_rating DESC;

### Answer / Result:
Returns product categories where the average review rating is greater than 4.0, showing the category name, number of unique products in that category, and the average rating rounded to two decimal places.

### Insight:
This query combines aggregation and filtering on grouped data. 
GROUP BY p.category groups all products by category, AVG(r.rating) calculates the mean rating for each category, and HAVING AVG(r.rating) > 4.0 filters the grouped results to include only high-rated categories. 
COUNT(DISTINCT p.product_id) ensures each product is counted once, even if it has multiple reviews.



### Question:
For each warehouse, calculate the total number of shipments, total employees, and show capacity information. Display warehouse name, location, employee count, shipment count, and capacity.

SQL Query:

```
SELECT 
w.warehouse_name AS warehouse_name,
w.location AS location,
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

### Answer / Result:
Returns a summary for each warehouse showing its name, location, number of employees, number of shipments handled, and capacity, ordered by warehouses with the highest shipment count first.

### Insight:
The query uses LEFT JOINs so that warehouses are still included even if they have no employees or no shipments. 
COUNT(DISTINCT ...) prevents duplicate counting that could occur due to multiple joins. 
The GROUP BY clause aggregates the data at the warehouse level, producing a summarized operational overview for each warehouse.



### Question:
Find customers who have placed more than 1 order. Show customer name, email, total orders, and total amount spent. Order the results by total amount spent in descending order.

SQL Query:

```
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

### Answer / Result:
Returns customers who have placed more than one order, displaying their full name, email, total number of orders, and total money spent, sorted from highest spender to lowest.

### Insight:
This query combines aggregation and filtering. COUNT(o.order_id) calculates how many orders each customer placed, while SUM(o.total_amount) computes the total spending. 
The HAVING clause filters grouped results to include only customers with more than one order, and ORDER BY total_spent DESC highlights the most valuable customers first.



### Question:
Find all employees whose salary is higher than the average salary of their department. Show employee name, department name, employee salary, department average salary, and salary difference.

SQL Query:

```
WITH department_averages AS (
    SELECT 
    department_id,
    CAST(AVG(salary) AS INTEGER) AS dept_avg_salary
    FROM employees
    GROUP BY department_id
)

SELECT 
CONCAT(e.first_name, ' ', e.last_name) AS employee_name,
d.department_name,
e.salary AS employee_salary,
da.dept_avg_salary,
e.salary - da.dept_avg_salary AS salary_difference
FROM employees e
INNER JOIN departments d
ON e.department_id = d.department_id
INNER JOIN department_averages da
ON e.department_id = da.department_id
WHERE e.salary > da.dept_avg_salary
ORDER BY salary_difference DESC;

### Answer / Result:
Returns employees who earn more than the average salary of their department, displaying their name, department, salary, the department’s average salary, and the difference between the two, ordered by the largest salary difference first.

### Insight:
This query uses a Common Table Expression (CTE) called department_averages to first calculate the average salary for each department. 
The main query then joins this result with the employees and departments tables to compare each employee’s salary against their department’s average. 
The WHERE clause filters only employees whose salary exceeds the department average, and the calculated salary_difference highlights how much higher their salary is.



### Question:
Create a seller performance report showing seller name, total products sold, total revenue, average product rating, and revenue rank. Include only sellers with at least 2 products sold, and rank sellers by revenue using a window function.

SQL Query:

```
SELECT 
s.seller_name as seller_name,
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

### Answer / Result:
Returns a performance report for sellers including their name, number of products sold, total revenue generated, average product rating, and their rank based on revenue, sorted from the highest revenue to the lowest.

### Insight:
This query combines aggregation, joins, and a window function. 
SUM(oi.price * oi.quantity) calculates total revenue per seller, while AVG(r.rating) computes the average rating across their products. 
The window function RANK() OVER (ORDER BY revenue DESC) assigns a revenue-based rank to each seller without collapsing rows further.
The HAVING clause ensures that only sellers with at least two sold products are included in the final report.



### Question:
Create a hierarchical employee report showing each employee with their manager’s name, department, salary difference from their manager, and their rank within their department by salary. Also include the department average salary for comparison.

```
SQL Query:

WITH dept_stats AS (
    SELECT 
        department_id,
        ROUND(AVG(salary), 2) AS dept_avg_salary
    FROM employees
    GROUP BY department_id
)

SELECT 
CONCAT(e.first_name,' ',e.last_name) AS employee_name,
CONCAT(m.first_name,' ',m.last_name) AS manager_name,
d.department_name,
e.salary AS employee_salary,
m.salary AS manager_salary,
COALESCE(m.salary - e.salary, 0) AS salary_difference,
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

### Answer / Result:
Returns a hierarchical employee dataset showing each employee’s name, their manager’s name, department, employee salary, manager salary, salary difference,
their rank within the department by salary, and the average salary of the department.

### Insight:
This query demonstrates several advanced SQL concepts. 
A Common Table Expression (CTE) (dept_stats) calculates the average salary per department.
A self-join (employees e joined to employees m) links each employee to their manager. 
The window function RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) ranks employees by salary within each department. 
COALESCE() ensures that if an employee has no manager, the salary difference defaults to 0 instead of NULL.



### Question:
Identify products that may need restocking. Calculate days of inventory remaining, revenue impact, warehouse location, and assign a restock priority rank. Use order history to estimate sales and rank products based on low stock and high sales volume.

SQL Query:

```
WITH sales_data AS (
    SELECT 
        oi.product_id,
        SUM(oi.quantity) AS total_sold,
        SUM(oi.price * oi.quantity) AS revenue
    FROM order_items oi
    GROUP BY oi.product_id
)

SELECT 
p.product_name as product_name,
p.category as category,
w.location AS warehouse_location,
p.stock_quantity AS current_stock,
COALESCE(ROUND(sd.total_sold / 8.0, 2), 0) AS avg_daily_sales,
CASE 
  WHEN COALESCE(sd.total_sold, 0) > 0 
  THEN ROUND(p.stock_quantity * 1.0 / (sd.total_sold / 8.0), 1)
  ELSE 999
  END AS days_remaining,
  COALESCE(ROUND(sd.revenue, 2), 0) AS potential_revenue_loss,
  RANK() OVER (ORDER BY p.stock_quantity ASC, sd.total_sold DESC) AS restock_rank
FROM products p
INNER JOIN warehouses w
ON p.warehouse_id = w.warehouse_id
LEFT JOIN sales_data sd
ON p.product_id = sd.product_id
ORDER BY restock_rank
LIMIT 5;

### Answer / Result:
Returns the top 5 products most likely needing restocking, including product name, category, warehouse location, current stock level, average daily sales, estimated days of inventory remaining, revenue generated, and a restock priority rank.

### Insight:
This query uses a CTE (sales_data) to summarize product sales and revenue from order history. 
avg_daily_sales estimates daily demand by dividing total sales by a fixed period (8 days in this case). 
The CASE statement calculates days of remaining inventory while preventing division by zero. 
COALESCE() handles products without sales history.
A window function (RANK()) prioritizes products for restocking by ranking low stock first and high sales volume second, helping identify the most urgent inventory risks.



### Question:
Identify each customer’s highest value order. Show customer name, order ID, order amount, and the rank of the order within that customer’s orders.

SQL Query:

```
WITH customer_order_ranks AS (
    SELECT 
        o.order_id,
        CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
        o.total_amount AS order_amount,
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
    order_amount,
    order_rank
FROM customer_order_ranks
WHERE order_rank = 1
ORDER BY order_amount DESC;

### Answer / Result:
Returns the highest-value order for each customer, showing the customer’s name, order ID, order amount, and the rank of that order among the customer’s orders, sorted from largest order value to smallest.

### Insight:
This query uses a CTE (customer_order_ranks) combined with a window function.
RANK() OVER (PARTITION BY o.customer_id ORDER BY o.total_amount DESC) ranks orders within each customer group, placing the largest order at rank 1. 
The outer query filters to only those rows where order_rank = 1, effectively returning each customer’s highest-value purchase.



### Question:
Calculate the longest gap (in days) between consecutive orders for each customer who has placed at least 3 orders. Show customer name, email, total orders, average gap in days, and maximum gap in days.

```
SQL Query:

WITH order_gaps AS (
    SELECT 
        o.customer_id as customer_id,
        o.order_date as order_date,
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
        ROUND(AVG(gap_days), 1) AS avg_gap_days,
        ROUND(MAX(gap_days), 1) AS max_gap_days
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

### Answer / Result:
Returns customers who have placed at least three orders, displaying their name, email, total number of orders, average number of days between orders, and the longest gap between any two consecutive orders, sorted by the largest gap first.

### Insight:
This query uses the window function LAG() to access the previous order date for each customer. 
By subtracting the two dates using julianday(), it calculates the gap in days between consecutive orders. 
The first CTE (order_gaps) computes individual order gaps, while the second CTE (customer_gap_stats) aggregates these gaps to calculate average and maximum gaps per customer. 
Filtering for customers with at least three orders ensures the analysis is meaningful for repeat buyers.



### Question:
Create a seller performance report showing seller name, total products sold, total revenue, average product rating, and revenue rank. Include only sellers with at least 2 products sold 
and rank sellers by revenue using a window function.

SQL Query:

```
SELECT 
s.seller_name AS seller_name,
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

### Answer / Result:
Returns a seller performance summary including seller name, number of products sold, total revenue generated, average product rating, and their rank based on revenue, sorted from highest to lowest revenue.

### Insight:
This query combines aggregation and window functions for analytical reporting. 
SUM(oi.price * oi.quantity) calculates each seller’s total revenue, while AVG(r.rating) computes the average rating across their products. 
The window function RANK() OVER (ORDER BY revenue DESC) ranks sellers based on revenue without affecting the grouped results. 
The HAVING clause filters the dataset to include only sellers who have sold at least two products, ensuring the report focuses on active sellers.
