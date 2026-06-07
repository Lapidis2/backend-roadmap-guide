# Lesson 04 — Keys & Relationships

> **Where this fits:** In Lesson 03, we split data into separate, focused tables to avoid redundancy. But split data is useless if you can't reconnect it. **Keys** are how individual rows are uniquely identified, and **relationships** are how tables reference each other — together, they're what makes a *relational* database actually relational.

---

## 1. Primary Keys

### What it is
A **primary key** is a column (or combination of columns) that **uniquely identifies each row** in a table. No two rows can share the same primary key value, and it can never be `NULL`.

```sql
CREATE TABLE users (
    id INTEGER PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE
);
```

**Line-by-line explanation:**
- `CREATE TABLE users (...)` — defines a new table named `users` with the listed columns.
- `id INTEGER PRIMARY KEY` — declares `id` as an integer column that uniquely identifies each row. Most databases can auto-generate this value for you on insert (e.g., `SERIAL` in PostgreSQL, `AUTO_INCREMENT` in MySQL).
- `username VARCHAR(50) NOT NULL` — a text column (max 50 characters) that cannot be left empty.
- `email VARCHAR(100) NOT NULL UNIQUE` — a text column that cannot be empty *and* cannot be duplicated across rows (a separate constraint from being the primary key).

### Why it matters
Every other table that needs to reference "this specific user" does so via their `id` — not their name (names change and aren't guaranteed unique) and not their position in the table (rows can be reordered, deleted, etc.). The primary key is the one stable, unique "handle" for a row.

> 💡 **Why use a numeric `id` instead of, say, the email as the primary key?** Numeric IDs are smaller, faster to compare and index, and never need to change — even if the user updates their email. This is a classic "why this, not that" interview moment (see Lesson 13).

---

## 2. Foreign Keys

### What it is
A **foreign key** is a column in one table that stores the primary key value of a row in *another* table — creating a link between them.

```sql
CREATE TABLE orders (
    id INTEGER PRIMARY KEY,
    user_id INTEGER NOT NULL,
    total_amount DECIMAL(10, 2) NOT NULL,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

**Line-by-line explanation:**
- `user_id INTEGER NOT NULL` — stores the ID of the user who placed this order. Every order *must* belong to a user.
- `FOREIGN KEY (user_id) REFERENCES users(id)` — tells the database: "the value in `user_id` must correspond to an existing `id` in the `users` table." This is called **referential integrity** — the database itself enforces that you can't create an order pointing to a user that doesn't exist.

### Real backend example
```sql
-- Find all orders placed by the user with id = 1
SELECT orders.id, orders.total_amount
FROM orders
WHERE orders.user_id = 1;
```
This is the foundation of nearly every "show me X's stuff" feature — a user's orders, a post's comments, a project's tasks.

> 💡 **What happens if you try to delete a user who has orders?** By default, most databases will block the deletion (to protect referential integrity) unless you've defined a behavior like `ON DELETE CASCADE` (delete the orders too) or `ON DELETE SET NULL` (keep the orders, but clear the link). Knowing this exists — even without memorizing exact syntax — is a great signal in an interview.

---

## 3. The Three Relationship Types

This is one of the most commonly tested concepts — make sure you can both *explain* and *draw* (even just in words) each of these.

### One-to-One (1:1)
**Each row in Table A relates to exactly one row in Table B, and vice versa.**

Example: A `user` and their `profile` (bio, avatar, preferences) — kept in a separate table, perhaps because the profile data is large, optional, or rarely accessed alongside the core user record.

```sql
CREATE TABLE profiles (
    id INTEGER PRIMARY KEY,
    user_id INTEGER NOT NULL UNIQUE,   -- UNIQUE enforces the "one-to-one" rule
    bio TEXT,
    avatar_url VARCHAR(255),
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```
The `UNIQUE` constraint on `user_id` is what makes this 1:1 instead of 1:many — it guarantees no user can have *two* profile rows.

### One-to-Many (1:N) — by far the most common
**One row in Table A can relate to *many* rows in Table B, but each row in Table B relates to only *one* row in Table A.**

Example: One `user` can place *many* `orders`, but each `order` belongs to exactly *one* `user`. This is exactly the `orders` table we defined above — the foreign key lives on the "many" side.

```sql
-- One user, many orders
SELECT * FROM orders WHERE user_id = 1;
```

### Many-to-Many (N:N)
**Many rows in Table A can relate to many rows in Table B.**

Example: A `student` can enroll in many `courses`, and a `course` can have many `students`. Neither table can simply hold the other's foreign key (a student has *multiple* courses — which one would go in the column?). The solution is a **join table** (also called an associative or junction table) sitting between them:

```sql
CREATE TABLE enrollments (
    student_id INTEGER NOT NULL,
    course_id INTEGER NOT NULL,
    enrolled_at DATE NOT NULL,
    PRIMARY KEY (student_id, course_id),   -- a "composite" primary key
    FOREIGN KEY (student_id) REFERENCES students(id),
    FOREIGN KEY (course_id) REFERENCES courses(id)
);
```

**Line-by-line explanation:**
- `enrollments` exists *purely* to connect `students` and `courses` — it has no independent meaning of its own beyond representing "this student is enrolled in this course."
- `PRIMARY KEY (student_id, course_id)` — a **composite primary key**: the *combination* of these two columns must be unique. This naturally prevents the same student from enrolling in the same course twice.
- Two foreign keys link back to the two parent tables.

```sql
-- All courses a given student (id = 7) is enrolled in
SELECT courses.title
FROM courses
JOIN enrollments ON courses.id = enrollments.course_id
WHERE enrollments.student_id = 7;
```

---

## 4. Putting it together: a quick visual mental model

```
ONE-TO-ONE:        user  ───── profile
                    (1)         (1)

ONE-TO-MANY:       user  ─────< orders
                    (1)         (many)

MANY-TO-MANY:    students >──── enrollments ────< courses
                  (many)      (join table)       (many)
```

A simple rule of thumb for spotting which type you need:
> "Can the 'one' side legitimately have more than one of the 'other' thing at the same time? If yes on *both* sides, you need a join table (N:N). If yes on only one side, the foreign key goes on the 'many' side (1:N). If no on both sides, it's 1:1."

---

## 5. Common Beginner Mistakes

1. **Putting the foreign key on the wrong side of a 1:N relationship.** The foreign key always goes on the "many" side (each `order` has one `user_id` — not each `user` having a list of order IDs crammed into a column, which would violate 1NF from Lesson 03).

2. **Trying to model many-to-many without a join table** — usually by stuffing comma-separated IDs into a column (`course_ids: "3,7,12"`). This breaks normalization, makes queries painful, and prevents the database from enforcing integrity.

3. **Forgetting `NOT NULL` and `UNIQUE` constraints**, leaving the database unable to enforce rules your application logic assumes are always true. (If your code "guarantees" something but the database doesn't enforce it, a bug — or a different system touching the same database — *will* eventually violate that guarantee.)

4. **Confusing a primary key with "the first column"** — a primary key is defined by its *uniqueness and identity role*, not its position.

---

## 6. Likely Interview Questions & Strong Answers

**Q: What's the difference between a primary key and a foreign key?**

> *"A primary key uniquely identifies each row within its own table and can never be null or duplicated. A foreign key is a column that stores another table's primary key value, creating a link between the two — and the database uses it to enforce referential integrity, so you can't reference a row that doesn't exist."*

**Q: How would you model a many-to-many relationship, like students and courses?**

> *"Neither table can hold a single foreign key to the other, since each side can relate to multiple rows on the other side. I'd create a join table — say, `enrollments` — with foreign keys to both `students` and `courses`, and typically a composite primary key on those two columns to prevent duplicate enrollments. This also gives me a natural place to store relationship-specific data, like the enrollment date."*

**Q: Why use an auto-incrementing integer ID instead of something like an email as a primary key?**

> *"Integers are small, fast to compare and index, and never need to change. An email can change when a user updates their account, and propagating that change to every table referencing it would be costly and risky. Keeping a stable surrogate key (the ID) and enforcing uniqueness on the email *separately* gives us the best of both — stability for relationships, and uniqueness where the business actually needs it."*

---

## 7. Tips From a Senior Interviewer

- **Draw it out — even with words.** "So `users` has many `orders`, and the foreign key `user_id` lives on the `orders` table" said slowly and clearly demonstrates structured thinking far better than rushing to write SQL.
- **Always justify *where* the foreign key goes.** This single detail ("on the many side") is one of the most telling signals of whether someone *understands* relationships or has just memorized the term.
- **If asked to design a schema, start by naming the entities, then ask "how do these relate?" out loud before writing any SQL.** This mirrors how real schema design conversations happen on the job — and interviewers notice.

---

## 8. Summary

- A **primary key** uniquely identifies a row; a **foreign key** references another table's primary key, linking rows together.
- **One-to-one**: each row pairs with exactly one row elsewhere (enforced via a `UNIQUE` foreign key).
- **One-to-many**: the most common relationship — the foreign key lives on the "many" side.
- **Many-to-many**: requires a **join table** with foreign keys to both sides, often with a composite primary key.
- Constraints (`NOT NULL`, `UNIQUE`, `FOREIGN KEY`) aren't bureaucracy — they're the database actively protecting your data's correctness, even when application code has bugs.

**Next up →** [Lesson 05: ORM & the N+1 Query Problem](05-orm-and-n-plus-one.md), where we see how backend code actually talks to these related tables — and a notorious performance trap that lurks in the process.
