# Lesson 14 — Mock Interview Q&A (Final Rehearsal)

> **Where this fits:** This is your dress rehearsal. Every question below is realistic, drawn directly from the topics in Lessons 01–13, and roughly ordered the way a real interview often flows — warm-up, technical depth, system-thinking, behavioral, and closing. **Read each question, pause, answer it OUT LOUD before reading the model answer.** This single habit — speaking before peeking — is what makes the difference between "I recognize this" and "I can produce this under pressure."

---

## How to use this lesson

1. **Cover the model answer** (scroll slowly, or use a sheet of paper / your hand on the screen).
2. **Answer out loud**, as if someone is sitting across from you, timing yourself to roughly 60–90 seconds per answer.
3. **Then compare** — not to judge yourself harshly, but to notice: *Did I name the trade-off? Did I give a concrete example? Did I explain the "why"?*
4. **Repeat the ones that felt shaky** a day later. Spaced repetition beats cramming, every time.
5. Treat the model answers as **a style and structure to internalize** — not a script to memorize word-for-word. An interviewer can always tell the difference, and genuine reasoning in your own words will always outperform a recitation.

---

## Part 1 — Warm-Up & Fundamentals

**Q1: Can you explain, in simple terms, what a backend engineer actually does day to day?**

> *"A backend engineer builds and maintains the parts of an application that run on servers — handling business logic, storing and retrieving data, enforcing rules like authentication and permissions, and exposing that functionality through APIs that frontends or other systems can use. Day to day, that might mean designing a database schema, writing and testing API endpoints, debugging a performance issue, reviewing a teammate's code, or figuring out why something behaves differently in production than it did locally."*

**Q2: What's the difference between a list and a dictionary, and when would you use each?** *(Lesson 01)*

> *"A list is an ordered collection — great when sequence matters or when I need to process every item, like a feed of recent orders. A dictionary maps keys to values, giving near-instant lookup by a meaningful identifier — like finding a user by their ID. The deciding factor is the access pattern: am I iterating in order, or looking something up by a key? That single question usually settles it."*

**Q3: What is Big-O notation, and why does it matter in backend work?** *(Lesson 02)*

> *"It's a way of describing how an algorithm's run time or memory use grows as the input grows, independent of hardware or specific data. It matters because production data is almost always larger and messier than test data — code that feels instant with 50 records can grind to a halt with 5 million. Thinking in Big-O helps me catch those problems during design and code review, rather than discovering them as live incidents."*

---

## Part 2 — Databases & Data Modeling

**Q4: What is database normalization, and can you walk me through an example?** *(Lesson 03)*

> *"Normalization is structuring tables so each fact lives in exactly one place, avoiding redundancy and the inconsistencies that come with it. For example, instead of repeating a customer's name and email on every order row — which means an email change requires updating every order — I'd store customers in their own table and reference them from orders via a foreign key. Now that fact lives in one place, and updating it is instant and consistent everywhere it matters."*

**Q5: Explain the difference between a primary key and a foreign key, and describe how you'd model a many-to-many relationship.** *(Lesson 04)*

> *"A primary key uniquely identifies a row within its table — never null, never duplicated. A foreign key stores another table's primary key value, creating a link the database can enforce for integrity. For a many-to-many relationship — say, students and courses — neither table can hold a single foreign key to the other, since each side relates to many on the other side. I'd introduce a join table, like `enrollments`, with foreign keys to both, often with a composite primary key on the pair to prevent duplicate enrollments — and that table is also a natural place to store relationship-specific data, like the enrollment date."*

**Q6: Is it ever acceptable to denormalize a database? When?** *(Lesson 03)*

> *"Yes — deliberately, and for a clear reason. Highly normalized schemas can require expensive joins for common reads, and in read-heavy systems, a controlled amount of redundancy can be a reasonable trade-off for performance. The key word is *deliberate*: I'd want a clear, documented reason — for example, storing a snapshot of a product's price on an order, so historical orders stay accurate even if the product's price changes later — rather than redundancy that crept in by accident."*

---

## Part 3 — ORMs, Performance & Code Quality

**Q7: What is the N+1 query problem, and how would you fix it?** *(Lesson 05)*

> *"It happens when fetching a list triggers one query, and then accessing related data for each item triggers a separate query per item — turning what should be a small, constant number of database round-trips into something that scales with the size of the result set. It's dangerous because it's invisible with small test data and devastating with real production data. I'd detect it by logging or counting generated SQL queries during a single request, and fix it with eager loading — fetching related data in a batched query upfront, via something like `joinedload` or `selectinload`, instead of lazily, one at a time, inside a loop."*

**Q8: What makes code 'clean,' in your opinion — and can you give a concrete example of cleaning up a messy function?** *(Lesson 06)*

> *"Clean code is code another engineer — or me, months later — can understand without needing to ask the original author. Concretely: instead of one large `process_order` function that validates, calculates tax, and sends an email all at once, I'd split it into `validate_order`, `apply_tax`, and `send_order_confirmation` — each with one clear job, a descriptive name, and the ability to be tested and changed independently. The orchestrating function then reads almost like a checklist in plain English, which makes the whole system easier to reason about."*

**Q9: What is dependency injection, and why does it matter for testing?** *(Lesson 06)*

> *"It means passing a function or class's dependencies in from the outside — like a database connection — rather than having it create them internally. This makes testing dramatically easier: I can substitute a fake or in-memory version of that dependency in tests, verifying my logic in isolation without needing a real database. It also makes the code more flexible across environments — development, testing, production — without changing the core logic itself."*

---

## Part 4 — APIs, Testing & Quality Gates

**Q10: Why might you reach for FastAPI, and how does it handle request validation?** *(Lesson 07)*

> *"FastAPI uses Python type hints as a single source of truth for both validation and documentation — you define a Pydantic model describing the expected shape of the data, and FastAPI automatically validates incoming requests against it, returning a clear, structured error if something doesn't match — all before your route function even runs. That removes a whole category of defensive checks from business logic, and the interactive docs it generates from the same definitions keep documentation from drifting out of sync with the code."*

**Q11: How would you approach testing a function that calculates a discounted total based on price, quantity, and discount rate?** *(Lesson 08)*

> *"I'd start with the happy path — typical, valid inputs — then deliberately focus on edge cases: zero quantity, a 0% and a 100% discount, and invalid inputs like a negative price or a discount over 100%. For the invalid cases, I'd verify the function handles them in a well-defined way — raising a clear error, for instance — because that's exactly the kind of boundary where production bugs tend to hide, and where a test suite earns its keep."*

**Q12: Is 100% code coverage a good target? Why or why not?** *(Lesson 09)*

> *"Not by itself. Coverage measures whether code *executed* during tests — not whether the results were actually *verified*. A test can run every line of a function and assert nothing, showing 100% coverage on a completely broken function. Chasing the last few percent often produces shallow tests that cost more to maintain than they protect. I'd rather use coverage as a flashlight to find overlooked areas — prioritizing meaningful tests on business logic, edge cases, and code that's broken before — than treat the percentage itself as the goal."*

---

## Part 5 — Infrastructure & Collaboration

**Q13: At a high level, what problem does Docker solve, and how is a container different from a virtual machine?** *(Lesson 10)*

> *"Docker solves the 'works on my machine' problem — packaging an application with everything it needs to run into one portable, consistent unit, so it behaves the same in development, testing, and production. A virtual machine virtualizes hardware and runs a full guest operating system, which is heavy and slow to start; a container virtualizes at the OS level, sharing the host's kernel, which makes it dramatically lighter and faster — while still keeping each application's dependencies isolated."*

**Q14: What makes a commit message good or bad, and why does it matter on a team?** *(Lesson 11)*

> *"A good commit message clearly explains what changed and, ideally, why — written in the imperative mood, like 'Fix pagination off-by-one error,' rather than something vague like 'fix' or 'update stuff.' It matters because commit history is a real working tool: it's how teams debug regressions, understand why code looks the way it does, and safely revert specific changes. A history of vague, sprawling commits turns all of that into slow, frustrating guesswork — while a clean one quietly builds trust and saves real time, every single day."*

**Q15: What is CI, and how does it change the way a team works?** *(Lesson 12)*

> *"Continuous Integration automatically builds and tests code on every push, so problems are caught the moment they're introduced — rather than relying on every developer remembering to run checks manually, every time, forever. It shifts routine verification from human memory to automation, which builds real trust in the shared codebase — anyone can pull the latest `main` and build on it confidently — and frees code review to focus on design and reasoning, the things machines genuinely can't evaluate well."*

---

## Part 6 — Behavioral & Reflective Questions

**Q16: Tell me about a time you had to learn something completely new in a short amount of time. How did you approach it?**

> *"[Describe a real, specific situation.] I started by figuring out the smallest possible version of the problem I could get working, rather than trying to understand everything before writing any code — that gave me a concrete foothold and fast feedback. When I got stuck, I tried to identify *exactly* what I didn't understand, rather than feeling generally lost — that turned a vague feeling of confusion into a specific, answerable question I could research or ask about. That approach — start small, get concrete feedback fast, narrow down confusion into specific questions — is something I now apply to most unfamiliar problems."*

**Q17: Describe a situation where you disagreed with an approach — a teammate's, a tutorial's, or even your own earlier decision. How did you handle it?**

> *"[Describe a real, specific situation, however small.] I tried to understand *why* the existing approach was chosen — there's often a reason that isn't obvious at first glance, and assuming there isn't one is a common mistake. Once I understood the reasoning, I could either see why it actually made sense, or articulate specifically *where* I thought a different approach would do better and *why* — framed as 'here's a trade-off I think we should weigh' rather than 'this is wrong.' That framing turns disagreement into a shared problem-solving conversation instead of a confrontation — which, in my experience, is what actually moves things forward."*

**Q18: Why are you interested in backend engineering specifically — and why this kind of role?**

> *"[Be genuinely specific and personal here — vague answers like 'I like solving problems' apply to literally every job and signal that you haven't actually reflected on this question.] What draws me to backend work specifically is [a real reason — e.g., 'the satisfaction of designing a system that quietly, reliably does its job at scale,' or 'the puzzle of figuring out why something that *should* work doesn't, and tracing it back to its root cause']. I'm especially drawn to [something specific about this role/company/team, if you know it] because [a genuine, specific reason tied to what you've learned about them]."*

*(This question rewards genuine reflection far more than any other on this list — and punishes generic answers the most visibly. Spend real time on your own honest answer to this one before your interview — it shows.)*

---

## Part 7 — Curveballs & "I'm not sure" Practice

These are deliberately the hardest ones — practice them *especially*, because handling them gracefully is often what separates a good impression from a forgettable one. *(Review [Lesson 13](13-interview-why-this-not-that.md) before attempting these.)*

**Q19: Here's a function with a nested loop over the same list — what's the time complexity, and how would you improve it?**
```python
def has_duplicate(items):
    for i in range(len(items)):
        for j in range(len(items)):
            if i != j and items[i] == items[j]:
                return True
    return False
```

> *"This is O(n²) — for every item, we're scanning the entire list again, so the work grows with the square of the input size. I'd improve it by using a set to track items I've already seen: loop through once, and for each item, check whether it's already in the set — an O(1) check on average — and add it if not. That brings the whole thing down to O(n), which is a massive improvement as the list grows."*
```python
def has_duplicate(items):
    seen = set()
    for item in items:
        if item in seen:
            return True
        seen.add(item)
    return False
```

**Q20: Suppose I told you this ORM uses a method called `prefetch_related` instead of what you might know as `selectinload` — does that change your answer about fixing N+1 queries?**

> *"Not conceptually, no — that's just this ORM's name for the same underlying idea: batch-fetching related data upfront via something like a follow-up `WHERE ... IN (...)` query, instead of triggering one query per item lazily inside a loop. The specific method name is something I'd confirm in the documentation, but the approach — eager loading related data based on what I know the endpoint will actually use — is exactly the same regardless of which ORM or language I'm working in."*

*(This is a deliberately constructed example of exactly the "concept vs. syntax" separation taught in Lesson 13 — notice how calmly it pivots from "I don't know this specific name" to "but I understand the underlying idea completely.")*

**Q21: Honestly — is there a topic from today's interview you feel less confident about than others?**

> *"[Name something real and specific — this question is often a direct, deliberate test of self-awareness and honesty, and a vague non-answer like 'no, I feel good about everything' usually reads as either inexperience or evasiveness.] I'd say [specific topic] — I understand the core idea of [what you do understand], but I haven't yet had hands-on experience with [the specific gap], so I'd want to build that through [a specific, realistic plan — a project, a course, deliberate practice]. I find that I learn concepts like this fastest by [your genuine learning style — building something small and real, reading source code, working through structured exercises, etc.]."*

*(A thoughtful, specific, honest answer to this question often leaves a stronger, more memorable impression than a generic "I'm confident in everything" — because it's the only answer in this entire list that **cannot** be memorized in advance. It has to be genuinely yours.)*

---

## Final Pre-Interview Checklist

Before you walk in (or log on), make sure you can do *all* of the following — out loud, fluidly, in your own words:

- [ ] Explain the difference between a list and a dictionary, with a real backend example for each (Lesson 01)
- [ ] Define O(1), O(n), and O(n²) — and *spot* an O(n²) pattern in a snippet of code on sight (Lesson 02)
- [ ] Explain normalization with a concrete before/after example (Lesson 03)
- [ ] Model a one-to-many *and* a many-to-many relationship, including where foreign keys live (Lesson 04)
- [ ] Explain the N+1 query problem and describe eager loading as its fix (Lesson 05)
- [ ] Explain dependency injection and *why* it matters for testing (Lesson 06)
- [ ] Describe how FastAPI validates requests and walk through a basic CRUD endpoint (Lesson 07)
- [ ] Explain the testing pyramid and identify meaningful edge cases for a given function (Lesson 08)
- [ ] Explain why 100% code coverage isn't automatically a good goal (Lesson 09)
- [ ] Explain what Docker solves and the container-vs-VM distinction (Lesson 10)
- [ ] Write (and critique) a clear, well-structured commit message (Lesson 11)
- [ ] Explain what CI does and why it matters to a team (Lesson 12)
- [ ] Use the trade-off template fluidly, on *any* topic, on the spot (Lesson 13)
- [ ] Have a genuine, specific, well-reflected answer ready for "why backend, why this role?" (this lesson, Q18)
- [ ] Have at least one real, specific story ready for a behavioral question — built, broken, fixed, or learned by *you* (this lesson, Q16–Q17)

---

## A closing word

If you've worked through all fourteen lessons, typed out the code, answered the questions out loud, and reflected honestly on the gaps you found — **you are genuinely ready.** Not because you've memorized every fact in this roadmap (nobody does, and no interviewer expects that), but because you've built the thing that actually matters: **the ability to reason clearly about backend engineering problems, explain your thinking honestly, and keep learning what you don't yet know.**

That combination — clear reasoning, honest communication, and a visible appetite to keep growing — is *exactly* what a strong junior backend engineer looks like on day one of a real job. It's also exactly what AmaliTech, and any team worth joining, is looking for.

Go in there, think out loud, stay curious, and trust the work you've put in.

**Good luck. You've got this.** 🚀

---

*[← Back to Lesson 13: Interview Mindset](13-interview-why-this-not-that.md) · [↑ Back to README](README.md)*
