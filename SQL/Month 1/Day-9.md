# 🚀 SQL Roadmap 2026 — Part 9

## 🔤 SQL String Functions — Cleaning & Transforming Text Data

In real-world databases, a huge amount of information is stored as **text**:

- Customer names
- Email addresses
- Phone numbers
- Product names
- Cities
- Categories
- Addresses
- Job titles

But text data is rarely perfectly clean. You may encounter:

```text
'  Alice  '
'alice@example.com'
'ALICE@EXAMPLE.COM'
'Premium Customer'
'  Mumbai'
```

> SQL string functions allow you to **clean, search, extract, combine, and transform** text directly inside your queries.

---

## 🧠 1. What Are String Functions?

String functions are SQL functions that operate on **text values**.

Common functions include:

| | | |
| --- | --- | --- |
| `LENGTH()` | `UPPER()` | `LOWER()` |
| `TRIM()` | `LTRIM()` | `RTRIM()` |
| `SUBSTRING()` | `LEFT()` | `RIGHT()` |
| `CONCAT()` | `REPLACE()` | `POSITION()` |
| `CHAR_LENGTH()` | | |

> ⚠️ Exact function names and syntax can vary slightly between databases such as PostgreSQL, MySQL, SQL Server, and Oracle.

---

## 🔠 2. UPPER()

Converts text to **uppercase**.

```sql
SELECT
    customer_name,
    UPPER(customer_name) AS uppercase_name
FROM customers;
```

**Example:** `Alice` becomes `ALICE`

**Useful for:**

- Standardizing text
- Case-insensitive comparisons
- Creating reports
- Data cleaning

---

## 🔡 3. LOWER()

Converts text to **lowercase**.

```sql
SELECT
    LOWER(email) AS email
FROM customers;
```

**Example:** `ALICE@EXAMPLE.COM` becomes `alice@example.com`

A common data-cleaning pattern is:

```sql
SELECT
    LOWER(TRIM(email)) AS cleaned_email
FROM customers;
```

> This handles both unnecessary spaces **and** inconsistent capitalization.

---

## 🧹 4. TRIM()

Removes **leading and trailing** spaces.

```sql
SELECT
    TRIM(customer_name) AS cleaned_name
FROM customers;
```

For example `'   Alice   '` becomes `'Alice'`

> This is extremely useful when importing data from Excel, CSV files, APIs, and external systems.

---

## ↩️ 5. LTRIM() and RTRIM()

| Function | Removes spaces from | Example |
| -------- | ------------------- | ------- |
| `LTRIM()` | the **beginning** | `SELECT LTRIM(customer_name) FROM customers;` |
| `RTRIM()` | the **end** | `SELECT RTRIM(customer_name) FROM customers;` |
| `TRIM()` | **both sides** (generally) | `SELECT TRIM(customer_name) FROM customers;` |

---

## 📏 6. LENGTH()

Returns the **number of characters** in a string.

```sql
SELECT
    customer_name,
    LENGTH(customer_name) AS name_length
FROM customers;
```

**Example:**

```text
Alice  → 5
Robert → 6
```

> ⚠️ Function behavior can vary across SQL dialects, particularly with multibyte characters.

---

## 🔍 7. Finding Long or Short Values

String length can be useful for **data-quality checks**.

```sql
-- Can help identify potentially invalid phone numbers
SELECT * FROM customers WHERE LENGTH(phone) < 10;

-- Can identify unusually long product descriptions
SELECT * FROM products WHERE LENGTH(product_name) > 100;
```

---

## ✂️ 8. SUBSTRING()

`SUBSTRING()` extracts **part** of a string. A common form is:

```sql
SUBSTRING(column_name, start_position, length)
```

```sql
SELECT SUBSTRING(customer_name, 1, 3) AS first_three_characters
FROM customers;
```

- For `Alexander` the result would be `Ale`
- ⚠️ Syntax differs by database — always check the dialect you're using

---

## 👈 9. LEFT()

Returns characters from the **beginning** of a string.

```sql
SELECT LEFT(product_code, 3) AS category_code
FROM products;
```

- If `product_code = 'ELE12345'` → Result: `ELE`
- This can be useful when codes contain meaningful **prefixes**.

---

## 👉 10. RIGHT()

Returns characters from the **end** of a string.

```sql
SELECT RIGHT(account_number, 4) AS last_four_digits
FROM accounts;
```

- Example: `1234567890` → Result: `7890`
- This is commonly useful for reporting or identifying records **without displaying the complete identifier**.

---

## 🔗 11. CONCAT()

Combines multiple strings.

```sql
SELECT CONCAT(first_name, ' ', last_name) AS full_name
FROM customers;
```

**Example:** `first_name = 'Alice'`, `last_name = 'Smith'` → Result: `Alice Smith`

---

## ⚠️ 12. CONCAT vs `+` Operator

Some SQL dialects allow string concatenation using operators:

```sql
first_name +  ' ' +  last_name   -- some dialects
first_name || ' ' || last_name   -- others
```

> `CONCAT()` provides a more **portable and readable** approach, although `NULL` behavior can still vary by database.

---

## 🔄 13. REPLACE()

Replaces one piece of text with another.

```sql
SELECT REPLACE(phone, '-', '') AS cleaned_phone
FROM customers;
```

**Example:** `987-654-3210` becomes `9876543210`

```sql
SELECT REPLACE(product_name, 'Old', 'New') AS updated_name
FROM products;
```

---

## 📧 14. Extracting Information from Email Addresses

Suppose `email = 'alice@gmail.com'` and you want to identify the **domain**. One approach is database-specific string manipulation.

For example, in **PostgreSQL**:

```sql
SELECT SPLIT_PART(email, '@', 2) AS email_domain
FROM customers;
```

Result: `gmail.com`

**This is useful for:**

- customer segmentation
- domain analysis
- corporate vs personal email analysis
- detecting invalid domains

---

## 📊 15. Grouping Customers by Email Domain

Once you extract the domain, you can aggregate it.

```sql
SELECT
    SPLIT_PART(LOWER(TRIM(email)), '@', 2) AS email_domain,
    COUNT(*)                               AS customer_count
FROM customers
WHERE email IS NOT NULL
GROUP BY SPLIT_PART(LOWER(TRIM(email)), '@', 2)
ORDER BY customer_count DESC;
```

This combines several concepts:

```text
TRIM() → LOWER() → SPLIT_PART() → GROUP BY → COUNT() → ORDER BY
```

> This is much closer to **real-world analytics work**.

---

## 🔎 16. POSITION()

`POSITION()` finds **where** a substring occurs.

```sql
SELECT POSITION('@' IN email) AS at_position
FROM customers;
```

- For `alice@gmail.com` it returns the position of `@`
- This can help identify whether a string contains a particular character.

---

## 🧪 17. String Functions for Data Validation

Suppose you want to identify potentially invalid emails.

```sql
SELECT *
FROM customers
WHERE email IS NOT NULL
  AND POSITION('@' IN email) = 0;
```

- This doesn't **prove** an email is valid, but it can identify obviously problematic records.
- For serious validation, application-level validation or dedicated data-quality tools may be more appropriate.

---

## 🏷️ 18. Standardizing Categories

Suppose your database contains `Premium`, `premium`, `PREMIUM`, `  Premium` — these may represent the **same** business category.

You can standardize them:

```sql
SELECT UPPER(TRIM(customer_type)) AS standardized_type
FROM customers;
```

- Now they all become `PREMIUM`
- This is particularly useful **before grouping**.

---

## 📈 19. String Functions + GROUP BY

**Without cleaning:**

```sql
SELECT customer_type, COUNT(*) AS customer_count
FROM customers
GROUP BY customer_type;
```

You might get **separate groups** for `Premium`, `premium`, `PREMIUM`.

**Instead:**

```sql
SELECT
    UPPER(TRIM(customer_type)) AS customer_type,
    COUNT(*)                   AS customer_count
FROM customers
GROUP BY UPPER(TRIM(customer_type));
```

> Now logically equivalent values can be grouped together.

---

## 🧹 20. Cleaning Product Names

Suppose product names contain unnecessary spaces and inconsistent capitalization.

```sql
SELECT UPPER(TRIM(product_name)) AS cleaned_product_name
FROM products;
```

You can also remove unwanted characters:

```sql
SELECT REPLACE(TRIM(product_name), '-', ' ') AS cleaned_product_name
FROM products;
```

**Example:** `'  wireless-earbuds  '` can become `wireless earbuds`

---

## 💼 21. Real-World Business Example

Suppose an e-commerce company stores customer names inconsistently:

```text
' alice ', 'ALICE', 'Alice', ' alice'
```

You can create a normalized version:

```sql
SELECT UPPER(TRIM(customer_name)) AS normalized_name
FROM customers;
```

This produces: `ALICE`, `ALICE`, `ALICE`, `ALICE`

- Now the cleaned value can be used for analysis or as part of a data-matching strategy.
- ⚠️ String normalization alone does **not** guarantee that two records represent the same person.

---

## 🧩 22. Combining Multiple String Functions

SQL becomes particularly powerful when functions are **combined**.

```sql
SELECT UPPER(TRIM(customer_name)) AS cleaned_name  FROM customers;
SELECT LOWER(TRIM(email))         AS cleaned_email FROM customers;
```

Think of it as a **pipeline**:

```text
Raw Data → TRIM() → LOWER()/UPPER() → REPLACE() → Clean Data
```

---

## ⚠️ 23. Common Mistakes

**Mistake 1 — Ignoring spaces**

`'Alice'` and `' Alice'` may behave as different values depending on the database and comparison context. Use `TRIM(customer_name)` when appropriate.

**Mistake 2 — Ignoring capitalization**

`Premium`, `premium`, `PREMIUM` can create inconsistent groups. Use `UPPER(TRIM(customer_type))` when the business meaning is case-insensitive.

**Mistake 3 — Assuming all databases use the same syntax**

String functions differ between PostgreSQL, MySQL, SQL Server, and Oracle. Always verify the syntax for your SQL dialect.

**Mistake 4 — Modifying data unnecessarily**

There is a difference between `SELECT TRIM(name)` and actually **updating** the stored value. Always understand whether you're transforming data for analysis or permanently modifying the database.

---

## 🎤 SQL Interview Questions

**Q1. What is the purpose of string functions?**

They are used to manipulate, clean, transform, search, and extract text data.

**Q2. What does `TRIM()` do?**

It removes leading and trailing spaces from a string.

**Q3. Difference between `UPPER()` and `LOWER()`?**

`UPPER()` converts text to uppercase; `LOWER()` converts text to lowercase.

**Q4. What does `CONCAT()` do?**

It combines multiple strings into one value.

**Q5. What does `REPLACE()` do?**

It replaces occurrences of one substring with another.

**Q6. How can you find the length of a string?**

Commonly `LENGTH(column_name)` or, depending on the database, `CHAR_LENGTH(column_name)`.

**Q7. How would you standardize customer categories?**

For example `UPPER(TRIM(customer_type))` — this removes surrounding spaces and standardizes capitalization.

**Q8. How can you extract the last four characters of a value?**

In databases supporting it: `RIGHT(column_name, 4)`

**Q9. How can you combine first and last names?**

`CONCAT(first_name, ' ', last_name)`

**Q10. Why are string functions important for data analysts?**

Because real-world text data often contains inconsistent capitalization, spaces, formats, prefixes, suffixes, and unwanted characters.

---

## 📝 Practice Questions

**Practice 1:** Convert customer names to uppercase.

```sql
SELECT UPPER(customer_name) AS customer_name FROM customers;
```

**Practice 2:** Remove unnecessary spaces from product names.

```sql
SELECT TRIM(product_name) AS product_name FROM products;
```

**Practice 3:** Create a full name from first and last name.

```sql
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM customers;
```

**Practice 4:** Remove hyphens from phone numbers.

```sql
SELECT REPLACE(phone, '-', '') AS cleaned_phone FROM customers;
```

**Practice 5:** Find products whose names contain more than 50 characters.

```sql
SELECT * FROM products WHERE LENGTH(product_name) > 50;
```

---

## 🧪 Mini SQL Challenge

You have this table:

```text
customers: customer_id, first_name, last_name, email, customer_type, phone
```

Write a query that returns:

- Customer ID
- Cleaned full name
- Cleaned lowercase email
- Standardized customer type
- Phone number without hyphens

**Solution:**

```sql
SELECT
    customer_id,
    CONCAT(TRIM(first_name), ' ', TRIM(last_name)) AS full_name,
    LOWER(TRIM(email))                             AS cleaned_email,
    UPPER(TRIM(customer_type))                     AS customer_type,
    REPLACE(TRIM(phone), '-', '')                  AS cleaned_phone
FROM customers;
```

> This single query demonstrates a practical **data-cleaning workflow** using several string functions.

---

📌 💡 **Double Tap ❤️ For More**
