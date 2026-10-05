# 🚀 SQL Roadmap 2026 — Part 14

## 🪟 SQL Window Functions — Advanced Analytics Without Losing Rows

Window Functions are one of the most powerful features in SQL.

They allow you to perform calculations **across related rows without collapsing the rows** into a single result.

This is the key difference between:

| Feature             | What it does                                  |
| ------------------- | --------------------------------------------- |
| **GROUP BY**        | Combines rows                                 |
| **Window Function** | Keeps the rows and calculates across them     |

Window Functions are heavily used for:

- Rankings
- Running totals
- Moving averages
- Previous/next row comparisons
- Customer analysis
- Sales analysis
- Time-series analysis
- Percentage calculations
- Top-N analysis

---

## 🧠 1. The Problem Window Functions Solve

Suppose we have:

| order_id | customer_id | amount |
| -------- | ----------- | ------ |
| 1        | 101         | 500    |
| 2        | 101         | 800    |
| 3        | 102         | 300    |
| 4        | 102         | 700    |

If we use:

```sql
SELECT
    customer_id,
    SUM(amount) AS total_spending
FROM orders
GROUP BY customer_id;
```

we get:

| customer_id | total_spending |
| ----------- | -------------- |
| 101         | 1300           |
| 102         | 1000           |

**The individual orders disappear.**

But what if we want:

| order_id | customer_id | amount | customer_total |
| -------- | ----------- | ------ | -------------- |
| 1        | 101         | 500    | 1300           |
| 2        | 101         | 800    | 1300           |
| 3        | 102         | 300    | 1000           |
| 4        | 102         | 700    | 1000           |

This is where a **Window Function** is useful.

---

## 🪟 2. Basic Window Function Syntax

The general structure is:

```sql
function_name(...) OVER (
    PARTITION BY ...
    ORDER BY ...
)
```

**Example:**

```sql
SELECT
    order_id,
    customer_id,
    amount,
    SUM(amount) OVER (
        PARTITION BY customer_id
    ) AS customer_total
FROM orders;
```

The result keeps **every order** while calculating the customer's total.

---

## 🔹 3. What Does OVER() Mean?

`OVER()` tells SQL:

> «Perform this calculation across a **window** of rows.»

For example:

```sql
SUM(amount) OVER ()
```

means:

> «Calculate the sum across **all rows** while keeping every row.»

**Example:**

```sql
SELECT
    order_id,
    amount,
    SUM(amount) OVER () AS total_revenue
FROM orders;
```

Every row will show the overall revenue.

---

## 📊 4. PARTITION BY

`PARTITION BY` divides the data into logical groups.

```sql
SUM(amount) OVER (
    PARTITION BY customer_id
)
```

means:

> «Calculate the sum **separately for each customer**.»

```text
All Orders
    ↓
Partition by customer
    ↓
Customer 101 → calculate separately
Customer 102 → calculate separately
Customer 103 → calculate separately
```

Unlike `GROUP BY`, the original rows remain visible.

---

## 🆚 5. GROUP BY vs Window Function

**GROUP BY:**

```sql
SELECT
    customer_id,
    SUM(amount) AS total_spending
FROM orders
GROUP BY customer_id;
```

Result: **1 row per customer**

**Window Function:**

```sql
SELECT
    order_id,
    customer_id,
    amount,
    SUM(amount) OVER (
        PARTITION BY customer_id
    ) AS total_spending
FROM orders;
```

Result: **1 row per order**

Remember:

- `GROUP BY` **reduces** rows.
- Window Functions **preserve** rows.

---

## 🏆 6. RANK()

`RANK()` assigns a ranking to rows.

```sql
SELECT
    employee_name,
    salary,
    RANK() OVER (
        ORDER BY salary DESC
    ) AS salary_rank
FROM employees;
```

| salary | salary_rank |
| ------ | ----------- |
| 90000  | 1           |
| 80000  | 2           |
| 70000  | 3           |

---

## 🥇 7. Ranking with Ties

Suppose salaries are `90000, 80000, 80000, 70000`.

Using `RANK()`:

```text
90000 → 1
80000 → 2
80000 → 2
70000 → 4
```

Notice that **rank 3 is skipped**. That's how `RANK()` handles ties.

---

## 🔢 8. DENSE_RANK()

`DENSE_RANK()` also handles ties but **does not skip** the next rank.

```text
90000 → 1
80000 → 2
80000 → 2
70000 → 3
```

**Difference:**

```text
RANK()       → 1, 2, 2, 4
DENSE_RANK() → 1, 2, 2, 3
```

---

## 🔢 9. ROW_NUMBER()

`ROW_NUMBER()` assigns a **unique sequential number** to every row.

```sql
SELECT
    employee_name,
    salary,
    ROW_NUMBER() OVER (
        ORDER BY salary DESC
    ) AS row_num
FROM employees;
```

Even if two employees have the same salary, they receive **different** row numbers:

```text
90000 → 1
80000 → 2
80000 → 3
70000 → 4
```

⚠️ Which of the two tied rows gets `2` vs `3` is **not guaranteed**. Add a tie-breaker (e.g., `ORDER BY salary DESC, employee_id`) if you need deterministic results.

---

## 🆚 10. RANK vs DENSE_RANK vs ROW_NUMBER

| Function       | Ties                | Gaps after ties | Example (90k, 80k, 80k, 70k) |
| -------------- | ------------------- | --------------- | ---------------------------- |
| `ROW_NUMBER()` | Unique number each  | —               | 1, 2, 3, 4                   |
| `RANK()`       | Share rank          | ✅ Yes          | 1, 2, 2, 4                   |
| `DENSE_RANK()` | Share rank          | ❌ No           | 1, 2, 2, 3                   |

---

## 🏆 11. Top Customer per Region

Suppose we want the highest-spending customer in each region.

First calculate spending, then rank:

```sql
WITH customer_sales AS (
    SELECT
        customer_id,
        region,
        SUM(amount) AS total_spending
    FROM orders
    GROUP BY
        customer_id,
        region
)
SELECT
    customer_id,
    region,
    total_spending,
    RANK() OVER (
        PARTITION BY region
        ORDER BY total_spending DESC
    ) AS regional_rank
FROM customer_sales;
```

The important part is:

```sql
PARTITION BY region
```

This **restarts the ranking** for each region.

---

## 🎯 12. Top 1 Per Group

If we want **only** the top customer from each region:

```sql
WITH customer_sales AS (
    SELECT
        customer_id,
        region,
        SUM(amount) AS total_spending
    FROM orders
    GROUP BY
        customer_id,
        region
),
ranked_customers AS (
    SELECT
        customer_id,
        region,
        total_spending,
        ROW_NUMBER() OVER (
            PARTITION BY region
            ORDER BY total_spending DESC
        ) AS rn
    FROM customer_sales
)
SELECT
    customer_id,
    region,
    total_spending
FROM ranked_customers
WHERE rn = 1;
```

This is one of the **most frequently used** Window Function patterns in SQL interviews.

💡 **Why the extra CTE?** Window functions are evaluated *after* `WHERE`, so you **cannot** write `WHERE ROW_NUMBER() OVER (...) = 1` directly. Compute the rank first, then filter it in an outer query / CTE.

💡 **ROW_NUMBER vs RANK here:** `ROW_NUMBER()` returns exactly one customer per region. Use `RANK()` if tied customers should *all* be returned.

---

## ➕ 13. Running Total

Suppose we have:

| order_date | amount |
| ---------- | ------ |
| Jan 1      | 100    |
| Jan 2      | 200    |
| Jan 3      | 150    |
| Jan 4      | 300    |

We want:

| order_date | amount | running_total |
| ---------- | ------ | ------------- |
| Jan 1      | 100    | 100           |
| Jan 2      | 200    | 300           |
| Jan 3      | 150    | 450           |
| Jan 4      | 300    | 750           |

Use:

```sql
SELECT
    order_date,
    amount,
    SUM(amount) OVER (
        ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING
                 AND CURRENT ROW
    ) AS running_total
FROM orders;
```

The calculation builds cumulatively:

```text
100
100 + 200
100 + 200 + 150
100 + 200 + 150 + 300
```

**Understanding the frame:**

| Frame clause            | Meaning                          |
| ----------------------- | -------------------------------- |
| `UNBOUNDED PRECEDING`   | From the first row of the window |
| `N PRECEDING`           | N rows before the current row    |
| `CURRENT ROW`           | The current row                  |
| `N FOLLOWING`           | N rows after the current row     |
| `UNBOUNDED FOLLOWING`   | To the last row of the window    |

Why write `ROWS BETWEEN ...` explicitly? With only `ORDER BY`, the default frame is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`, which treats rows with the **same `order_date`** as one group — they'd all show the same running total. `ROWS` counts row by row.

---

## 📅 14. Running Total by Customer

We can combine `PARTITION BY` and `ORDER BY`.

```sql
SELECT
    customer_id,
    order_date,
    amount,
    SUM(amount) OVER (
        PARTITION BY customer_id
        ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING
                 AND CURRENT ROW
    ) AS running_total
FROM orders;
```

Now each customer's running total is calculated **independently** — it resets to zero at the start of each customer.

---

## ⏮️ 15. LAG()

`LAG()` lets you access a value from a **previous row**.

```sql
SELECT
    order_date,
    amount,
    LAG(amount) OVER (
        ORDER BY order_date
    ) AS previous_amount
FROM orders;
```

| order_date | amount | previous_amount |
| ---------- | ------ | --------------- |
| Jan 1      | 100    | NULL            |
| Jan 2      | 200    | 100             |
| Jan 3      | 150    | 200             |
| Jan 4      | 300    | 150             |

The first row has no previous row, so it returns `NULL`.

Full signature: `LAG(column, offset, default)` — e.g. `LAG(amount, 1, 0)` returns `0` instead of `NULL`.

This is extremely useful for:

- Month-over-month analysis
- Previous transaction comparison
- Trend analysis
- Customer behavior

---

## ⏭️ 16. LEAD()

`LEAD()` does the opposite — it looks at the **next row**.

```sql
SELECT
    order_date,
    amount,
    LEAD(amount) OVER (
        ORDER BY order_date
    ) AS next_amount
FROM orders;
```

| order_date | amount | next_amount |
| ---------- | ------ | ----------- |
| Jan 1      | 100    | 200         |
| Jan 2      | 200    | 150         |
| Jan 3      | 150    | 300         |
| Jan 4      | 300    | NULL        |

Easy memory:

```text
LAG  → Look backward
LEAD → Look forward
```

---

## 📈 17. Calculate Change from Previous Row

`LAG()` becomes more useful when combined with arithmetic.

```sql
SELECT
    order_date,
    amount,
    amount - LAG(amount) OVER (
        ORDER BY order_date
    ) AS change_from_previous
FROM orders;
```

If previous = `100` and current = `200`, then `200 - 100 = 100`.

---

## 📊 18. Percentage Change

```sql
SELECT
    order_date,
    amount,
    (
        amount - LAG(amount) OVER (ORDER BY order_date)
    ) * 100.0
    / NULLIF(
        LAG(amount) OVER (ORDER BY order_date),
        0
    ) AS percentage_change
FROM orders;
```

Here we use:

- `LAG()` → previous value
- `* 100.0` → converts to a percentage (and avoids integer division)
- `NULLIF()` → prevents division by zero

This combines concepts from earlier parts.

💡 Repeating `LAG(...)` twice is noisy. A cleaner way is to compute it once in a CTE, then do the math — you'll see this pattern in Part 15.

---

## 📐 19. Moving Average

Suppose we want a **3-row moving average**.

```sql
SELECT
    order_date,
    amount,
    AVG(amount) OVER (
        ORDER BY order_date
        ROWS BETWEEN 2 PRECEDING
                 AND CURRENT ROW
    ) AS moving_average
FROM orders;
```

For each row, SQL considers:

```text
Current row + Previous row + 2nd previous row
```

| order_date | amount | moving_average        |
| ---------- | ------ | --------------------- |
| Jan 1      | 100    | 100.00 (only 1 row)   |
| Jan 2      | 200    | 150.00 (2 rows)       |
| Jan 3      | 150    | 150.00                |
| Jan 4      | 300    | 216.67                |

This is useful for **smoothing short-term fluctuations** in time-series data.

---

## 🧮 20. Window Functions for Percentage of Total

Each product's percentage of total revenue:

```sql
SELECT
    product_id,
    revenue,
    revenue * 100.0
    / SUM(revenue) OVER () AS percentage_of_total
FROM product_sales;
```

Example:

```text
Product A → 20%
Product B → 30%
Product C → 50%
```

The key advantage: **we don't need a separate query to calculate total revenue.**

---

## 📊 21. Percentage Within a Group

Each customer's percentage of **their region's** revenue:

```sql
SELECT
    customer_id,
    region,
    revenue,
    revenue * 100.0
    / SUM(revenue) OVER (
        PARTITION BY region
    ) AS region_percentage
FROM customer_sales;
```

The denominator is calculated **separately for each region**.

---

## 🧠 22. Window Functions with CASE

You can combine Window Functions with `CASE`.

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
        WHEN RANK() OVER (
            ORDER BY total_spending DESC
        ) <= 10
        THEN 'Top 10'
        ELSE 'Other'
    END AS customer_group
FROM customer_sales;
```

This allows analytical results to be converted into **business categories**.

---

## ⚠️ 23. Window Functions Don't Replace GROUP BY

Window Functions and `GROUP BY` solve **different problems**.

**GROUP BY** — use when you want **1 row per group**:

```sql
SELECT
    region,
    SUM(revenue) AS total_revenue
FROM sales
GROUP BY region;
```

**Window Function** — use when you want **original rows + group-level calculation**:

```sql
SELECT
    customer_id,
    region,
    revenue,
    SUM(revenue) OVER (
        PARTITION BY region
    ) AS regional_revenue
FROM sales;
```

---

## ⚠️ 24. Window Function Order Matters

```sql
RANK() OVER (ORDER BY revenue DESC)
```

and

```sql
RANK() OVER (ORDER BY revenue ASC)
```

produce **completely different** rankings.

Always ask:

> «What should rank 1 represent?»

- Highest value? → `DESC`
- Lowest value? → `ASC`

---

## 🔍 25. Multiple Window Functions

You can use several Window Functions in one query.

```sql
SELECT
    customer_id,
    order_date,
    amount,

    ROW_NUMBER() OVER (
        PARTITION BY customer_id
        ORDER BY order_date
    ) AS order_number,

    SUM(amount) OVER (
        PARTITION BY customer_id
        ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING
                 AND CURRENT ROW
    ) AS running_spending,

    LAG(amount) OVER (
        PARTITION BY customer_id
        ORDER BY order_date
    ) AS previous_order_amount

FROM orders;
```

Now we have:

```text
Order number + Running spending + Previous order amount
```

all while **preserving the order-level rows**.

💡 PostgreSQL and MySQL 8+ support a named `WINDOW` clause to avoid repeating the same `OVER (...)`:

```sql
SELECT
    customer_id,
    order_date,
    ROW_NUMBER() OVER w AS order_number,
    LAG(amount)  OVER w AS previous_order_amount
FROM orders
WINDOW w AS (PARTITION BY customer_id ORDER BY order_date);
```

---

## 🏢 26. Real-World Business Example

A company wants to analyze customer transactions:

- Transaction date
- Transaction amount
- Previous transaction
- Difference from previous transaction
- Running customer spending

```sql
SELECT
    customer_id,
    transaction_date,
    amount,

    LAG(amount) OVER (
        PARTITION BY customer_id
        ORDER BY transaction_date
    ) AS previous_amount,

    amount - LAG(amount) OVER (
        PARTITION BY customer_id
        ORDER BY transaction_date
    ) AS amount_change,

    SUM(amount) OVER (
        PARTITION BY customer_id
        ORDER BY transaction_date
        ROWS BETWEEN UNBOUNDED PRECEDING
                 AND CURRENT ROW
    ) AS running_spending

FROM transactions;
```

This is a **realistic analytical SQL pattern**.

---

## 🎤 SQL Interview Questions

**Q1. What is a Window Function?**

A function that performs calculations across a set of related rows while retaining the individual rows in the result.

**Q2. What does `OVER()` do?**

It defines the window of rows over which the function operates.

**Q3. What does `PARTITION BY` do?**

It divides rows into logical groups for the Window Function.

**Q4. Difference between `GROUP BY` and Window Functions?**

`GROUP BY` collapses rows into groups. Window Functions calculate across rows without collapsing them.

**Q5. Difference between `RANK()` and `DENSE_RANK()`?**

`RANK()` leaves gaps after ties; `DENSE_RANK()` does not.

```text
RANK       → 1, 2, 2, 4
DENSE_RANK → 1, 2, 2, 3
```

**Q6. Difference between `ROW_NUMBER()` and `RANK()`?**

`ROW_NUMBER()` gives every row a unique number. `RANK()` gives tied rows the same rank.

**Q7. What does `LAG()` do?**

Returns a value from a previous row.

**Q8. What does `LEAD()` do?**

Returns a value from a subsequent row.

**Q9. How do you calculate a running total?**

```sql
SUM(amount) OVER (
    ORDER BY date_column
    ROWS BETWEEN UNBOUNDED PRECEDING
             AND CURRENT ROW
)
```

**Q10. How do you find the top customer in each region?**

Use a ranking Window Function with:

```sql
PARTITION BY region
ORDER BY total_spending DESC
```

Then filter the ranking in an outer query or CTE (window functions can't be used directly in `WHERE`).

---

## 📝 Practice Questions

**Practice 1 — Rank employees by salary.**

```sql
SELECT
    employee_name,
    salary,
    RANK() OVER (
        ORDER BY salary DESC
    ) AS salary_rank
FROM employees;
```

**Practice 2 — Calculate total spending for each customer while keeping every order.**

```sql
SELECT
    order_id,
    customer_id,
    amount,
    SUM(amount) OVER (
        PARTITION BY customer_id
    ) AS customer_total
FROM orders;
```

**Practice 3 — Find the previous order amount for every customer.**

```sql
SELECT
    customer_id,
    order_date,
    amount,
    LAG(amount) OVER (
        PARTITION BY customer_id
        ORDER BY order_date
    ) AS previous_amount
FROM orders;
```

**Practice 4 — Calculate a running total for each customer.**

```sql
SELECT
    customer_id,
    order_date,
    amount,
    SUM(amount) OVER (
        PARTITION BY customer_id
        ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING
                 AND CURRENT ROW
    ) AS running_total
FROM orders;
```

**Practice 5 — Assign a unique order number to each customer's orders.**

```sql
SELECT
    customer_id,
    order_id,
    order_date,
    ROW_NUMBER() OVER (
        PARTITION BY customer_id
        ORDER BY order_date
    ) AS order_number
FROM orders;
```

---

## 🧪 Mini SQL Challenge

You have:

```text
sales (sale_id, customer_id, sale_date, amount)
```

Write a query that returns:

- Customer ID
- Sale date
- Amount
- Previous sale amount
- Difference from previous sale
- Running customer spending
- Customer's transaction number

**Solution:**

```sql
SELECT
    customer_id,
    sale_date,
    amount,

    LAG(amount) OVER (
        PARTITION BY customer_id
        ORDER BY sale_date
    ) AS previous_amount,

    amount - LAG(amount) OVER (
        PARTITION BY customer_id
        ORDER BY sale_date
    ) AS difference_from_previous,

    SUM(amount) OVER (
        PARTITION BY customer_id
        ORDER BY sale_date
        ROWS BETWEEN UNBOUNDED PRECEDING
                 AND CURRENT ROW
    ) AS running_spending,

    ROW_NUMBER() OVER (
        PARTITION BY customer_id
        ORDER BY sale_date
    ) AS transaction_number

FROM sales;
```

**What did we use?**

| Tool            | Purpose                                   |
| --------------- | ----------------------------------------- |
| `LAG()`         | Previous transaction                      |
| `SUM() OVER()`  | Running spending                          |
| `ROW_NUMBER()`  | Transaction sequence                      |
| `PARTITION BY`  | Separate calculations for each customer   |
| `ORDER BY`      | Establish chronological order             |

---

💡 **Double Tap ❤️ For More**
