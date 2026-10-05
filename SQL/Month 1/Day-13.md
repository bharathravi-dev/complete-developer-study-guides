# 🚀 SQL Roadmap 2026 — Part 13

## 🧱 SQL CTEs — Writing Complex Queries in Simple Steps

As SQL queries become more advanced, they can become difficult to read.

You may have:

- JOINs
- Subqueries
- GROUP BY
- CASE
- Aggregations
- Multiple calculations

all inside one query.

This is where **CTEs** become extremely useful.

CTE stands for:

> «**Common Table Expression**»

A CTE lets you create a **temporary named result set** and then use it in your main query.

Think of it as:

```text
Step 1 → Prepare the data
Step 2 → Transform the data
Step 3 → Analyze the data
Step 4 → Return the result
```

---

## 🧠 1. Basic CTE Syntax

A CTE starts with `WITH`:

```sql
WITH cte_name AS (
    SELECT
        ...
    FROM ...
)
SELECT *
FROM cte_name;
```

**Example:**

```sql
WITH customer_sales AS (
    SELECT
        customer_id,
        SUM(amount) AS total_spending
    FROM orders
    GROUP BY customer_id
)
SELECT *
FROM customer_sales;
```

The CTE `customer_sales` acts like a temporary result set **for the duration of the query**.

---

## 📊 2. Why Use CTEs?

Without a CTE, a complex query can become difficult to understand.

**Derived table (subquery):**

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
) AS customer_totals
WHERE total_spending > 50000;
```

**With a CTE:**

```sql
WITH customer_totals AS (
    SELECT
        customer_id,
        SUM(amount) AS total_spending
    FROM orders
    GROUP BY customer_id
)
SELECT
    customer_id,
    total_spending
FROM customer_totals
WHERE total_spending > 50000;
```

The second version reads **top to bottom** — much easier to follow.

---

## 🧩 3. CTEs Break Complex Problems into Steps

Business question:

> «Find customers whose total spending is above the average customer spending.»

(This is the Mini Challenge from Part 12 — where we had to write the same subquery twice.)

Instead of one large nested query, break it into logical steps.

### Step 1 — Calculate spending per customer

```sql
WITH customer_totals AS (
    SELECT
        customer_id,
        SUM(amount) AS total_spending
    FROM orders
    GROUP BY customer_id
)
```

### Step 2 — Calculate average spending

```sql
WITH customer_totals AS (
    SELECT
        customer_id,
        SUM(amount) AS total_spending
    FROM orders
    GROUP BY customer_id
),
average_spending AS (
    SELECT
        AVG(total_spending) AS avg_spending
    FROM customer_totals
)
```

### Step 3 — Compare customers with the average

```sql
WITH customer_totals AS (
    SELECT
        customer_id,
        SUM(amount) AS total_spending
    FROM orders
    GROUP BY customer_id
),
average_spending AS (
    SELECT
        AVG(total_spending) AS avg_spending
    FROM customer_totals
)
SELECT
    c.customer_id,
    c.total_spending
FROM customer_totals c
CROSS JOIN average_spending a
WHERE c.total_spending > a.avg_spending;
```

`average_spending` has exactly **one row**, so the `CROSS JOIN` simply attaches the average to every customer row.

Now the logic is much easier to follow — and `customer_totals` is written **only once**.

---

## 🔗 4. Multiple CTEs

A single query can contain multiple CTEs.

**Structure:**

```sql
WITH first_cte AS (
    ...
),
second_cte AS (
    ...
),
third_cte AS (
    ...
)
SELECT ...
FROM third_cte;
```

- Only **one** `WITH` keyword at the start.
- CTEs are separated by **commas**.
- Later CTEs can reference earlier CTEs.

---

## 🏢 5. Real-World Example

An e-commerce company wants:

> «Customers with spending above ₹50,000 **and** at least 5 orders.»

First calculate customer-level metrics:

```sql
WITH customer_metrics AS (
    SELECT
        customer_id,
        COUNT(*)    AS order_count,
        SUM(amount) AS total_spending
    FROM orders
    GROUP BY customer_id
)
SELECT
    customer_id,
    order_count,
    total_spending
FROM customer_metrics
WHERE order_count >= 5
  AND total_spending > 50000;
```

The CTE changes the grain to: **1 row = 1 customer**

That makes the final filtering straightforward — plain `WHERE`, no `HAVING` needed.

---

## 📈 6. CTE + JOIN

CTEs become even more useful when combined with JOINs.

```sql
WITH customer_sales AS (
    SELECT
        customer_id,
        SUM(amount) AS total_spending
    FROM orders
    GROUP BY customer_id
)
SELECT
    c.customer_name,
    cs.total_spending
FROM customers c
JOIN customer_sales cs
    ON c.customer_id = cs.customer_id;
```

- The CTE handles the **aggregation**.
- The main query handles the **customer information**.

This is the **pre-aggregation** pattern from Part 11 — aggregate first, then join — which avoids double counting.

---

## 💰 7. CTE + COALESCE

Want to include customers who haven't placed orders?

```sql
WITH customer_sales AS (
    SELECT
        customer_id,
        SUM(amount) AS total_spending
    FROM orders
    GROUP BY customer_id
)
SELECT
    c.customer_id,
    c.customer_name,
    COALESCE(cs.total_spending, 0) AS total_spending
FROM customers c
LEFT JOIN customer_sales cs
    ON c.customer_id = cs.customer_id;
```

Now customers without orders appear with `total_spending = 0` instead of `NULL`.

---

## 🧮 8. CTE + CASE

We can also create business segments.

```sql
WITH customer_sales AS (
    SELECT
        customer_id,
        SUM(amount) AS total_spending
    FROM orders
    GROUP BY customer_id
)
SELECT
    customer_id,
    total_spending,
    CASE
        WHEN total_spending >= 100000 THEN 'VIP'
        WHEN total_spending >=  50000 THEN 'Premium'
        ELSE 'Standard'
    END AS customer_segment
FROM customer_sales;
```

- The CTE creates the **metric**.
- `CASE` converts the metric into **business categories**.

---

## 🔍 9. CTE vs Subquery

Both can solve similar problems.

**Subquery:**

```sql
SELECT *
FROM (
    SELECT
        customer_id,
        SUM(amount) AS total_spending
    FROM orders
    GROUP BY customer_id
) AS customer_sales
WHERE total_spending > 50000;
```

**CTE:**

```sql
WITH customer_sales AS (
    SELECT
        customer_id,
        SUM(amount) AS total_spending
    FROM orders
    GROUP BY customer_id
)
SELECT *
FROM customer_sales
WHERE total_spending > 50000;
```

| Aspect                     | Subquery                  | CTE                          |
| -------------------------- | ------------------------- | ---------------------------- |
| Reading order              | Inside-out                | Top-to-bottom                |
| Reuse in same statement    | Must repeat the code      | Reference by name            |
| Multi-step logic           | Gets deeply nested        | Clean sequence of steps      |
| Recursion                  | ❌                        | ✅ (`WITH RECURSIVE`)        |

The CTE often makes multi-step logic easier to read.

---

## 🧠 10. CTE vs Temporary Table

A CTE is **not** the same as a table.

**CTE:**

```sql
WITH sales AS (...)
SELECT ...;
```

Generally exists only for the duration of **that one SQL statement**.

**Temporary table:**

```sql
CREATE TEMP TABLE sales AS
SELECT ...;
```

A temporary table can generally be referenced by **multiple statements during its session**, depending on the database.

Simple distinction:

- **CTE** → temporary named query result
- **Temporary table** → temporary database object

---

## ⚡ 11. CTE Does Not Automatically Mean Faster

A common misconception:

> *«CTEs make queries faster.»* — **Not necessarily.**

CTEs primarily improve:

- readability
- organization
- maintainability
- debugging
- step-by-step logic

Performance depends on the database engine and how it optimizes the query. Some databases may **inline** a CTE (treat it like a subquery), while others may **materialize** it (compute and store it once) in certain situations.

So: **Use CTEs for clear logic, not simply because you expect better performance.**

---

## 🧪 12. CTE for Data Quality Analysis

Suppose we want to identify customers with missing contact information.

First create a cleaned customer dataset:

```sql
WITH cleaned_customers AS (
    SELECT
        customer_id,
        TRIM(customer_name)  AS customer_name,
        LOWER(TRIM(email))   AS email
    FROM customers
)
SELECT *
FROM cleaned_customers
WHERE email IS NULL;
```

This creates a **clean intermediate layer** before analysis.

---

## 📊 13. CTE for KPI Calculation

> «Revenue per customer.»

```sql
WITH customer_revenue AS (
    SELECT
        customer_id,
        SUM(amount) AS revenue
    FROM orders
    GROUP BY customer_id
)
SELECT
    SUM(revenue)                        AS total_revenue,
    COUNT(*)                            AS active_customers,
    SUM(revenue) / NULLIF(COUNT(*), 0)  AS revenue_per_customer
FROM customer_revenue;
```

The CTE first creates: **1 row = 1 customer**

Then the final query calculates KPIs from that customer-level dataset. `NULLIF(..., 0)` protects against division by zero.

---

## 🧱 14. CTE for Multi-Step Analytics

Let's build a slightly more realistic analysis.

### Step 1 — Calculate customer sales

```sql
WITH customer_sales AS (
    SELECT
        customer_id,
        COUNT(*)    AS order_count,
        SUM(amount) AS total_spending
    FROM orders
    GROUP BY customer_id
)
```

### Step 2 — Create customer segments

```sql
, segmented_customers AS (
    SELECT
        customer_id,
        order_count,
        total_spending,
        CASE
            WHEN total_spending >= 100000 THEN 'VIP'
            WHEN total_spending >=  50000 THEN 'Premium'
            ELSE 'Standard'
        END AS segment
    FROM customer_sales
)
```

### Step 3 — Analyze segments

```sql
SELECT
    segment,
    COUNT(*)            AS customers,
    SUM(total_spending) AS revenue
FROM segmented_customers
GROUP BY segment
ORDER BY revenue DESC;
```

### The complete query

```sql
WITH customer_sales AS (
    SELECT
        customer_id,
        COUNT(*)    AS order_count,
        SUM(amount) AS total_spending
    FROM orders
    GROUP BY customer_id
),
segmented_customers AS (
    SELECT
        customer_id,
        order_count,
        total_spending,
        CASE
            WHEN total_spending >= 100000 THEN 'VIP'
            WHEN total_spending >=  50000 THEN 'Premium'
            ELSE 'Standard'
        END AS segment
    FROM customer_sales
)
SELECT
    segment,
    COUNT(*)            AS customers,
    SUM(total_spending) AS revenue
FROM segmented_customers
GROUP BY segment
ORDER BY revenue DESC;
```

This is a good example of **structured analytical SQL**.

---

## 🪟 15. CTE + Window Functions Preview

CTEs become especially powerful when combined with window functions.

```sql
WITH customer_sales AS (
    SELECT
        customer_id,
        SUM(amount) AS total_spending
    FROM orders
    GROUP BY customer_id
)
SELECT
    customer_id,
    total_spending,
    RANK() OVER (
        ORDER BY total_spending DESC
    ) AS spending_rank
FROM customer_sales;
```

- The CTE creates the **customer-level metric**.
- The window function **ranks** the customers.
- This pattern is extremely common in analytics.

---

## 🔄 16. Recursive CTEs

There is another advanced type of CTE: **Recursive CTE**

It allows a query to **repeatedly reference itself**.

Common use cases:

- organizational hierarchies
- employee-manager structures
- category trees
- folder structures
- graph-like relationships
- generating sequences

**Example structure:**

```sql
WITH RECURSIVE employee_tree AS (
    -- Anchor member: start at the top (CEO)
    SELECT
        employee_id,
        employee_name,
        manager_id
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- Recursive member: find direct reports of rows already found
    SELECT
        e.employee_id,
        e.employee_name,
        e.manager_id
    FROM employees e
    JOIN employee_tree t
        ON e.manager_id = t.employee_id
)
SELECT *
FROM employee_tree;
```

How it works:

```text
Anchor      → CEO
Iteration 1 → people reporting to the CEO
Iteration 2 → people reporting to them
...         → stops when no new rows are found
```

Note: SQL Server and Oracle write just `WITH` (no `RECURSIVE` keyword).

Recursive CTEs are an advanced topic, but understanding their purpose is important.

---

## 🧠 17. Why CTEs Are Valuable for Data Analysts

Imagine a business request:

> «Identify customers who increased their spending, classify them by segment, compare them against the average, and calculate their rank.»

Trying to write everything as one giant query can become difficult.

CTEs allow you to think in stages:

```text
Raw Orders
  ↓
Customer Metrics
  ↓
Customer Segmentation
  ↓
Average Comparison
  ↓
Ranking
  ↓
Final Report
```

Each step has a clear purpose.

---

## ⚠️ 18. Common CTE Mistakes

**Mistake 1 — Forgetting the CTE name**

```sql
-- ❌ Incorrect
WITH ( SELECT ... )

-- ✅ Correct
WITH customer_sales AS ( SELECT ... )
```

**Mistake 2 — Forgetting the comma between CTEs (or repeating `WITH`)**

```sql
-- ❌ Incorrect
WITH cte1 AS ( ... )
WITH cte2 AS ( ... )

-- ✅ Correct
WITH cte1 AS ( ... ),
     cte2 AS ( ... )
```

**Mistake 3 — Using a CTE without understanding its grain**

A CTE might produce **1 row = 1 order** while you think it produces **1 row = 1 customer**. Always validate the grain.

**Mistake 4 — Creating too many unnecessary CTEs**

CTEs should make logic clearer. If every two-line transformation becomes its own CTE, the query can become harder to follow. Use them when they improve structure.

---

## 🔎 19. Debugging with CTEs

One major advantage is **easier debugging**.

Suppose your final query produces incorrect revenue. Instead of debugging one huge query, test each stage:

```sql
WITH customer_sales AS (
    ...
)
SELECT *
FROM customer_sales;
```

- Check the results (row count, totals, grain).
- Then add the next CTE and inspect it.
- This lets you identify **exactly where** the numbers become incorrect.

---

## 🎤 SQL Interview Questions

**Q1. What is a CTE?**

A Common Table Expression is a named temporary result set defined using the `WITH` clause and available to the query that follows it.

**Q2. What is the syntax of a CTE?**

```sql
WITH cte_name AS (
    SELECT ...
)
SELECT ... FROM cte_name;
```

**Q3. Can you create multiple CTEs?**

Yes.

```sql
WITH cte1 AS ( ... ),
     cte2 AS ( ... )
SELECT ... FROM cte2;
```

**Q4. Can one CTE reference another CTE?**

Yes. A later CTE can generally reference an earlier CTE in the same `WITH` clause.

**Q5. What is the difference between a CTE and a subquery?**

Both can represent intermediate query results, but CTEs often make multi-step logic easier to read and can be referenced multiple times within the same statement.

**Q6. Does a CTE permanently store data?**

No. A standard CTE is associated only with the SQL statement in which it is defined.

**Q7. Does using a CTE always improve performance?**

No. CTEs primarily improve query organization and readability. Performance depends on the database engine and execution plan.

**Q8. What is a recursive CTE?**

A CTE that references itself, typically used for hierarchical or recursive data. It has an anchor member and a recursive member combined with `UNION ALL`.

**Q9. Why are CTEs useful in analytics?**

They allow complex analytical logic to be divided into clear, manageable stages.

**Q10. What should you check when using multiple CTEs?**

The grain, row count, joins, aggregations, and filters at each stage.

---

## 📝 Practice Questions

**Practice 1 — Calculate total spending per customer using a CTE.**

```sql
WITH customer_sales AS (
    SELECT customer_id, SUM(amount) AS total_spending
    FROM orders
    GROUP BY customer_id
)
SELECT * FROM customer_sales;
```

**Practice 2 — Find customers spending more than ₹50,000.**

```sql
WITH customer_sales AS (
    SELECT customer_id, SUM(amount) AS total_spending
    FROM orders
    GROUP BY customer_id
)
SELECT *
FROM customer_sales
WHERE total_spending > 50000;
```

**Practice 3 — Calculate order count and revenue per customer.**

```sql
WITH customer_metrics AS (
    SELECT
        customer_id,
        COUNT(*)    AS order_count,
        SUM(amount) AS total_revenue
    FROM orders
    GROUP BY customer_id
)
SELECT * FROM customer_metrics;
```

**Practice 4 — Rank customers by total spending.**

```sql
WITH customer_sales AS (
    SELECT customer_id, SUM(amount) AS total_spending
    FROM orders
    GROUP BY customer_id
)
SELECT
    customer_id,
    total_spending,
    RANK() OVER (ORDER BY total_spending DESC) AS spending_rank
FROM customer_sales;
```

**Practice 5 — Create customer segments based on spending.**

```sql
WITH customer_sales AS (
    SELECT customer_id, SUM(amount) AS total_spending
    FROM orders
    GROUP BY customer_id
)
SELECT
    customer_id,
    total_spending,
    CASE
        WHEN total_spending >= 100000 THEN 'VIP'
        WHEN total_spending >=  50000 THEN 'Premium'
        ELSE 'Standard'
    END AS segment
FROM customer_sales;
```

---

## 🧪 Mini SQL Challenge

You have:

```text
orders (order_id, customer_id, amount, order_date)
```

> Find the **top 5 customers by total spending**, but only consider customers who have placed **at least 3 orders**.

**Solution:**

```sql
WITH customer_metrics AS (
    SELECT
        customer_id,
        COUNT(*)    AS order_count,
        SUM(amount) AS total_spending
    FROM orders
    GROUP BY customer_id
),
qualified_customers AS (
    SELECT
        customer_id,
        order_count,
        total_spending
    FROM customer_metrics
    WHERE order_count >= 3
)
SELECT
    customer_id,
    order_count,
    total_spending
FROM qualified_customers
ORDER BY total_spending DESC
LIMIT 5;
```

**Logic:**

```text
Orders
  ↓
GROUP BY customer
  ↓
Calculate order count + spending
  ↓
Keep customers with ≥ 3 orders
  ↓
Sort by spending
  ↓
Return top 5
```

Note: `LIMIT` works in PostgreSQL, MySQL, and SQLite. SQL Server uses `SELECT TOP 5 ...`, and Oracle/standard SQL use `FETCH FIRST 5 ROWS ONLY`.

---

💡 **Double Tap ❤️ For More**
