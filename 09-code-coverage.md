# Lesson 09 — Code Coverage

> **Where this fits:** In Lesson 08 we learned *how* to write tests. Now: how do we know whether we've written *enough* of them — and in the right places? **Code coverage** is the metric teams use to answer that question — and the nuance around it is a favorite "do you actually understand testing, or just do it by rote" interview topic.

---

## 1. What is code coverage?

**Code coverage** is a measurement of *how much of your source code is executed* while your test suite runs — usually expressed as a percentage.

```
Total lines of code:        200
Lines executed by tests:    170
Coverage:                   170 / 200 = 85%
```

Tools (like `coverage.py` in Python, often run alongside `pytest` via the `pytest-cov` plugin) instrument your code while tests run, tracking exactly which lines, branches, or functions were "touched."

```bash
pytest --cov=app --cov-report=term-missing
```

**What this command does:**
- `--cov=app` — measure coverage for the `app` package/module.
- `--cov-report=term-missing` — print a report showing the percentage *and* the specific line numbers that were never executed by any test — extremely useful for finding exactly what to test next.

### Types of coverage (good to know the names)
- **Line coverage** — did each line of code execute at least once?
- **Branch coverage** — did each possible branch of an `if`/`else` (or similar) get exercised? *(This is stricter and more meaningful than line coverage — a line can "execute" while only ever taking one of its possible paths.)*

---

## 2. Why measure it at all?

Coverage is a **diagnostic tool**, not a report card. Its real value:

- **Finding untested code you forgot about.** It's easy to write tests for the logic you were focused on and completely forget the error-handling branch three lines down. Coverage reports surface these blind spots.
- **Giving teams a shared, objective signal** about testing thoroughness — useful in code review ("this PR adds a new branch with no corresponding test — can we add one?").
- **Catching "dead code"** — code that's never executed by tests *or* by the application might be entirely unused and safe to delete (always worth investigating, not assuming).

---

## 3. Why 100% coverage is NOT always the goal — the crucial nuance

This is the part that separates a thoughtful answer from a textbook-recitation answer. Here's the honest picture:

### Coverage measures *execution*, not *correctness*
```python
def calculate_discounted_total(price, quantity, discount_rate):
    return price * quantity * (1 - discount_rate)

# This test gives the function 100% line coverage...
def test_calculate_discounted_total_runs():
    calculate_discounted_total(100, 2, 0.1)   # ...but asserts NOTHING!
```
This test *executes every line* of the function — 100% coverage! — but verifies **absolutely nothing** about whether the result is *correct*. A function could be completely broken and still show 100% coverage if the tests never check the output. **Coverage tells you what ran. It does not tell you what was verified.**

### Chasing 100% can produce *worse* tests, not better ones
To hit that last few percent, teams sometimes end up writing tests that exist purely to "touch" a line — with weak or no real assertions — just to make the number go up. This **wastes effort and adds maintenance burden** (more tests to update when code changes) **without adding real protection against bugs**. It can also create a false sense of security — "we're at 100%, we must be safe!" — which is more dangerous than knowing you have gaps.

### Some code genuinely isn't worth the effort to cover
- Trivial getters/setters with no logic
- Defensive code for conditions that are structurally impossible to reach (though — be *very* careful with this reasoning; "impossible" conditions have a habit of becoming possible as code evolves)
- Auto-generated code, simple configuration, or thin framework glue

### The senior framing
> "Coverage is a *flashlight*, not a *finish line*. It helps me find places I haven't thought to test — but the goal is always meaningful tests on the code that matters most: business logic, edge cases, error handling, and anything that's bitten the team before. A high percentage with weak assertions is worse than a slightly lower percentage with tests that actually verify behavior."

A common, healthy team norm is something like **"aim for roughly 80%, with critical business logic close to 100%, and don't sweat chasing the last few percent on trivial code."** The exact number matters far less than understanding *why* that's a reasonable target rather than a religious rule.

---

## 4. How to use coverage well (practical guidance)

1. **Run coverage regularly** (often automated in CI — see Lesson 12) and *read the gaps*, not just the headline percentage.
2. **Prioritize coverage for:**
   - Business logic (calculations, validation rules, state transitions)
   - Error handling and edge cases
   - Code that has caused bugs before (a great argument for adding a regression test *every time* you fix a bug)
3. **Be skeptical of a sudden coverage spike** in a PR — it might mean genuinely thorough new tests, or it might mean someone wrote shallow tests to satisfy a CI gate. Code review should look at *what's actually being asserted*, not just the number.
4. **Don't treat a coverage threshold as a substitute for thinking.** A CI gate requiring "80% coverage" is a useful trip-wire for *forgotten* code — it is not, by itself, evidence of a *good* test suite.

---

## 5. Common Beginner Mistakes

1. **Treating the coverage percentage as the goal**, rather than as a tool for finding gaps in *meaningful* testing.
2. **Writing assertion-free or weak-assertion tests** purely to inflate the number — a practice that actively harms a codebase's long-term health (false confidence + maintenance cost, with none of the benefit).
3. **Assuming "100% covered" means "bug-free."** It very much does not — it only means every line *ran* at least once during some test.
4. **Ignoring branch coverage.** A function might show high *line* coverage while one entire `else` branch — perhaps the one handling a critical error case — has never actually been exercised.
5. **Getting discouraged by a "low" percentage on a legacy codebase** instead of focusing pragmatically: "what's the most valuable code to add coverage to *next*?"

---

## 6. Likely Interview Questions & Strong Answers

**Q: What is code coverage, and how do you measure it?**

> *"It's a measurement of how much of your source code gets executed while your tests run, usually as a percentage of lines or branches. In Python, tools like coverage.py — often run via the pytest-cov plugin — instrument the code during test runs and report exactly which lines were and weren't exercised, which is very useful for finding gaps you didn't realize existed."*

**Q: Is 100% code coverage a good goal?**

> *"Not by itself — coverage measures whether code *ran* during tests, not whether it was *correctly verified*. A test can execute every line of a function without making a single meaningful assertion, and still show 100% coverage on a completely broken function. Chasing the last few percent often leads teams to write low-value tests just to satisfy the number, which adds maintenance cost without adding real protection. I'd rather have strong, meaningful tests on the code that matters most — business logic, edge cases, error handling — than a hollow 100%."*

**Q: How would you decide what to prioritize if your team's coverage is currently low?**

> *"I'd start with the code that's riskiest if it breaks — core business logic, payment or auth-related code, and anything that's caused production bugs before. I'd also make it a habit to add a regression test every time we fix a bug, since that's a very efficient way to grow meaningful coverage exactly where the codebase has proven to be fragile, rather than spreading effort evenly across code of wildly different importance."*

---

## 7. Tips From a Senior Interviewer

- **This question is a "do you understand testing, or do you just run the tool" filter.** Anyone can say "we use coverage.py." Articulating *why 100% isn't automatically good* is what shows real understanding — and it's a very common follow-up, so be ready for it.
- **Use the "flashlight, not finish line" framing (or your own version of it).** A vivid, memorable analogy like this sticks with interviewers far better than a dry definition — and signals that you've internalized the idea rather than memorized a talking point.
- **If you don't know the exact tool name for a language you haven't used, say so — and pivot to the concept.** *"I haven't used a coverage tool in [language X], but conceptually I'd expect something that instruments test runs and reports which lines/branches were executed — the underlying idea is the same across ecosystems."* This shows transferable understanding (see Lesson 13 for more on handling "I don't know" gracefully).

---

## 8. Summary

- **Code coverage** measures how much source code is executed during test runs — a useful *diagnostic* for finding untested areas.
- It measures **execution, not correctness** — a fully "covered" function can still be completely broken if the tests don't assert the right things.
- **100% is not automatically the goal.** Chasing it can produce shallow, low-value tests that cost more to maintain than they're worth.
- The senior approach: use coverage to find *gaps*, then prioritize *meaningful* tests on the code that matters most — business logic, edge cases, error handling, and previously-buggy code.

**Next up →** [Lesson 10: Docker (High Level)](10-docker-high-level.md), where we shift gears to how backend applications are packaged and run consistently across different environments.
