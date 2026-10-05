# 🚀 SQL Roadmap 2026 — Part 2

## 📤 SQL SELECT Statement & Retrieving Data

Now that you understand databases, tables, rows, columns, primary keys, and foreign keys, it's time to learn the most fundamental SQL command: **`SELECT`**.

> Almost every SQL analysis starts with retrieving data.

---

## 1️⃣ What is SELECT?

`SELECT` is used to retrieve data from one or more columns in a table.

**Basic syntax:**

```sql
SELECT column_name
FROM table_name;
```

**Example:**

```sql
SELECT customer_name
FROM customers;
```

---

## 2️⃣ Select Multiple Columns

You can retrieve multiple columns by separating them with commas.

```sql
SELECT
    customer_id,
    customer_name,
    city
FROM customers;
```

**Result:**

| customer_id | customer_name | city   |
| ----------- | ------------- | ------ |
| 1           | Rahul         | Mumbai |
| 2           | Priya         | Delhi  |
| 3           | Amit          | Pune   |

---

## 3️⃣ Select All Columns Using `*`

If you want every column:

```sql
SELECT *
FROM customers;
```

`*` means **all columns**.

> ⚠️ **Interview Tip:** Although `SELECT *` is convenient while exploring data, avoid relying on it in production queries. Prefer selecting only needed columns.

---

## 4️⃣ Column Aliases

Use `AS` to give a column a different name in the result.

```sql
SELECT
    customer_name AS name,
    city AS location
FROM customers;
```

> The original table is **not** changed.

---

## 5️⃣ Aliases Without AS

```sql
SELECT
    customer_name name,
    city location
FROM customers;
```

However, using `AS` is generally clearer for beginners.

---

## 6️⃣ Calculations Inside SELECT

```sql
SELECT
    product_name,
    price,
    price * 0.90 AS discounted_price
FROM products;
```

---

## 7️⃣ Arithmetic Operators

| Operator | Meaning |
| -------- | -------------- |
| `+`      | Addition       |
| `-`      | Subtraction    |
| `*`      | Multiplication |
| `/`      | Division       |

```sql
SELECT
    product_name,
    selling_price,
    cost_price,
    selling_price - cost_price AS profit
FROM products;
```

---

## 8️⃣ Using Expressions

```sql
SELECT
    product_name,
    quantity,
    unit_price,
    quantity * unit_price AS total_value
FROM order_items;
```

---

## 9️⃣ DISTINCT

Removes duplicate values.

```sql
SELECT DISTINCT city
FROM customers;
```

---

## 🔟 DISTINCT Across Multiple Columns

```sql
SELECT DISTINCT
    city,
    customer_segment
FROM customers;
```

---

## 1️⃣1️⃣ Using SELECT With Text

```sql
SELECT
    customer_name,
    'Active Customer' AS status
FROM customers;
```

---

## 1️⃣2️⃣ Combining Columns

```sql
SELECT
    first_name,
    last_name,
    CONCAT(first_name, ' ', last_name) AS full_name
FROM employees;
```

---

## 1️⃣3️⃣ SELECT With a Condition

```sql
SELECT
    customer_name,
    city
FROM customers
WHERE city = 'Mumbai';
```

---

## 1️⃣4️⃣ SQL Query Structure

At this stage, learn this basic pattern:

```sql
SELECT column1, column2
FROM table_name;
```

```sql
SELECT column1, column2
FROM table_name
WHERE condition;
```

---

## 1️⃣5️⃣ A Real-World Example

**Manager asks:** *"Show me product name, selling price, cost price, and profit."*

```sql
SELECT
    product_name,
    selling_price,
    cost_price,
    selling_price - cost_price AS profit
FROM products;
```

---

## 🧠 Common Beginner Mistakes

**❌ Mistake 1: Forgetting `FROM`**

```sql
-- Wrong
SELECT customer_name;

-- Correct
SELECT customer_name FROM customers;
```

**❌ Mistake 2: Using commas incorrectly**

```sql
-- Wrong
SELECT customer_id customer_name city

-- Correct — use commas
SELECT customer_id, customer_name, city
```

**❌ Mistake 3: Using quotes around column names unnecessarily**

`SELECT 'customer_name'` treats it as **text**, not a column.

**❌ Mistake 4: Confusing `*`**

`SELECT *` means return **all columns**, not all rows.

---

## 🎯 Practice Questions

1. Display all columns from `employees`
2. Display `employee_name`, `salary`, `department_id`
3. Display unique cities from `customers`
4. Display `product_name`, `price` and 15% discounted price
5. Display `product_name`, `selling_price`, `cost_price` and profit
6. Display customer names with alias `Customer`
7. Display employee name and salary increased by 10%

### ✅ Answers

```sql
-- A1
SELECT * FROM employees;

-- A2
SELECT employee_name, salary, department_id FROM employees;

-- A3
SELECT DISTINCT city FROM customers;

-- A4
SELECT product_name, price, price * 0.85 AS discounted_price FROM products;

-- A5
SELECT product_name, selling_price, cost_price, selling_price - cost_price AS profit FROM products;

-- A6
SELECT customer_name AS Customer FROM customers;

-- A7
SELECT employee_name, salary, salary * 1.10 AS increased_salary FROM employees;
```

---

## 💼 Interview Questions

**1. What does `SELECT` do?**

Retrieves data from columns.

**2. What does `SELECT *` mean?**

Retrieves all columns.

**3. What is `DISTINCT`?**

Removes duplicate combinations.

**4. What is an alias?**

A temporary name for clarity.

**5. Can SQL perform calculations?**

Yes — arithmetic expressions can be written directly in queries.

---

💡 **Double Tap ❤️ For Part-3**
