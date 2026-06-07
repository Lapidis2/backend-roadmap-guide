# Backend Engineering Interview Roadmap

Welcome! This repository is a **lesson-by-lesson study guide** designed to take a complete beginner from "I don't know what backend engineering is" to "I can confidently pass a backend engineering interview" — specifically tuned for the kind of questions asked in **AmaliTech backend engineering interviews**.

> 🎯 **Goal:** By the end of this roadmap, you should be able to explain core backend concepts clearly, write simple working code, and answer interview questions with the confidence of someone who *understands* — not someone who *memorized*.

---

## 👤 Who is this for?

- Complete beginners to backend engineering
- Junior developers preparing for technical interviews
- Self-taught programmers who know how to code but have never been interviewed
- Anyone who wants a structured, no-fluff path to interview readiness

You do **not** need prior backend experience. You do need to be able to read basic code and be willing to type things out yourself (reading is not enough — typing builds memory).

---

## 🗺️ How to use this roadmap

1. **Go in order.** Each lesson builds on the one before it. Don't skip ahead, even if a topic looks "easy" — the early lessons set up vocabulary used later.
2. **Type out every code example yourself.** Don't copy-paste. Typing forces your brain to notice details you'd otherwise skim past.
3. **Answer the "Likely Interview Questions" out loud**, as if someone is sitting across from you. Saying an answer out loud is very different from thinking you know it.
4. **Re-read the "Common Beginner Mistakes" sections a second time** after finishing the lesson — they'll make more sense once you've seen the full picture.
5. **Finish with Lesson 14 (Mock Interview Q&A)** as a final rehearsal, ideally a day or two before your real interview.

---

## 📚 Lesson Index

| # | Lesson | What you'll learn |
|---|--------|-------------------|
| 01 | [Data Structures & Algorithms](01-data-structures-and-algorithms.md) | Arrays, hash maps, when to use which, common interview traps |
| 02 | [Time & Space Complexity](02-time-complexity.md) | Big-O notation, O(1)/O(n)/O(n²), why performance matters |
| 03 | [Databases & Normalization](03-databases-and-normalization.md) | Tables, rows, columns, 1NF/2NF/3NF, why normalization matters |
| 04 | [Keys & Relationships](04-keys-and-relationships.md) | Primary/foreign keys, 1:1, 1:N, N:N, SQL examples |
| 05 | [ORM & the N+1 Query Problem](05-orm-and-n-plus-one.md) | What an ORM is, when to avoid it, detecting and fixing N+1 |
| 06 | [Python for Backend](06-python-backend-fundamentals.md) | Clean code, functions, classes, dependency injection |
| 07 | [FastAPI](07-fastapi.md) | Pydantic validation, CRUD APIs, error handling, async basics |
| 08 | [Testing: Unit, Integration, E2E](08-testing-unit-integration-e2e.md) | Testing pyramid, pytest examples, what interviewers expect |
| 09 | [Code Coverage](09-code-coverage.md) | What coverage measures, why 100% isn't the goal |
| 10 | [Docker (High Level)](10-docker-high-level.md) | Containers vs VMs, Dockerfiles, why companies use Docker |
| 11 | [Git & Commit Messages](11-git-and-commit-messages.md) | Git workflow, writing clean commits, examples of good vs bad |
| 12 | [GitHub Actions & CI](12-github-actions-ci.md) | What CI is, why it matters, a simple pipeline example |
| 13 | [Interview Mindset: "Why This, Not That?"](13-interview-why-this-not-that.md) | Explaining trade-offs, handling "I don't know" moments |
| 14 | [Mock Interview Q&A](14-mock-interview-qna.md) | Full rehearsal with model answers across every topic |

---

## 🧠 What makes a good backend interview answer?

Before you start, internalize this one idea — it will be repeated throughout the roadmap because it matters more than any single fact:

> **Interviewers are not grading you on whether you know the "right" answer. They are grading you on whether you can *think out loud*, justify a choice, and recognize trade-offs.**

A junior who says *"I'd use a hash map here because I need fast lookups by key, even though it costs a bit more memory"* will often outscore someone who blurts out *"hash map"* with no reasoning — even if both gave the "same" answer.

Keep that in mind in every lesson. We don't just teach you *what* — we teach you *why*, because *why* is what gets scored.

---

## ✅ Suggested Study Plan

| Week | Focus |
|------|-------|
| Week 1 | Lessons 01–04 (DSA, Big-O, Databases, Keys) |
| Week 2 | Lessons 05–07 (ORM, Python, FastAPI) |
| Week 3 | Lessons 08–10 (Testing, Coverage, Docker) |
| Week 4 | Lessons 11–14 (Git, CI, Interview Mindset, Mock Q&A) |

If your interview is sooner, you can compress this — but don't skip Lesson 13 and 14. They are what tie everything together.

---

## 📌 A note on tone

Every lesson is written the way a senior engineer would mentor a junior they actually care about: clearly, patiently, and honestly — including admitting where things are genuinely confusing at first. There is no "just memorize this" anywhere in this roadmap. If something feels like memorization, it means we haven't explained the *why* well enough — re-read the real-world example in that section, and it should click.

Good luck. You've got this. 🚀

**Start here →** [Lesson 01: Data Structures & Algorithms](01-data-structures-and-algorithms.md)
