# 🚀 SQL Roadmap 2026 — Part 11

## 🔗 Advanced JOINs — Multiple Tables, Many-to-Many Relationships & Avoiding Double Counting

JOINs are one of the most important SQL skills for a Data Analyst.

In Part 10, we learned the basic JOIN types. Now we'll go deeper into how JOINs behave in real-world analytical datasets — especially when multiple tables are involved.

Basic JOIN syntax is easy to learn. **The difficult part is understanding what happens to your data after the JOIN.**

A JOIN can:

- duplicate rows
- multiply records
- inflate revenue
- distort averages
- produce incorrect KPIs
- create many-to-many relationships

So the real skill is not simply knowing how to write:

```sql
JOIN table2 ON table1.id = table2.id
```

It's understanding **the relationship between the tables** and **the grain of the resulting dataset**.

---

## 🧠 1. What Is Table Grain?

The **grain** of a table means:

> «What does exactly one row represent?»

For example:

| Table         | 1 row = ?                  |
| ------------- | -------------------------- |
| `customers`   | 1 customer                 |
| `orders`      | 1 order                    |
| `order_items` | 1 product within an order  |
| `payments`    | 1 payment                  |

Before joining tables, always ask: **"What does one row represent in each table?"**

---

## 🔗 2. One-to-One Relationship

```text
customers          (customer_id, customer_name)
customer_profiles  (customer_id, date_of_birth)
```

If each customer has exactly one profile: **1 customer → 1 profile**

This is a **One-to-One** relationship.

```sql
SELECT
    c.customer_id,
    c.customer_name,
    p.date_of_birth
FROM customers c
JOIN customer_profiles p
    ON c.customer_id = p.customer_id;
```

---

## 👥 3. One-to-Many Relationship

**Example:** `customers` ↓ `orders`

One customer can have many orders.

```text
Customer 101 → Order 1, Order 2, Order 3
```

So: **1 customer → many orders**

```sql
SELECT
    c.customer_name,
    o.order_id,
    o.amount
FROM customers c
JOIN orders o
    ON c.customer_id = o.customer_id;
```

---

## 🔀 4. Many-to-Many Relationship

Many rows in Table A ↕ Many rows in Table B

**Example:** `orders` ↕ `products`

- An order can contain multiple products.
- A product can appear in many orders.

Therefore: **Many Orders ↔ Many Products**

A **junction table** is commonly used: `orders`, `order_items`, `products`

---

## 🧩 5. The Role of a Junction Table

```text
orders       (order_id, customer_id)
products     (product_id, product_name)
order_items  (order_id, product_id, quantity)
```

`order_items` connects `orders` and `products`:

```text
orders ↓ order_items ↓ products
```

---

## 🛒 6. Joining Orders to Products

```sql
SELECT
    o.order_id,
    p.product_name,
    oi.quantity
FROM orders o
JOIN order_items oi
    ON o.order_id = oi.order_id
JOIN products p
    ON oi.product_id = p.product_id;
```

**Result:** Order 1001 → Laptop, Mouse

One order becomes multiple rows **because its grain has changed**.

---

## 📏 7. JOIN Changes the Grain

| Stage                        | 1 row = ?    |
| ---------------------------- | ------------ |
| Before the JOIN (`orders`)   | 1 order      |
| After joining `order_items`  | 1 order item |

> ⚠️ If you don't recognize the change in grain, you can accidentally calculate incorrect KPIs.

---

## 💰 8. The Double-Counting Problem

Suppose an order has `order_id = 1001`, `order_amount = ₹5,000`, and contains **three items**.

After joining:

```text
1001 | ₹5,000
1001 | ₹5,000
1001 | ₹5,000
```

If you run `SUM(o.order_amount)` you could calculate:

```text
₹5,000 × 3 = ₹15,000   ❌   instead of   ₹5,000   ✅
```

That's **double counting** caused by grain mismatch.

---

## ⚠️ 9. Never Assume JOIN + SUM Is Safe

```sql
SELECT
    c.customer_id,
    SUM(o.order_amount) AS total_revenue
FROM customers c
JOIN orders o
    ON c.customer_id = o.customer_id
GROUP BY c.customer_id;
```

This may be correct **if** `orders` has 1 row = 1 order.

But if you additionally join another one-to-many table:

```sql
JOIN payments p ON o.order_id = p.order_id
```

and an order has multiple payments, the order may appear multiple times.

**Your revenue can then be inflated.**

---

## 🧮 10. Example of Row Multiplication

Suppose one order has **2 order items** and **3 payment records**.

Joining both tables can potentially create:

```text
2 × 3 = 6 rows for that single order
```

This is called **row multiplication**.

If the order revenue is ₹1,000:

```text
₹1,000 × 6 = ₹6,000   ❌   A simple SUM() would therefore be wrong.
```

---

## 🛠️ 11. Solution: Aggregate Before Joining

```sql
WITH payment_summary AS (
    SELECT
        order_id,
        SUM(amount) AS total_paid
    FROM payments
    GROUP BY order_id
)
SELECT
    o.order_id,
    o.order_amount,
    p.total_paid
FROM orders o
LEFT JOIN payment_summary p
    ON o.order_id = p.order_id;
```

Now:

```text
payments (many rows per order)
    ↓ GROUP BY order_id
1 row per order
    ↓ JOIN orders
The grain is controlled ✅
```

---

## 📦 12. Aggregate Order Items Before Joining

```sql
WITH item_summary AS (
    SELECT
        order_id,
        SUM(quantity) AS total_quantity
    FROM order_items
    GROUP BY order_id
)
SELECT
    o.order_id,
    o.order_amount,
    i.total_quantity
FROM orders o
LEFT JOIN item_summary i
    ON o.order_id = i.order_id;
```

---

## 🧠 13. Pre-Aggregation

The technique above is called **pre-aggregation**.

Instead of joining detailed data immediately:

```text
✅ Detailed Table  ↓ Aggregate ↓ JOIN
❌ Table A ↓ JOIN ↓ Detailed Table ↓ Aggregate
```

Pre-aggregation can improve both:

- accuracy
- query performance

---

## 🔍 14. Finding Duplicate Keys

```sql
SELECT
    customer_id,
    COUNT(*) AS row_count
FROM customers
GROUP BY customer_id
HAVING COUNT(*) > 1;
```

If this returns rows, `customer_id` **isn't unique** in the table.

That could indicate:

- duplicate records
- unexpected table grain
- data-quality problems
- a key that isn't actually a primary key

---

## 🔑 15. Checking Key Uniqueness

```sql
SELECT
    order_id,
    COUNT(*) AS occurrences
FROM orders
GROUP BY order_id
HAVING COUNT(*) > 1;
```

No results generally means there are **no duplicates** for that key in the checked data.

---

## 🔎 16. Finding Orphan Records

An **orphan record** is a record that references something that doesn't exist in the parent table.

**Example:** `orders.customer_id` references `customers.customer_id`

```sql
SELECT
    o.order_id,
    o.customer_id
FROM orders o
LEFT JOIN customers c
    ON o.customer_id = c.customer_id
WHERE c.customer_id IS NULL;
```

---

## 🧪 17. EXISTS vs JOIN

```sql
SELECT DISTINCT c.customer_id
FROM customers c
JOIN orders o
    ON c.customer_id = o.customer_id;
```

You can use:

```sql
SELECT c.customer_id
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);
```

---

## 🚫 18. Finding Customers Without Orders Using NOT EXISTS

```sql
SELECT
    c.customer_id,
    c.customer_name
FROM customers c
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);
```

This means: «Return customers for whom no matching order exists.»

It is another form of an **anti-join**.

---

## 🔄 19. JOIN Conditions with Multiple Columns

Sometimes one column isn't enough to identify a relationship.

**Example:** sales are identified by `store_id`, `product_id`

```sql
SELECT *
FROM sales s
JOIN targets t
    ON  s.store_id   = t.store_id
    AND s.product_id = t.product_id;
```

Both conditions must match. This is called a **composite join condition**.

---

## 📊 20. JOIN + CASE for Business Segmentation

```sql
SELECT
    c.customer_name,
    SUM(o.amount) AS total_spending,
    CASE
        WHEN SUM(o.amount) >= 100000 THEN 'VIP'
        WHEN SUM(o.amount) >=  50000 THEN 'Premium'
        ELSE 'Standard'
    END AS customer_segment
FROM customers c
JOIN orders o
    ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name;
```

---

## 📈 21. Example: Revenue by Product Category

```sql
SELECT
    p.category,
    SUM(oi.quantity * oi.unit_price) AS revenue
FROM order_items oi
JOIN products p
    ON oi.product_id = p.product_id
GROUP BY p.category
ORDER BY revenue DESC;
```

Notice that we're calculating revenue at the **order-item grain**, where the required fields exist.

---

## 🎯 22. Join Only What You Need

A common beginner mistake is joining every available table.

Ask: «What business question am I answering?» Then join only the tables necessary to answer it.

Fewer unnecessary JOINs can mean:

- simpler queries
- fewer opportunities for row multiplication
- easier debugging
- better performance

---

## ⚡ 23. JOIN Performance Basics

- **Select only required columns** instead of `SELECT *`
- **Filter data appropriately:** `WHERE o.order_date >= '2026-01-01'`
- **Join using appropriate keys:** `ON c.customer_id = o.customer_id`
- **Understand indexes:** indexes on commonly joined or filtered columns can improve performance

---

## 🧠 24. The Most Important JOIN Checklist

- 1️⃣ **What is the grain of each table?** `1 row = ?`
- 2️⃣ **What is the relationship?** `1:1`, `1:Many`, `Many:Many`
- 3️⃣ **Which column connects them?** `customer_id`? `order_id`? `product_id`?
- 4️⃣ **Will rows multiply?**
- 5️⃣ **Am I going to `SUM` or `COUNT` after the JOIN?**
- 6️⃣ **Could this cause double counting?**
- 7️⃣ **Do I need all the tables?**

---

## 🎤 SQL Interview Questions

**Q1. What is table grain?**

It describes what one row represents in a table.

**Q2. What is row multiplication?**

When a JOIN causes one row to match multiple rows, producing multiple output rows.

**Q3. What is a many-to-many relationship?**

A relationship where multiple rows in one table can relate to multiple rows in another table.

**Q4. How can JOINs cause incorrect revenue?**

If a transaction-level value is repeated because of a one-to-many JOIN, `SUM()` may count the same transaction multiple times.

**Q5. What is pre-aggregation?**

Aggregating a detailed table to the required grain before joining it to another table.

**Q6. How do you find orphan records?**

```sql
SELECT o.*
FROM orders o
LEFT JOIN customers c
    ON o.customer_id = c.customer_id
WHERE c.customer_id IS NULL;
```

**Q7. When can EXISTS be preferable to JOIN?**

When you only need to know whether a matching record exists and don't need to retrieve columns from the other table.

**Q8. What is a junction table?**

A table used to represent relationships between entities, commonly used for many-to-many relationships.

**Q9. Why is `SELECT DISTINCT` not always a proper solution to duplicate rows?**

Because it may hide the underlying grain or JOIN problem rather than fixing the cause of duplication.

**Q10. What should you determine before joining two tables?**

Their grain, relationship, join keys, and how the JOIN will affect row counts and aggregations.

---

💡 **Double Tap ❤️ For More**
