# 🚀 SQL Roadmap 2026 — Part 7

## 🧠 CASE Statements — Adding Business Logic to SQL

In the previous parts, you learned how to:

- Retrieve data with `SELECT`
- Filter data with `WHERE`
- Sort results with `ORDER BY`
- Summarize data with aggregate functions
- Group data with `GROUP BY`

Now it's time to learn how to make SQL **think in business categories**.

For example:

- Is this customer **High Value**, **Medium Value**, or **Low Value**?
- Is this employee's salary **High**, **Medium**, or **Low**?
- Is this order **Small**, **Medium**, or **Large**?

That's what the `CASE` expression helps you do.

---

## 1️⃣ What is CASE?

`CASE` allows you to create **conditional logic** inside SQL. It's similar to:

```text
IF   condition
THEN result
ELSE result
```

**Basic Syntax:**

```sql
SELECT
    column_name,
    CASE
        WHEN condition THEN result
        WHEN condition THEN result
        ELSE result
    END AS new_column
FROM table_name;
```

---

## 2️⃣ Simple CASE Example

Suppose we have employee salaries and want to classify employees based on salary.

```sql
SELECT
    employee_name,
    salary,
    CASE
        WHEN salary >= 1000000 THEN 'High'
        WHEN salary >=  600000 THEN 'Medium'
        ELSE 'Low'
    END AS salary_category
FROM employees;
```

**Result:**

| employee_name | salary  | salary_category |
| ------------- | ------- | --------------- |
| Rahul         | 1200000 | High            |
| Priya         | 850000  | Medium          |
| Amit          | 500000  | Low             |

---

## 3️⃣ How CASE Works

SQL checks conditions **from top to bottom**.

```sql
CASE
    WHEN salary >= 1000000 THEN 'High'
    WHEN salary >=  600000 THEN 'Medium'
    ELSE 'Low'
END
```

SQL effectively asks:

```text
Is salary >= 1,000,000?  YES → High
NO → Is salary >= 600,000?  YES → Medium
NO → Low
```

> Once a matching `WHEN` condition is found, SQL returns that result.

---

## 4️⃣ Order of WHEN Conditions Matters

Consider this **problematic** version:

```sql
CASE
    WHEN salary >=  600000 THEN 'Medium'
    WHEN salary >= 1000000 THEN 'High'
    ELSE 'Low'
END
```

**Why?** Someone earning ₹12 lakh satisfies `salary >= 600000` first, so SQL labels them **Medium** instead of **High**.

✅ **Better:**

```sql
CASE
    WHEN salary >= 1000000 THEN 'High'
    WHEN salary >=  600000 THEN 'Medium'
    ELSE 'Low'
END
```

> **Rule:** Put more specific or higher-priority conditions **before** broader conditions.

---

## 5️⃣ CASE With Text Conditions

You can also classify based on text.

```sql
SELECT
    employee_name,
    department,
    CASE
        WHEN department = 'IT'      THEN 'Technology'
        WHEN department = 'Finance' THEN 'Corporate'
        WHEN department = 'HR'      THEN 'Corporate'
        ELSE 'Other'
    END AS department_group
FROM employees;
```

---

## 6️⃣ CASE With Multiple Conditions

You can use `AND` and `OR` inside `WHEN`.

```sql
SELECT
    customer_name,
    city,
    customer_segment,
    CASE
        WHEN customer_segment = 'Premium'
             AND city = 'Mumbai'
        THEN 'Premium Mumbai'

        WHEN customer_segment = 'Premium'
        THEN 'Other Premium'

        ELSE 'Standard'
    END AS customer_group
FROM customers;
```

---

## 7️⃣ CASE With IN

You can combine `CASE` with `IN`.

```sql
SELECT
    customer_name,
    city,
    CASE
        WHEN city IN ('Mumbai', 'Pune', 'Nashik')
            THEN 'Maharashtra'
        WHEN city IN ('Delhi', 'Noida', 'Gurgaon')
            THEN 'NCR'
        ELSE 'Other'
    END AS region
FROM customers;
```

> This is useful for creating **business regions**.

---

## 8️⃣ CASE With BETWEEN

```sql
SELECT
    employee_name,
    salary,
    CASE
        WHEN salary BETWEEN 0 AND 500000
            THEN 'Entry Level'

        WHEN salary BETWEEN 500001 AND 1000000
            THEN 'Mid Level'

        ELSE 'Senior Level'
    END AS salary_band
FROM employees;
```

However, for numeric ranges, **inequality conditions are often easier to maintain**:

```sql
CASE
    WHEN salary <  500000 THEN 'Entry Level'
    WHEN salary < 1000000 THEN 'Mid Level'
    ELSE 'Senior Level'
END
```

...because the conditions are evaluated from top to bottom.

---

## 9️⃣ CASE for Customer Segmentation

Customer segmentation is a common analytics use case. Suppose `spend` represents total customer spending.

```sql
SELECT
    customer_id,
    customer_name,
    total_spend,
    CASE
        WHEN total_spend >= 100000 THEN 'VIP'
        WHEN total_spend >=  50000 THEN 'High Value'
        WHEN total_spend >=  10000 THEN 'Medium Value'
        ELSE 'Low Value'
    END AS customer_segment
FROM customers;
```

> This transforms raw spending into a **business classification**.

---

## 🔟 CASE for Order Size

```sql
SELECT
    order_id,
    amount,
    CASE
        WHEN amount >= 10000 THEN 'Large'
        WHEN amount >=  5000 THEN 'Medium'
        ELSE 'Small'
    END AS order_size
FROM orders;
```

**Result:**

| order_id | amount | order_size |
| -------- | ------ | ---------- |
| 101      | 12000  | Large      |
| 102      | 7000   | Medium     |
| 103      | 2500   | Small      |

---

## 1️⃣1️⃣ CASE for Order Status

You can simplify several statuses into broader business categories.

```sql
SELECT
    order_id,
    order_status,
    CASE
        WHEN order_status = 'Completed'
            THEN 'Successful'

        WHEN order_status = 'Cancelled'
            THEN 'Unsuccessful'

        ELSE 'Pending'
    END AS business_status
FROM orders;
```

---

## 1️⃣2️⃣ CASE With Dates

You can classify orders based on when they were placed.

```sql
SELECT
    order_id,
    order_date,
    CASE
        WHEN order_date < '2026-01-01'
            THEN 'Previous Year'
        ELSE 'Current Year'
    END AS order_period
FROM orders;
```

---

## 1️⃣3️⃣ CASE for Profitability

Suppose you have `selling_price` and `cost_price`. You can classify products based on profit.

```sql
SELECT
    product_name,
    selling_price,
    cost_price,
    selling_price - cost_price AS profit,

    CASE
        WHEN selling_price - cost_price >= 10000
            THEN 'Highly Profitable'

        WHEN selling_price - cost_price > 0
            THEN 'Profitable'

        ELSE 'Loss'
    END AS profitability
FROM products;
```

---

## 1️⃣4️⃣ CASE With Aggregate Functions

> This is where `CASE` becomes extremely powerful.

Suppose you want to count completed orders.

```sql
SELECT
    SUM(
        CASE
            WHEN order_status = 'Completed'
            THEN 1
            ELSE 0
        END
    ) AS completed_orders
FROM orders;
```

**Why does this work?** Each row becomes:

```text
Completed → 1
Other     → 0
```

Then `SUM()` adds them.

---

## 1️⃣5️⃣ Conditional Counting

You can calculate several metrics at once.

```sql
SELECT
    COUNT(*) AS total_orders,

    SUM(
        CASE
            WHEN order_status = 'Completed'
            THEN 1 ELSE 0
        END
    ) AS completed_orders,

    SUM(
        CASE
            WHEN order_status = 'Cancelled'
            THEN 1 ELSE 0
        END
    ) AS cancelled_orders
FROM orders;
```

This is called **conditional aggregation**. It is one of the most useful SQL techniques for dashboard development.

---

## 1️⃣6️⃣ Calculate Success Rate

You can combine `CASE`, `SUM`, and `COUNT`.

```sql
SELECT
    ROUND(
        100.0 *
        SUM(
            CASE
                WHEN order_status = 'Completed'
                THEN 1
                ELSE 0
            END
        ) / NULLIF(COUNT(*), 0),
        2
    ) AS completion_rate
FROM orders;
```

**The logic:** Completed orders ÷ Total orders × 100

---

## 1️⃣7️⃣ Conditional Revenue

Suppose you want only revenue from completed orders.

```sql
SELECT
    SUM(
        CASE
            WHEN order_status = 'Completed'
            THEN amount
            ELSE 0
        END
    ) AS completed_revenue
FROM orders;
```

> This is especially useful when you need several conditional metrics in one query.

---

## 1️⃣8️⃣ Multiple Conditional Metrics

You can build an entire KPI summary:

```sql
SELECT
    COUNT(*) AS total_orders,

    SUM(
        CASE
            WHEN order_status = 'Completed'
            THEN 1 ELSE 0
        END
    ) AS completed_orders,

    SUM(
        CASE
            WHEN order_status = 'Cancelled'
            THEN 1 ELSE 0
        END
    ) AS cancelled_orders,

    SUM(
        CASE
            WHEN order_status = 'Completed'
            THEN amount ELSE 0
        END
    ) AS completed_revenue,

    SUM(
        CASE
            WHEN order_status = 'Cancelled'
            THEN amount ELSE 0
        END
    ) AS cancelled_value
FROM orders;
```

> This is very close to the kind of SQL used behind BI dashboards.

---

## 1️⃣9️⃣ CASE With GROUP BY

You can create categories and then aggregate them.

```sql
SELECT
    CASE
        WHEN amount >= 10000 THEN 'Large'
        WHEN amount >=  5000 THEN 'Medium'
        ELSE 'Small'
    END AS order_size,
    COUNT(*) AS order_count
FROM orders
GROUP BY
    CASE
        WHEN amount >= 10000 THEN 'Large'
        WHEN amount >=  5000 THEN 'Medium'
        ELSE 'Small'
    END;
```

**Result:**

| order_size | order_count |
| ---------- | ----------- |
| Large      | 120         |
| Medium     | 450         |
| Small      | 980         |

---

## 2️⃣0️⃣ CASE + GROUP BY + SUM

You can also calculate revenue by order category.

```sql
SELECT
    CASE
        WHEN amount >= 10000 THEN 'Large'
        WHEN amount >=  5000 THEN 'Medium'
        ELSE 'Small'
    END AS order_size,
    SUM(amount) AS revenue
FROM orders
GROUP BY
    CASE
        WHEN amount >= 10000 THEN 'Large'
        WHEN amount >=  5000 THEN 'Medium'
        ELSE 'Small'
    END;
```

---

## 2️⃣1️⃣ Simple CASE vs Searched CASE

There are two common forms.

**Searched CASE** — this is what we've mainly used. It evaluates **conditions**.

```sql
CASE
    WHEN salary >= 1000000 THEN 'High'
    WHEN salary >=  600000 THEN 'Medium'
    ELSE 'Low'
END
```

**Simple CASE** — useful when comparing **one expression** against specific values.

```sql
CASE department
    WHEN 'IT'      THEN 'Technology'
    WHEN 'HR'      THEN 'People'
    WHEN 'Finance' THEN 'Corporate'
    ELSE 'Other'
END
```

Think:

| Form | Use |
| ---- | --- |
| **Simple CASE** | Compare one value |
| **Searched CASE** | Evaluate different conditions |

---

## 2️⃣2️⃣ CASE and NULL

You can explicitly handle `NULL`.

```sql
SELECT
    employee_name,
    CASE
        WHEN manager_id IS NULL
            THEN 'No Manager Assigned'
        ELSE 'Manager Assigned'
    END AS manager_status
FROM employees;
```

> This is much better than comparing `NULL` using `=`.

---

## 2️⃣3️⃣ CASE and COALESCE

Sometimes you want to replace `NULL` with a default value.

```sql
SELECT
    employee_name,
    COALESCE(bonus, 0) AS bonus
FROM employees;
```

You can combine this with `CASE`:

```sql
SELECT
    employee_name,
    CASE
        WHEN COALESCE(bonus, 0) > 10000
            THEN 'High Bonus'
        ELSE 'Standard Bonus'
    END AS bonus_category
FROM employees;
```

---

## 2️⃣4️⃣ CASE in Data Cleaning

`CASE` can also standardize inconsistent values. Suppose a dataset contains: `M`, `Male`, `male`, `MALE`

You can standardize them:

```sql
SELECT
    employee_name,
    CASE
        WHEN LOWER(gender) = 'm'
          OR LOWER(gender) = 'male'
        THEN 'Male'

        WHEN LOWER(gender) = 'f'
          OR LOWER(gender) = 'female'
        THEN 'Female'

        ELSE 'Unknown'
    END AS standardized_gender
FROM employees;
```

> This is a practical **data-cleaning** technique.

---

## 2️⃣5️⃣ CASE for Business Rules

Imagine a company wants to classify customers:

| Rule | Segment |
| ---- | ------- |
| Spend ≥ ₹100,000 | VIP |
| Spend ≥ ₹50,000 | High Value |
| Spend ≥ ₹10,000 | Regular |
| Otherwise | Low Value |

**SQL:**

```sql
SELECT
    customer_name,
    total_spend,
    CASE
        WHEN total_spend >= 100000 THEN 'VIP'
        WHEN total_spend >=  50000 THEN 'High Value'
        WHEN total_spend >=  10000 THEN 'Regular'
        ELSE 'Low Value'
    END AS customer_segment
FROM customers;
```

> 💡 This is an important Data Analyst mindset: **convert business rules into SQL logic.**

---

## 🧠 Common CASE Mistakes

**❌ Mistake 1: Forgetting `END`**

```sql
-- Wrong
CASE WHEN salary > 500000 THEN 'High'

-- Correct
CASE WHEN salary > 500000 THEN 'High' ELSE 'Low' END
```

**❌ Mistake 2: Incorrect condition order**

```sql
-- Wrong
CASE WHEN salary > 500000 THEN 'Medium'
     WHEN salary > 1000000 THEN 'High' END
```

The second condition won't be reached for salaries above ₹1 million because they already satisfy the first condition.

```sql
-- Better
CASE WHEN salary > 1000000 THEN 'High'
     WHEN salary >  500000 THEN 'Medium'
     ELSE 'Low' END
```

**❌ Mistake 3: Forgetting `ELSE`**

You can omit `ELSE`, but if no `WHEN` condition matches, SQL generally returns `NULL`.

```sql
-- Better when appropriate
CASE WHEN status = 'Completed' THEN 'Success'
     WHEN status = 'Cancelled' THEN 'Failure'
     ELSE 'Other' END
```

**❌ Mistake 4: Confusing `CASE` with filtering**

- `CASE` **creates or transforms** a value.
- `WHERE` **filters** rows.

For example, `CASE WHEN salary > 800000 THEN 'High' ELSE 'Low' END` doesn't remove rows — it **categorizes** them.

---

## 💼 SQL Interview Questions

**Q1. What is `CASE` in SQL?**

`CASE` is an expression used to implement conditional logic and return different values based on specified conditions.

**Q2. Can `CASE` be used with aggregate functions?**

Yes — `SUM(CASE WHEN status = 'Completed' THEN 1 ELSE 0 END)`

**Q3. What happens if no `WHEN` condition matches?**

If there is an `ELSE`, its value is returned. Otherwise, the result is generally `NULL`.

**Q4. Does `CASE` stop after the first matching condition?**

For a searched `CASE`, SQL returns the result associated with the **first** matching `WHEN` condition.

**Q5. Can `CASE` be used with `GROUP BY`?**

Yes. You can group by a `CASE` expression or, depending on the SQL dialect, an alias representing that expression.

**Q6. What is conditional aggregation?**

Using expressions such as `CASE` inside aggregate functions to calculate metrics for selected conditions.

---

## 🎯 Practice Questions

Try solving these yourself first.

1. Classify employees as: **High** → salary ≥ 1,000,000; **Medium** → salary ≥ 600,000; **Low** → everything else
2. Classify orders as: **Large** → amount ≥ 10,000; **Medium** → amount ≥ 5,000; **Small** → everything else
3. Count completed and cancelled orders using conditional aggregation.
4. Calculate successful transaction value.
5. Classify customers as VIP if spending is greater than ₹100,000.
6. Create a column that says *Has Manager* or *No Manager* based on `manager_id`.
7. Create salary bands and count employees in each band.
8. Calculate completed revenue and cancelled revenue in the same query.

### ✅ Answers

```sql
-- A1
SELECT employee_name, salary,
       CASE WHEN salary >= 1000000 THEN 'High'
            WHEN salary >=  600000 THEN 'Medium'
            ELSE 'Low' END AS salary_category
FROM employees;

-- A2
SELECT order_id, amount,
       CASE WHEN amount >= 10000 THEN 'Large'
            WHEN amount >=  5000 THEN 'Medium'
            ELSE 'Small' END AS order_size
FROM orders;

-- A3
SELECT SUM(CASE WHEN order_status = 'Completed' THEN 1 ELSE 0 END) AS completed_orders,
       SUM(CASE WHEN order_status = 'Cancelled' THEN 1 ELSE 0 END) AS cancelled_orders
FROM orders;

-- A4
SELECT SUM(CASE WHEN transaction_status = 'Success' THEN amount ELSE 0 END) AS successful_transaction_value
FROM transactions;

-- A5
SELECT customer_name, total_spend,
       CASE WHEN total_spend >= 100000 THEN 'VIP' ELSE 'Regular' END AS customer_category
FROM customers;

-- A6
SELECT employee_name,
       CASE WHEN manager_id IS NULL THEN 'No Manager' ELSE 'Has Manager' END AS manager_status
FROM employees;

-- A7
SELECT CASE WHEN salary >= 1000000 THEN 'High'
            WHEN salary >=  600000 THEN 'Medium'
            ELSE 'Low' END AS salary_band,
       COUNT(*) AS employee_count
FROM employees
GROUP BY CASE WHEN salary >= 1000000 THEN 'High'
              WHEN salary >=  600000 THEN 'Medium'
              ELSE 'Low' END;

-- A8
SELECT SUM(CASE WHEN order_status = 'Completed' THEN amount ELSE 0 END) AS completed_revenue,
       SUM(CASE WHEN order_status = 'Cancelled' THEN amount ELSE 0 END) AS cancelled_revenue
FROM orders;
```

---

## 🔥 Mini Challenge

Imagine an e-commerce company wants this dashboard:

- Total Orders
- Completed Orders
- Cancelled Orders
- Completed Revenue
- Cancelled Revenue
- Completion Rate

**Write one SQL query to calculate all six metrics.**

Think about:

| Tool | Gives you |
| ---- | --------- |
| `COUNT(*)` | Total orders |
| `SUM(CASE...)` | Conditional counts / revenue |
| `COUNT` + `SUM` | Completion rate |

**Solution:**

```sql
SELECT
    COUNT(*) AS total_orders,
    SUM(CASE WHEN order_status = 'Completed' THEN 1 ELSE 0 END) AS completed_orders,
    SUM(CASE WHEN order_status = 'Cancelled' THEN 1 ELSE 0 END) AS cancelled_orders,
    SUM(CASE WHEN order_status = 'Completed' THEN amount ELSE 0 END) AS completed_revenue,
    SUM(CASE WHEN order_status = 'Cancelled' THEN amount ELSE 0 END) AS cancelled_revenue,
    ROUND(
        100.0 * SUM(CASE WHEN order_status = 'Completed' THEN 1 ELSE 0 END)
        / NULLIF(COUNT(*), 0),
        2
    ) AS completion_rate
FROM orders;
```

---

> 📌 Once you become comfortable with `CASE`, you'll be able to build much more meaningful analytical queries instead of simply retrieving raw data.

💡 **Double Tap ❤️ For Part-8**
