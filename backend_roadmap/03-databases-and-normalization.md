# Lesson 03 — Databases & Normalization

> **Where this fits:** Backend systems exist largely to manage data — accepting it, storing it, retrieving it, and keeping it consistent. **Relational databases** are the most common long-term storage mechanism, and **normalization** is the discipline of organizing that data so it stays correct and efficient as your application grows.

---

## 1. What is a backend, anyway? (A quick grounding)

Before databases make sense, let's ground the bigger picture in one paragraph:

A **backend** is the part of an application that runs on a server (not on the user's device) and is responsible for business logic, data storage, authentication, and communicating with other systems. When you tap "Buy Now" on a shopping app, your phone (the **frontend**) sends a request to a server (the **backend**), which checks your account, talks to a **database**, processes payment, and sends a response back. The database is where all of that persistent information — users, products, orders — actually lives between requests.

---

## 2. What is a relational database?

A **relational database** stores data in **tables** — grids of rows and columns, similar to a spreadsheet — and lets you define **relationships** between those tables (we'll dig into relationships in Lesson 04).

Examples you'll hear in interviews: **PostgreSQL, MySQL, SQLite, SQL Server**. They're called "relational" because data in one table can *relate* to data in another (e.g., an order *relates* to the user who placed it).

### Tables, Rows, and Columns

Imagine a table called `users`:

| id | username   | email                  | created_at |
|----|------------|------------------------|------------|
| 1  | amina_dev  | amina@example.com      | 2026-01-04 |
| 2  | ben_codes  | ben@example.com        | 2026-02-11 |
| 3  | zee123     | zee@example.com        | 2026-03-02 |

- **Table**: the whole grid — `users` — represents *one type of entity*.
- **Column** (a.k.a. **field** or **attribute**): a vertical slice describing *one property* shared by every row — e.g., `email`. Every row has a value (or `NULL`) for every column.
- **Row** (a.k.a. **record**): a horizontal slice representing *one specific instance* — e.g., the user "amina_dev" is one row.

A simple SQL query to fetch this data:

```sql
SELECT id, username, email
FROM users
WHERE username = 'amina_dev';
```

**Line-by-line explanation:**
- `SELECT id, username, email` — choose which columns to return (don't grab everything if you don't need it — this matters for performance).
- `FROM users` — specifies which table to read from.
- `WHERE username = 'amina_dev'` — filters rows, returning only the one(s) matching this condition.

### Real backend example
When a user logs in, your backend runs something conceptually like:
```sql
SELECT id, password_hash FROM users WHERE email = 'amina@example.com';
```
It then checks whether the submitted password matches the stored (hashed) password. This single, simple-looking query is the backbone of nearly every authentication system you'll ever build.

---

## 3. Why organize data carefully? The problem normalization solves

Imagine, instead of separate tables, you stored *everything* about an order in one giant table:

| order_id | customer_name | customer_email     | product_name | product_price |
|----------|---------------|--------------------|--------------|---------------|
| 1001     | Amina Uwase   | amina@example.com  | Laptop       | 1200          |
| 1002     | Amina Uwase   | amina@example.com  | Mouse        | 25            |
| 1003     | Ben Mugisha   | ben@example.com    | Laptop       | 1200          |

Spot the problems?
- **Repetition**: "Amina Uwase" and her email are duplicated across every order she places.
- **Update anomalies**: If Amina changes her email, you must update it in *every row* she appears in — miss one, and now your data disagrees with itself.
- **Wasted space**: The same strings ("Laptop", 1200) are stored over and over.
- **Insert/delete anomalies**: You can't store a new product until someone orders it; if you delete the only order for a product, you lose all information about that product too.

**Normalization** is the process of structuring tables to eliminate this kind of redundancy and these anomalies — typically by splitting one big table into several smaller, focused tables connected by relationships.

---

## 4. Normal Forms — explained simply

You'll often hear **1NF, 2NF, 3NF** thrown around. Here's what they actually mean, in plain English (not textbook jargon):

### 1NF — First Normal Form
**Rule: Each cell holds a single, indivisible value. No lists or repeating groups crammed into one field.**

```
🚫 Bad (violates 1NF):
| order_id | products              |
|----------|-----------------------|
| 1001     | Laptop, Mouse, Bag    |

✅ Good (1NF — one product per row):
| order_id | product   |
|----------|-----------|
| 1001     | Laptop    |
| 1001     | Mouse     |
| 1001     | Bag       |
```
**Why it matters:** If "products" is one comma-separated string, you cannot easily search for "all orders containing a Mouse," count products, or update just one of them. Splitting into rows makes the data queryable.

### 2NF — Second Normal Form
**Rule: Meet 1NF, *and* every non-key column must depend on the *whole* primary key — not just part of it.** (This becomes relevant with composite keys — keys made of more than one column.)

```
🚫 Bad (violates 2NF):
order_items(order_id, product_id, product_name, quantity)
   -- product_name depends only on product_id, not on the combination
   -- (order_id, product_id) — so it doesn't belong here.

✅ Good:
order_items(order_id, product_id, quantity)
products(product_id, product_name)
```
**Why it matters:** Storing `product_name` inside `order_items` means it's duplicated for every order containing that product — and if the product is renamed, you'd have to update every single order row.

### 3NF — Third Normal Form
**Rule: Meet 2NF, *and* no column should depend on another *non-key* column** (this is called eliminating "transitive dependencies").

```
🚫 Bad (violates 3NF):
employees(employee_id, department_id, department_name)
   -- department_name depends on department_id, not directly on employee_id

✅ Good:
employees(employee_id, department_id)
departments(department_id, department_name)
```
**Why it matters:** If `department_name` lives inside the `employees` table, renaming a department means updating every employee row in that department. Move it to its own `departments` table, and you update it **once**.

### The simple mental model
> "Each fact should be stored in exactly one place. If you ever find yourself needing to update the *same* piece of information in multiple rows to keep your data consistent, it's a sign your tables need to be split further."

---

## 5. Why normalization matters (the "so what")

- **Data integrity**: A fact lives in one place, so it can't silently disagree with itself.
- **Easier updates**: Change a product's price once — every order referencing it sees the new price (or, if you need historical accuracy, you store a price *snapshot* at order time — a deliberate design choice, not an accident).
- **Smaller storage footprint**: No duplicated strings bloating your tables.
- **Clearer modeling**: Separate tables for separate concepts (`users`, `products`, `orders`) make the system easier to reason about and extend.

### A quick, honest caveat (this is what separates juniors from seniors)
Normalization isn't free — splitting data into many tables means you often need to **join** them back together when reading data, which has its own performance cost. In some read-heavy systems, engineers deliberately **denormalize** (reintroduce some redundancy) to make reads faster. **The senior answer isn't "always normalize" — it's "normalize by default for integrity, and denormalize deliberately, with a clear reason, when read performance demands it."**

---

## 6. Common Beginner Mistakes

1. **Cramming multiple values into one column** (e.g., `tags: "sale,electronics,featured"`) — making search, filtering, and updates painful.
2. **Duplicating descriptive data** (names, prices, addresses) across rows "to make queries simpler" — this is exactly what normalization warns against; use relationships instead (Lesson 04).
3. **Treating normalization as an all-or-nothing religious rule.** In real systems, deliberate denormalization for performance is common and valid — the key word is *deliberate*.
4. **Confusing a table with the data it stores.** A table is a *structure* (the columns/schema); rows are the actual *data* living inside that structure.

---

## 7. Likely Interview Questions & Strong Answers

**Q: What is database normalization, and why does it matter?**

> *"Normalization is organizing data into tables so that each fact is stored in exactly one place, which prevents inconsistencies and reduces redundancy. For example, instead of repeating a customer's email on every order row, I'd store customers in their own table and reference them by ID — so updating an email happens in one place and is instantly reflected everywhere it's needed."*

**Q: Can you explain 1NF, 2NF, and 3NF in simple terms?**

> *"1NF means each field holds a single value — no comma-separated lists crammed into one column. 2NF means every column depends on the *entire* primary key, not just part of it — relevant when keys are made of multiple columns. 3NF means no column depends on another non-key column — for example, a department's name shouldn't live inside the employees table, because it depends on the department, not the employee."*

**Q: Is it ever okay to *not* fully normalize a database?**

> *"Yes — deliberate denormalization is a real, valid technique, usually applied to optimize read-heavy workloads by avoiding expensive joins. The key is that it should be a conscious trade-off you can explain — for example, storing a snapshot of a product's price on an order so historical orders remain accurate even if the product's price changes later — not an accident from not understanding relationships."*

---

## 8. Tips From a Senior Interviewer

- **Use a concrete example, every time.** "Imagine an e-commerce order..." is far more convincing than reciting "1NF means atomic values."
- **Show you understand trade-offs, not just rules.** Mentioning *when* denormalization makes sense signals real-world maturity well beyond "junior who memorized the normal forms."
- **If you forget the exact NF number, describe the *problem* it solves instead.** "I don't remember if this is 2NF or 3NF exactly, but the issue here is that this column depends on something other than the full primary key, which could cause update inconsistencies" — that's a *strong* answer that shows understanding over memorization (see Lesson 13 for more on this).

---

## 9. Summary

- A **relational database** organizes data into **tables** (entities), made of **rows** (records) and **columns** (fields/attributes).
- **Normalization** restructures tables to eliminate redundancy and anomalies — the guiding idea is "each fact lives in exactly one place."
- **1NF**: atomic values. **2NF**: depend on the whole key. **3NF**: no dependency on non-key columns.
- Normalization isn't dogma — **deliberate denormalization** for performance is a valid, senior-level trade-off, as long as it's a conscious choice with a clear justification.

**Next up →** [Lesson 04: Keys & Relationships](04-keys-and-relationships.md), where we connect these tables together — which is what makes a *relational* database relational.
