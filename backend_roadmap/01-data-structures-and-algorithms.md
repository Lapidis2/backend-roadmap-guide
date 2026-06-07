# Lesson 01 — Data Structures & Algorithms (DSA)

> **Where this fits:** Before we touch databases, APIs, or frameworks, we need to understand how to *store* and *organize* data in memory, and how to *operate* on it efficiently. This is the bedrock of every backend system — and the most common warm-up topic in interviews.

---

## 1. What is a Data Structure, really?

A **data structure** is just a *way of organizing data* so that you can use it efficiently for a particular job.

Think of it like furniture in a kitchen:
- A **drawer** (array/list) is great when you want to grab the 3rd item quickly, or go through everything in order.
- A **labeled spice rack** (hash map / dictionary) is great when you want to grab "the cinnamon" by name, instantly, without checking every jar.

Neither is "better" — they're suited to different tasks. **Choosing the right one is the actual skill.**

An **algorithm** is simply a *step-by-step procedure* for solving a problem — e.g., "how do I find the largest number in this list?" or "how do I check if this username already exists?"

---

## 2. Arrays / Lists

### What it is
An **array** (called a `list` in Python) is an ordered collection of items, stored so that each item can be accessed by its **index** (its position number, starting at 0).

```python
fruits = ["apple", "banana", "cherry"]

print(fruits[0])  # "apple"  -> index 0 is the first item
print(fruits[2])  # "cherry" -> index 2 is the third item
```

**Line-by-line explanation:**
- `fruits = ["apple", "banana", "cherry"]` — creates a list with three string elements. Python stores them in order, side-by-side conceptually.
- `fruits[0]` — square brackets access an element by its index. Index `0` is always the *first* element (this trips up many beginners coming from human counting, where we'd call "apple" the "1st" item).
- `fruits[2]` — index `2` is the *third* element, `"cherry"`.

### Why it matters in backend
Arrays/lists show up constantly:
- A list of user objects returned from a database query
- A list of items in a shopping cart
- A list of log entries to process in order

### Strengths
- **Fast access by position**: `fruits[1]` is instant — O(1) (we'll explain this notation in Lesson 02).
- **Preserves order** — the order you insert is the order you get back.
- **Simple to iterate** — looping through every item is straightforward and fast.

### Weaknesses
- **Slow to search by value**: "Does this list contain `'mango'`?" requires checking each element one by one, in the worst case checking *all* of them — O(n).
- **Slow to insert/remove from the middle or front**: everything after that position has to shift over.

### Real backend example
Imagine an endpoint that returns the 10 most recent orders for a user:

```python
recent_orders = ["order_101", "order_102", "order_103"]

for order_id in recent_orders:
    print(f"Processing {order_id}")
```
Here, **order matters** (most recent first) and we process every item — a perfect job for a list.

---

## 3. Hash Maps (Dictionaries)

### What it is
A **hash map** (called a `dict` in Python, sometimes "dictionary" or "object" in other languages) stores data as **key–value pairs**. Instead of looking something up by position, you look it up by a meaningful **key**.

```python
user = {
    "id": 42,
    "username": "amina_dev",
    "email": "amina@example.com"
}

print(user["username"])  # "amina_dev"
```

**Line-by-line explanation:**
- `user = { ... }` — creates a dictionary. Each entry is a `key: value` pair separated by commas.
- `"id": 42` — the key `"id"` maps to the value `42`.
- `user["username"]` — instead of an index number, we use the *key* `"username"` to instantly retrieve its value, `"amina_dev"`.

### How it works (just enough to sound smart in an interview)
A hash map uses a **hash function** to convert a key into a number, which tells it *exactly* where to store (or find) the value — without scanning anything. That's why lookup is (on average) **instant**, regardless of how many items are in the map.

You don't need to implement a hash function for an interview — you just need to be able to say:
> "A hash map converts the key into a memory address using a hash function, so lookup, insertion, and deletion are all O(1) on average."

### Strengths
- **Very fast lookup, insertion, and deletion by key** — O(1) on average.
- **Self-documenting** — `user["email"]` is much clearer than `user[2]`.

### Weaknesses
- **No guaranteed order** (in many languages — Python dicts *do* preserve insertion order since 3.7, but you shouldn't rely on dict order for business logic).
- **Slow to search by value** instead of by key — that still requires scanning.
- Slightly more memory overhead than a plain array.

### Real backend example
Imagine you need to check whether a username is already taken, and you have 100,000 users:

```python
# Slow approach: O(n) — checks every user, one by one
existing_usernames_list = ["amina_dev", "ben_codes", "zee123", ...]
if "amina_dev" in existing_usernames_list:
    print("Username taken!")

# Fast approach: O(1) — instant lookup
existing_usernames_set = {"amina_dev": True, "ben_codes": True, "zee123": True}
if "amina_dev" in existing_usernames_set:
    print("Username taken!")
```

The second version doesn't get slower as your user base grows from 100 to 100 million. **This is exactly the kind of design decision interviewers want to hear you reason about.**

> 💡 In Python, a `set` is essentially a hash map that only stores keys (no values) — also O(1) lookup. Knowing when to reach for a `set` vs a `dict` vs a `list` is a strong signal of backend maturity.

---

## 4. Why choose one over another?

| Need | Best choice | Why |
|------|------------|-----|
| Keep things in a specific order | List/Array | Preserves insertion order, fast iteration |
| Access the 5th item directly | List/Array | O(1) index access |
| Look something up by a unique identifier (username, ID, email) | Hash Map / Dict | O(1) average lookup by key |
| Check if something exists in a large collection | Set / Hash Map | O(1) average membership check |
| Store related fields about one entity (a user, a product) | Dict / Object | Keys describe meaning, easy to extend |

**The interview-winning phrase:** *"It depends on the access pattern — how will this data be read and written?"* That single sentence shows you think like an engineer, not a coder who memorized syntax.

---

## 5. Common Beginner Mistakes

1. **Using a list to check membership repeatedly.**
   ```python
   # 🚫 Bad: O(n) check inside a loop = O(n²) overall
   for user_id in incoming_ids:
       if user_id in all_user_ids_list:   # scans the whole list each time
           ...
   ```
   ```python
   # ✅ Good: convert to a set first, O(1) checks
   all_user_ids_set = set(all_user_ids_list)
   for user_id in incoming_ids:
       if user_id in all_user_ids_set:
           ...
   ```

2. **Forgetting that dictionary keys must be unique.** Adding a second value with the same key *overwrites* the first — it doesn't create a second entry.

3. **Assuming index 1 is the "first" item.** In almost every backend language, indexing starts at `0`.

4. **Mutating a list while iterating over it** — this causes skipped elements or crashes. (Iterate over a copy, or build a new list instead.)

5. **Confusing "ordered" with "sorted."** A list preserves the order you put things in — it does **not** automatically sort them.

---

## 6. Likely Interview Questions & Strong Answers

**Q: What's the difference between a list and a dictionary, and when would you use each?**

> *"A list is an ordered collection best suited for when I care about sequence or need to process every item — like a feed of recent activity. A dictionary maps keys to values and is best when I need to look something up by a unique identifier — like finding a user by their ID. I'd choose based on how the data will be accessed: by position/order, or by a meaningful key."*

**Q: How would you check if a value exists in a large collection efficiently?**

> *"I'd convert the collection to a set or use a dictionary, because membership checks are O(1) on average — versus O(n) for scanning a list. This becomes critical as the collection grows; a list-based check that's 'fine' at 100 items can become a real bottleneck at 100,000."*

**Q: Can you give an example of a real backend scenario where the wrong data structure caused a performance problem?**

> *"A common one is checking 'has this email already registered?' against a list of all emails inside a loop — that's an O(n) check repeated for every signup, so as the user base grows, signups get slower and slower. Switching that lookup list to a set (or, more realistically, an indexed database column) makes the check constant-time regardless of how many users exist."*

---

## 7. Tips From a Senior Interviewer

- **Don't jump to code immediately.** Say out loud what data structure you're considering and *why* before writing anything. Interviewers are listening for your reasoning process more than your syntax.
- **It's okay to say "I'd start with a list because it's simplest, but if lookups become a bottleneck, I'd switch to a hash map."** This shows you understand trade-offs *and* that you don't over-engineer prematurely.
- **Relate it back to a real system whenever you can** — "this is like checking if a username is taken" lands far better than reciting a textbook definition.

---

## 8. Summary

- **Arrays/Lists** are ordered collections, great for sequence and iteration, but slow for searching by value.
- **Hash Maps/Dictionaries** map keys to values, giving near-instant lookup, insertion, and deletion by key.
- **Sets** are like dictionaries with only keys — perfect for fast membership checks.
- The right choice depends entirely on **how the data will be accessed** — this is the question to ask yourself (and to voice out loud in interviews).
- Mastering *when* to use which structure — not just *how* — is what separates a junior who passes from one who doesn't.

**Next up →** [Lesson 02: Time & Space Complexity](02-time-complexity.md), where we put precise language (Big-O) around the "fast" and "slow" claims we made in this lesson.
