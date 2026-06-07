# Lesson 08 — Testing: Unit, Integration, and End-to-End

> **Where this fits:** We've written code (Lessons 06–07) — now, how do we *prove* it works, and keep proving it as the codebase grows and changes? Testing is one of the clearest signals interviewers use to separate "can write code that works once" from "can build something a team can rely on long-term."

---

## 1. Why testing matters (the honest pitch)

Imagine you fix a bug in your `calculate_discounted_total` function. Six months later, a teammate changes how discounts are calculated elsewhere — and accidentally breaks your function in a subtle way. Without tests, **nobody finds out until a customer complains** that their total looks wrong.

> **Tests are a safety net that lets you (and your team) change code with confidence — instead of fear.**

This reframe matters: testing isn't busywork imposed by process-obsessed managers. It's what makes it *possible* to keep improving a system over years without it slowly collapsing under the weight of undetected regressions.

---

## 2. The Testing Pyramid: Unit → Integration → End-to-End

Picture a pyramid. The bottom layer is the widest (you write the *most* of these); the top is the narrowest (you write the *fewest*).

```
        ▲
       / \         End-to-End (E2E) Tests
      /   \        — few, slow, test the whole system together
     /-----\
    /       \      Integration Tests
   /         \     — moderate number, test how pieces work together
  /-----------\
 /             \   Unit Tests
/_______________\  — many, fast, test one small piece in isolation
```

### Unit Tests
**Test a single function or class, in complete isolation from everything else** (databases, networks, other modules — all faked/mocked out).

```python
# The function we're testing (from Lesson 06)
def calculate_discounted_total(price, quantity, discount_rate):
    return price * quantity * (1 - discount_rate)

# A unit test for it
def test_calculate_discounted_total_applies_discount_correctly():
    result = calculate_discounted_total(price=100, quantity=2, discount_rate=0.1)
    assert result == 180.0   # 100 * 2 * (1 - 0.1) = 180
```

**Line-by-line explanation:**
- `def test_calculate_discounted_total_applies_discount_correctly():` — the test function's *name* describes exactly what behavior is being verified. (Yes, long test names are not just okay — they're *good practice*. A failing test name should tell you what broke without opening the file.)
- `result = calculate_discounted_total(...)` — calls the function with known, controlled inputs.
- `assert result == 180.0` — checks that the output matches the expected value. If it doesn't, the test fails loudly, telling you *exactly* what went wrong.

**Why isolation matters:** Unit tests should be **fast** (milliseconds) and **deterministic** (same result every single time) — so you can run hundreds or thousands of them in seconds, on every single change. That's only possible if they don't depend on a real database, network, or filesystem, all of which are slow and occasionally flaky.

### Integration Tests
**Test how multiple pieces work together** — e.g., does your API endpoint correctly save data to a real (often temporary/test) database and return the right response?

```python
def test_create_book_endpoint_saves_to_database(test_client, test_db):
    response = test_client.post("/books", json={
        "title": "Clean Code",
        "author": "Robert Martin",
        "price": 35.0
    })

    assert response.status_code == 201
    saved_book = test_db.query(Book).filter(Book.title == "Clean Code").first()
    assert saved_book is not None
    assert saved_book.price == 35.0
```

**What's different here:** This test exercises the *real* path — HTTP request → validation → route handler → database write → response — using a real (test) database and a real (test) HTTP client. It catches problems that unit tests, by design, can't: e.g., "the function works in isolation, but the route never actually calls it," or "the data gets validated but never actually persists."

### End-to-End (E2E) Tests
**Test the entire system from the user's perspective**, simulating real usage across the whole stack — frontend, backend, database, sometimes even third-party services.

Example: a script that opens a browser, signs up a new user, logs in, adds an item to a cart, checks out, and verifies the order appears in the order history — exactly as a real user would experience it.

**Why they're at the top of the pyramid (few of them):**
- **Slow** — they may take seconds or minutes per test, since they exercise the full system.
- **Brittle** — a small UI change can break a dozen E2E tests that don't actually reflect a real underlying problem.
- **Expensive to maintain** — they require the most setup (running the full stack, seed data, etc.).

But they're irreplaceable for catching the kind of bug that *only* shows up when everything is wired together for real — which is exactly why a *small* number of well-chosen E2E tests, covering your most critical user journeys (e.g., "can a user successfully complete a purchase?"), is so valuable.

---

## 3. The Pyramid Shape — Why?

> **"Write lots of fast, focused unit tests; a moderate number of integration tests for the seams between components; and a small number of E2E tests for your most critical user journeys."**

This isn't an arbitrary rule — it's a direct trade-off between **speed/reliability** and **realism**:

| | Unit | Integration | E2E |
|---|------|-------------|-----|
| Speed | ⚡ Very fast | 🚶 Moderate | 🐢 Slow |
| Realism | Low (isolated) | Medium | High (full system) |
| Cost to write/maintain | Low | Medium | High |
| How many you should have | Many | Some | Few |

A healthy test suite runs the bulk of its tests (the unit layer) in seconds, giving you near-instant feedback on most changes — while still having enough integration and E2E coverage to catch the kinds of bugs that only emerge when components meet.

---

## 4. pytest Examples

**pytest** is the most widely used Python testing framework — simple syntax, powerful features, and the de facto standard in most Python backend teams.

```python
import pytest

# A simple test
def test_addition():
    assert 2 + 2 == 4

# Testing that an error is raised correctly
def test_calculate_discounted_total_rejects_negative_price():
    with pytest.raises(ValueError):
        calculate_discounted_total(price=-10, quantity=2, discount_rate=0.1)

# Fixtures: reusable setup code, injected into tests as arguments
@pytest.fixture
def sample_cart():
    cart = ShoppingCart(user_id=1)
    cart.add_item("Laptop", 1200.00)
    return cart

def test_cart_total_sums_item_prices(sample_cart):
    assert sample_cart.total() == 1200.00

# Parametrize: run the same test logic against multiple input/output pairs
@pytest.mark.parametrize("price, quantity, discount, expected", [
    (100, 1, 0.0, 100.0),
    (100, 2, 0.5, 100.0),
    (50, 3, 0.1, 135.0),
])
def test_calculate_discounted_total_various_cases(price, quantity, discount, expected):
    assert calculate_discounted_total(price, quantity, discount) == expected
```

**Line-by-line explanation:**
- `assert 2 + 2 == 4` — pytest's core idea: just use plain `assert` statements. If the condition is false, pytest shows you a detailed, readable failure report automatically — no special assertion methods to memorize.
- `with pytest.raises(ValueError):` — verifies that the code inside the block raises a `ValueError` — essential for testing that your *error handling* (Lesson 07) works as intended, not just the happy path.
- `@pytest.fixture` — marks `sample_cart` as reusable setup logic. Any test that takes `sample_cart` as a parameter automatically receives a freshly created cart — avoiding repetitive setup code across many tests (a direct application of the DRY principle from Lesson 06).
- `@pytest.mark.parametrize(...)` — runs the *same* test function multiple times with different inputs/expected outputs, listed in a clear table-like format. This is far more maintainable than copy-pasting near-identical test functions for each case.

---

## 5. What interviewers expect (be honest with yourself here)

Interviewers aren't usually expecting you to recite pytest's entire API. They *are* checking whether you:

1. **Understand *why* testing matters** — not as a checkbox, but as a tool for confidence and safety when changing code.
2. **Can identify what's worth testing.** Good signal: "I'd test the discount calculation with edge cases — zero quantity, 100% discount, negative price — because those are where bugs tend to hide." Weak signal: "I'd write a test that checks if the function returns a number."
3. **Understand the difference between unit and integration tests**, and *why* you'd reach for one over the other in a given situation.
4. **Think about edge cases, not just the happy path.** What happens with empty input? Negative numbers? Extremely large values? Concurrent requests?
5. **Can explain testing trade-offs** — e.g., "I wouldn't write an E2E test for every single field validation; that's better covered by fast unit tests on the validation logic itself."

---

## 6. Common Beginner Mistakes

1. **Only testing the happy path.** A test suite that only confirms "it works when everything goes right" misses the entire point — bugs live in edge cases and error conditions.
2. **Writing tests that are too tightly coupled to implementation details**, so they break every time you refactor — even when the actual *behavior* hasn't changed. (Test *behavior*, not *internals*.)
3. **Slow, flaky test suites** that take so long or fail randomly that the team starts ignoring (or skipping) them — defeating their entire purpose.
4. **Vague test names** like `test_function1` or `test_it_works`. A good test name documents the expected behavior — `test_calculate_discounted_total_returns_zero_for_full_discount` tells you immediately what broke, without opening the file.
5. **Treating "I wrote some tests" as the finish line**, rather than thinking about *what* meaningfully needs coverage (this connects directly to Lesson 09 — code coverage).

---

## 7. Likely Interview Questions & Strong Answers

**Q: What's the difference between unit, integration, and end-to-end tests?**

> *"Unit tests check a single function or class in isolation, with everything else mocked out — they're fast and pinpoint exactly where a bug is. Integration tests check how multiple pieces work together, like an API endpoint actually writing to a database. End-to-end tests simulate a real user journey across the whole system. I'd write the most unit tests because they're fast and precise, a moderate number of integration tests for critical seams, and only a few E2E tests for the most important user flows — because those are slow and expensive to maintain."*

**Q: How would you test this function?** *(shown `calculate_discounted_total`)*

> *"I'd start with the happy path — a normal price, quantity, and discount — then focus on edge cases: a 0% discount, a 100% discount, a quantity of zero, and invalid inputs like a negative price or a discount above 100%. For the invalid inputs, I'd check that the function either raises a clear error or handles them in a well-defined way — whichever the team's contract specifies — because untested edge cases are where production bugs usually hide."*

**Q: Why might you choose to mock the database in a unit test, but use a real (test) database in an integration test?**

> *"In a unit test, I want to isolate the *logic* I'm testing from external systems — mocking the database keeps the test fast, deterministic, and focused purely on whether my function behaves correctly given certain inputs. In an integration test, the whole point is to verify that the pieces — the route, the validation, the database layer — correctly work *together*, so using a real test database is exactly what catches the kind of bug that mocking would hide."*

---

## 8. Tips From a Senior Interviewer

- **If asked "how would you test X," think out loud about edge cases first.** This single habit is one of the strongest differentiators between juniors and more experienced engineers — it shows you're thinking about *where bugs actually live*, not just confirming the obvious case works.
- **It's fine — even good — to say "I'd also want to test what happens when..." and name a scenario you're not 100% sure how the system should handle.** That's not a weakness; it's exactly the kind of clarifying question a thoughtful engineer asks before writing code.
- **Connect testing back to confidence and team velocity**, not "because we're told to." *"Good tests let the team ship changes quickly without fear of breaking something else"* is a senior-level framing that resonates strongly with interviewers (most of whom have been burned by untested code at some point in their careers).

---

## 9. Summary

- **Tests are a safety net** — they let you and your team change code confidently instead of fearfully.
- The **testing pyramid**: many fast **unit tests** (isolated), some **integration tests** (pieces working together), and a few **E2E tests** (full user journeys).
- **pytest** uses plain `assert` statements, `pytest.raises` for error testing, `@pytest.fixture` for reusable setup, and `@pytest.mark.parametrize` for testing multiple cases cleanly.
- Interviewers care most about whether you can **identify what's worth testing** — especially edge cases — not whether you've memorized framework syntax.

**Next up →** [Lesson 09: Code Coverage](09-code-coverage.md), where we look at how to *measure* how much of your code your tests actually exercise — and why chasing 100% isn't the goal.
