# 🚀 SQL Roadmap 2026 — Part 4

## 🔢 Sorting, Limiting & Selecting the Right Records

In the previous part, you learned how to filter data using `WHERE`.

Now we'll learn how to control **which records appear first, last, or how many records are returned.**

> These concepts are simple, but they are extremely important for SQL interviews and real-world analytics.

---

## 1️⃣ ORDER BY

`ORDER BY` is used to **sort** query results.

**Syntax:**

```sql
SELECT column1, column2
FROM table_name
ORDER BY column_name;
```

By default, SQL sorts in **ascending order** (`ASC`).

**Example:**

```sql
SELECT
    employee_name,
    salary
FROM employees
ORDER BY salary;
```

This displays employees from the lowest salary to the highest.

---

## 2️⃣ ASC — Ascending Order

You can explicitly specify `ASC`.

```sql
SELECT
    employee_name,
    salary
FROM employees
ORDER BY salary ASC;
```

| Type | Sorted result |
| ---- | ------------- |
| Numbers | `100, 250, 500, 1000` |
| Text | `Amit, Neha, Priya, Rahul` |

---

## 3️⃣ DESC — Descending Order

Use `DESC` when you want the **highest values first**.

```sql
SELECT
    employee_name,
    salary
FROM employees
ORDER BY salary DESC;
```

**Result:**

| employee_name | salary  |
| ------------- | ------- |
| Amit          | 1200000 |
| Priya         | 950000  |
| Rahul         | 850000  |
| Neha          | 650000  |

> This is one of the most commonly used SQL patterns.

---

## 4️⃣ Real-World Example: Top Salaries

**Business requirement:** *Find the highest-paid employees.*

```sql
SELECT
    employee_name,
    salary
FROM employees
ORDER BY salary DESC;
```

But this might return thousands of employees. That's where `LIMIT` becomes useful.

---

## 5️⃣ LIMIT

`LIMIT` restricts the number of rows returned.

```sql
SELECT
    employee_name,
    salary
FROM employees
ORDER BY salary DESC
LIMIT 5;
```

This returns only the top 5 employees by salary. Think of it as:

```text
ORDER BY DESC → Highest first → LIMIT 5 → Keep first 5
```

---

## 6️⃣ Top 10 Products by Price

```sql
SELECT
    product_name,
    price
FROM products
ORDER BY price DESC
LIMIT 10;
```

> Very common in analytics.

---

## 7️⃣ LIMIT Without ORDER BY

You technically *can* write:

```sql
SELECT *
FROM customers
LIMIT 10;
```

But this means:

> Give me 10 rows.

It does **not** mean:

> Give me the first 10 rows according to some meaningful business order.

⚠️ Without `ORDER BY`, the returned order should generally **not** be relied upon.

If you want the top 10 customers by revenue:

```sql
SELECT
    customer_id,
    revenue
FROM customer_revenue
ORDER BY revenue DESC
LIMIT 10;
```

---

## 8️⃣ OFFSET

`OFFSET` allows you to **skip** a number of rows.

```sql
SELECT
    employee_name,
    salary
FROM employees
ORDER BY salary DESC
LIMIT 5 OFFSET 5;
```

This skips the first 5 rows and returns the next 5. Conceptually:

```text
Rows 1–5  → Skip
Rows 6–10 → Return
```

---

## 9️⃣ Pagination

`LIMIT` and `OFFSET` are often used for **pagination**.

```sql
-- Page 1
SELECT *
FROM customers
ORDER BY customer_id
LIMIT 10 OFFSET 0;

-- Page 2
SELECT *
FROM customers
ORDER BY customer_id
LIMIT 10 OFFSET 10;

-- Page 3
SELECT *
FROM customers
ORDER BY customer_id
LIMIT 10 OFFSET 20;
```

The general pattern is:

| Page | Offset |
| ---- | ------ |
| Page 1 | `OFFSET 0` |
| Page 2 | `OFFSET 10` |
| Page 3 | `OFFSET 20` |

---

## 🔟 Sorting by Multiple Columns

You can sort using more than one column.

```sql
SELECT
    employee_name,
    department,
    salary
FROM employees
ORDER BY department ASC, salary DESC;
```

SQL first sorts by `department`, then within each department by `salary DESC`.

**Example:**

```text
Finance: 950000
Finance: 750000
IT:     1200000
IT:      850000
IT:      700000
```

---

## 1️⃣1️⃣ Why Multiple Sorting Columns Matter

Suppose several products have the same price:

```text
Laptop: 50000
Phone:  50000
Tablet: 50000
```

You can add a second sorting condition:

```sql
SELECT
    product_name,
    price
FROM products
ORDER BY
    price DESC,
    product_name ASC;
```

Now SQL uses the product name to **break ties**.

---

## 1️⃣2️⃣ Sorting by Calculated Values

You can sort using an expression.

```sql
SELECT
    product_name,
    selling_price,
    cost_price,
    selling_price - cost_price AS profit
FROM products
ORDER BY profit DESC;
```

This displays the products with the highest calculated profit first.

---

## 1️⃣3️⃣ Sorting by an Alias

You can usually sort using a column alias defined in the `SELECT` list.

```sql
SELECT
    product_name,
    selling_price - cost_price AS profit
FROM products
ORDER BY profit DESC;
```

This is convenient and makes the query easier to read.

---

## 1️⃣4️⃣ Sorting by Column Position

Some SQL dialects allow:

```sql
SELECT
    product_name,
    price
FROM products
ORDER BY 2 DESC;
```

Here `1 → product_name`, `2 → price` — so SQL sorts by the second selected column.

> ⚠️ **Best Practice:** Although positional ordering may be supported, prefer `ORDER BY price DESC;` because it is easier to understand and less fragile if the `SELECT` list changes.

---

## 1️⃣5️⃣ NULL Values and ORDER BY

`NULL` values require special attention.

```text
Rahul: 5000
Priya: NULL
Amit:  8000
```

The position of `NULL` values when sorting can **vary by database system and sort direction**.

Some systems allow explicit control:

```sql
ORDER BY bonus DESC NULLS LAST;
-- or
ORDER BY bonus ASC NULLS FIRST;
```

> 💼 **Interview Tip:** Don't assume `NULL` sorting behavior is identical across MySQL, PostgreSQL, SQL Server, and Oracle.

---

## 1️⃣6️⃣ ORDER BY With WHERE

You can combine filtering and sorting.

**Requirement:** *Find Mumbai customers and display the highest spenders first.*

```sql
SELECT
    customer_name,
    city,
    total_spend
FROM customers
WHERE city = 'Mumbai'
ORDER BY total_spend DESC;
```

Execution conceptually works as:

```text
FROM → WHERE → SELECT → ORDER BY
```

> The detailed logical processing order has a few nuances, but this is a useful beginner mental model.

---

## 1️⃣7️⃣ ORDER BY With LIMIT

This combination is extremely important.

**Requirement:** *Find the top 3 customers by spending.*

```sql
SELECT
    customer_name,
    total_spend
FROM customers
ORDER BY total_spend DESC
LIMIT 3;
```

> This pattern appears constantly in SQL interviews.

---

## 1️⃣8️⃣ Top N Per Category

Here's an important distinction.

For **top 3 products overall**, you can use:

```sql
ORDER BY revenue DESC LIMIT 3;
```

But if the requirement is **top 3 products in every category**, `LIMIT 3` alone isn't enough. You'll eventually need window functions such as `ROW_NUMBER()` or `DENSE_RANK()`.

```sql
WITH ranked_products AS (
    SELECT
        product_name,
        category,
        revenue,
        ROW_NUMBER() OVER (
            PARTITION BY category
            ORDER BY revenue DESC
        ) AS rn
    FROM product_sales
)
SELECT
    product_name,
    category,
    revenue
FROM ranked_products
WHERE rn <= 3;
```

> Don't worry if this looks advanced — you'll learn window functions later.

---

## 1️⃣9️⃣ DISTINCT & ORDER BY

You can combine `DISTINCT` and `ORDER BY`.

```sql
SELECT DISTINCT city
FROM customers
ORDER BY city ASC;
```

**Result:** Bangalore, Delhi, Hyderabad, Mumbai, Pune

---

## 2️⃣0️⃣ ORDER BY Multiple Columns With Different Directions

```sql
SELECT
    department,
    employee_name,
    salary
FROM employees
ORDER BY
    department ASC,
    salary DESC;
```

**Meaning:** Department → A to Z, Salary → Highest to Lowest within department

---

## 2️⃣1️⃣ Real-World Business Example

**Requirement:** *Show the 5 most expensive products that are currently active.*

```sql
SELECT
    product_name,
    category,
    price
FROM products
WHERE product_status = 'Active'
ORDER BY price DESC
LIMIT 5;
```

Notice the combination:

| Clause | Role |
| ------ | ---- |
| `WHERE` | Filter active products |
| `ORDER BY` | Highest price first |
| `LIMIT` | Keep only 5 |

---

## 2️⃣2️⃣ Another Example

**Requirement:** *Find the 10 customers with the highest total spending.*

```sql
SELECT
    customer_id,
    customer_name,
    total_spend
FROM customers
ORDER BY total_spend DESC
LIMIT 10;
```

> This is a classic Data Analyst query.

---

## 🧠 Common Beginner Mistakes

**❌ Mistake 1: Forgetting `DESC`**

If you want the highest values first, use `ORDER BY salary DESC;` — not `ORDER BY salary;`, because the default is typically ascending.

**❌ Mistake 2: Using `LIMIT` without `ORDER BY`**

```sql
-- Doesn't reliably identify the "top 5" by any business metric
SELECT * FROM products LIMIT 5;

-- Instead
SELECT * FROM products ORDER BY revenue DESC LIMIT 5;
```

**❌ Mistake 3: Confusing `LIMIT` with filtering**

`LIMIT` doesn't filter rows based on a condition.

| Clause | Meaning |
| ------ | ------- |
| `LIMIT 10` | Return **at most** 10 rows |
| `WHERE salary > 800000` | Return rows **satisfying a condition** |

**❌ Mistake 4: Using `LIMIT` for Top N per Group**

`ORDER BY revenue DESC LIMIT 3;` returns 3 rows **overall**. It does **not** return 3 rows from every category.

---

## 💼 SQL Interview Questions

**Q1. What is `ORDER BY`?**

`ORDER BY` sorts the result set according to one or more columns or expressions.

**Q2. What is the default sorting direction?**

Ascending (`ASC`) is the default in standard SQL usage.

**Q3. How do you find the highest-paid employee?**

```sql
SELECT employee_name, salary FROM employees ORDER BY salary DESC LIMIT 1;
```

**Q4. How do you find the top 5 products by revenue?**

```sql
SELECT product_name, revenue FROM products ORDER BY revenue DESC LIMIT 5;
```

**Q5. What does `OFFSET` do?**

`OFFSET` skips a specified number of rows before returning the remaining rows, subject to `LIMIT` or the database's equivalent pagination mechanism.

**Q6. Can you sort by multiple columns?**

Yes — `ORDER BY department, salary DESC;`

**Q7. Can you use an alias in `ORDER BY`?**

In most common SQL systems, yes.

```sql
SELECT salary * 12 AS annual_salary FROM employees ORDER BY annual_salary DESC;
```

---

## 🎯 Practice Questions

Try these yourself first.

1. Display all employees sorted by salary from highest to lowest.
2. Find the top 5 highest-priced products.
3. Display customers alphabetically by name.
4. Find the 10 customers with the highest spending.
5. Display employees by department alphabetically and salary from highest to lowest within each department.
6. Find the 3 cheapest products.
7. Display unique customer cities alphabetically.
8. Return the second page of 10 customers ordered by `customer_id`.
9. Find the 5 most profitable products.
10. Explain why `ORDER BY revenue DESC LIMIT 3` cannot directly find the top 3 products in each category.

### ✅ Answers

```sql
-- A1
SELECT employee_name, salary FROM employees ORDER BY salary DESC;

-- A2
SELECT product_name, price FROM products ORDER BY price DESC LIMIT 5;

-- A3
SELECT customer_name FROM customers ORDER BY customer_name ASC;

-- A4
SELECT customer_name, total_spend FROM customers ORDER BY total_spend DESC LIMIT 10;

-- A5
SELECT employee_name, department, salary FROM employees ORDER BY department ASC, salary DESC;

-- A6
SELECT product_name, price FROM products ORDER BY price ASC LIMIT 3;

-- A7
SELECT DISTINCT city FROM customers ORDER BY city ASC;

-- A8
SELECT * FROM customers ORDER BY customer_id LIMIT 10 OFFSET 10;

-- A9
SELECT product_name, profit FROM products ORDER BY profit DESC LIMIT 5;
```

**A10.** Because `LIMIT 3` applies to the **entire result**, not separately to each category. To get the top 3 within every category, you need a window function such as `ROW_NUMBER()` or `DENSE_RANK()`.

---

## 🔥 Mini Challenge

You have products:

| id | product  | category    | revenue |
| -- | -------- | ----------- | ------- |
| 1  | Laptop   | Electronics | 90000   |
| 2  | Phone    | Electronics | 70000   |
| 3  | Monitor  | Electronics | 50000   |
| 4  | Chair    | Furniture   | 80000   |
| 5  | Desk     | Furniture   | 60000   |

**Business Requirement:** Find the 3 products generating the highest revenue overall.

**Steps:**

1. Retrieve products
2. Sort revenue highest → lowest
3. Keep 3 rows

**Solution:**

```sql
SELECT product_name, category, revenue
FROM products
ORDER BY revenue DESC
LIMIT 3;
```

---

💡 **Double Tap ❤️ For Part-5**
