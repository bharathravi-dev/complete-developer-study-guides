# 🚀 SQL Roadmap 2026 — Part 8

## 🕳️ NULL Handling, COALESCE & NULLIF — Managing Missing Data in SQL

In real-world databases, **missing data is extremely common**:

- Customers may not have a phone number.
- Orders may not have a discount.
- Employees may not have a resignation date.
- Transactions may have missing reference values.

SQL uses `NULL` to represent an **unknown or missing** value.

> Understanding `NULL` properly is essential for accurate SQL queries and data analysis.

---

## 🧠 1. What is NULL?

`NULL` means:

> «The value is missing, unknown, or not available.»

**Example:**

| customer_id | customer_name | phone      |
| ----------- | ------------- | ---------- |
| 101         | Alice         | 9876543210 |
| 102         | Bob           | `NULL`     |
| 103         | Charlie       | 9123456780 |

Bob's phone number is **not stored**.

It does **not** necessarily mean:

- `0`
- an empty string `''`
- `'Unknown'`
- `'N/A'`

> These are **different values**.

---

## ⚠️ 2. NULL Is Not Equal to 0

```sql
SELECT *
FROM customers
WHERE credit_limit = 0;
```

This finds customers whose credit limit is **actually zero**. It will **not** find customers whose credit limit is missing.

To find missing values:

```sql
SELECT *
FROM customers
WHERE credit_limit IS NULL;
```

---

## ⚠️ 3. Never Use `= NULL`

```sql
-- ❌ Incorrect — won't correctly identify NULL values
SELECT *
FROM customers
WHERE phone = NULL;
```

Use:

```sql
-- ✅ Correct
SELECT *
FROM customers
WHERE phone IS NULL;
```

And for non-`NULL` values:

```sql
SELECT *
FROM customers
WHERE phone IS NOT NULL;
```

**Remember:**

| Expression | Valid? |
| ---------- | ------ |
| `= NULL` | ❌ |
| `<> NULL` | ❌ |
| `IS NULL` | ✅ |
| `IS NOT NULL` | ✅ |

---

## 🔢 4. NULL in Calculations

Suppose:

| order_id | price | discount |
| -------- | ----- | -------- |
| 1        | 1000  | 100      |
| 2        | 800   | `NULL`   |

Now:

```sql
SELECT
    price,
    discount,
    price - discount AS final_price
FROM orders;
```

For order 2, the result may be `NULL`, because:

```text
800 - NULL = NULL
```

> SQL generally cannot determine the result when one operand is unknown.

---

## 🛠️ 5. COALESCE()

`COALESCE()` is one of the most important functions for handling `NULL` values. It returns **the first non-NULL value**.

**Syntax:**

```sql
COALESCE(value1, value2, value3, ...)
```

**Example:**

```sql
SELECT
    customer_name,
    COALESCE(phone, 'Not Available') AS phone
FROM customers;
```

If `phone` is `NULL`:

```text
NULL → Not Available
```

---

## 🎯 6. COALESCE with Multiple Values

You can provide several fallback values.

```sql
SELECT
    customer_name,
    COALESCE(phone, email, 'No Contact Information') AS contact
FROM customers;
```

SQL checks in order:

```text
phone
  ↓
email
  ↓
No Contact Information
```

> The **first non-NULL** value is returned.

---

## 💰 7. COALESCE for Financial Calculations

Suppose discounts can be `NULL`.

```sql
-- Instead of
SELECT price - discount AS final_price FROM orders;

-- Use
SELECT price - COALESCE(discount, 0) AS final_price FROM orders;
```

Now a missing discount is treated as zero.

| Price | Discount | Final Price |
| ----- | -------- | ----------- |
| 1000  | 100      | 900         |
| 800   | `NULL`   | 800         |

> This is extremely common in analytics.

---

## 📊 8. COALESCE with Aggregations

Suppose there are no matching transactions for a customer. You may want to display `0` instead of `NULL`.

```sql
SELECT
    customer_id,
    COALESCE(SUM(amount), 0) AS total_spending
FROM transactions
GROUP BY customer_id;
```

> This makes reports easier to interpret.

---

## 🧮 9. NULL and COUNT()

These two queries behave differently:

```sql
SELECT COUNT(*) FROM customers;      -- Counts all rows
SELECT COUNT(phone) FROM customers;  -- Counts only rows where phone is NOT NULL
```

**Example:**

| customer | phone  |
| -------- | ------ |
| A        | 12345  |
| B        | `NULL` |
| C        | 67890  |

```text
COUNT(*)      → 3
COUNT(phone)  → 2
```

> 💼 This difference is frequently tested in interviews.

---

## 📈 10. NULL and SUM(), AVG(), MIN(), MAX()

Most aggregate functions **ignore** `NULL` values.

**Example salaries:** `50000`, `60000`, `NULL`, `70000`

```sql
SELECT AVG(salary) FROM employees;
```

The `NULL` salary is generally ignored, so the average is calculated using `50000, 60000, 70000` — **not four values**.

**Important:**

| Function | Behavior |
| -------- | -------- |
| `COUNT(*)` | Counts **rows** |
| `COUNT(column)` | Ignores `NULL` |
| `SUM()`, `AVG()`, `MIN()`, `MAX()` | Generally ignore `NULL` values |

---

## 🔄 11. NULL with CASE

`NULL` can be handled using `CASE`.

```sql
SELECT
    customer_name,
    CASE
        WHEN phone IS NULL THEN 'Missing'
        ELSE 'Available'
    END AS phone_status
FROM customers;
```

**Result:**

| customer | phone_status |
| -------- | ------------ |
| Alice    | Available    |
| Bob      | Missing      |
| Charlie  | Available    |

---

## 🧹 12. Handling NULL in Data Cleaning

Suppose customer cities contain missing values.

```sql
SELECT
    customer_name,
    COALESCE(city, 'Unknown') AS city
FROM customers;
```

This can make reports more readable. **But be careful:**

> ⚠️ Replacing `NULL` does not mean the original data wasn't missing. For analysis, it may still be important to track missingness.

---

## 🧨 13. NULLIF()

`NULLIF()` returns `NULL` when two expressions are **equal**.

**Syntax:**

```sql
NULLIF(value1, value2)
```

**Examples:**

```sql
SELECT NULLIF(10, 10);  -- Result: NULL
SELECT NULLIF(10, 5);   -- Result: 10
```

---

## 🚨 14. NULLIF() for Division by Zero

> This is one of the most useful real-world applications.

```sql
SELECT
    revenue / orders AS revenue_per_order
FROM sales;
```

If `orders = 0`, some database systems will raise a **division-by-zero error**.

Use:

```sql
SELECT
    revenue / NULLIF(orders, 0) AS revenue_per_order
FROM sales;
```

If `orders = 0`, then `NULLIF(orders, 0)` returns `NULL`, so the calculation becomes `revenue / NULL` and returns `NULL` instead of attempting division by zero.

You can then provide a fallback:

```sql
SELECT
    COALESCE(
        revenue / NULLIF(orders, 0),
        0
    ) AS revenue_per_order
FROM sales;
```

This combines:

| Function | Role |
| -------- | ---- |
| `NULLIF` | Prevent invalid division |
| `COALESCE` | Provide fallback value |

---

## 🔗 15. NULL in JOINs

`NULL` becomes especially important with joins.

Suppose `customers` contains **all** customers, while `orders` contains only customers who placed orders.

```sql
SELECT
    c.customer_id,
    c.customer_name,
    o.order_id
FROM customers c
LEFT JOIN orders o
    ON c.customer_id = o.customer_id;
```

Customers without orders may have `order_id = NULL`. You can identify them with:

```sql
WHERE o.order_id IS NULL;
```

> This is a common technique for finding: «Customers who have never placed an order.»

---

## 📦 16. NULL in GROUP BY

`NULL` values can also appear **as a group**.

```sql
SELECT
    department,
    COUNT(*) AS employee_count
FROM employees
GROUP BY department;
```

If some employees have no department, the result can contain a group where `department = NULL`.

You can make it more readable:

```sql
SELECT
    COALESCE(department, 'Unassigned') AS department,
    COUNT(*) AS employee_count
FROM employees
GROUP BY COALESCE(department, 'Unassigned');
```

---

## ↕️ 17. NULL and ORDER BY

`NULL` sorting behavior can **differ between database systems**.

```sql
SELECT *
FROM employees
ORDER BY salary DESC;
```

Depending on the database, `NULL` values may appear at the beginning or end.

Some systems support:

```sql
ORDER BY salary DESC NULLS LAST;
```

> ⚠️ Always check the SQL dialect you're using when `NULL` ordering matters.

---

## 🧠 18. NULL vs Empty String

These are **not** necessarily the same: `NULL` and `''`

| Value | Meaning |
| ----- | ------- |
| `NULL` | «No known value.» |
| `''` | «A string exists but contains no characters.» |

For example, `phone = ''` is different from `phone IS NULL`.

> This distinction matters during data cleaning.

---

## 🏢 19. Real-World Analytics Example

Imagine an e-commerce dataset:

| order_id | revenue | discount | shipping_cost |
| -------- | ------- | -------- | ------------- |
| 1        | 2000    | 200      | 100           |
| 2        | 1500    | `NULL`   | 80            |
| 3        | 3000    | 300      | `NULL`        |

Calculate profit safely:

```sql
SELECT
    order_id,
    revenue,
    COALESCE(discount, 0)      AS discount,
    COALESCE(shipping_cost, 0) AS shipping_cost,
    revenue
        - COALESCE(discount, 0)
        - COALESCE(shipping_cost, 0) AS net_revenue
FROM orders;
```

> This prevents missing values from turning the entire calculation into `NULL`.

---

## 🎯 20. Business KPI Example — Conversion Rate

Suppose `conversions = 50` and `visitors = 0`. A safe calculation is:

```sql
SELECT
    COALESCE(
        conversions * 100.0 / NULLIF(visitors, 0),
        0
    ) AS conversion_rate
FROM marketing;
```

The logic is:

```text
NULLIF(visitors, 0)
        ↓
Prevents division by zero
        ↓
Returns NULL if visitors = 0
        ↓
COALESCE(..., 0)
        ↓
Displays 0 instead of NULL
```

> This pattern is highly useful for KPI dashboards.

---

## ⚠️ Common NULL Mistakes

**❌ Mistake 1: Using `= NULL`**

```sql
WHERE salary = NULL;   -- ❌ Incorrect
WHERE salary IS NULL;  -- ✅ Use this
```

**❌ Mistake 2: Assuming NULL means zero**

`NULL ≠ 0`

**❌ Mistake 3: Ignoring NULL during calculations**

`price - discount` may produce `NULL` when `discount` is `NULL`. Consider `price - COALESCE(discount, 0)` when treating a missing discount as zero is appropriate.

**❌ Mistake 4: Using COALESCE blindly**

Replacing every `NULL` with `0` can **distort analysis**. For example, *missing salary → 0* does not mean the employee earns zero.

> The correct replacement depends on the **business meaning** of the missing value.

---

## 🎤 SQL Interview Questions

**Q1. What is NULL?**

`NULL` represents a missing, unknown, or unavailable value.

**Q2. How do you check for NULL?**

```sql
WHERE column_name IS NULL;
```

**Q3. How do you check for non-NULL values?**

```sql
WHERE column_name IS NOT NULL;
```

**Q4. Why doesn't `= NULL` work?**

Because `NULL` represents an unknown value and comparisons with `NULL` do not evaluate to `TRUE` in the normal way. SQL provides `IS NULL` and `IS NOT NULL` specifically for this purpose.

**Q5. What does `COALESCE()` do?**

It returns the first non-`NULL` expression — e.g. `COALESCE(phone, email, 'No Contact')`

**Q6. What does `NULLIF()` do?**

It returns `NULL` when two expressions are equal — `NULLIF(value1, value2)`

**Q7. Difference between `COUNT(*)` and `COUNT(column)`?**

`COUNT(*)` counts rows; `COUNT(column)` counts non-`NULL` values in that column.

**Q8. How can you prevent division by zero?**

```sql
revenue / NULLIF(orders, 0)
```

**Q9. Does `AVG()` normally include NULL values?**

No. `NULL` values are generally ignored when calculating the average.

**Q10. What is the difference between NULL and 0?**

`0` is an actual numeric value; `NULL` represents an unknown or missing value.

---

## 📝 Practice Questions

**Practice 1:** Find customers whose email is missing.

```sql
SELECT *
FROM customers
WHERE email IS NULL;
```

**Practice 2:** Display `'Unknown'` when a customer's city is `NULL`.

```sql
SELECT
    customer_name,
    COALESCE(city, 'Unknown') AS city
FROM customers;
```

**Practice 3:** Calculate final price assuming a missing discount means zero.

```sql
SELECT
    price - COALESCE(discount, 0) AS final_price
FROM orders;
```

**Practice 4:** Calculate revenue per order without dividing by zero.

```sql
SELECT
    revenue / NULLIF(order_count, 0) AS revenue_per_order
FROM sales;
```

**Practice 5:** Count how many customers have a phone number.

```sql
SELECT COUNT(phone) AS customers_with_phone
FROM customers;
```

---

## 🧪 Mini SQL Challenge

You have a table `sales` with columns: `sale_id`, `revenue`, `discount`, `orders`

Write a query that returns:

- `sale_id`
- revenue
- discount, treating `NULL` as `0`
- revenue after discount
- revenue per order
- safely handle `orders = 0`

**Solution:**

```sql
SELECT
    sale_id,
    revenue,
    COALESCE(discount, 0) AS discount,

    revenue - COALESCE(discount, 0)
        AS revenue_after_discount,

    COALESCE(
        revenue / NULLIF(orders, 0),
        0
    ) AS revenue_per_order

FROM sales;
```

---

💡 **Double Tap ❤️ For Part-9**
