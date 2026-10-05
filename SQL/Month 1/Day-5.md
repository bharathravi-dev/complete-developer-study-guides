🚀 *SQL Roadmap 2026 — Part 5*

*Aggregate Functions: COUNT, SUM, AVG, MIN & MAX* 📊

So far, you've learned how to retrieve, filter, and sort individual rows.

Now we're moving to one of the most important skills for a Data Analyst:
> Turning thousands of rows into meaningful business metrics.

For example:
How many customers do we have?
What is our total revenue?
What is the average order value?
What is the highest salary?
What is the lowest product price?

That's exactly what aggregate functions are designed for.

*1️⃣ What Are Aggregate Functions?*

Aggregate functions perform a calculation across multiple rows and return a summarized result.

The five essential functions are:

- COUNT() Counts rows/values
- SUM() Calculates total
- AVG() Calculates average
- MIN() Finds minimum
- MAX() Finds maximum


*2️⃣ COUNT()*

COUNT() is used to count records or non-NULL values.

Count all rows

SELECT COUNT(*) AS total_customers
FROM customers;

If there are 5,000 customers: total_customers = 5000

*3️⃣ COUNT(*) vs COUNT(column)*

This distinction is extremely important.

COUNT(*) Counts rows.

SELECT COUNT(*)
FROM employees;

COUNT(column) Counts non-NULL values in that column.

SELECT COUNT(manager_id)
FROM employees;

Suppose: employee A | 101, B | 102, C | NULL, D | 103

Then: COUNT(_) = 4, COUNT(manager_id) = 3

Because one manager_id is NULL.

Interview Tip: 
> COUNT(*) counts rows; COUNT(column) counts non-NULL values in that column.

*4️⃣ COUNT(DISTINCT)*

Use COUNT(DISTINCT ...) when you want to count unique values.

Example:
SELECT
    COUNT(DISTINCT customer_id) AS unique_customers
FROM orders;

Suppose: customer_id 101, 101, 102, 103, 103, 103

Then: COUNT(*) = 6, COUNT(DISTINCT id) = 3

This is extremely common in analytics.

*5️⃣ Real-World Example: Active Customers*

Suppose your orders table contains thousands of orders.

The business asks: 
> How many unique customers placed an order?

SELECT
    COUNT(DISTINCT customer_id) AS active_customers
FROM orders;

Notice that we're counting customers, not orders. One customer may have placed 20 orders, but should still count as one unique customer.

*6️⃣ SUM()*

SUM() calculates the total of a numeric column.

Example:
SELECT
    SUM(amount) AS total_revenue
FROM orders;

If the amounts are: 1000, 2000, 1500, 3000 then: SUM = 7500

*7️⃣ SUM With a Condition*

You can combine SUM() with WHERE.

Example: 
> Calculate revenue from completed orders only.

SELECT
    SUM(amount) AS completed_revenue
FROM orders
WHERE order_status = 'Completed';

This is a very common business query.

*8️⃣ AVG()*

AVG() calculates the average of non-NULL numeric values.

Example:
SELECT
    AVG(salary) AS average_salary
FROM employees;

If salaries are: 50000, 60000, 70000 then: Average = 60000

*9️⃣ AVG and NULL Values*

AVG() generally ignores NULL values.

Suppose: salary 50000, 60000, NULL, 70000

The average is: (50000 + 60000 + 70000) / 3 = 60000

It doesn't divide by 4. This is important when working with incomplete real-world data.

*🔟 MIN()*

MIN() finds the smallest value.

Example:
SELECT
    MIN(salary) AS lowest_salary
FROM employees;
For products:
SELECT
    MIN(price) AS lowest_price
FROM products;

*1️⃣1️⃣ MAX()*

MAX() finds the largest value.

SELECT
    MAX(salary) AS highest_salary
FROM employees;

SELECT
    MAX(amount) AS largest_order
FROM orders;

*1️⃣2️⃣ Using Multiple Aggregate Functions*

You can use several aggregate functions in the same query.

SELECT
    COUNT(*) AS total_orders,
    SUM(amount) AS total_revenue,
    AVG(amount) AS average_order_value,
    MIN(amount) AS smallest_order,
    MAX(amount) AS largest_order
FROM orders;

This single query gives you a basic sales summary.

*1️⃣3️⃣ Aggregate Functions With WHERE*

Example: 
> Analyze completed orders only.

SELECT
    COUNT(*) AS completed_orders,
    SUM(amount) AS revenue,
    AVG(amount) AS average_order_value,
    MIN(amount) AS smallest_order,
    MAX(amount) AS largest_order
FROM orders
WHERE order_status = 'Completed';

This is a powerful analytical pattern.

*1️⃣4️⃣ NULL and SUM()*

SUM() generally ignores NULL values.

Suppose: 
amount 1000, 2000, NULL, 3000 

Then: SUM(amount) = 6000

However, if all values are NULL, the result can be NULL rather than 0.

You can handle that later using COALESCE().

Example:
SELECT
    COALESCE(SUM(amount), 0) AS total_revenue
FROM orders
WHERE order_status = 'Completed';

*1️⃣5️⃣ Aggregate Functions Are the Foundation of KPIs*

Most business dashboards are built using aggregate functions.

For example:
Revenue = SUM(amount)
Number of Orders = COUNT(*)
Customers = COUNT(DISTINCT customer_id)
Average Order Value = AVG(amount)
Largest Order = MAX(amount)

This is why mastering aggregates is critical.

*1️⃣6️⃣ Calculating Average Order Value*

A common e-commerce KPI is AOV — Average Order Value.

A simple version:

SELECT
    AVG(amount) AS average_order_value
FROM orders
WHERE order_status = 'Completed';
Another formulation is:
SELECT
    SUM(amount) / COUNT(*) AS average_order_value
FROM orders
WHERE order_status = 'Completed';

The AVG() version is usually clearer when each row represents one order.

*1️⃣7️⃣ Calculating Revenue Per Customer*

Suppose the business asks: 
> What is the average revenue generated per unique customer?

You need to be careful not to divide revenue by the number of orders.

SELECT
    SUM(amount) /
    COUNT(DISTINCT customer_id) AS revenue_per_customer
FROM orders
WHERE order_status = 'Completed';

This is a good example of translating a business metric into SQL.

*1️⃣8️⃣ Aggregate Functions + Expressions*

You can aggregate calculations.

Example:
SELECT
    SUM(quantity * unit_price) AS total_sales
FROM order_items;

SQL first evaluates: quantity × unit_price for each row, then sums those values.

*1️⃣9️⃣ Aggregate Functions + CASE*

You can create conditional metrics.

Example:
SELECT
    COUNT(*) AS total_orders,
    SUM(
        CASE
            WHEN order_status = 'Completed'
            THEN 1
            ELSE 0
        END
    ) AS completed_orders
FROM orders;

This technique becomes extremely important when building dashboards.

*2️⃣0️⃣ Example: Success Rate*

Suppose you have payment transactions. You want: 
> Percentage of successful transactions.

SELECT
    100.0 *
    SUM(
        CASE
            WHEN status = 'Success'
            THEN 1
            ELSE 0
        END
    ) / COUNT(*) AS success_rate
FROM transactions;

This combines: COUNT + SUM + CASE + Arithmetic. 

*2️⃣1️⃣ Why GROUP BY Comes Next*

At the moment:
SELECT
    SUM(amount)
FROM orders;
gives you one total.

But what if the business asks: 
> What is the revenue for each city?

Now you need:

SELECT
    city,
    SUM(amount) AS revenue
FROM orders
GROUP BY city;

For now, understand the difference: Aggregate only ↓ One summary, GROUP BY + Aggregate ↓ One summary per group

*2️⃣2️⃣ COUNT DISTINCT in Business Analytics*

Suppose orders table has 5 orders, customer 101 appears twice, 103 appears twice. Total orders = COUNT(*) = 5, Unique customers = COUNT(DISTINCT customer_id) = 3. This distinction is fundamental.

*2️⃣3️⃣ Common Mistake: COUNT(*) vs COUNT(DISTINCT)*

If a customer places multiple orders: Customer 101 ↓ Order 1, Order 2, Order 3

Then: COUNT(*) counts: 3 while: COUNT(DISTINCT customer_id) counts: 1


*2️⃣4️⃣ Real-World Dashboard Query*

Imagine your manager asks for a quick sales summary.

SELECT
    COUNT(*) AS total_orders,
    COUNT(DISTINCT customer_id) AS unique_customers,
    SUM(amount) AS total_revenue,
    AVG(amount) AS average_order_value,
    MIN(amount) AS minimum_order,
    MAX(amount) AS maximum_order
FROM orders
WHERE order_status = 'Completed';

This gives you six useful business metrics in one query.

🧠 *Common Beginner Mistakes*

❌ Mistake 1: Counting the wrong thing. 
Don't automatically use: COUNT(*) when the requirement says: 
> Number of customers. Use: COUNT(DISTINCT customer_id) when appropriate.

❌ Mistake 2: Assuming NULL is zero. 
NULL → Missing/unknown, 0 → Actual numeric zero

❌ Mistake 3: Using SUM on text. 
SUM() is designed for numeric expressions. This is invalid or inappropriate: SUM(customer_name)

❌ Mistake 4: Forgetting the business definition. 
"Revenue" might mean: Gross revenue, Net revenue, Completed-order revenue, Revenue after discounts, Revenue excluding refunds. Always understand the business definition before writing the SQL.

💼 *SQL Interview Questions*

Q1. What is an aggregate function? 
An aggregate function performs a calculation over multiple rows and returns a summarized value.

Q2. Name five common aggregate functions. COUNT(), SUM(), AVG(), MIN(), MAX()

Q3. Difference between COUNT(*) and COUNT(column)? 
COUNT(*) counts rows, while COUNT(column) counts non-NULL values in that column.

Q4. What does COUNT(DISTINCT customer_id) do? 
It counts the number of unique non-NULL customer IDs.

Q5. Does AVG ignore NULL values? 
Yes, AVG() normally ignores NULL values.

Q6. How do you calculate total revenue? 
SELECT SUM(amount) FROM orders;

Q7. How do you find the highest salary?
SELECT MAX(salary) FROM employees;

Q8. Can multiple aggregate functions be used together? 
Yes.

*🎯 Practice Questions*

Q1. Find the total number of employees.
Q2. Find the average employee salary.
Q3. Find the highest product price.
Q4. Find the lowest product price.
Q5. Calculate total revenue from completed orders.
Q6. Count the number of unique customers who placed an order.
Q7. Find the largest order amount.
Q8. Calculate the average order value for completed orders.
Q9. Count the number of completed orders.
Q10. Calculate total revenue and total unique customers from completed orders.

✅ *Answers*

Answer 1
SELECT COUNT(*) AS total_employees
FROM employees;

Answer 2
SELECT AVG(salary) AS average_salary
FROM employees;

Answer 3
SELECT MAX(price) AS highest_price
FROM products;

Answer 4
SELECT MIN(price) AS lowest_price
FROM products;

Answer 5
SELECT
    SUM(amount) AS total_revenue
FROM orders
WHERE order_status = 'Completed';

Answer 6
SELECT
    COUNT(DISTINCT customer_id) AS unique_customers
FROM orders;

Answer 7
SELECT
    MAX(amount) AS largest_order
FROM orders;

Answer 8
SELECT
    AVG(amount) AS average_order_value
FROM orders
WHERE order_status = 'Completed';

Answer 9
SELECT
    COUNT(*) AS completed_orders
FROM orders
WHERE order_status = 'Completed';

Answer 10
SELECT
    SUM(amount) AS total_revenue,
    COUNT(DISTINCT customer_id) AS unique_customers
FROM orders
WHERE order_status = 'Completed';

🔥 *Mini Challenge*

You have an orders table: order_id | customer_id | amount | status

Business requirement: Calculate: Total completed orders, Unique completed customers, Total completed revenue, Average completed order value, Largest completed order

*Solution*
SELECT
    COUNT(*) AS completed_orders,
    COUNT(DISTINCT customer_id) AS unique_customers,
    SUM(amount) AS total_revenue,
    AVG(amount) AS average_order_value,
    MAX(amount) AS largest_order
FROM orders
WHERE status = 'Completed';

Expected result: completed_orders = 4, unique_customers = 3, total_revenue = 15500, average_order_value = 3875, largest_order = 6000

*Double Tap ❤️ For Part-6*