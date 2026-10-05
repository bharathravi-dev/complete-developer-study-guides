# 🚀 SQL Roadmap 2026 — Part 12

## 🧩 SQL Subqueries — Using One Query Inside Another

So far, we've learned how to retrieve, filter, group, transform, and combine data.

But sometimes a business question requires **one query to use the result of another query**.

For example:

> «Find employees whose salary is higher than the average salary.»

First, we need to calculate:

```sql
SELECT AVG(salary)
FROM employees;
```

Then compare every employee against that result.

A **subquery** allows us to do both inside one SQL statement.

---

## 🧠 1. What Is a Subquery?

A subquery is a SQL query written **inside another SQL query**.

```sql
SELECT *
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
);
```

- The **inner query** `SELECT AVG(salary) FROM employees` calculates the average salary.
- The **outer query** then uses that result.

Think of it as:

```text
Outer Query → Needs an answer → Subquery calculates the answer → Outer Query uses it
```

---

## 🔹 2. Basic Subquery Structure

```sql
SELECT column_name
FROM table_name
WHERE column_name operator (
    SELECT ...
);
```

The inner query is always enclosed in parentheses `( ... )`.

---

## 📊 3. Subquery Returning One Value

A **scalar subquery** returns a **single value** (one row, one column).

```sql
SELECT
    employee_name,
    salary
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
);
```

The inner query returns something like `65000`. The outer query then finds employees earning more than `65000`.

---

## 💰 4. Employees Earning Above Average

Classic interview question.

```sql
SELECT
    employee_name,
    salary
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
);
```

**Logic:**

```text
Calculate average salary → Compare every employee → Keep salary > average
```

---

## 🏆 5. Products More Expensive Than Average

```sql
SELECT
    product_name,
    price
FROM products
WHERE price > (
    SELECT AVG(price)
    FROM products
);
```

---

## 🔢 6. Subquery with COUNT()

Customers who have placed **more than 5 orders**:

```sql
SELECT
    customer_id,
    customer_name
FROM customers
WHERE customer_id IN (
    SELECT customer_id
    FROM orders
    GROUP BY customer_id
    HAVING COUNT(*) > 5
);
```

The subquery produces the list of qualifying `customer_id`s; the outer query fetches their details.

---

## 🔗 7. Subquery with IN

Useful when the subquery returns **multiple values**.

```sql
SELECT *
FROM customers
WHERE customer_id IN (
    SELECT customer_id
    FROM orders
);
```

Returns customers who have **at least one order**.

---

## ⚠️ 8. Single Value vs Multiple Values

| Subquery returns | Use                            |
| ---------------- | ------------------------------ |
| One value        | `=`, `>`, `<`, `>=`, `<=`, `<>` |
| Multiple values  | `IN`, `NOT IN`, `EXISTS`       |

**One value:**

```sql
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
)
```

**Multiple values:**

```sql
WHERE customer_id IN (
    SELECT customer_id
    FROM orders
)
```

If you use `=` and the subquery returns more than one row, most databases raise an error like *"subquery returns more than one row"*.

---

## 🚫 9. NOT IN

```sql
SELECT
    customer_id,
    customer_name
FROM customers
WHERE customer_id NOT IN (
    SELECT customer_id
    FROM orders
);
```

### ⚠️ Important NULL Warning

`NOT IN` can produce unexpected results if the subquery contains a `NULL`.

Suppose the subquery returns `(1, 2, NULL)`. Then:

```text
customer_id NOT IN (1, 2, NULL)
= customer_id <> 1 AND customer_id <> 2 AND customer_id <> NULL
= ... AND UNKNOWN
= never TRUE
```

Result: **zero rows**, even if many customers have no orders.

Fixes:

- Filter NULLs: `WHERE customer_id IS NOT NULL` inside the subquery, or
- Use `NOT EXISTS` (safer for anti-matching).

---

## 🔎 10. EXISTS

```sql
SELECT
    c.customer_id,
    c.customer_name
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);
```

Means: «Return the customer if **at least one** matching order exists.»

`SELECT 1` is a convention — `EXISTS` only cares whether a row exists, not what columns it returns.

---

## 🚫 11. NOT EXISTS

```sql
SELECT
    c.customer_id,
    c.customer_name
FROM customers c
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);
```

Means: «Return customers for whom **no matching order** exists.»

Unlike `NOT IN`, this is not affected by NULLs in `orders.customer_id`.

---

## 🧠 12. EXISTS vs IN

| Operator | What it does                              |
| -------- | ----------------------------------------- |
| `IN`     | Compares against a **set of returned values** |
| `EXISTS` | Checks whether **a matching row exists**      |

---

## 🔄 13. Correlated Subquery

A **correlated subquery** depends on the **current row of the outer query**. It is conceptually re-evaluated for every outer row.

```sql
SELECT
    e.employee_name,
    e.salary,
    e.department_id
FROM employees e
WHERE e.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department_id = e.department_id
);
```

> «Is this employee's salary higher than the average salary of **their own department**?»

Notice `e2.department_id = e.department_id` — the inner query references `e` from the outer query.

---

## 🏢 14. Above-Department-Average Salary

Sample data:

| employee_name | department | salary |
| ------------- | ---------- | ------ |
| Alice         | Sales      | 50000  |
| Bob           | Sales      | 70000  |
| Charlie       | IT         | 80000  |
| David         | IT         | 100000 |

Department averages:

| department | avg_salary |
| ---------- | ---------- |
| Sales      | 60000      |
| IT         | 90000      |

The correlated subquery compares **Bob against the Sales average** (70000 > 60000 ✅) and **David against the IT average** (100000 > 90000 ✅).

Result: **Bob, David**

Note: Charlie (80000) earns more than Bob, but is *below* his own department average — so he's excluded.

---

## 📦 15. Subquery in FROM

A subquery can appear in the `FROM` clause.

```sql
SELECT
    customer_id,
    total_spending
FROM (
    SELECT
        customer_id,
        SUM(amount) AS total_spending
    FROM orders
    GROUP BY customer_id
) AS customer_sales;
```

This is often called a **derived table**. Most databases require it to have an alias (`AS customer_sales`).

---

## 📈 16. Finding High-Value Customers

```sql
SELECT
    customer_id,
    total_spending
FROM (
    SELECT
        customer_id,
        SUM(amount) AS total_spending
    FROM orders
    GROUP BY customer_id
) AS customer_sales
WHERE total_spending > 50000;
```

The derived table changes the grain to **1 row = 1 customer**, so we can filter on `total_spending` with a normal `WHERE`.

---

## 🧩 17. Subquery in SELECT

```sql
SELECT
    c.customer_name,
    (
        SELECT COUNT(*)
        FROM orders o
        WHERE o.customer_id = c.customer_id
    ) AS order_count
FROM customers c;
```

A subquery in `SELECT` must return **a single value per row**.

---

## ⚠️ 18. But Be Careful with Correlated Subqueries

Instead of a correlated subquery in `SELECT`, you could use:

```sql
SELECT
    c.customer_name,
    COUNT(o.order_id) AS order_count
FROM customers c
LEFT JOIN orders o
    ON c.customer_id = o.customer_id
GROUP BY
    c.customer_id,
    c.customer_name;
```

The better choice depends on the database optimizer, table size, indexes, etc.

---

## 🧱 19. Nested Subqueries

```sql
SELECT *
FROM products
WHERE price > (
    SELECT AVG(price)
    FROM products
    WHERE category_id IN (
        SELECT category_id
        FROM categories
        WHERE category_name = 'Electronics'
    )
);
```

Deeply nested queries can become difficult to read. Use **CTEs** instead (next part 👉).

---

## 🧠 20. Subqueries vs JOINs

**Subquery:**

```sql
SELECT *
FROM customers
WHERE customer_id IN (
    SELECT customer_id
    FROM orders
);
```

**JOIN:**

```sql
SELECT DISTINCT
    c.*
FROM customers c
JOIN orders o
    ON c.customer_id = o.customer_id;
```

The JOIN needs `DISTINCT` because a customer with many orders would otherwise appear many times (one-to-many!). The subquery doesn't have this problem.

Choose based on **readability**, **business logic**, and **performance**.

---

## 🏆 21. Subqueries for Business Analysis

**Find products above average:**

```sql
SELECT product_name, price
FROM products
WHERE price > (SELECT AVG(price) FROM products);
```

**Find customers with at least one order:**

```sql
SELECT customer_id, customer_name
FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id
);
```

**Find customers with more than 10 orders:**

```sql
SELECT customer_id, customer_name
FROM customers
WHERE customer_id IN (
    SELECT customer_id
    FROM orders
    GROUP BY customer_id
    HAVING COUNT(*) > 10
);
```

---

## 🧮 22. Subquery for KPI Comparison

Orders larger than the average order value:

```sql
SELECT order_id, customer_id, amount
FROM orders
WHERE amount > (
    SELECT AVG(amount) FROM orders
);
```

---

## ⚠️ 23. Common Subquery Mistakes

**Mistake 1 — Returning multiple rows with `=`**

Use `IN` instead of `=` when multiple rows are expected.

**Mistake 2 — Forgetting NULL behavior with `NOT IN`**

A single `NULL` in the subquery can make `NOT IN` return nothing. Prefer `NOT EXISTS`.

**Mistake 3 — Ignoring duplicates**

Replacing an `IN` subquery with a JOIN can multiply rows.

**Mistake 4 — Making the query unnecessarily complicated**

If a simple JOIN or CTE is clearer, use it.

---

## 🎤 SQL Interview Questions

**Q1. What is a subquery?**

A query nested inside another query.

**Q2. What is a scalar subquery?**

A subquery that returns a single value (one row, one column).

**Q3. When do you use `IN`?**

When the subquery returns a set of values.

**Q4. What does `EXISTS` do?**

Checks whether at least one matching row exists.

**Q5. What is a correlated subquery?**

A subquery that references a column from the outer query, so it is evaluated per outer row.

**Q6. What is a derived table?**

A subquery in the `FROM` clause, treated as a temporary result set.

**Q7. Can a subquery be used in `SELECT`?**

Yes — as long as it returns a single value per row.

**Q8. Difference between `IN` and `EXISTS`?**

`IN` compares against a set of values; `EXISTS` checks for the existence of matching rows.

**Q9. Why is `NOT IN` dangerous with NULL?**

Because of three-valued logic: comparing with `NULL` yields `UNKNOWN`, so the whole `NOT IN` condition can never be `TRUE`.

**Q10. Can every subquery be replaced with a JOIN?**

Many can, but the best approach depends on the logic, readability, and performance.

---

## 📝 Practice Questions

**Practice 1 — Employees earning more than average**

```sql
SELECT employee_name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

**Practice 2 — Customers with at least one order**

```sql
SELECT customer_id, customer_name
FROM customers
WHERE customer_id IN (SELECT customer_id FROM orders);
```

**Practice 3 — Customers with more than 5 orders**

```sql
SELECT customer_id, customer_name
FROM customers
WHERE customer_id IN (
    SELECT customer_id
    FROM orders
    GROUP BY customer_id
    HAVING COUNT(*) > 5
);
```

**Practice 4 — Products higher than average price**

```sql
SELECT product_name, price
FROM products
WHERE price > (SELECT AVG(price) FROM products);
```

**Practice 5 — Customers who never placed an order**

```sql
SELECT c.customer_id, c.customer_name
FROM customers c
WHERE NOT EXISTS (
    SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id
);
```

---

## 🧪 Mini SQL Challenge

> Find customers whose total spending is greater than the average customer spending.

**Solution:**

```sql
SELECT customer_id, total_spending
FROM (
    SELECT customer_id, SUM(amount) AS total_spending
    FROM orders
    GROUP BY customer_id
) AS customer_totals
WHERE total_spending > (
    SELECT AVG(total_spending)
    FROM (
        SELECT customer_id, SUM(amount) AS total_spending
        FROM orders
        GROUP BY customer_id
    ) AS totals
);
```

**Logic:**

```text
Orders
  ↓
Total spending per customer (derived table)
  ↓
Average of those totals (nested subquery)
  ↓
Keep customers above that average
```

Notice the customer-totals logic is written **twice**. In Part 13, we'll see how **CTEs** remove this repetition.

---

💡 **Double Tap ❤️ For More**
