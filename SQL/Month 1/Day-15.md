# 🚀 SQL Roadmap 2026 — Part 15

## 📅 SQL Date & Time Functions — Working with Dates, Time & Time-Based Analytics

Almost every real-world analytics project involves dates.

Examples:

- When was the order placed?
- How many days did delivery take?
- What was the revenue in January?
- Which month had the highest sales?
- How many customers purchased this year?
- How long has an account been inactive?
- What is the month-over-month growth?
- Which transactions breached the SLA?

SQL provides powerful date and time functions to answer these questions.

⚠️ **Dialect note:** Date functions vary more between databases than almost any other SQL feature. The examples below use **PostgreSQL** syntax unless stated otherwise; see the cheat sheet in section 28 for MySQL and SQL Server equivalents.

---

## 🧠 1. Common Date & Time Data Types

Different databases support different date/time types, but commonly you'll encounter:

| Type        | Stores           | Example               |
| ----------- | ---------------- | --------------------- |
| `DATE`      | Date only        | `2026-09-18`          |
| `TIME`      | Time only        | `14:30:25`            |
| `TIMESTAMP` | Date + time      | `2026-09-18 14:30:25` |
| `DATETIME`  | Date + time (MySQL / SQL Server) | `2026-09-18 14:30:25` |

The exact data types and behavior vary by database.

---

## 📅 2. Current Date

```sql
SELECT CURRENT_DATE;
```

Result: `2026-09-18`

The exact function can vary by SQL dialect (e.g. SQL Server uses `CAST(GETDATE() AS DATE)`).

---

## ⏰ 3. Current Date and Time

```sql
SELECT CURRENT_TIMESTAMP;
```

Returns the current date and time, e.g. `2026-09-18 21:39:00`.

Exact formatting depends on the database.

---

## 🔍 4. Extracting Year, Month & Day

Suppose `order_date = 2026-09-18`. We may want:

```text
Year  → 2026
Month → 9
Day   → 18
```

A common (ANSI standard) SQL approach:

```sql
SELECT
    EXTRACT(YEAR  FROM order_date) AS year,
    EXTRACT(MONTH FROM order_date) AS month,
    EXTRACT(DAY   FROM order_date) AS day
FROM orders;
```

Some databases (MySQL, SQL Server) use `YEAR(order_date)`, `MONTH(order_date)`, `DAY(order_date)`.

So always check the syntax for your SQL dialect.

---

## 📊 5. Revenue by Year

```sql
SELECT
    EXTRACT(YEAR FROM order_date) AS order_year,
    SUM(amount) AS revenue
FROM orders
GROUP BY EXTRACT(YEAR FROM order_date)
ORDER BY order_year;
```

| order_year | revenue |
| ---------- | ------- |
| 2024       | 850000  |
| 2025       | 1120000 |
| 2026       | 1380000 |

This lets us analyze **year-over-year** business performance.

---

## 📆 6. Revenue by Month

```sql
SELECT
    EXTRACT(YEAR  FROM order_date) AS year,
    EXTRACT(MONTH FROM order_date) AS month,
    SUM(amount) AS revenue
FROM orders
GROUP BY
    EXTRACT(YEAR  FROM order_date),
    EXTRACT(MONTH FROM order_date)
ORDER BY
    year,
    month;
```

⚠️ **Important:** Don't group **only** by month number if your data spans multiple years.

January 2025 and January 2026 both have `month = 1`. Grouping only by month would incorrectly **combine them**.

---

## 🗓️ 7. Month-Level Grouping

Some databases provide functions that **truncate** a date to the beginning of a period.

PostgreSQL:

```sql
SELECT
    DATE_TRUNC('month', order_date) AS month,
    SUM(amount) AS revenue
FROM orders
GROUP BY DATE_TRUNC('month', order_date)
ORDER BY month;
```

This produces values such as `2026-01-01`, `2026-02-01` — one value per month, **year included**. This is often convenient for time-series analysis.

---

## 📈 8. Month-over-Month Analysis

Suppose monthly revenue is: Jan `100000`, Feb `120000`, Mar `110000`. We can use `LAG()` (from Part 14):

```sql
WITH monthly_sales AS (
    SELECT
        DATE_TRUNC('month', order_date) AS month,
        SUM(amount) AS revenue
    FROM orders
    GROUP BY DATE_TRUNC('month', order_date)
)
SELECT
    month,
    revenue,
    LAG(revenue) OVER (ORDER BY month) AS previous_month_revenue
FROM monthly_sales;
```

| month      | revenue | previous_month_revenue |
| ---------- | ------- | ---------------------- |
| 2026-01-01 | 100000  | NULL                   |
| 2026-02-01 | 120000  | 100000                 |
| 2026-03-01 | 110000  | 120000                 |

---

## 📊 9. Month-over-Month Growth %

```sql
WITH monthly_sales AS (
    SELECT
        DATE_TRUNC('month', order_date) AS month,
        SUM(amount) AS revenue
    FROM orders
    GROUP BY DATE_TRUNC('month', order_date)
),
monthly_comparison AS (
    SELECT
        month,
        revenue,
        LAG(revenue) OVER (ORDER BY month) AS previous_revenue
    FROM monthly_sales
)
SELECT
    month,
    revenue,
    previous_revenue,
    (revenue - previous_revenue) * 100.0
        / NULLIF(previous_revenue, 0) AS mom_growth
FROM monthly_comparison
ORDER BY month;
```

| month      | revenue | previous_revenue | mom_growth |
| ---------- | ------- | ---------------- | ---------- |
| 2026-01-01 | 100000  | NULL             | NULL       |
| 2026-02-01 | 120000  | 100000           | 20.00      |
| 2026-03-01 | 110000  | 120000           | -8.33      |

Computing `LAG()` once in a CTE keeps the final formula clean (compare with Part 14, section 18).

---

## ⏳ 10. Calculating Date Differences

> «How many days were there between two dates?»

**PostgreSQL** (subtracting two `DATE`s returns an integer number of days):

```sql
SELECT
    order_id,
    delivery_date - order_date AS delivery_days
FROM orders;
```

**SQL Server:**

```sql
SELECT
    order_id,
    DATEDIFF(day, order_date, delivery_date) AS delivery_days
FROM orders;
```

**MySQL** (note the reversed argument order — end date first):

```sql
SELECT
    order_id,
    DATEDIFF(delivery_date, order_date) AS delivery_days
FROM orders;
```

The exact syntax depends on the database.

---

## 🚚 11. Delivery Time Analysis

```sql
SELECT
    order_id,
    order_date,
    delivery_date,
    delivery_date - order_date AS delivery_duration
FROM orders;
```

This can help answer: **Which orders took longer than the target SLA?**

---

## 🎯 12. SLA Analysis

Suppose the target delivery time is **3 days**.

```sql
SELECT
    order_id,
    order_date,
    delivery_date,
    CASE
        WHEN delivery_date - order_date <= 3 THEN 'Within SLA'
        ELSE 'SLA Breach'
    END AS sla_status
FROM orders;
```

⚠️ If `delivery_date` is `NULL` (not yet delivered), the comparison is `UNKNOWN`, so the row falls to `ELSE` and is labelled `'SLA Breach'`. Add `WHEN delivery_date IS NULL THEN 'Pending'` first if that matters.

---

## 🧮 13. Calculating Ageing

Ageing is common in banking, finance, accounts payable, receivables, operations, and ticket management.

Days overdue:

```sql
SELECT
    invoice_id,
    due_date,
    CURRENT_DATE - due_date AS days_overdue
FROM invoices
WHERE due_date < CURRENT_DATE;
```

---

## 💰 14. Ageing Buckets

Combine date calculations with `CASE`:

```sql
SELECT
    invoice_id,
    due_date,
    CASE
        WHEN CURRENT_DATE - due_date <= 0  THEN 'Not Due'
        WHEN CURRENT_DATE - due_date <= 30 THEN '1-30 Days'
        WHEN CURRENT_DATE - due_date <= 60 THEN '31-60 Days'
        WHEN CURRENT_DATE - due_date <= 90 THEN '61-90 Days'
        ELSE '90+ Days'
    END AS ageing_bucket
FROM invoices;
```

`CASE` checks conditions **top to bottom**, so each bucket only needs an upper bound.

---

## 📆 15. Filtering by Date

Orders from 2026:

```sql
SELECT *
FROM orders
WHERE order_date >= '2026-01-01'
  AND order_date <  '2027-01-01';
```

This **half-open range** (`>=` start, `<` next period's start) is especially useful when `order_date` is a timestamp.

💡 Prefer this over `WHERE EXTRACT(YEAR FROM order_date) = 2026` — wrapping the column in a function usually prevents the database from using an index on it.

---

## ⚠️ 16. Why BETWEEN Can Be Risky with Timestamps

Suppose `order_timestamp` contains `2026-01-31 15:30:00`.

```sql
WHERE order_timestamp BETWEEN '2026-01-01' AND '2026-01-31'
```

`'2026-01-31'` is interpreted as `2026-01-31 00:00:00`, so anything **later on January 31 is excluded**.

A safer pattern:

```sql
WHERE order_timestamp >= '2026-01-01'
  AND order_timestamp <  '2026-02-01'
```

---

## 🕐 17. Extracting the Hour

Analyze activity by hour:

```sql
SELECT
    EXTRACT(HOUR FROM transaction_timestamp) AS transaction_hour,
    COUNT(*) AS transaction_count
FROM transactions
GROUP BY EXTRACT(HOUR FROM transaction_timestamp)
ORDER BY transaction_hour;
```

This can help identify:

- peak transaction times
- system load
- customer activity patterns

---

## 📅 18. Day-of-Week Analysis

PostgreSQL provides `EXTRACT(DOW FROM ...)`; MySQL provides `DAYOFWEEK()`.

```sql
SELECT
    EXTRACT(DOW FROM transaction_date) AS day_of_week,
    COUNT(*) AS transactions
FROM transactions
GROUP BY EXTRACT(DOW FROM transaction_date)
ORDER BY day_of_week;
```

⚠️ Numbering differs by database:

| Function                     | Sunday | Monday | Saturday |
| ---------------------------- | ------ | ------ | -------- |
| PostgreSQL `EXTRACT(DOW ..)` | 0      | 1      | 6        |
| PostgreSQL `EXTRACT(ISODOW ..)` | 7   | 1      | 6        |
| MySQL `DAYOFWEEK()`          | 1      | 2      | 7        |

---

## 🏪 19. Weekend vs Weekday Analysis

```sql
SELECT
    transaction_date,
    CASE
        WHEN EXTRACT(DOW FROM transaction_date) IN (0, 6) THEN 'Weekend'
        ELSE 'Weekday'
    END AS day_type
FROM transactions;
```

(`0` = Sunday, `6` = Saturday in PostgreSQL.)

---

## 📊 20. Business Example — Peak Sales Day

Which date generated the highest revenue?

```sql
WITH daily_sales AS (
    SELECT
        order_date,
        SUM(amount) AS daily_revenue
    FROM orders
    GROUP BY order_date
)
SELECT
    order_date,
    daily_revenue
FROM daily_sales
ORDER BY daily_revenue DESC
LIMIT 1;
```

If `order_date` is a timestamp, group by `DATE_TRUNC('day', order_date)` or `CAST(order_date AS DATE)` — otherwise every distinct time becomes its own "day".

---

## 🧠 21. Date Truncation vs Date Extraction

**Extraction** gets a component:

```sql
EXTRACT(YEAR FROM order_date)      -- 2026
```

**Truncation** rounds down to a period bucket:

```sql
DATE_TRUNC('month', order_date)    -- 2026-09-01
```

| Function     | Returns                 | Use for                         |
| ------------ | ----------------------- | ------------------------------- |
| `EXTRACT`    | A **number** (the part) | "All Januaries", hour-of-day    |
| `DATE_TRUNC` | A **date** (the period) | Time-series, monthly trends     |

---

## 🔄 22. Adding Intervals to Dates

PostgreSQL:

```sql
SELECT
    order_date,
    order_date + INTERVAL '7 days' AS follow_up_date
FROM orders;
```

MySQL: `DATE_ADD(order_date, INTERVAL 7 DAY)` · SQL Server: `DATEADD(day, 7, order_date)`

Useful for:

- follow-up dates
- SLA deadlines
- subscription periods
- reminder dates

---

## 📉 23. Finding Inactive Customers

First, find each customer's last order:

```sql
SELECT
    customer_id,
    MAX(order_date) AS last_order_date
FROM orders
GROUP BY customer_id;
```

Then filter:

```sql
WITH customer_activity AS (
    SELECT
        customer_id,
        MAX(order_date) AS last_order_date
    FROM orders
    GROUP BY customer_id
)
SELECT
    customer_id,
    last_order_date
FROM customer_activity
WHERE last_order_date < CURRENT_DATE - INTERVAL '90 days';
```

Note: this only covers customers who ordered **at least once**. Customers who never ordered won't appear in `orders` — use `NOT EXISTS` (Part 12) to find them.

---

## 📈 24. Year-over-Year Analysis

```sql
WITH monthly_sales AS (
    SELECT
        DATE_TRUNC('month', order_date) AS month,
        SUM(amount) AS revenue
    FROM orders
    GROUP BY DATE_TRUNC('month', order_date)
)
SELECT
    month,
    revenue,
    LAG(revenue, 12) OVER (ORDER BY month) AS previous_year_revenue
FROM monthly_sales;
```

Here `LAG(..., 12)` looks **12 rows** backward.

⚠️ If any months are missing (no sales that month), 12 rows back is **not** the same calendar month one year earlier. For robust YoY, explicitly align dates (e.g. self-join on `month - INTERVAL '1 year'`) or build a complete calendar (next section).

---

## 📅 25. Calendar Tables

A **calendar table** contains one row for each date in a period.

| date       | year | month | month_name | weekday  |
| ---------- | ---- | ----- | ---------- | -------- |
| 2026-01-01 | 2026 | 1     | January    | Thursday |
| 2026-01-02 | 2026 | 1     | January    | Friday   |

Calendar tables are extremely useful for:

- time-series analysis
- missing-date detection
- reporting
- fiscal calendars
- holidays
- business days
- YoY comparisons

PostgreSQL can generate one on the fly:

```sql
SELECT generate_series(
    DATE '2026-01-01',
    DATE '2026-12-31',
    INTERVAL '1 day'
)::date AS calendar_date;
```

`LEFT JOIN` your sales to this, and days with zero sales show up instead of silently disappearing.

---

## 🏦 26. Financial Year Analysis

Business calendars don't always follow January–December. In India, for example, the financial year runs **April → March**.

```sql
CASE
    WHEN EXTRACT(MONTH FROM order_date) >= 4
        THEN EXTRACT(YEAR FROM order_date) + 1
    ELSE EXTRACT(YEAR FROM order_date)
END AS fiscal_year
```

With this convention, `2026-05-10` → FY **2027** (FY 2026-27), and `2026-02-10` → FY **2026** (FY 2025-26).

---

## ⚠️ 27. Time Zones Matter

A timestamp may be stored in UTC while the business operates in IST (UTC+5:30).

```text
UTC   : 2026-09-18 23:30
India : 2026-09-19 05:00   ← a different date!
```

An order at 23:30 UTC counts toward **Sep 18** in a UTC report but **Sep 19** in an India report.

For global analytics, always understand:

- source timezone
- database timezone
- reporting timezone
- daylight-saving rules

PostgreSQL: `order_ts AT TIME ZONE 'Asia/Kolkata'`

---

## 🗂️ 28. Dialect Cheat Sheet

| Task              | PostgreSQL                       | MySQL                              | SQL Server                          |
| ----------------- | -------------------------------- | ---------------------------------- | ----------------------------------- |
| Current date      | `CURRENT_DATE`                   | `CURRENT_DATE` / `CURDATE()`       | `CAST(GETDATE() AS DATE)`           |
| Current timestamp | `CURRENT_TIMESTAMP` / `NOW()`    | `CURRENT_TIMESTAMP` / `NOW()`      | `CURRENT_TIMESTAMP` / `GETDATE()`   |
| Year              | `EXTRACT(YEAR FROM d)`           | `YEAR(d)`                          | `YEAR(d)` / `DATEPART(year, d)`     |
| Month bucket      | `DATE_TRUNC('month', d)`         | `DATE_FORMAT(d, '%Y-%m-01')`       | `DATETRUNC(month, d)` (2022+)       |
| Days between      | `end - start`                    | `DATEDIFF(end, start)`             | `DATEDIFF(day, start, end)`         |
| Add 7 days        | `d + INTERVAL '7 days'`          | `DATE_ADD(d, INTERVAL 7 DAY)`      | `DATEADD(day, 7, d)`                |

---

## 🎤 SQL Interview Questions

**Q1. What is the difference between `DATE` and `TIMESTAMP`?**

`DATE` generally stores a calendar date. `TIMESTAMP` generally stores both date and time.

**Q2. How do you extract the year from a date?**

A common approach is `EXTRACT(YEAR FROM date_column)`. Some databases also support `YEAR(date_column)`.

**Q3. How do you get the current date?**

`CURRENT_DATE` (varies slightly by dialect).

**Q4. What is `DATE_TRUNC()` used for?**

It truncates a date/time to a specified period such as year, month, day, or hour. Commonly used for time-based grouping.

**Q5. How can you calculate month-over-month growth?**

Aggregate by month → use `LAG()` to get the previous month → calculate the percentage difference (with `NULLIF` to avoid division by zero).

**Q6. Why can `BETWEEN` be problematic with timestamps?**

Because an upper bound such as `'2026-01-31'` means midnight at the *start* of that day, excluding the rest of the day. Use a half-open range instead.

**Q7. What is a calendar table?**

A table containing one row per date with attributes such as year, month, weekday, fiscal period, and holidays.

**Q8. How can you calculate customer inactivity?**

Find the customer's latest transaction date using `MAX(transaction_date)` and compare it with the current date.

**Q9. What is `LAG()` useful for in time-series analysis?**

It allows comparison with a previous row, such as the previous month's revenue.

**Q10. Why are time zones important in analytics?**

The same timestamp can represent different local dates and times depending on the timezone, which can shift records between days or months in reports.

---

## 📝 Practice Questions

**Practice 1 — Calculate annual revenue.**

```sql
SELECT
    EXTRACT(YEAR FROM order_date) AS year,
    SUM(amount) AS revenue
FROM orders
GROUP BY EXTRACT(YEAR FROM order_date)
ORDER BY year;
```

**Practice 2 — Calculate monthly revenue.**

```sql
SELECT
    DATE_TRUNC('month', order_date) AS month,
    SUM(amount) AS revenue
FROM orders
GROUP BY DATE_TRUNC('month', order_date)
ORDER BY month;
```

**Practice 3 — Find the last order date for every customer.**

```sql
SELECT
    customer_id,
    MAX(order_date) AS last_order_date
FROM orders
GROUP BY customer_id;
```

**Practice 4 — Find orders placed in 2026.**

```sql
SELECT *
FROM orders
WHERE order_date >= '2026-01-01'
  AND order_date <  '2027-01-01';
```

**Practice 5 — Calculate delivery duration.**

```sql
SELECT
    order_id,
    delivery_date - order_date AS delivery_duration
FROM orders;
```

---

## 🧪 Mini SQL Challenge

You have:

```text
orders (order_id, customer_id, order_date, delivery_date, amount)
```

Create a query that returns:

- Order ID
- Customer ID
- Order month
- Order amount
- Delivery duration
- SLA status (SLA = 3 days)
- Running customer revenue

**Solution:**

```sql
SELECT
    order_id,
    customer_id,
    DATE_TRUNC('month', order_date) AS order_month,
    amount,
    delivery_date - order_date AS delivery_duration,
    CASE
        WHEN delivery_date - order_date <= 3 THEN 'Within SLA'
        ELSE 'SLA Breach'
    END AS sla_status,
    SUM(amount) OVER (
        PARTITION BY customer_id
        ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING
                 AND CURRENT ROW
    ) AS running_customer_revenue
FROM orders;
```

**What did we use?**

| Tool                        | Purpose                              |
| --------------------------- | ------------------------------------ |
| `DATE_TRUNC('month', ...)`  | Order month bucket                   |
| Date subtraction            | Delivery duration                    |
| `CASE`                      | SLA classification                   |
| `SUM() OVER (...)`          | Running customer revenue (Part 14)   |

---

💡 **Double Tap ❤️ For More**
