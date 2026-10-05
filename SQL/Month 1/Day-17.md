# 🚀 SQL Roadmap 2026 — Part 17

## 🪟 SQL Views — Creating Reusable Virtual Tables

A SQL **View** is a saved SQL query that behaves like a virtual table.

Instead of writing the same complex query again and again, you create a View once and query it like a table.

Views are especially useful in:

- Data analytics
- Reporting
- Business Intelligence
- Power BI datasets
- Data warehouses
- Reusable business logic
- Data security

⚠️ **Dialect note:** Examples use **PostgreSQL** syntax unless stated otherwise. View features such as `CREATE OR REPLACE` and materialized views differ between databases — see the cheat sheet in section 20.

---

## 🧠 1. What Is a View?

Suppose you frequently need this query:

```sql
SELECT
    customer_id,
    customer_name,
    country,
    total_sales
FROM customers
WHERE country = 'India'
  AND total_sales > 100000;
```

Instead of writing it every time, create a View:

```sql
CREATE VIEW high_value_indian_customers AS
SELECT
    customer_id,
    customer_name,
    country,
    total_sales
FROM customers
WHERE country = 'India'
  AND total_sales > 100000;
```

Now you can simply write:

```sql
SELECT *
FROM high_value_indian_customers;
```

From the user's point of view, the View behaves like a table.

---

## ❓ 2. Why Do We Need Views?

Imagine a reporting query that contains:

- 4 JOINs
- 3 CASE expressions
- Multiple filters
- Several calculated columns
- Aggregations

You don't want every analyst and every report rewriting that logic (and getting it slightly different each time).

A View lets you **define the logic once** and reuse it everywhere.

---

## 🛠️ 3. Basic CREATE VIEW Syntax

```sql
CREATE VIEW view_name AS
SELECT
    column1,
    column2,
    column3
FROM table_name
WHERE condition;
```

Example:

```sql
CREATE VIEW active_customers AS
SELECT
    customer_id,
    customer_name,
    country
FROM customers
WHERE status = 'Active';
```

Query the View:

```sql
SELECT *
FROM active_customers;
```

---

## 🔗 4. A View Can Contain JOINs

Views aren't limited to simple `SELECT`s.

```sql
CREATE VIEW customer_orders AS
SELECT
    c.customer_id,
    c.customer_name,
    o.order_id,
    o.order_date,
    o.order_amount
FROM customers c
JOIN orders o
    ON c.customer_id = o.customer_id;
```

Now:

```sql
SELECT *
FROM customer_orders;
```

returns the result of that JOIN — no need to rewrite it.

---

## 📊 5. Views with Aggregation

Suppose management regularly needs regional sales:

```sql
CREATE VIEW regional_sales AS
SELECT
    region,
    SUM(order_amount) AS total_sales,
    COUNT(*)          AS total_orders
FROM orders
GROUP BY region;
```

```sql
SELECT *
FROM regional_sales;
```

| region | total_sales | total_orders |
| ------ | ----------- | ------------ |
| North  | 450000      | 1250         |
| South  | 620000      | 1580         |
| West   | 510000      | 1390         |
| East   | 380000      | 980          |

You can filter the View further, just like a table:

```sql
SELECT *
FROM regional_sales
WHERE total_sales > 500000;
```

---

## 🏷️ 6. Views with CASE Expressions

Views are great for **standardizing business logic**.

```sql
CREATE VIEW customer_segments AS
SELECT
    customer_id,
    customer_name,
    total_sales,
    CASE
        WHEN total_sales >= 100000 THEN 'High Value'
        WHEN total_sales >= 50000  THEN 'Medium Value'
        ELSE 'Low Value'
    END AS customer_segment
FROM customers;
```

Now every report uses the same segmentation:

```sql
SELECT
    customer_segment,
    COUNT(*) AS customers
FROM customer_segments
GROUP BY customer_segment;
```

No more slightly different definitions of "High Value Customer" across reports.

---

## 🗃️ 7. View vs Table

This is an important distinction.

| Object | Stores                    |
| ------ | ------------------------- |
| Table  | The actual data           |
| View   | Only the query definition |

```text
Table : customers (data stored)
View  : customers → saved query → result computed when you query it
```

Because a normal View stores only the query, it always reflects the **current** data in the underlying tables.

---

## ⏳ 8. View vs Temporary Table

These are also different.

**View** — reusable logic, usually permanent, shareable with other users (subject to permissions):

```sql
CREATE VIEW customer_summary AS
SELECT ...
```

**Temporary table** — stores intermediate results, visible only to your session, and dropped when the session ends:

```sql
CREATE TEMPORARY TABLE temp_customer_data AS
SELECT *
FROM customers
WHERE status = 'Active';
```

(SQL Server uses `SELECT ... INTO #temp_customer_data` instead.)

```text
VIEW       → Reusable saved query (data computed on read)
TEMP TABLE → Temporary stored copy of a result (a snapshot)
```

---

## ✏️ 9. Can You Update Data Through a View?

Sometimes.

Simple Views — one table, no aggregation — are often **updatable**:

```sql
CREATE VIEW active_customers AS
SELECT
    customer_id,
    customer_name,
    status
FROM customers
WHERE status = 'Active';
```

```sql
UPDATE active_customers
SET customer_name = 'New Name'
WHERE customer_id = 101;
```

This updates the underlying `customers` table.

Views that contain any of the following are generally **not** directly updatable:

- Aggregations / `GROUP BY`
- `DISTINCT`
- Set operators (`UNION`, etc.)
- Many kinds of JOINs
- Window functions

So don't assume every View can be modified.

💡 `WITH CHECK OPTION` stops users from inserting or updating rows through the View that the View itself would hide:

```sql
CREATE VIEW active_customers AS
SELECT customer_id, customer_name, status
FROM customers
WHERE status = 'Active'
WITH CHECK OPTION;

-- ❌ Rejected: the row would disappear from the view
UPDATE active_customers SET status = 'Inactive' WHERE customer_id = 101;
```

---

## 🔄 10. CREATE OR REPLACE VIEW

To change a View's definition:

```sql
CREATE OR REPLACE VIEW active_customers AS
SELECT
    customer_id,
    customer_name,
    status,
    country
FROM customers
WHERE status = 'Active';
```

This changes the query behind the View while keeping its name (and usually its permissions).

⚠️ **Dialect notes:**

- **SQL Server** uses `CREATE OR ALTER VIEW` (2016 SP1+) or `ALTER VIEW`.
- **PostgreSQL** only lets `CREATE OR REPLACE` **add new columns at the end** — it can't drop, rename or reorder existing ones (that's why `country` goes last above). For those changes, `DROP VIEW` and recreate it.

---

## 🗑️ 11. Dropping a View

```sql
DROP VIEW active_customers;
```

This removes only the View **definition**. The underlying tables and their data are untouched.

```sql
DROP VIEW customer_orders;
-- customers and orders tables still exist, with all their rows
```

Use `DROP VIEW IF EXISTS customer_orders;` to avoid an error when the View doesn't exist.

---

## 🔒 12. Views for Data Security

Views can control which **columns** (and rows) users can see.

Suppose `employees` contains:

```text
employee_id, employee_name, department, salary, bank_account
```

You don't want every analyst seeing salaries or bank details:

```sql
CREATE VIEW employee_directory AS
SELECT
    employee_id,
    employee_name,
    department
FROM employees;
```

Then grant access to the View, not the table:

```sql
GRANT SELECT ON employee_directory TO analyst_role;
```

⚠️ A View on its own is **not** a security solution. It only protects data when users are denied direct access to the underlying table through proper permissions.

---

## 📐 13. Views for Business Logic

Suppose the business defines:

```text
High Value Customer = sales >= 100,000
```

Encode it in one place:

```sql
CREATE VIEW customer_classification AS
SELECT
    customer_id,
    customer_name,
    total_sales,
    CASE
        WHEN total_sales >= 100000 THEN 'High Value'
        ELSE 'Standard'
    END AS customer_type
FROM customers;
```

Every report can now use:

```sql
SELECT
    customer_type,
    COUNT(*) AS customer_count
FROM customer_classification
GROUP BY customer_type;
```

If the threshold ever changes, you update **one** View instead of dozens of reports.

---

## 📈 14. Views for BI and Reporting

Views are very common in reporting environments.

Imagine Power BI needs: customer, order, product, region, revenue, profit and order date.

Instead of exposing many raw tables, the data team creates a reporting View:

```sql
CREATE VIEW sales_reporting AS
SELECT
    o.order_id,
    o.order_date,
    c.customer_id,
    c.customer_name,
    p.product_name,
    o.region,
    o.sales_amount,
    o.profit
FROM orders o
JOIN customers c
    ON o.customer_id = c.customer_id
JOIN products p
    ON o.product_id = p.product_id;
```

The BI tool consumes:

```sql
SELECT *
FROM sales_reporting;
```

This simplifies the data model and keeps transformation logic in the database.

---

## ⚡ 15. Views and Query Performance

A very important point:

> A normal View does **not** automatically make a query faster.

```sql
SELECT *
FROM customer_orders;
```

The database still runs the underlying JOIN every time. A View is mainly a tool for **abstraction and reuse**.

Performance still depends on:

- Indexes
- Query structure
- Join conditions
- Filtering
- Data volume
- The database optimizer and statistics
- The execution plan

Usually the optimizer "inlines" the View's query into your query, so a filter like `WHERE customer_id = 101` on the View can still use indexes on the base tables.

---

## 💾 16. Materialized Views

A **materialized view** physically stores the result of its query.

```text
Normal View       → Stores the query definition
Materialized View → Stores the query result
```

```sql
CREATE MATERIALIZED VIEW monthly_sales AS
SELECT
    DATE_TRUNC('month', order_date) AS month,
    SUM(order_amount)               AS total_sales
FROM orders
GROUP BY DATE_TRUNC('month', order_date);
```

Because the result is stored, it becomes **stale** as the base tables change and must be refreshed:

```sql
REFRESH MATERIALIZED VIEW monthly_sales;
```

Materialized views are useful when:

- Queries are expensive
- Data is large
- The same aggregation is requested repeatedly
- Near-real-time data isn't required

⚠️ **Dialect notes:** PostgreSQL and Oracle support `CREATE MATERIALIZED VIEW`. SQL Server's equivalent is an **indexed view**. MySQL has no materialized views — teams typically use a summary table refreshed on a schedule.

---

## 🧱 17. View with a CTE

A View can contain a CTE (Part 13):

```sql
CREATE VIEW customer_summary AS
WITH customer_orders AS (
    SELECT
        customer_id,
        COUNT(*)          AS order_count,
        SUM(order_amount) AS total_sales
    FROM orders
    GROUP BY customer_id
)
SELECT
    c.customer_id,
    c.customer_name,
    co.order_count,
    co.total_sales
FROM customers c
JOIN customer_orders co
    ON c.customer_id = co.customer_id;
```

Now the multi-step logic is reusable:

```sql
SELECT *
FROM customer_summary;
```

---

## 🪜 18. Views Can Be Layered

One View can reference another:

```text
Raw Tables → View 1 → View 2 → Reporting View → Dashboard
```

This is powerful, but deep layering makes debugging and performance tuning hard — and you can't drop or change a View that others depend on without breaking them.

Keep dependencies shallow and understandable.

---

## ⚠️ 19. Common Mistakes

❌ **Mistake 1: Assuming a View stores a copy of the data** — a normal View stores only the query.

❌ **Mistake 2: Assuming Views always improve performance** — they improve abstraction and reuse, not speed.

❌ **Mistake 3: Creating too many nested Views** — long View chains are hard to maintain and debug.

❌ **Mistake 4: Forgetting the filters hidden inside a View** — if a View contains `WHERE status = 'Active'`, anyone counting "all customers" from it will get the wrong answer.

❌ **Mistake 5: Using `SELECT *` in a View**

Instead of:

```sql
CREATE VIEW customer_report AS
SELECT *
FROM customers;
```

list the columns explicitly:

```sql
CREATE VIEW customer_report AS
SELECT
    customer_id,
    customer_name,
    country,
    status
FROM customers;
```

Many databases expand `*` into a fixed column list **when the View is created**, so columns added to the table later won't show up, and the View may break when columns are dropped. An explicit list makes the View's structure clear and controlled.

❌ **Mistake 6: Putting `ORDER BY` inside a View** — the order isn't guaranteed when you query the View (SQL Server rejects it outright). Sort in the query that *uses* the View.

---

## 🗂️ 20. Dialect Cheat Sheet

| Feature             | PostgreSQL                     | MySQL                     | SQL Server                     | Oracle                         |
| ------------------- | ------------------------------ | ------------------------- | ------------------------------ | ------------------------------ |
| Create              | `CREATE VIEW`                  | `CREATE VIEW`             | `CREATE VIEW`                  | `CREATE VIEW`                  |
| Replace             | `CREATE OR REPLACE VIEW`       | `CREATE OR REPLACE VIEW`  | `CREATE OR ALTER VIEW`         | `CREATE OR REPLACE VIEW`       |
| Drop if exists      | `DROP VIEW IF EXISTS`          | `DROP VIEW IF EXISTS`     | `DROP VIEW IF EXISTS` (2016+)  | `DROP VIEW` (23ai adds `IF EXISTS`) |
| Materialized        | `CREATE MATERIALIZED VIEW`     | ❌ (use summary tables)   | Indexed view                   | `CREATE MATERIALIZED VIEW`     |
| Refresh             | `REFRESH MATERIALIZED VIEW`    | —                         | Automatic                      | `DBMS_MVIEW.REFRESH`           |

---

## 💼 21. Real-World Data Analyst Example

Suppose you build a monthly sales report every week and write this each time:

```sql
SELECT
    region,
    DATE_TRUNC('month', order_date) AS month,
    SUM(order_amount)               AS revenue,
    COUNT(DISTINCT customer_id)     AS customers,
    COUNT(*)                        AS orders
FROM orders
GROUP BY
    region,
    DATE_TRUNC('month', order_date);
```

Turn it into a View:

```sql
CREATE VIEW monthly_sales_report AS
SELECT
    region,
    DATE_TRUNC('month', order_date) AS month,
    SUM(order_amount)               AS revenue,
    COUNT(DISTINCT customer_id)     AS customers,
    COUNT(*)                        AS orders
FROM orders
GROUP BY
    region,
    DATE_TRUNC('month', order_date);
```

Your analysis becomes:

```sql
SELECT *
FROM monthly_sales_report;
```

And you can filter and sort it:

```sql
SELECT *
FROM monthly_sales_report
WHERE month >= DATE '2026-01-01'
ORDER BY revenue DESC;
```

The View becomes a reusable reporting layer.

---

## 🎤 SQL Interview Questions

**Q1. What is a SQL View?**

A named database object that represents the result of a saved query and can be queried like a table.

**Q2. Does a normal View physically store data?**

No. It stores the query definition; the data is computed from the underlying tables each time you query it.

**Q3. What is the difference between a View and a table?**

A table physically stores data. A normal View is a saved query over one or more tables.

**Q4. Can a View contain JOINs?**

Yes.

```sql
CREATE VIEW customer_orders AS
SELECT ...
FROM customers c
JOIN orders o
    ON c.customer_id = o.customer_id;
```

**Q5. Can a View contain `GROUP BY`?**

Yes.

```sql
CREATE VIEW regional_sales AS
SELECT
    region,
    SUM(amount) AS total_sales
FROM sales
GROUP BY region;
```

**Q6. Can every View be updated?**

No. Simple single-table Views often can be; Views with aggregation, `DISTINCT`, set operators, many JOINs or window functions generally can't. The rules depend on the database.

**Q7. Does creating a View automatically improve performance?**

No. A normal View is for abstraction, reuse and centralized logic. The underlying query still runs each time.

**Q8. What is a materialized view?**

A View whose query result is stored physically. It is faster to read but must be refreshed to reflect changes in the base tables.

**Q9. Why are Views useful in BI?**

They provide a clean, reusable reporting layer and keep SQL transformations and business definitions in one place.

**Q10. What is one security benefit of Views?**

Combined with proper permissions, a View can expose only selected columns or rows instead of giving users access to the whole underlying table.

---

## 📝 Practice Questions

**Practice 1 — Create a View containing active customers.**

```sql
CREATE VIEW active_customers AS
SELECT
    customer_id,
    customer_name,
    country
FROM customers
WHERE status = 'Active';
```

**Practice 2 — Query only active customers from India.**

```sql
SELECT *
FROM active_customers
WHERE country = 'India';
```

**Practice 3 — Create a View showing total sales by region.**

```sql
CREATE VIEW regional_sales AS
SELECT
    region,
    SUM(order_amount) AS total_sales
FROM orders
GROUP BY region;
```

**Practice 4 — Find regions with sales above 500,000.**

```sql
SELECT *
FROM regional_sales
WHERE total_sales > 500000;
```

**Practice 5 — Create a View combining customers and their orders.**

```sql
CREATE VIEW customer_orders AS
SELECT
    c.customer_id,
    c.customer_name,
    o.order_id,
    o.order_date,
    o.order_amount
FROM customers c
JOIN orders o
    ON c.customer_id = o.customer_id;
```

---

## 🧪 Mini SQL Challenge

You have two tables:

```text
customers : customer_id, customer_name, region, status
orders    : order_id, customer_id, order_date, order_amount
```

Create a View called `customer_sales_summary` containing:

- Customer ID
- Customer name
- Region
- Number of orders
- Total sales
- Average order value

**Solution**

```sql
CREATE VIEW customer_sales_summary AS
SELECT
    c.customer_id,
    c.customer_name,
    c.region,
    COUNT(o.order_id)               AS order_count,
    COALESCE(SUM(o.order_amount), 0) AS total_sales,
    AVG(o.order_amount)             AS average_order_value
FROM customers c
LEFT JOIN orders o
    ON c.customer_id = o.customer_id
GROUP BY
    c.customer_id,
    c.customer_name,
    c.region;
```

Why these choices?

- `LEFT JOIN` keeps customers who have **no orders**.
- `COUNT(o.order_id)` (not `COUNT(*)`) gives those customers `0` instead of `1`.
- `COALESCE(SUM(...), 0)` turns their `NULL` total into `0`. `AVG` is left as `NULL`, since there's no average without orders.

Now you can analyze high-value customers:

```sql
SELECT *
FROM customer_sales_summary
WHERE total_sales > 100000
ORDER BY total_sales DESC;
```

**What did we use?**

| Tool                    | Purpose                                    |
| ----------------------- | ------------------------------------------ |
| `CREATE VIEW`           | Save the logic as a reusable virtual table |
| `LEFT JOIN`             | Keep customers without orders              |
| `COUNT` / `SUM` / `AVG` | Per-customer metrics                       |
| `COALESCE`              | Replace `NULL` totals with `0`             |

---

## 🎯 Key Takeaway

```text
TABLE             → Stores data
VIEW              → Stores a reusable query definition
MATERIALIZED VIEW → Stores the query result (needs refreshing)
```

A strong Data Analyst doesn't just write SQL queries — they build reusable, maintainable and consistent data logic.

---

💡 **Double Tap ❤️ For More**
