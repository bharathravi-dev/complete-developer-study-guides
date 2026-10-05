# 🚀 SQL Roadmap 2026 — Part 16

## 🔗 SQL Set Operators — UNION, UNION ALL, INTERSECT & EXCEPT

Set operators let you combine the results of multiple `SELECT` queries into one result.

They are extremely useful when you have similar datasets and want to:

- Combine records from different sources
- Remove duplicates
- Find common records
- Find records that exist in one dataset but not another
- Compare two datasets
- Combine current and historical data

The four important set operators are:

| Operator    | Returns                                   |
| ----------- | ----------------------------------------- |
| `UNION`     | All rows from both queries, deduplicated  |
| `UNION ALL` | All rows from both queries, duplicates kept |
| `INTERSECT` | Rows present in **both** queries          |
| `EXCEPT`    | Rows in the first query but **not** the second |

⚠️ **Dialect note:** Oracle uses `MINUS` instead of `EXCEPT` (Oracle 21c+ also accepts `EXCEPT`). MySQL only supports `INTERSECT` and `EXCEPT` from **8.0.31** onward.

---

## 🧠 1. What Are Set Operators?

Suppose you have two tables, `customers_2025` and `customers_2026`. Both contain `customer_id` and `customer_name`, and you want one result containing customers from both years.

```sql
SELECT customer_id, customer_name FROM customers_2025
UNION
SELECT customer_id, customer_name FROM customers_2026;
```

The two result sets are combined into one.

---

## ➕ 2. UNION

`UNION` combines two result sets and **removes duplicate rows**.

```sql
SELECT customer_id FROM customers_2025
UNION
SELECT customer_id FROM customers_2026;
```

If customer `101` appears in both tables, it appears only once in the final result.

```text
2025   : 101, 102, 103
2026   : 102, 103, 104
Result : 101, 102, 103, 104
```

**When to use `UNION`?** When duplicate rows should be removed.

```sql
SELECT email FROM online_customers
UNION
SELECT email FROM store_customers;
```

This produces a unique list of customer emails across both channels.

💡 Duplicates are removed across the **whole** result, including duplicates that exist *within* a single table.

---

## 📚 3. UNION ALL

`UNION ALL` also combines result sets, but **keeps duplicates**.

```sql
SELECT customer_id FROM customers_2025
UNION ALL
SELECT customer_id FROM customers_2026;
```

Using the previous example:

```text
Result : 101, 102, 103, 102, 103, 104
```

The duplicate records remain.

---

## ⚖️ 4. UNION vs UNION ALL

This is one of the most common SQL interview questions.

| Operator    | Combines results | Duplicates |
| ----------- | ---------------- | ---------- |
| `UNION`     | ✅               | Removed    |
| `UNION ALL` | ✅               | Kept       |

**Performance consideration:** `UNION` needs extra work (usually a sort or hash) to find and remove duplicates. `UNION ALL` skips that step.

So when duplicates are valid, or you already know the inputs can't overlap, `UNION ALL` is generally the better choice.

---

## 📏 5. Rules for Using Set Operators

The queries being combined must be **compatible**.

✅ Valid — both queries return 2 columns:

```sql
SELECT customer_id, customer_name FROM customers
UNION
SELECT customer_id, customer_name FROM archived_customers;
```

❌ Invalid — the number of columns doesn't match:

```sql
SELECT customer_id, customer_name FROM customers
UNION
SELECT customer_id FROM archived_customers;
```

**Rules:** every participating `SELECT` must have:

1. The **same number** of columns
2. **Compatible data types** in corresponding positions

The column names in the final result come from the **first** `SELECT`.

---

## 🔢 6. Column Order Matters

Set operators match columns **by position, not by name**.

```sql
-- ⚠️ Runs without error if both are text — but the data is scrambled!
SELECT customer_name, city FROM customers
UNION ALL
SELECT city, customer_name FROM archived_customers;
```

Because the types match, the database won't complain — names and cities just end up in the wrong columns. Always list columns in the same order:

```sql
SELECT customer_id, customer_name FROM customers
UNION ALL
SELECT customer_id, customer_name FROM archived_customers;
```

---

## 🔃 7. ORDER BY with UNION

To sort the final combined result, put a single `ORDER BY` at the **end**.

```sql
SELECT customer_id, customer_name FROM customers_2025
UNION
SELECT customer_id, customer_name FROM customers_2026
ORDER BY customer_id;
```

The `ORDER BY` applies to the whole combined result, not just the last query. It must refer to column names (or positions) from the **first** `SELECT`.

---

## 🤝 8. INTERSECT

`INTERSECT` returns rows that exist in **both** result sets.

```sql
SELECT customer_id FROM customers_2025
INTERSECT
SELECT customer_id FROM customers_2026;
```

```text
2025   : 101, 102, 103
2026   : 102, 103, 104
Result : 102, 103
```

**Real-world use** — customers who purchased in both years:

```sql
SELECT customer_id FROM purchases_2025
INTERSECT
SELECT customer_id FROM purchases_2026;
```

---

## ➖ 9. EXCEPT

`EXCEPT` returns rows from the **first** query that don't exist in the **second**.

```sql
SELECT customer_id FROM customers_2025
EXCEPT
SELECT customer_id FROM customers_2026;
```

```text
Result : 101
```

Order matters: `A EXCEPT B` is not the same as `B EXCEPT A` (which here would return `104`).

In Oracle, write `MINUS` instead of `EXCEPT`.

---

## 🧮 10. Understanding the Four Operators

Suppose `A = {1, 2, 3}` and `B = {2, 3, 4}`:

| Operation         | Result             |
| ----------------- | ------------------ |
| `A UNION B`       | `{1, 2, 3, 4}`     |
| `A UNION ALL B`   | `{1, 2, 3, 2, 3, 4}` |
| `A INTERSECT B`   | `{2, 3}`           |
| `A EXCEPT B`      | `{1}`              |

This is the easiest way to remember them.

💡 Like `UNION`, plain `INTERSECT` and `EXCEPT` also **remove duplicates**. PostgreSQL (and some others) support `INTERSECT ALL` / `EXCEPT ALL` to keep them.

---

## 🛒 11. Combining Similar Data

```sql
SELECT order_id, customer_id, sales_amount FROM online_sales
UNION ALL
SELECT order_id, customer_id, sales_amount FROM store_sales;
```

Tip: add a literal column so you still know where each row came from.

```sql
SELECT 'Online' AS channel, order_id, customer_id, sales_amount FROM online_sales
UNION ALL
SELECT 'Store'  AS channel, order_id, customer_id, sales_amount FROM store_sales;
```

---

## 🗄️ 12. Current + Historical Data

```sql
SELECT * FROM current_transactions
UNION ALL
SELECT * FROM historical_transactions;
```

⚠️ `SELECT *` only works if both tables have exactly the same columns in exactly the same order. If someone adds a column to one table, the query breaks (or worse, silently misaligns). Listing columns explicitly is safer.

---

## 👥 13. Finding Common Customers

```sql
SELECT customer_id FROM product_a_users
INTERSECT
SELECT customer_id FROM product_b_users;
```

Customers who use **both** product A and product B.

---

## 🚪 14. Finding Customers in One Group but Not Another

```sql
SELECT customer_id FROM product_a_users
EXCEPT
SELECT customer_id FROM product_b_users;
```

Customers who use product A but **not** product B.

The same pattern answers churn questions — e.g. customers active last year but not this year:

```sql
SELECT customer_id FROM active_users_2025
EXCEPT
SELECT customer_id FROM active_users_2026;
```

---

## 🔀 15. Set Operators vs JOINs

**JOIN** combines **columns** from related tables:

```sql
SELECT c.customer_id, c.customer_name, o.order_amount
FROM customers c
JOIN orders o
    ON c.customer_id = o.customer_id;
```

**UNION** combines **rows** from compatible queries.

```text
JOIN  → Add columns (horizontal)
UNION → Add rows    (vertical)
```

---

## 📋 16. UNION vs JOIN — Example

```text
Table A              Table B
101  Rahul           103  Amit
102  Priya           104  Neha
```

To stack the records:

```sql
SELECT customer_id, name FROM table_a
UNION ALL
SELECT customer_id, name FROM table_b;
```

| customer_id | name  |
| ----------- | ----- |
| 101         | Rahul |
| 102         | Priya |
| 103         | Amit  |
| 104         | Neha  |

---

## 🔍 17. Using Set Operators with Filters

Each `SELECT` can have its own `WHERE` clause:

```sql
SELECT customer_id FROM customers_2025 WHERE country = 'India'
UNION
SELECT customer_id FROM customers_2026 WHERE country = 'India';
```

---

## 📊 18. Set Operators with Aggregation

Each `SELECT` can also aggregate on its own. Add a label column so you can tell the years apart:

```sql
SELECT 2025 AS year, region, SUM(sales_amount) AS total_sales
FROM sales_2025
GROUP BY region

UNION ALL

SELECT 2026 AS year, region, SUM(sales_amount) AS total_sales
FROM sales_2026
GROUP BY region;
```

For **one combined total per region**, stack the raw rows first, then aggregate:

```sql
SELECT region, SUM(sales_amount) AS total_sales
FROM (
    SELECT region, sales_amount FROM sales_2025
    UNION ALL
    SELECT region, sales_amount FROM sales_2026
) s
GROUP BY region;
```

⚠️ Use `UNION ALL` here, not `UNION`. With `UNION`, two different sales that happen to have the same `(region, sales_amount)` would collapse into one row and your total would be too low.

---

## ⚫ 19. NULL Values with Set Operators

`NULL`s can take part in set operations. For duplicate removal, set operators treat two `NULL`s as **equal** — unlike `WHERE a = b`, where `NULL = NULL` is not true.

```text
A : 1, NULL        B : NULL, 2

A UNION B     → 1, NULL, 2      (only one NULL)
A INTERSECT B → NULL
A EXCEPT B    → 1
```

This makes `EXCEPT` a handy NULL-safe alternative to `NOT IN`, which returns no rows at all if the subquery contains a `NULL`.

---

## 🧩 20. Operator Precedence

When you mix operators, `INTERSECT` is evaluated **before** `UNION` and `EXCEPT` (in PostgreSQL, SQL Server and the SQL standard). `UNION` and `EXCEPT` run left to right.

```sql
SELECT id FROM a
UNION
SELECT id FROM b
INTERSECT
SELECT id FROM c;
-- means: a UNION (b INTERSECT c)
```

Use parentheses to make your intent obvious:

```sql
(SELECT id FROM a UNION SELECT id FROM b)
INTERSECT
SELECT id FROM c;
```

---

## ⚠️ 21. Common Mistakes

❌ **Mistake 1: Different number of columns** — every `SELECT` must return the same number of columns.

❌ **Mistake 2: Incompatible data types** — e.g. a `DATE` column in the same position as an `INT` column.

❌ **Mistake 3: Columns in a different order** — the query runs, but the data ends up in the wrong columns.

❌ **Mistake 4: Using `UNION` when duplicates matter** — counts and sums come out wrong. Use `UNION ALL`.

❌ **Mistake 5: `ORDER BY` in the middle** — put a single `ORDER BY` at the end of the whole statement.

❌ **Mistake 6: Confusing JOIN and UNION** — JOIN combines related **columns**, UNION combines compatible **rows**.

---

## 💼 22. Business Example

Total number of payments across two years:

```sql
SELECT COUNT(*) AS payment_count
FROM (
    SELECT payment_id FROM payments_2025
    UNION ALL
    SELECT payment_id FROM payments_2026
) p;
```

Total payment value across two years:

```sql
SELECT SUM(payment_amount) AS total_payment_value
FROM (
    SELECT payment_amount FROM payments_2025
    UNION ALL
    SELECT payment_amount FROM payments_2026
) p;
```

Both must use `UNION ALL` — `UNION` would merge different payments that happen to share the same amount.

---

## 🎤 SQL Interview Questions

**Q1. What is `UNION`?**

It combines results from multiple `SELECT` statements and removes duplicate rows.

**Q2. What is `UNION ALL`?**

It combines results and keeps duplicate rows.

**Q3. Which is generally faster: `UNION` or `UNION ALL`?**

`UNION ALL`, because it skips the duplicate-removal step. Use it whenever deduplication isn't needed.

**Q4. What is the difference between `UNION` and `JOIN`?**

`JOIN` combines related tables horizontally by adding columns. `UNION` combines compatible result sets vertically by adding rows.

**Q5. What does `INTERSECT` do?**

It returns rows that appear in both result sets.

**Q6. What does `EXCEPT` do?**

It returns rows present in the first result set but absent from the second.

**Q7. What is `MINUS`?**

Oracle's name for `EXCEPT`.

**Q8. Do the columns need the same names?**

No. They need the same number of columns and compatible data types in the same positions. The result uses the column names from the first `SELECT`.

**Q9. Can you use `ORDER BY` with `UNION`?**

Yes. Put a single `ORDER BY` after the complete set operation; it sorts the combined result.

**Q10. When would you choose `UNION ALL` over `UNION`?**

When duplicate rows are valid (e.g. for counting or summing), or when you know the inputs can't overlap and want to avoid the deduplication cost.

---

## 📝 Practice Questions

**Practice 1 — Combine customers from two tables, keeping duplicates.**

```sql
SELECT customer_id FROM customers_a
UNION ALL
SELECT customer_id FROM customers_b;
```

**Practice 2 — Find customers who appear in both tables.**

```sql
SELECT customer_id FROM customers_a
INTERSECT
SELECT customer_id FROM customers_b;
```

**Practice 3 — Find customers in `customers_a` but not in `customers_b`.**

```sql
SELECT customer_id FROM customers_a
EXCEPT
SELECT customer_id FROM customers_b;
```

**Practice 4 — Combine two years of transactions and calculate the total value.**

```sql
SELECT SUM(amount) AS total_amount
FROM (
    SELECT amount FROM transactions_2025
    UNION ALL
    SELECT amount FROM transactions_2026
) t;
```

**Practice 5 — Find unique customer IDs across two sales channels.**

```sql
SELECT customer_id FROM online_sales
UNION
SELECT customer_id FROM store_sales;
```

---

## 🧪 Mini SQL Challenge

You have two tables, `orders_2025` and `orders_2026`, each with:

```text
order_id, customer_id, order_amount
```

Write SQL to:

1. Combine all orders from both years
2. Calculate the total order value
3. Calculate the total number of orders
4. Find customers who ordered in both years
5. Find customers who ordered in 2025 but not in 2026

**Solution**

```sql
-- 1. Combine all orders
SELECT order_id, customer_id, order_amount FROM orders_2025
UNION ALL
SELECT order_id, customer_id, order_amount FROM orders_2026;

-- 2. Total order value
SELECT SUM(order_amount) AS total_order_value
FROM (
    SELECT order_amount FROM orders_2025
    UNION ALL
    SELECT order_amount FROM orders_2026
) o;

-- 3. Total number of orders
SELECT COUNT(*) AS total_orders
FROM (
    SELECT order_id FROM orders_2025
    UNION ALL
    SELECT order_id FROM orders_2026
) o;

-- 4. Customers who ordered in both years
SELECT customer_id FROM orders_2025
INTERSECT
SELECT customer_id FROM orders_2026;

-- 5. Customers who ordered in 2025 but not 2026
SELECT customer_id FROM orders_2025
EXCEPT
SELECT customer_id FROM orders_2026;
```

**What did we use?**

| Tool                       | Purpose                                 |
| -------------------------- | --------------------------------------- |
| `UNION ALL`                | Stack every order, keeping duplicates   |
| Subquery + `SUM` / `COUNT` | Aggregate over the combined rows        |
| `INTERSECT`                | Customers present in both years         |
| `EXCEPT`                   | Customers present only in 2025          |

---

💡 **Double Tap ❤️ For More**
