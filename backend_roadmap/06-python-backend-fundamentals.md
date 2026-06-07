# Lesson 06 — Python for Backend Fundamentals

> **Where this fits:** We've covered *what* backend systems need to do — store data, model relationships, query efficiently. Now let's look at *how* we write the code that does it, in a way that's clean, testable, and something a teammate (or future-you) can actually maintain. We'll use Python, since it's what FastAPI (Lesson 07) and most modern backend interviews are built around.

---

## 1. Clean Code Principles

"Clean code" isn't about being fancy — it's about writing code that's **easy for a human to read, understand, and safely change**. Your code is read far more often than it's written.

### Use descriptive names
```python
# 🚫 Unclear
def calc(a, b, c):
    return a * b * (1 - c)

# ✅ Clear — the names explain the purpose without needing a comment
def calculate_discounted_total(price, quantity, discount_rate):
    return price * quantity * (1 - discount_rate)
```
The second version tells you *what* it computes just by reading the signature — no need to dig into the implementation or hunt for a comment that might be outdated.

### Keep functions small and focused (Single Responsibility)
```python
# 🚫 One function doing three different jobs
def process_order(order):
    # validate
    if order.total <= 0:
        raise ValueError("Order total must be positive")
    # calculate tax
    order.tax = order.total * 0.18
    # send email
    send_email(order.user.email, "Your order is confirmed!")

# ✅ Each function has ONE clear job — easy to test, reuse, and reason about
def validate_order(order):
    if order.total <= 0:
        raise ValueError("Order total must be positive")

def apply_tax(order, tax_rate=0.18):
    order.tax = order.total * tax_rate

def send_order_confirmation(order):
    send_email(order.user.email, "Your order is confirmed!")

def process_order(order):
    validate_order(order)
    apply_tax(order)
    send_order_confirmation(order)
```
Now `process_order` reads almost like a checklist in plain English — and each piece can be tested and changed independently (this connects directly to Lesson 08 on testing).

### Avoid "magic numbers" and "magic strings"
```python
# 🚫 What does 0.18 mean? What if it changes?
order.tax = order.total * 0.18

# ✅ Named constant — self-documenting, and changeable in one place
VAT_RATE = 0.18
order.tax = order.total * VAT_RATE
```

### Don't Repeat Yourself (DRY) — but don't over-abstract either
If you find yourself copy-pasting the same logic in three places, extract it into a function. But resist the urge to build elaborate, "flexible" abstractions for things you don't actually need yet — that's **premature optimization of structure**, and it often makes code *harder* to follow, not easier. (We'll revisit this idea — match your solution to the actual problem — throughout the roadmap.)

---

## 2. Functions

A **function** is a named, reusable block of code that performs a task — optionally taking inputs (**parameters**) and producing an output (**return value**).

```python
def calculate_total_price(unit_price: float, quantity: int) -> float:
    """Return the total cost for a given quantity at a unit price."""
    return unit_price * quantity

total = calculate_total_price(unit_price=9.99, quantity=3)
print(total)  # 29.97
```

**Line-by-line explanation:**
- `def calculate_total_price(unit_price: float, quantity: int) -> float:` — defines a function named `calculate_total_price`. The `: float` and `: int` are **type hints** — they document (and let tools check) what types are expected; `-> float` documents the return type.
- The docstring (the triple-quoted string) explains *what* the function does — useful for both humans and auto-generated documentation.
- `return unit_price * quantity` — computes and returns the result; the function ends here.
- `calculate_total_price(unit_price=9.99, quantity=3)` — calling the function with **keyword arguments**, which makes the call self-documenting (you can tell what each value represents without checking the function definition).

### Why functions matter in backend
Nearly everything in a backend system — request handlers, validation rules, database queries, business calculations — is organized as functions (or methods, which are functions that belong to a class). Clear functions are the fundamental unit of a maintainable codebase.

---

## 3. Classes

A **class** is a blueprint for creating objects that bundle together **data** (attributes) and **behavior** (methods) that belong together conceptually.

```python
class ShoppingCart:
    def __init__(self, user_id: int):
        self.user_id = user_id
        self.items = []                 # starts empty

    def add_item(self, product_name: str, price: float):
        self.items.append({"product": product_name, "price": price})

    def total(self) -> float:
        return sum(item["price"] for item in self.items)


cart = ShoppingCart(user_id=42)
cart.add_item("Laptop", 1200.00)
cart.add_item("Mouse", 25.00)
print(cart.total())  # 1225.0
```

**Line-by-line explanation:**
- `class ShoppingCart:` — defines a new type called `ShoppingCart`.
- `def __init__(self, user_id: int):` — the **constructor** — runs automatically when you create a new `ShoppingCart`. `self` refers to *this specific instance* being created.
- `self.user_id = user_id` and `self.items = []` — store data *on this instance*, so each cart has its own independent state.
- `def add_item(self, ...):` — a **method**: a function that belongs to the class and operates on `self` (this particular cart's data).
- `cart = ShoppingCart(user_id=42)` — creates an **instance** (an actual object) from the class blueprint.
- `cart.add_item(...)` — calls a method on that specific instance, modifying *its* `items` list (a different `cart2` would have its own separate list).

### Why classes matter in backend
Classes are how we model real-world entities (users, orders, carts) and how frameworks like FastAPI and ORMs like SQLAlchemy structure their building blocks (models, schemas, services). Understanding classes is a prerequisite for understanding nearly every framework you'll touch.

---

## 4. Dependency Injection (DI) — the concept, demystified

This phrase sounds intimidating, but the idea is simple — and FastAPI leans on it heavily (Lesson 07), so it's worth understanding *now*, in plain terms.

### The problem DI solves
```python
# 🚫 This function creates its OWN database connection internally
def get_user_by_id(user_id: int):
    db = connect_to_production_database()   # hardcoded — hard to test, hard to swap
    return db.query(User).filter(User.id == user_id).first()
```
What if you want to test this function without hitting a real production database? You can't — the dependency is *baked in*.

### The DI approach
**Dependency Injection** means: instead of a function/class *creating* the things it depends on, those things are *passed in* ("injected") from the outside.

```python
# ✅ The database connection is passed in, not created internally
def get_user_by_id(db, user_id: int):
    return db.query(User).filter(User.id == user_id).first()

# In production:
get_user_by_id(production_db, user_id=1)

# In a test:
get_user_by_id(fake_test_db, user_id=1)
```

**Why this is powerful:**
- **Testability**: you can swap in a fake/mock database for tests, without touching the function itself (more in Lesson 08).
- **Flexibility**: the same function can work with different databases, configurations, or services in different environments (development, staging, production).
- **Decoupling**: the function doesn't need to *know* how to create a database connection — it just needs *something* that behaves like one. This is a cornerstone of writing maintainable, large-scale systems.

> 💡 In FastAPI, this concept appears directly via the `Depends()` system — the framework automatically "injects" things like database sessions or the current authenticated user into your route functions. We'll see this in Lesson 07 — and now you'll understand *why* it's designed that way, not just *how* to use it.

---

## 5. Common Beginner Mistakes

1. **Vague names** (`data`, `temp`, `x`, `do_stuff`) — these force every reader (including future-you) to dig into the implementation to understand intent.
2. **Giant functions that "do everything."** If you can't summarize a function's job in one short sentence, it's probably doing too much — split it.
3. **Hardcoding dependencies** (database connections, API keys, file paths) directly inside functions, making testing and configuration painful.
4. **Mutating shared state unexpectedly** — e.g., a function that modifies a list passed into it without the caller expecting that, causing confusing bugs elsewhere.
5. **Skipping type hints "to save time."** They cost almost nothing to write and pay for themselves many times over in catching bugs early and making code self-documenting.

---

## 6. Likely Interview Questions & Strong Answers

**Q: What makes code "clean," in your view?**

> *"Code that another engineer — or me, six months later — can read and understand without needing to ask the original author. That usually means descriptive names, small functions that each do one clear thing, and avoiding hidden 'magic' values. Clean code isn't about being clever; it's about reducing the mental effort needed to safely change it later."*

**Q: What is dependency injection, and why does it matter?**

> *"It means passing the things a function or class depends on — like a database connection or a configuration object — in from the outside, rather than having it create them internally. This makes code far easier to test, because you can substitute a fake version of the dependency in tests, and easier to adapt to different environments without changing the core logic."*

**Q: How do you decide when to use a function versus a class?**

> *"If I just need to perform an operation on some inputs and return a result, a function is simplest and clearest. If I need to bundle related data and behavior together — something that has both 'state' and 'actions on that state,' like a shopping cart that holds items and can compute its total — a class models that more naturally. I try not to reach for a class just because it 'feels more professional' — the simplest tool that clearly expresses the idea wins."*

---

## 7. Tips From a Senior Interviewer

- **If asked to write code live, narrate your naming choices.** "I'll call this `calculate_discounted_total` so it's clear what it returns" shows the *thinking* behind clean code, not just the result.
- **Don't over-engineer a simple answer.** If the problem calls for a function, don't build an elaborate class hierarchy "to be safe" — that often reads as *over*-engineering, which is its own red flag.
- **When discussing DI, ground it in testing.** "...which makes it much easier to test in isolation" is the single most convincing justification you can give — because it's the *real* reason senior engineers value it.

---

## 8. Summary

- **Clean code** means optimizing for the human reader: descriptive names, small focused functions, no magic numbers, and avoiding both excessive duplication *and* excessive abstraction.
- **Functions** are the basic reusable unit of behavior; **classes** bundle related data and behavior into a coherent whole.
- **Dependency Injection** means passing dependencies in from the outside rather than creating them internally — the single biggest unlock for writing testable, flexible, maintainable backend code.
- These aren't "nice to haves" — they're what determines whether a codebase stays pleasant to work in, or turns into something everyone dreads touching.

**Next up →** [Lesson 07: FastAPI](07-fastapi.md), where we put all of this together to build real, working API endpoints.
