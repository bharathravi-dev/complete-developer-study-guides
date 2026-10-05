# 🚀 SQL Roadmap 2026 — Part 1

## ✅ Database Basics You Should Know

Thanks for the amazing response to the SQL roadmap! 🔥

Let's start from the absolute foundation. Before learning `SELECT`, `JOIN`, or window functions, you need to understand what a database actually is and how data is organized inside it.

---

## 1️⃣ What is a Database?

A **database** is an organized collection of data that allows us to store, manage, search, and analyze information efficiently.

For example, an e-commerce company may need to store:

- Customers
- Orders
- Products
- Payments
- Employees
- Reviews

Instead of keeping everything in separate Excel files, the company can store this information in a database.

> 💡 Think of a database as a **structured digital storage system** for data.

---

## 2️⃣ What is DBMS?

**DBMS = Database Management System**

A DBMS is software used to create, store, manage, retrieve, and manipulate data in databases.

**Examples:** MySQL, PostgreSQL, Microsoft SQL Server, Oracle Database, SQLite

The flow looks like this:

```text
You → SQL Query → DBMS → Database → Result
```

When you write:

```sql
SELECT * FROM customers;
```

the DBMS processes your SQL query and returns the requested data.

---

## 3️⃣ SQL vs DBMS

| | **SQL** | **DBMS** |
| --- | --- | --- |
| **What it is** | A language used to communicate with a relational database | The software that manages the database |
| **Example** | `SELECT customer_name FROM customers;` | MySQL, PostgreSQL, SQL Server, Oracle |

> **SQL = Language, DBMS = Software** that understands and executes that language.

---

## 4️⃣ What is a Relational Database?

A relational database stores data in **tables** and establishes **relationships** between those tables.

**Customers**

| customer_id | customer_name | city   |
| ----------- | ------------- | ------ |
| 1           | Rahul         | Mumbai |
| 2           | Priya         | Delhi  |
| 3           | Amit          | Pune   |

**Orders**

| order_id | customer_id | amount |
| -------- | ----------- | ------ |
| 101      | 1           | 2500   |
| 102      | 2           | 1800   |
| 103      | 1           | 4200   |

The link between them:

```text
Customers.customer_id → Orders.customer_id
```

This relationship allows us to answer: **How much has each customer spent?**

---

## 5️⃣ What is a Table?

A **table** is where data is actually organized and stored. Think of it like an Excel sheet.

**employees**

| employee_id | name  | department | salary |
| ----------- | ----- | ---------- | ------ |
| 1           | Rahul | IT         | 80000  |
| 2           | Priya | Finance    | 75000  |
| 3           | Amit  | IT         | 90000  |

A database can contain hundreds or thousands of tables.

---

## 6️⃣ What is a Row?

A **row** represents one record.

```text
1 | Rahul | IT | 80000     →   1 row = 1 record
```

---

## 7️⃣ What is a Column?

A **column** represents an attribute or field.

```text
Row    → Record
Column → Attribute
```

---

## 8️⃣ What is a Primary Key?

A **Primary Key** uniquely identifies each row in a table — e.g. `employee_id = 101` cannot repeat.

**Properties:**

- ✅ Must be unique
- ✅ Cannot normally be `NULL`
- ✅ Identifies a specific record

---

## 9️⃣ What is a Foreign Key?

A **Foreign Key** creates a relationship between tables.

`orders.customer_id` is a Foreign Key referring to `customers.customer_id`.

---

## 🔟 Primary Key vs Foreign Key

| Key | Purpose |
| --- | --- |
| **PRIMARY KEY** | Uniquely identifies a record |
| **FOREIGN KEY** | Connects one table to another |

---

## 1️⃣1️⃣ What is NULL?

`NULL` means the value is **missing, unknown, or not available**.

> ⚠️ It does **NOT** mean `0`, an empty string, or the text `"NULL"`.

✅ **Correct query:**

```sql
SELECT * FROM employees WHERE manager_id IS NULL;
```

❌ **Wrong:**

```sql
WHERE manager_id = NULL;
```

---

## 1️⃣2️⃣ What are Data Types?

| Data type | Example |
| --- | --- |
| `INT` | `101` |
| `DECIMAL(10,2)` | `85000.50` |
| `VARCHAR(100)` | `'Rahul Sharma'` |
| `DATE` | `2026-08-25` |
| `TIMESTAMP` | `2026-08-25 10:30:00` |

---

## 1️⃣3️⃣ One Database Can Have Multiple Tables

```text
DATABASE → Customers, Products, Employees → Orders → Payments
```

The power of SQL comes from being able to **analyze these related tables together**.

---

## 1️⃣4️⃣ Example: E-Commerce Database

Tables: `customers`, `products`, `orders`, `order_items`

Now the business can ask:

- Which city generated the highest revenue?
- Who are the top 10 customers?
- Which products sell the most?

---

## 🧠 Your First SQL Query

```sql
SELECT * FROM customers;
```

| Keyword | Meaning |
| --- | --- |
| `SELECT` | Retrieve data |
| `*` | All columns |
| `FROM` | From this table |

Selecting specific columns:

```sql
SELECT customer_id, customer_name, city FROM customers;
```

---

## 💼 Interview Question

**Q. What is the difference between a Primary Key and a Foreign Key?**

**A.** A Primary Key uniquely identifies each record within its own table, while a Foreign Key is used to reference a key in another table and establish a relationship between tables.

---

> 📌 If you understand how data is structured and how tables relate to each other, learning `JOIN`, `GROUP BY`, CTEs, and Window Functions becomes much easier.

💡 **Double Tap ❤️ For Part-2**
