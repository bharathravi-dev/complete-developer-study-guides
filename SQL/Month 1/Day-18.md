# 🚀 SQL Roadmap 2026 — Part 18

## ⚡ SQL Indexes & Query Performance

Writing a correct SQL query is important.

But in real-world analytics, where tables hold millions or billions of rows, another question matters:

> **How efficiently does the database find the data?**

This is where **indexes** come in.

Used well, indexes can make data retrieval dramatically faster — but they also cost storage and slow down writes.

⚠️ **Dialect note:** Examples use **PostgreSQL** syntax unless stated otherwise. Index features and `EXPLAIN` output differ between databases; the ideas are the same everywhere.

---

## 🧠 1. What Is a SQL Index?

An index is a separate data structure that helps the database find rows without reading the whole table.

Think about a book.

**Without an index:**

```text
Search for a topic
↓
Read page 1, page 2, page 3, ...
↓
Eventually find the topic
```

**With an index:**

```text
Search for a topic
↓
Check the index at the back
↓
Find the page number
↓
Go directly to that page
```

A database index works the same way. Most indexes are **B-trees**: a sorted, balanced tree of key values, each pointing to where the matching rows live.

---

## ❓ 2. Why Do We Need Indexes?

Imagine a table with **10 million** customers:

```sql
SELECT *
FROM customers
WHERE email = 'customer@example.com';
```

Without a suitable index, the database may have to check every row.

With an index:

```sql
CREATE INDEX idx_customers_email
ON customers(email);
```

the database can walk the B-tree and jump almost straight to the matching row — a handful of steps instead of millions.

The **query optimizer** decides which strategy to actually use.

---

## 🛠️ 3. Creating an Index

Basic syntax:

```sql
CREATE INDEX index_name
ON table_name(column_name);
```

Example:

```sql
CREATE INDEX idx_customers_email
ON customers(email);
```

The database now has an index on `customers.email`.

💡 A common naming convention is `idx_<table>_<columns>`.

---

## 🔍 4. Querying an Indexed Column

You **don't** change your SQL after creating an index. You still write:

```sql
SELECT *
FROM customers
WHERE email = 'customer@example.com';
```

The optimizer decides whether using the index is worthwhile.

> Creating an index does **not** guarantee the database will use it.

---

## 🎯 5. Indexes and WHERE Conditions

Indexes are most useful on columns you **filter** on frequently.

```sql
SELECT *
FROM orders
WHERE customer_id = 101;
```

```sql
CREATE INDEX idx_orders_customer_id
ON orders(customer_id);
```

Common candidates:

- `customer_id`, `order_id`, `transaction_id`, `account_id`
- Date columns
- Foreign keys
- Status columns (sometimes — see section 28)

Whether an index actually helps depends on the data, query patterns, database engine and existing indexes.

---

## 🔗 6. Indexes and JOINs

```sql
SELECT
    c.customer_name,
    o.order_amount
FROM customers c
JOIN orders o
    ON c.customer_id = o.customer_id;
```

An index on the join column can help the database find matching rows efficiently:

```sql
CREATE INDEX idx_orders_customer_id
ON orders(customer_id);
```

This is why **foreign-key columns** are often indexed. Note that PostgreSQL and SQL Server do **not** create these indexes automatically (MySQL/InnoDB does).

Base your indexing on actual workload and execution plans, not habit.

---

## 🔃 7. Indexes and ORDER BY

```sql
SELECT *
FROM orders
ORDER BY order_date;
```

Because a B-tree index is already sorted, an index on `order_date` can let the database read rows in order and **skip the sort** entirely:

```sql
CREATE INDEX idx_orders_order_date
ON orders(order_date);
```

This shines with `ORDER BY ... LIMIT` queries such as "latest 10 orders". Again, the optimizer decides whether it's worth it.

---

## 📊 8. Indexes and GROUP BY

```sql
SELECT
    region,
    COUNT(*)
FROM orders
GROUP BY region;
```

An index on `region` *can* help in some databases, but this query still has to count every row. For large aggregations the optimizer often prefers a full scan with hash aggregation.

Indexes don't automatically make every aggregation faster.

---

## 🔑 9. Primary Keys and Indexes

```sql
CREATE TABLE customers (
    customer_id   INT PRIMARY KEY,
    customer_name VARCHAR(100)
);
```

Declaring a primary key **automatically creates a unique index** on it (in PostgreSQL, MySQL, SQL Server and Oracle). That index enforces uniqueness and makes lookups by ID fast.

In MySQL (InnoDB) and, by default, SQL Server, the primary key is also the **clustered index** — the table rows themselves are stored in primary-key order.

---

## 🦄 10. Unique Index

A unique index prevents duplicate values in the indexed column(s):

```sql
CREATE UNIQUE INDEX idx_customers_email
ON customers(email);
```

```text
customer1@example.com  ✅
customer2@example.com  ✅
customer1@example.com  ❌ duplicate — rejected
```

`NULL` handling varies: PostgreSQL, MySQL and Oracle allow many `NULL`s in a unique index, while SQL Server allows only one.

A `UNIQUE` constraint does the same job and is usually implemented with a unique index behind the scenes.

---

## 1️⃣ 11. Single-Column Index

```sql
CREATE INDEX idx_orders_customer
ON orders(customer_id);
```

Useful when queries frequently filter with:

```sql
WHERE customer_id = ...
```

---

## 🧩 12. Composite Index

An index can contain **multiple** columns:

```sql
CREATE INDEX idx_orders_customer_date
ON orders(customer_id, order_date);
```

This is a **composite** (multi-column) index. It suits queries like:

```sql
SELECT *
FROM orders
WHERE customer_id = 101
  AND order_date >= DATE '2026-01-01';
```

---

## 🔢 13. Column Order Matters

This is one of the most important ideas about composite indexes.

```sql
CREATE INDEX idx_orders_customer_date
ON orders(customer_id, order_date);
```

The index is sorted by `customer_id` first, then by `order_date` **within** each customer — like a phone book sorted by last name, then first name.

It works well for:

```sql
WHERE customer_id = 101
```

```sql
WHERE customer_id = 101
  AND order_date >= DATE '2026-01-01'
```

But a query filtering **only** on the second column:

```sql
WHERE order_date >= DATE '2026-01-01'
```

can't use it efficiently — just as a phone book sorted by last name doesn't help you find everyone named "Priya".

---

## ⬅️ 14. The Leftmost Prefix Rule

For:

```sql
CREATE INDEX idx_orders
ON orders(customer_id, order_date, status);
```

the index can be used efficiently when the query filters on a **leftmost prefix** of its columns:

| Filter columns                        | Uses index efficiently? |
| ------------------------------------- | ----------------------- |
| `customer_id`                         | ✅                      |
| `customer_id, order_date`             | ✅                      |
| `customer_id, order_date, status`     | ✅                      |
| `order_date`                          | ❌ skips leading column |
| `status`                              | ❌ skips leading column |
| `customer_id, status`                 | ⚠️ only `customer_id` part |

💡 **Rule of thumb:** put **equality** columns (`=`) first and **range** columns (`>`, `<`, `BETWEEN`) last. Once the index hits a range condition, the columns after it can't narrow the search as effectively.

---

## 🔤 15. Indexes and LIKE

```sql
SELECT *
FROM customers
WHERE customer_name LIKE 'Rah%';
```

A **prefix** search can use a B-tree index — it's a range: everything from `'Rah'` up to `'Rai'`.

But patterns that **start** with a wildcard:

```sql
WHERE customer_name LIKE '%Rah';
WHERE customer_name LIKE '%Rah%';
```

can't use a normal B-tree index, because the sort order is based on the first characters.

For those, databases offer special features — e.g. PostgreSQL's trigram (`pg_trgm`) indexes or full-text search.

💡 In PostgreSQL, a prefix `LIKE` only uses the index if the database uses the `C` collation or the index is created with `text_pattern_ops`.

---

## 🧮 16. Indexes and Functions

```sql
SELECT *
FROM customers
WHERE UPPER(customer_name) = 'RAHUL';
```

A normal index on `customer_name` can't be used here: the index stores `customer_name`, not `UPPER(customer_name)`.

Fix 1 — create an **expression index** that matches the query:

```sql
CREATE INDEX idx_customers_upper_name
ON customers(UPPER(customer_name));
```

(Supported by PostgreSQL, Oracle and MySQL 8.0.13+. SQL Server uses an indexed computed column instead.)

Fix 2 — rewrite the condition so the column stands alone. This matters a lot for dates:

```sql
-- ❌ Function on the column: can't use an index on order_date
WHERE EXTRACT(YEAR FROM order_date) = 2026

-- ✅ Same meaning, index-friendly (half-open range from Part 15)
WHERE order_date >= DATE '2026-01-01'
  AND order_date <  DATE '2027-01-01'
```

---

## ⚫ 17. Indexes and NULL

```sql
SELECT *
FROM customers
WHERE email IS NULL;
```

Behavior varies by database:

- **PostgreSQL, MySQL, SQL Server** include `NULL`s in B-tree indexes, so `IS NULL` can use the index.
- **Oracle** doesn't store rows in a single-column B-tree index when the key is entirely `NULL`, so `IS NULL` usually can't use it.

Don't assume every database handles `NULL` indexing the same way.

---

## 🚫 18. Why Not Index Every Column?

A common beginner assumption:

```text
More indexes = Faster database
```

That's not true. Every index must be kept up to date. On each

```text
INSERT / UPDATE / DELETE
```

the database updates the table **and** every affected index.

```text
More indexes
↓
More storage
↓
More maintenance
↓
Slower writes
```

Create indexes based on real query patterns, and remove ones nobody uses.

---

## 💽 19. Indexes Have a Storage Cost

On a table with **100 million** rows, each index can take up gigabytes of storage — sometimes as much as the table itself.

So indexing is always a trade-off:

```text
Read performance
       ↕
Write performance
       ↕
Storage
```

A good indexing strategy balances all three.

---

## 🏎️ 20. Query Performance Is More Than Indexes

Other factors include:

- Query structure
- JOIN strategy
- Filtering
- Data volume
- Table design and data types
- Statistics
- Partitioning
- Database engine
- Execution plan
- Sorting and aggregation
- Network transfer

A slow query isn't automatically an "index problem".

---

## 📦 21. SELECT * and Performance

```sql
SELECT *
FROM orders
WHERE customer_id = 101;
```

If you only need three columns, ask for three columns:

```sql
SELECT
    order_id,
    order_date,
    order_amount
FROM orders
WHERE customer_id = 101;
```

Fetching unnecessary columns increases:

- Data transfer
- Memory usage
- I/O
- Downstream processing work

It also makes your SQL less explicit — and it prevents **covering indexes** (next section) from working.

---

## 🧾 22. Covering Indexes

If an index contains **every** column a query needs, the database can answer the query from the index alone, without touching the table. This is called a **covering index** (an "index-only scan").

```sql
-- PostgreSQL 11+ / SQL Server: INCLUDE stores extra columns in the index
CREATE INDEX idx_orders_customer_covering
ON orders(customer_id)
INCLUDE (order_id, order_date, order_amount);
```

```sql
SELECT order_id, order_date, order_amount
FROM orders
WHERE customer_id = 101;   -- answered entirely from the index
```

In MySQL, add the extra columns to the index key itself instead.

---

## 🧹 23. Filter Early

```sql
SELECT
    customer_id,
    SUM(order_amount) AS total_sales
FROM orders
WHERE order_date >= DATE '2026-01-01'
GROUP BY customer_id;
```

Filtering **before** aggregating can drastically reduce the work:

```text
10 million rows → Filter → 2 million rows → Aggregate
```

instead of:

```text
10 million rows → Aggregate everything → Filter later
```

The optimizer can rearrange some queries internally, but clear `WHERE` conditions are still the best habit. Use `HAVING` only for conditions on aggregated values.

---

## 🏭 24. Let the Database Do the Heavy Lifting

```sql
SELECT
    transaction_id,
    amount
FROM transactions
WHERE status = 'Completed';
```

Don't pull every transaction into Python or Excel and filter it there when the database can filter it far more efficiently.

> Let the database do the filtering and aggregation whenever practical.

---

## 🗺️ 25. Execution Plans

The most important tool for understanding performance is the **execution plan** — the step-by-step strategy the database chooses for a query.

It can show things such as:

- Table scan (Seq Scan / Full Table Scan)
- Index scan / index seek
- Join strategy (nested loop, hash join, merge join)
- Sort
- Aggregation
- Estimated vs actual rows
- Cost

Terminology varies between databases.

---

## 🔬 26. EXPLAIN

Most databases support `EXPLAIN`:

```sql
EXPLAIN
SELECT *
FROM orders
WHERE customer_id = 101;
```

This shows the **planned** strategy without running the query.

```sql
EXPLAIN ANALYZE
SELECT *
FROM orders
WHERE customer_id = 101;
```

`EXPLAIN ANALYZE` (PostgreSQL, MySQL 8.0.18+) actually **runs** the query and reports real timings and row counts.

⚠️ Because `EXPLAIN ANALYZE` executes the statement, running it on an `UPDATE` or `DELETE` really changes data. Wrap it in a transaction and `ROLLBACK`.

A simplified PostgreSQL example:

```text
Index Scan using idx_orders_customer_id on orders
  Index Cond: (customer_id = 101)
  (actual rows=42)
```

SQL Server shows plans graphically ("Include Actual Execution Plan") or via `SET SHOWPLAN_ALL ON`.

---

## 📖 27. Table Scan vs Index Access

On a table with **10,000,000** rows:

**Table scan** — read every row and keep the matches:

```text
10 million rows → Check each row → Matching rows
```

**Index access** — use the index to go straight to the matches:

```text
Index → Locate matching keys → Fetch only those rows
```

For **highly selective** queries (few matching rows), the index is much faster.

But if a query needs a large share of the table, a scan can be cheaper — reading the table sequentially beats jumping back and forth between index and table millions of times. That's why the optimizer, not you, makes the choice.

---

## 🎚️ 28. Selectivity & Cardinality

**Selectivity** describes how much a condition narrows the data.

```sql
WHERE customer_id = 100245
```

If customer IDs are unique, this returns one row — **highly selective**. An index is ideal.

```sql
WHERE country = 'India'
```

If 60% of customers are Indian, this is **poorly selective**. Scanning the table may be cheaper.

**Low-cardinality columns** have only a few distinct values, like `status` with just `Active` / `Inactive`. If 95% of rows are `Active`, an index on `status` won't help `WHERE status = 'Active'`. It might still help the rare value (`WHERE status = 'Inactive'`), or as the second column of a composite index.

> An index isn't automatically useful just because a column appears in `WHERE`.

💡 PostgreSQL **partial indexes** can target just the rare rows:

```sql
CREATE INDEX idx_orders_pending
ON orders(order_date)
WHERE status = 'Pending';
```

---

## 🏢 29. Real-World Example

An `orders` table has **50 million** rows, and analysts often run:

```sql
SELECT
    order_id,
    order_date,
    order_amount
FROM orders
WHERE customer_id = 12345
  AND order_date >= DATE '2026-01-01';
```

A strong candidate:

```sql
CREATE INDEX idx_orders_customer_date
ON orders(customer_id, order_date);
```

Why?

- `customer_id` is an **equality** filter and highly selective → it goes first.
- `order_date` is a **range** filter → it goes second.
- Within one customer, the index is already sorted by date, so the database reads one contiguous slice.

Verify the impact with `EXPLAIN ANALYZE` on real data.

---

## ✅ 30. Query Optimization Checklist

When a query is slow, ask:

1. **Do I need every column?** Replace `SELECT *` with the columns you need.
2. **Can I filter earlier?** Push conditions into `WHERE`.
3. **Are the JOIN conditions correct?** A missing or wrong `ON` condition multiplies rows.
4. **Could an index help?** Look at columns used in `WHERE`, `JOIN` and `ORDER BY`.
5. **Is the index actually used?** Check the execution plan.
6. **Am I processing unnecessary rows?** Check the data volume at each step.
7. **Are functions blocking index use?** e.g. `WHERE UPPER(name) = ...` or `WHERE EXTRACT(YEAR FROM date) = ...`
8. **Do I have too many indexes?** Each one slows down writes.

---

## 💼 Data Analyst Example

A dashboard runs:

```sql
SELECT
    customer_id,
    SUM(order_amount) AS total_sales
FROM orders
WHERE order_date >= DATE '2026-01-01'
GROUP BY customer_id;
```

The table contains **100 million** orders. Things to investigate:

1. Is `order_date` indexed?
2. How selective is the date filter? (Last week vs. last 5 years is a huge difference.)
3. How many rows are processed?
4. Is the table partitioned by date?
5. What does `EXPLAIN ANALYZE` show?
6. Is the aggregation itself the expensive part?
7. Does the dashboard run this repeatedly? If so, a **materialized view** (Part 17) may be the better fix.

A strong analyst doesn't immediately say "create an index". They look at the execution plan and the workload first.

---

## 🎤 SQL Interview Questions

**Q1. What is an index?**

A separate data structure (usually a B-tree) that helps the database locate rows without scanning the whole table.

**Q2. Why are indexes useful?**

They speed up suitable read queries — especially filtering, joining and sorting on indexed columns.

**Q3. Can indexes slow down `INSERT`s?**

Yes. Every index on the table must also be updated when a row is inserted.

**Q4. Can indexes slow down `UPDATE` and `DELETE`?**

Yes, when the affected rows or indexed columns require index maintenance. (Indexes can also *speed up* finding the rows to update or delete.)

**Q5. What is a composite index?**

An index on multiple columns.

```sql
CREATE INDEX idx_customer_date
ON orders(customer_id, order_date);
```

**Q6. Does column order in a composite index matter?**

Yes. The index can be used efficiently only for filters on a leftmost prefix of its columns. Put equality columns before range columns.

**Q7. Does every query use an index if one exists?**

No. The optimizer uses an index only when it estimates that's cheaper than the alternatives, such as a full scan.

**Q8. What is `EXPLAIN`?**

A command that shows a query's execution plan. `EXPLAIN ANALYZE` also runs the query and reports actual timings and row counts.

**Q9. What is selectivity?**

How much a condition narrows the number of matching rows. Highly selective conditions benefit most from indexes.

**Q10. Why shouldn't you index every column?**

Indexes use storage and must be maintained on every write, so too many indexes slow down `INSERT`, `UPDATE` and `DELETE` without always helping reads.

---

## 📝 Practice Questions

**Practice 1 — Create an index on `customer_id`.**

```sql
CREATE INDEX idx_orders_customer_id
ON orders(customer_id);
```

**Practice 2 — Create a composite index on customer and date.**

```sql
CREATE INDEX idx_orders_customer_date
ON orders(customer_id, order_date);
```

**Practice 3 — Create a unique index on email.**

```sql
CREATE UNIQUE INDEX idx_customers_email
ON customers(email);
```

**Practice 4 — Write a query that could benefit from an index on `customer_id`.**

```sql
SELECT
    order_id,
    order_date,
    order_amount
FROM orders
WHERE customer_id = 101;
```

**Practice 5 — Inspect the execution plan.**

```sql
EXPLAIN
SELECT *
FROM orders
WHERE customer_id = 101;
```

If supported:

```sql
EXPLAIN ANALYZE
SELECT *
FROM orders
WHERE customer_id = 101;
```

---

## 🧪 Mini SQL Challenge

An `orders` table has **50 million** rows:

```text
order_id, customer_id, order_date, status, region, order_amount
```

This query is slow:

```sql
SELECT
    order_id,
    order_date,
    order_amount
FROM orders
WHERE customer_id = 5001
  AND order_date >= DATE '2026-01-01'
  AND status = 'Completed';
```

Think about:

1. Which columns are being filtered?
2. Is a composite index worth investigating?
3. Which column should come first?
4. Should you use `SELECT *`?
5. How would you inspect the execution plan?
6. Would the index always be used?
7. What happens as the table receives millions of new rows?

**Suggested answer**

1. `customer_id` (equality), `status` (equality) and `order_date` (range).
2. Yes — all three filters appear together, and `customer_id` is highly selective.
3. Equality columns first, range column last:

```sql
CREATE INDEX idx_orders_customer_status_date
ON orders(customer_id, status, order_date);
```

   With `(customer_id, status, order_date)`, the database jumps to customer 5001's *Completed* orders and reads only the dates from 2026 onward. With `(customer_id, order_date)`, it would read all of the customer's 2026 orders and throw away the non-completed ones.

   To make it **covering**, add the selected columns (PostgreSQL 11+ / SQL Server):

```sql
CREATE INDEX idx_orders_customer_status_date
ON orders(customer_id, status, order_date)
INCLUDE (order_id, order_amount);
```

4. No — select only `order_id`, `order_date` and `order_amount`, as the query already does. That's what makes the covering index possible.
5. Run it with `EXPLAIN` (plan) or `EXPLAIN ANALYZE` (plan plus real timings):

```sql
EXPLAIN ANALYZE
SELECT
    order_id,
    order_date,
    order_amount
FROM orders
WHERE customer_id = 5001
  AND order_date >= DATE '2026-01-01'
  AND status = 'Completed';
```

6. No. If customer 5001 has a huge share of all orders, the optimizer may still choose a scan. Out-of-date statistics can also lead to a bad choice.
7. Every insert now updates this index too, so writes get slightly slower and the index grows. Keep statistics up to date (`ANALYZE` in PostgreSQL) and check that the index is still pulling its weight.

Don't assume this is automatically the best index. Let the execution plan, data distribution and workload decide.

**What did we use?**

| Tool                        | Purpose                                   |
| --------------------------- | ----------------------------------------- |
| Composite index             | Match all three filter columns            |
| Equality-before-range order | Read one contiguous slice of the index    |
| `INCLUDE` (covering index)  | Answer the query from the index alone     |
| `EXPLAIN ANALYZE`           | Verify the index is used and actually helps |

---

## 🎯 Key Takeaway

```text
INDEX            → Helps the database find data efficiently
COMPOSITE INDEX  → Index on multiple columns (order matters!)
COVERING INDEX   → Index that contains every column a query needs
EXPLAIN          → Shows how the database plans to run a query
SELECTIVITY      → How much a filter narrows the data
```

And the most important principle:

```text
More indexes ≠ Always faster

Good indexing
+ Good query design
+ Execution-plan analysis
= Better SQL performance
```

A strong Data Analyst doesn't just write SQL that returns the right answer — they understand how it behaves when the data grows from thousands of rows to billions. 🚀

---

💡 **Double Tap ❤️ For More**
