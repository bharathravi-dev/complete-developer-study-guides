# 🚀 SQL Roadmap 2026 — Part 6

## 📊 GROUP BY & HAVING — Analyzing Data by Categories

In Part 5, you learned how aggregate functions answer questions like:

> - What is the total revenue?
> - How many customers do we have?

But real-world business questions are usually more specific:

> - What is the revenue **by city**?
> - How many employees are there **in each department**?
> - Which products generated the most revenue?

That's where `GROUP BY` comes in.

---

## 1️⃣ What is GROUP BY?

`GROUP BY` combines rows with the same value into **groups** so that aggregate functions can calculate a metric for each group.

**Basic Syntax:**

```sql
SELECT
    column_name,
    aggregate_function(column)
FROM table_name
GROUP BY column_name;
```

**Example:**

```sql
SELECT
    department,
    COUNT(*) AS employee_count
FROM employees
GROUP BY department;
```

Instead of getting one total employee count, you get a count for **each department**.

---

## 2️⃣ Why Do We Need GROUP BY?

**Without `GROUP BY`:**

```sql
SELECT COUNT(*) AS total_employees
FROM employees;
```

Result: `1000` — this answers *How many employees are there?*

**With `GROUP BY`:**

```sql
SELECT
    department,
    COUNT(*) AS employee_count
FROM employees
GROUP BY department;
```

**Result:**

| department | employee_count |
| ---------- | -------------- |
| IT         | 350            |
| Finance    | 200            |
| HR         | 120            |
| Sales      | 330            |

Now you can answer: *How many employees are in each department?*

---

## 3️⃣ GROUP BY With COUNT()

This is probably the most common `GROUP BY` pattern.

```sql
SELECT
    city,
    COUNT(*) AS customer_count
FROM customers
GROUP BY city;
```

---

## 4️⃣ GROUP BY With SUM()

Suppose you want revenue by city.

```sql
SELECT
    city,
    SUM(amount) AS total_revenue
FROM orders
GROUP BY city;
```

> This is a common business KPI.

---

## 5️⃣ GROUP BY With AVG()

Calculate average salary by department:

```sql
SELECT
    department,
    AVG(salary) AS average_salary
FROM employees
GROUP BY department;
```

---

## 6️⃣ GROUP BY With MIN() and MAX()

You can use multiple aggregate functions.

```sql
SELECT
    department,
    MIN(salary) AS minimum_salary,
    MAX(salary) AS maximum_salary,
    AVG(salary) AS average_salary
FROM employees
GROUP BY department;
```

---

## 7️⃣ Multiple Aggregations

You aren't limited to one metric.

```sql
SELECT
    department,
    COUNT(*)    AS employees,
    SUM(salary) AS total_salary,
    AVG(salary) AS average_salary,
    MIN(salary) AS minimum_salary,
    MAX(salary) AS maximum_salary
FROM employees
GROUP BY department;
```

> This is the foundation of many analytical reports.

---

## 8️⃣ GROUP BY Multiple Columns

You can group by more than one column.

**Example:** Count customers by city **and** customer segment.

```sql
SELECT
    city,
    customer_segment,
    COUNT(*) AS customer_count
FROM customers
GROUP BY
    city,
    customer_segment;
```

SQL creates a group for **each unique combination**.

---

## 9️⃣ Understanding Multiple GROUP BY Columns

Grouping by `GROUP BY city, segment` creates groups like:

- Mumbai + Premium
- Mumbai + Standard
- Delhi + Premium

> The **combination** matters.

---

## 🔟 GROUP BY With WHERE

`WHERE` filters rows **before** grouping.

**Example:** Calculate revenue by city for completed orders only.

```sql
SELECT
    city,
    SUM(amount) AS revenue
FROM orders
WHERE order_status = 'Completed'
GROUP BY city;
```

Conceptually:

```text
All Orders → WHERE Completed → GROUP BY City → SUM Revenue
```

---

## 1️⃣1️⃣ WHERE vs GROUP BY

| Clause | Answers |
| ------ | ------- |
| `WHERE` | Which rows should be included? |
| `GROUP BY` | How should those rows be divided into groups? |

---

## 1️⃣2️⃣ What is HAVING?

`HAVING` filters groups **after** aggregation.

**Example:** Find departments with more than 100 employees.

```sql
SELECT
    department,
    COUNT(*) AS employee_count
FROM employees
GROUP BY department
HAVING COUNT(*) > 100;
```

---

## 1️⃣3️⃣ WHERE vs HAVING

> This is one of the most frequently asked SQL interview questions.

| Clause | Filters | Example |
| ------ | ------- | ------- |
| `WHERE` | Individual **rows** | `WHERE salary > 500000` |
| `HAVING` | **Groups** | `HAVING AVG(salary) > 800000` |

**Remember:**

```text
WHERE    → Filter rows
GROUP BY → Create groups
HAVING   → Filter groups
```

---

## 1️⃣4️⃣ Example: WHERE + GROUP BY + HAVING

**Requirement:** Find departments whose average salary is greater than ₹8 lakh, considering only employees earning more than ₹5 lakh.

```sql
SELECT
    department,
    AVG(salary) AS average_salary
FROM employees
WHERE salary > 500000
GROUP BY department
HAVING AVG(salary) > 800000;
```

---

## 1️⃣5️⃣ GROUP BY With COUNT(DISTINCT)

Very useful for customer analytics.

```sql
SELECT
    DATE_TRUNC('month', order_date) AS month,
    COUNT(DISTINCT customer_id)     AS unique_customers
FROM orders
GROUP BY DATE_TRUNC('month', order_date)
ORDER BY month;
```

---

## 1️⃣6️⃣ GROUP BY With CASE

You can create business categories and then group them.

```sql
SELECT
    CASE
        WHEN salary >= 1000000 THEN 'High'
        WHEN salary >=  600000 THEN 'Medium'
        ELSE 'Low'
    END AS salary_band,
    COUNT(*) AS employee_count
FROM employees
GROUP BY
    CASE
        WHEN salary >= 1000000 THEN 'High'
        WHEN salary >=  600000 THEN 'Medium'
        ELSE 'Low'
    END;
```

---

## 1️⃣7️⃣ GROUP BY Dates

> This is extremely important for Data Analysts.

**Example:** Calculate monthly revenue.

```sql
SELECT
    DATE_TRUNC('month', order_date) AS month,
    SUM(amount)                     AS revenue
FROM orders
GROUP BY DATE_TRUNC('month', order_date)
ORDER BY month;
```

---

## 1️⃣8️⃣ Daily Sales

```sql
SELECT
    order_date,
    SUM(amount) AS daily_revenue
FROM orders
GROUP BY order_date
ORDER BY order_date;
```

---

## 1️⃣9️⃣ Monthly Order Count

```sql
SELECT
    DATE_TRUNC('month', order_date) AS month,
    COUNT(*)                        AS order_count
FROM orders
GROUP BY DATE_TRUNC('month', order_date)
ORDER BY month;
```

---

## 2️⃣0️⃣ Monthly Customer Count

```sql
SELECT
    DATE_TRUNC('month', order_date) AS month,
    COUNT(DISTINCT customer_id)     AS active_customers
FROM orders
GROUP BY DATE_TRUNC('month', order_date)
ORDER BY month;
```

> ⚠️ Notice: `COUNT(*)` counts **orders**, while `COUNT(DISTINCT customer_id)` counts **unique customers**.

---

## 2️⃣1️⃣ GROUP BY With ORDER BY

**Example:** Find departments with the highest average salary.

```sql
SELECT
    department,
    AVG(salary) AS average_salary
FROM employees
GROUP BY department
ORDER BY average_salary DESC;
```

---

## 2️⃣2️⃣ GROUP BY + HAVING + ORDER BY

A powerful analytical pattern:

```sql
SELECT
    customer_id,
    SUM(amount) AS total_spend
FROM orders
GROUP BY customer_id
HAVING SUM(amount) > 50000
ORDER BY total_spend DESC;
```

---

## 2️⃣3️⃣ Real-World Example: Top Revenue Categories

**Requirement:** Find categories generating more than ₹10 lakh in revenue.

```sql
SELECT
    p.category,
    SUM(oi.quantity * oi.selling_price) AS revenue
FROM products p
JOIN order_items oi
    ON p.product_id = oi.product_id
GROUP BY p.category
HAVING SUM(oi.quantity * oi.selling_price) > 1000000
ORDER BY revenue DESC;
```

---

## 2️⃣4️⃣ Common GROUP BY Error

```sql
SELECT
    department,
    employee_name,
    AVG(salary)
FROM employees
GROUP BY department;
```

> ❌ This is generally **invalid** because `employee_name` is neither grouped nor aggregated.

---

## 2️⃣5️⃣ The Golden Rule of GROUP BY

When using `GROUP BY`, every selected expression generally needs to be either:

1. Included in `GROUP BY`, **or**
2. Aggregated

> 💡 Think: **group columns describe the group; aggregate functions summarize the group.**

---

## 2️⃣6️⃣ SQL Query Pattern to Memorize

```sql
SELECT
    grouping_column,
    AGGREGATE_FUNCTION(value_column) AS metric
FROM table_name
WHERE row_condition
GROUP BY grouping_column
HAVING group_condition
ORDER BY metric DESC;
```

**Example:**

```sql
SELECT
    city,
    SUM(amount) AS revenue
FROM orders
WHERE order_status = 'Completed'
GROUP BY city
HAVING SUM(amount) > 100000
ORDER BY revenue DESC;
```

---

## 🧠 Logical Processing Order

A useful simplified model is:

```text
FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
```

> This helps explain why `WHERE SUM(amount) > 100000` is not valid. Use `HAVING` instead.

---

## 💼 SQL Interview Questions

**Q1. What is `GROUP BY`?**

Groups rows with the same values so aggregate functions can calculate metrics for each group.

**Q2. What is the difference between `WHERE` and `HAVING`?**

`WHERE` filters rows before grouping, while `HAVING` filters groups after aggregation.

**Q3. Can `GROUP BY` contain multiple columns?**

Yes — `GROUP BY city, category;`

**Q4. Can `GROUP BY` be used without an aggregate function?**

Yes, although `SELECT DISTINCT` is often clearer when the goal is simply to return unique combinations.

**Q5. Can you use aggregate functions in `WHERE`?**

Generally no. Use `HAVING`.

**Q6. Why do we use `COUNT(DISTINCT customer_id)`?**

To count unique customers rather than counting every transaction.

---

## 🎯 Practice Questions

1. Count employees in each department.
2. Calculate total revenue by product category.
3. Calculate average salary by department.
4. Find the highest salary in each department.
5. Count customers by city.
6. Find cities with more than 500 customers.
7. Calculate monthly revenue.
8. Calculate monthly unique customers.
9. Find customers whose total spending is greater than ₹50,000.
10. Find product categories generating more than ₹1 lakh revenue, sorted from highest to lowest.

### ✅ Answers

```sql
-- A1
SELECT department, COUNT(*) AS employee_count
FROM employees
GROUP BY department;

-- A2
SELECT category, SUM(amount) AS revenue
FROM sales
GROUP BY category;

-- A3
SELECT department, AVG(salary) AS average_salary
FROM employees
GROUP BY department;

-- A4
SELECT department, MAX(salary) AS highest_salary
FROM employees
GROUP BY department;

-- A5
SELECT city, COUNT(*) AS customer_count
FROM customers
GROUP BY city;

-- A6
SELECT city, COUNT(*) AS customer_count
FROM customers
GROUP BY city
HAVING COUNT(*) > 500;

-- A7
SELECT DATE_TRUNC('month', order_date) AS month, SUM(amount) AS revenue
FROM orders
GROUP BY DATE_TRUNC('month', order_date)
ORDER BY month;

-- A8
SELECT DATE_TRUNC('month', order_date) AS month, COUNT(DISTINCT customer_id) AS unique_customers
FROM orders
GROUP BY DATE_TRUNC('month', order_date)
ORDER BY month;

-- A9
SELECT customer_id, SUM(amount) AS total_spend
FROM orders
GROUP BY customer_id
HAVING SUM(amount) > 50000
ORDER BY total_spend DESC;

-- A10
SELECT category, SUM(amount) AS revenue
FROM sales
GROUP BY category
HAVING SUM(amount) > 100000
ORDER BY revenue DESC;
```

---

## 🔥 Mini Challenge

You have an `orders` table with columns: `order_id`, `customer_id`, `city`, `amount`, `status`

Find each city's:

- Completed order count
- Unique customers
- Total revenue
- Average order value

Only include cities where completed revenue is greater than ₹10,000. Sort by revenue from highest to lowest.

**Solution:**

```sql
SELECT
    city,
    COUNT(*)                    AS completed_orders,
    COUNT(DISTINCT customer_id) AS unique_customers,
    SUM(amount)                 AS revenue,
    AVG(amount)                 AS average_order_value
FROM orders
WHERE status = 'Completed'
GROUP BY city
HAVING SUM(amount) > 10000
ORDER BY revenue DESC;
```

---

💡 **Double Tap ❤️ For Part-7**
