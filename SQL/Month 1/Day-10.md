# 🚀 SQL Roadmap 2026 — Part 10

## 🔗 SQL JOINs — Combining Data from Multiple Tables

In real-world databases, information is rarely stored in one table.

For example: `customers`, `orders`, `products`, `payments`, `employees`, `departments`

A customer may exist in one table while their orders exist in another. **JOINs allow us to combine related data from multiple tables.**

> This is one of the most important SQL concepts for a Data Analyst.

---

## 🧠 1. Why Do We Need JOINs?

Suppose we have two tables.

**customers**

| customer_id | customer_name |
| ----------- | ------------- |
| 101         | Alice         |
| 102         | Bob           |
| 103         | Charlie       |

**orders**

| order_id | customer_id | amount |
| -------- | ----------- | ------ |
| 1        | 101         | 500    |
| 2        | 101         | 800    |
| 3        | 102         | 300    |

The customer name is stored in `customers`. The order amount is stored in `orders`.

To answer:

> «How much did each customer spend?»

We need to combine the tables. That's where `JOIN` comes in.

---

## 🔗 2. Basic JOIN Structure

```sql
SELECT
    c.customer_name,
    o.order_id,
    o.amount
FROM customers c
JOIN orders o
    ON c.customer_id = o.customer_id;
```

Here `customers → c` and `orders → o`. These are called **table aliases**.

The condition `ON c.customer_id = o.customer_id` tells SQL **how the tables are related**.

---

## 🔑 3. The JOIN Key

A JOIN usually connects tables through a related column.

```text
customers.customer_id ↓ orders.customer_id
```

Often: one table contains a **primary key**, another table contains the corresponding **foreign key**.

| Table | Column | Role |
| ----- | ------ | ---- |
| `customers` | `customer_id` | Primary Key |
| `orders` | `customer_id` | Foreign Key |

---

## 🧩 4. INNER JOIN

`INNER JOIN` returns **only rows that have a match in both tables**.

```sql
SELECT
    c.customer_name,
    o.order_id,
    o.amount
FROM customers c
INNER JOIN orders o
    ON c.customer_id = o.customer_id;
```

**Result:**

| customer_name | order_id | amount |
| ------------- | -------- | ------ |
| Alice         | 1        | 500    |
| Alice         | 2        | 800    |
| Bob           | 3        | 300    |

> Charlie is missing because Charlie has **no matching order**.

```text
Customers ∩ Orders — Only matching records
```

---

## 👈 5. LEFT JOIN

`LEFT JOIN` returns: **All rows from the left table + matching rows from the right table.**

```sql
SELECT
    c.customer_name,
    o.order_id,
    o.amount
FROM customers c
LEFT JOIN orders o
    ON c.customer_id = o.customer_id;
```

Result includes:

| customer_name | order_id | amount |
| ------------- | -------- | ------ |
| Charlie       | `NULL`   | `NULL` |

---

## 🎯 6. Finding Customers Who Never Ordered

> This is a very common interview and analytics problem.

```sql
SELECT
    c.customer_id,
    c.customer_name
FROM customers c
LEFT JOIN orders o
    ON c.customer_id = o.customer_id
WHERE o.customer_id IS NULL;
```

This technique is often called an **anti-join pattern**.

---

## 👉 7. RIGHT JOIN

`RIGHT JOIN` returns: **All rows from the right table + matching rows from the left table.**

> In practice, many analysts prefer rewriting a `RIGHT JOIN` as a `LEFT JOIN` by switching table order, because it is often easier to read.

---

## 🔄 8. FULL OUTER JOIN

`FULL OUTER JOIN` returns: **All rows from both tables, whether they match or not.**

Conceptually:

```text
LEFT JOIN + RIGHT JOIN
```

It can reveal:

- matching records
- customers without orders
- orders without matching customers

> ⚠️ Not every database supports `FULL OUTER JOIN` directly.

---

## 🆚 9. INNER JOIN vs LEFT JOIN

| JOIN type | Returns |
| --------- | ------- |
| `INNER JOIN` | Only customers **with** matching orders |
| `LEFT JOIN` | **All** customers, including those without orders |

**Simple rule:**

```text
INNER JOIN = matching records
LEFT JOIN  = keep everything from the left table
```

---

## 📊 10. JOIN + Aggregation

**Question:** «How much has each customer spent?»

```sql
SELECT
    c.customer_id,
    c.customer_name,
    SUM(o.amount) AS total_spending
FROM customers c
INNER JOIN orders o
    ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name;
```

---

## 💰 11. Include Customers with Zero Spending

```sql
SELECT
    c.customer_id,
    c.customer_name,
    COALESCE(SUM(o.amount), 0) AS total_spending
FROM customers c
LEFT JOIN orders o
    ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name;
```

---

## 🔢 12. JOIN + COUNT()

```sql
SELECT
    c.customer_id,
    c.customer_name,
    COUNT(o.order_id) AS order_count
FROM customers c
LEFT JOIN orders o
    ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name;
```

*Why `COUNT(o.order_id)` instead of `COUNT(*)`?*

> Because `COUNT(*)` would count the `LEFT JOIN` row even when the customer has **no matching order**.

---

## ⚠️ 13. A Very Common JOIN Mistake

```sql
-- ❌ This removes NULLs and behaves like an INNER JOIN
SELECT ...
WHERE o.amount > 500;

-- ✅ Correct
LEFT JOIN orders o
    ON c.customer_id = o.customer_id
   AND o.amount > 500;
```

> **Important concept:** With an OUTER JOIN, **the location of a filter can change the result.**

---

## 🔗 14. Joining More Than Two Tables

```sql
SELECT
    c.customer_name,
    o.order_id,
    p.product_name,
    o.amount
FROM customers c
JOIN orders o
    ON c.customer_id = o.customer_id
JOIN products p
    ON o.product_id = p.product_id;
```

---

## 🏢 15. Real-World Business Example

```sql
SELECT
    p.category,
    SUM(o.amount) AS total_revenue
FROM orders o
JOIN products p
    ON o.product_id = p.product_id
GROUP BY p.category
ORDER BY total_revenue DESC;
```

> This is a typical Data Analyst query.

---

## 📈 16. JOIN + WHERE + GROUP BY + HAVING

**Question:** «Find customers who spent more than ₹50,000.»

```sql
SELECT
    c.customer_id,
    c.customer_name,
    SUM(o.amount) AS total_spending
FROM customers c
JOIN orders o
    ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name
HAVING SUM(o.amount) > 50000
ORDER BY total_spending DESC;
```

**Logical flow:**

```text
JOIN → GROUP BY → HAVING → ORDER BY
```

---

## 🪞 17. SELF JOIN

A table can also be joined **to itself**.

```sql
SELECT
    e.employee_name AS employee,
    m.employee_name AS manager
FROM employees e
LEFT JOIN employees m
    ON e.manager_id = m.employee_id;
```

---

## 🔢 18. CROSS JOIN

`CROSS JOIN` produces **every possible combination** of rows.

```text
5 products × 4 regions = 20 rows
```

---

## 🚨 19. The Biggest JOIN Problem: Duplicate Rows

One customer has five orders → **the customer appears five times**. This is the natural result of a one-to-many relationship.

If you want unique customers:

```sql
SELECT COUNT(DISTINCT c.customer_id)
```

---

## ⚠️ 20. Double Counting in Multiple JOINs

If both `orders` and `payments` have multiple rows per customer, joining them directly can create a **many-to-many multiplication**.

**Example:** 2 orders × 3 payments = **6 joined rows**. `SUM()` will overcount.

> 💡 Understand the **grain** of each table before joining.

---

## 🧠 21. JOINs and Table Grain

Before writing a JOIN, identify:

| Table | Grain |
| ----- | ----- |
| Table 1 | One row = one customer |
| Table 2 | One row = one order |

→ **One-to-Many relationship**

Understanding table grain helps prevent:

- duplicate counts
- inflated revenue
- incorrect averages
- incorrect KPIs

---

## 🎤 SQL Interview Questions

**Q1. What is a JOIN?**

Combines rows from multiple tables using a related condition.

**Q2. What is the difference between INNER JOIN and LEFT JOIN?**

INNER returns only matching rows; LEFT returns all from the left + matching from the right.

**Q3. How do you find customers who never placed an order?**

`LEFT JOIN` + `WHERE o.customer_id IS NULL`

**Q4. What is a SELF JOIN?**

Joins a table to itself, for hierarchical relationships.

**Q5. What is a CROSS JOIN?**

Creates every possible combination.

**Q6. Why can JOINs create duplicate rows?**

Because of one-to-many or many-to-many relationships.

**Q7. Why should you understand table grain?**

Because grain determines how rows multiply and whether aggregations become inaccurate.

**Q8. What happens when there is no match in a LEFT JOIN?**

Columns from the right table become `NULL`.

**Q9. How do you count unique customers after a JOIN?**

`COUNT(DISTINCT customer_id)`

**Q10. Can a query contain multiple JOINs?**

Yes.

---

## 📝 Practice Questions

**Practice 1:** Return customer names and their orders.

```sql
SELECT c.customer_name, o.order_id
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id;
```

**Practice 2:** Find customers who have never ordered.

```sql
SELECT c.customer_id, c.customer_name
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE o.customer_id IS NULL;
```

**Practice 3:** Calculate total spending per customer.

```sql
SELECT c.customer_id, c.customer_name, SUM(o.amount) AS total_spending
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name;
```

**Practice 4:** Return all customers and their order counts, including zero orders.

```sql
SELECT c.customer_id, c.customer_name, COUNT(o.order_id) AS order_count
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name;
```

**Practice 5:** Find the number of unique customers who placed orders.

```sql
SELECT COUNT(DISTINCT c.customer_id) AS unique_customers
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id;
```

---

## 🧪 Mini SQL Challenge

Write a query that returns: Customer name, Product name, Category, Amount — only orders > ₹1,000.

**Solution:**

```sql
SELECT
    c.customer_name,
    p.product_name,
    p.category,
    o.amount
FROM customers c
JOIN orders o
    ON c.customer_id = o.customer_id
JOIN products p
    ON o.product_id = p.product_id
WHERE o.amount > 1000
ORDER BY o.amount DESC;
```

---

> 📌 JOINs are the bridge between database tables. But writing a JOIN is only half the skill. A strong Data Analyst also understands: **What each table represents → How tables are related → How rows will multiply → How that affects the KPI.**

💡 **Double Tap ❤️ For More**
