# 🚀 SQL Roadmap 2026 — Part 3

## 🔎 Filtering Data with WHERE, AND, OR & Comparison Operators

Now that you know how to retrieve data using `SELECT`, the next essential skill is **filtering data**.

In real-world analytics, you rarely need every row from a table. Instead, you'll ask questions like:

> - Which customers are from Mumbai?
> - Which products cost more than ₹10,000?
> - Which orders were placed in January?
> - Which employees earn more than ₹8 lakh?

That's where the `WHERE` clause comes in.

---

## 1️⃣ What is WHERE?

The `WHERE` clause is used to **filter rows** based on a condition.

**Basic syntax:**

```sql
SELECT column1, column2
FROM table_name
WHERE condition;
```

**Example:**

```sql
SELECT
    customer_name,
    city
FROM customers
WHERE city = 'Mumbai';
```

This returns only customers whose city is Mumbai.

---

## 2️⃣ Comparison Operators

| Operator | Meaning                  |
| -------- | ------------------------ |
| `=`      | Equal to                 |
| `<>`     | Not equal to             |
| `!=`     | Not equal to             |
| `>`      | Greater than             |
| `<`      | Less than                |
| `>=`     | Greater than or equal to |
| `<=`     | Less than or equal to    |

---

## 3️⃣ Equal To `=`

To find employees from IT:

```sql
SELECT *
FROM employees
WHERE department = 'IT';
```

---

## 4️⃣ Greater Than `>`

Find products costing more than ₹50,000:

```sql
SELECT product_name, price
FROM products
WHERE price > 50000;
```

---

## 5️⃣ Less Than `<`

Find products costing less than ₹5,000:

```sql
SELECT product_name, price
FROM products
WHERE price < 5000;
```

---

## 6️⃣ Greater Than or Equal To `>=`

Find employees earning at least ₹8 lakh:

```sql
SELECT employee_name, salary
FROM employees
WHERE salary >= 800000;
```

---

## 7️⃣ Less Than or Equal To `<=`

Find products priced at or below ₹10,000:

```sql
SELECT product_name, price
FROM products
WHERE price <= 10000;
```

---

## 8️⃣ Not Equal To

You can use either `<>` or `!=`

```sql
SELECT *
FROM employees
WHERE department <> 'IT';
```

---

## 9️⃣ Filtering Text Values

Text values are normally written inside **single quotes**.

```sql
-- ✅ Correct
SELECT * FROM customers WHERE city = 'Mumbai';

-- ❌ Incorrect
SELECT * FROM customers WHERE city = Mumbai;
```

> Remember: **Text → usually use single quotes.**

---

## 🔟 Filtering Numeric Values

Numbers don't require quotes.

```sql
-- ✅ Correct
SELECT * FROM products WHERE price > 50000;
```

---

## 1️⃣1️⃣ Filtering Dates

You can also filter dates.

```sql
SELECT * FROM orders WHERE order_date >= '2026-01-01';

SELECT * FROM orders WHERE order_date <  '2026-02-01';
```

---

## 1️⃣2️⃣ Using AND

`AND` allows you to apply multiple conditions.

Find employees from IT earning more than ₹8 lakh:

```sql
SELECT employee_name, department, salary
FROM employees
WHERE department = 'IT' AND salary > 800000;
```

> **Both** conditions must be true.

---

## 1️⃣3️⃣ Using OR

`OR` returns a row if **at least one** condition is true.

Find customers from Mumbai or Delhi:

```sql
SELECT customer_name, city
FROM customers
WHERE city = 'Mumbai' OR city = 'Delhi';
```

---

## 1️⃣4️⃣ AND vs OR

| Operator | Rule |
| -------- | ---- |
| `AND` | **Both** conditions must be true |
| `OR`  | **At least one** condition must be true |

---

## 1️⃣5️⃣ Combining AND and OR

Find Premium customers from Mumbai **or** Delhi:

```sql
SELECT *
FROM customers
WHERE customer_segment = 'Premium'
  AND (city = 'Mumbai' OR city = 'Delhi');
```

---

## 1️⃣6️⃣ Why Parentheses Matter

Consider:

```sql
WHERE city = 'Mumbai' OR city = 'Delhi' AND customer_segment = 'Premium';
```

A safer and clearer version is:

```sql
WHERE (city = 'Mumbai' OR city = 'Delhi') AND customer_segment = 'Premium';
```

> ✅ **Best practice:** When combining `AND` and `OR`, use parentheses to make your business logic obvious.

---

## 1️⃣7️⃣ Using IN

Instead of multiple `OR`s:

```sql
SELECT * FROM customers WHERE city IN ('Mumbai', 'Delhi', 'Pune');
```

`IN` means: **Match any value from this list.**

---

## 1️⃣8️⃣ NOT IN

```sql
SELECT * FROM customers WHERE city NOT IN ('Mumbai', 'Delhi');
```

---

## 1️⃣9️⃣ Using BETWEEN

Find products priced between ₹10,000 and ₹50,000:

```sql
SELECT product_name, price
FROM products
WHERE price BETWEEN 10000 AND 50000;
```

> `BETWEEN` is **inclusive**: `10000 ≤ price ≤ 50000`

---

## 2️⃣0️⃣ NOT BETWEEN

```sql
SELECT product_name, price
FROM products
WHERE price NOT BETWEEN 10000 AND 50000;
```

---

## 2️⃣1️⃣ LIKE Operator

`LIKE` is used for **pattern matching**.

```sql
SELECT * FROM customers WHERE customer_name LIKE 'A%';
```

`%` means: **any sequence of characters**.

| Pattern | Meaning |
| ------- | ---------- |
| `A%`    | Starts with A |
| `%A`    | Ends with A |
| `%A%`   | Contains A |

---

## 2️⃣2️⃣ LIKE Examples

```sql
WHERE customer_name LIKE 'R%';    -- beginning with R
WHERE customer_name LIKE '%a';    -- ending with a
WHERE customer_name LIKE '%sh%';  -- containing "sh"
```

---

## 2️⃣3️⃣ The `_` Wildcard

The underscore `_` represents **exactly one character**.

```sql
WHERE customer_name LIKE 'A_it';  -- Can match Amit
```

---

## 2️⃣4️⃣ IS NULL

```sql
SELECT * FROM employees WHERE manager_id IS NULL;
```

---

## 2️⃣5️⃣ IS NOT NULL

```sql
SELECT * FROM employees WHERE manager_id IS NOT NULL;
```

---

## 2️⃣6️⃣ WHERE With Calculations

```sql
SELECT product_name, selling_price, cost_price
FROM products
WHERE selling_price - cost_price > 5000;
```

---

## 2️⃣7️⃣ WHERE With Dates

Find orders placed during January 2026:

```sql
SELECT *
FROM orders
WHERE order_date >= '2026-01-01'
  AND order_date <  '2026-02-01';
```

---

## 2️⃣8️⃣ WHERE vs HAVING

| Clause | Filters |
| ------ | ------- |
| `WHERE`  | Individual **rows**, **before** aggregation |
| `HAVING` | **Groups**, **after** aggregation |

```sql
SELECT
    customer_id,
    SUM(amount) AS total_spent
FROM orders
GROUP BY customer_id
HAVING SUM(amount) > 50000;
```

---

## 2️⃣9️⃣ Real-World Business Example

Find completed orders above ₹5,000 placed by customers in Mumbai:

```sql
SELECT order_id, customer_id, order_date, amount
FROM orders
WHERE order_status  = 'Completed'
  AND amount        > 5000
  AND customer_city = 'Mumbai';
```

---

## 3️⃣0️⃣ Another Real-World Example

Find active customers from Mumbai, Delhi, or Pune:

```sql
SELECT customer_id, customer_name, city
FROM customers
WHERE customer_status = 'Active'
  AND city IN ('Mumbai', 'Delhi', 'Pune');
```

---

## 🧠 A Simple Way to Think About WHERE

```text
Imagine 100,000 rows → WHERE → 15,000 rows → SELECT → Your result
```

---

## 💼 Common SQL Interview Questions

**Q1. What is the purpose of `WHERE`?**

`WHERE` filters individual rows based on a specified condition.

**Q2. Can `WHERE` use multiple conditions?**

Yes. Multiple conditions can be combined using `AND`, `OR`, and parentheses.

**Q3. What is the difference between `=` and `LIKE`?**

`=` checks for an exact value, while `LIKE` performs pattern matching using wildcards such as `%` and `_`.

**Q4. What is the difference between `WHERE` and `HAVING`?**

`WHERE` filters rows before aggregation, while `HAVING` filters grouped results after aggregation.

**Q5. How do you check for NULL?**

`WHERE column_name IS NULL` or `WHERE column_name IS NOT NULL`

**Q6. What does `BETWEEN` do?**

It filters values within a specified **inclusive** range.

**Q7. What does `IN` do?**

It checks whether a value matches any value in a specified list.

---

## 🎯 Practice Questions

1. Find all employees whose salary is greater than ₹700,000.
2. Find customers from Mumbai.
3. Find products priced between ₹10,000 and ₹50,000.
4. Find employees from either IT or Finance.
5. Find customers whose names start with "S".
6. Find orders above ₹5,000 that are completed.
7. Find customers from Mumbai, Delhi, or Pune.
8. Find employees whose manager is NULL.
9. Find products that are not in the Electronics category.
10. Find orders placed during January 2026.

### ✅ Answers

```sql
-- A1
SELECT * FROM employees WHERE salary > 700000;

-- A2
SELECT * FROM customers WHERE city = 'Mumbai';

-- A3
SELECT * FROM products WHERE price BETWEEN 10000 AND 50000;

-- A4
SELECT * FROM employees WHERE department IN ('IT', 'Finance');

-- A5
SELECT * FROM customers WHERE customer_name LIKE 'S%';

-- A6
SELECT * FROM orders WHERE amount > 5000 AND order_status = 'Completed';

-- A7
SELECT * FROM customers WHERE city IN ('Mumbai', 'Delhi', 'Pune');

-- A8
SELECT * FROM employees WHERE manager_id IS NULL;

-- A9
SELECT * FROM products WHERE category <> 'Electronics';

-- A10
SELECT * FROM orders WHERE order_date >= '2026-01-01' AND order_date < '2026-02-01';
```

---

## 🔥 Mini Challenge

Find Premium customers from Mumbai, Pune, or Delhi whose spending is greater than ₹70,000.

```sql
SELECT customer_id, name, city, spend
FROM customers
WHERE segment = 'Premium'
  AND city IN ('Mumbai', 'Pune', 'Delhi')
  AND spend > 70000;
```

---

💡 **Double Tap ❤️ For Part-4**
