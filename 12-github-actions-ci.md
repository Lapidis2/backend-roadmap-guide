# Lesson 12 — GitHub Actions & Continuous Integration (CI)

> **Where this fits:** We've written code (Lessons 06–07), tested it (Lessons 08–09), containerized it (Lesson 10), and learned to collaborate on it via Git (Lesson 11). **CI** is the glue that ties all of this together automatically — running your tests, checks, and builds on every single change, so problems are caught *before* they reach your teammates or production.

---

## 1. What is Continuous Integration (CI)?

**Continuous Integration** is the practice of automatically building and testing code every time someone pushes a change — instead of relying on developers to remember to "run the tests before pushing" (a habit that, honestly, nobody follows perfectly, 100% of the time, forever).

> **The core idea: let a machine — not human memory or discipline — be the safety net that catches broken code before it spreads.**

Picture the alternative: a teammate pushes code with a failing test (perhaps they forgot to run the suite, or "it passed on my machine"). Without CI, that broken code merges into `main`, and now *everyone* who pulls the latest code inherits the breakage — possibly without immediately realizing it's not *their* fault. CI catches this automatically, the moment the change is pushed, and tells the author *immediately* — while the context is still fresh in their mind.

---

## 2. Why CI matters

- **Catches bugs early** — the moment they're introduced, not days later when someone stumbles onto the breakage in an unrelated task.
- **Removes "works on my machine" as an excuse** — tests run in a clean, consistent environment every single time (this connects directly to the consistency theme from Lesson 10).
- **Builds trust in the codebase** — if `main` always passes CI, teammates can pull the latest code and build on it with confidence, instead of crossing their fingers.
- **Frees humans from repetitive verification** — so code review can focus on *design and logic* ("does this approach make sense?") instead of "did you remember to run the tests?" (a machine already confirmed that, automatically, for every single change).
- **Enables continuous, confident delivery** — CI is usually the first link in a longer chain (CI/CD) that can extend all the way to automatically deploying validated changes.

---

## 3. A Simple CI Pipeline Example (GitHub Actions)

**GitHub Actions** is GitHub's built-in automation tool — you describe a "workflow" in a YAML file, and GitHub runs it automatically in response to events (like a push or a pull request).

```yaml
# .github/workflows/ci.yml
name: Run Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Check out the code
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Run tests with coverage
        run: pytest --cov=app --cov-report=term-missing

      - name: Run linter
        run: ruff check .
```

**Line-by-line explanation:**
- `name: Run Tests` — a human-readable name for this workflow, shown in GitHub's UI.
- `on: push: branches: [main]` and `pull_request: branches: [main]` — defines the **triggers**: this workflow runs automatically whenever code is pushed to `main`, or whenever a pull request targeting `main` is opened or updated. (This second trigger is what shows you the ✅ or ❌ check directly on a PR — exactly the safety net described above, *before* anything merges.)
- `jobs: test:` — defines a **job** named `test` — a set of steps that run together as a unit.
- `runs-on: ubuntu-latest` — specifies the operating system for the runner (a fresh, temporary virtual machine that GitHub provisions for this run — connecting back to the consistency theme of Lesson 10: every run starts from the exact same clean slate).
- `uses: actions/checkout@v4` — uses a pre-built, reusable "action" that checks out your repository's code onto the runner, so subsequent steps can access it.
- `uses: actions/setup-python@v5 / with: python-version: "3.12"` — installs and configures the specified Python version on the runner.
- `run: pip install -r requirements.txt` — runs a shell command directly: installing the project's dependencies.
- `run: pytest --cov=app --cov-report=term-missing` — runs the test suite with coverage reporting (straight out of Lessons 08–09!). **If any test fails, this step fails — and the whole workflow is marked as failed**, blocking the PR from being merged (when configured as a "required check").
- `run: ruff check .` — runs a **linter** (a tool that checks code for style issues, common bugs, and anti-patterns) — automating part of what code review would otherwise need to check by hand.

### What happens when this runs
1. A developer pushes a commit or opens a pull request.
2. GitHub spins up a clean virtual machine, checks out the code, sets up Python, and installs dependencies.
3. It runs the test suite and the linter.
4. **If everything passes** → a green ✅ check appears on the PR — reviewers and teammates can trust the code meets the baseline bar.
5. **If anything fails** → a red ❌ check appears, with logs showing exactly what broke — the author gets immediate, actionable feedback, *before* anyone else is affected.

---

## 4. How CI helps teams (the bigger picture)

Think about what CI *replaces*: a world where every developer must remember, every single time, to manually run the full test suite, the linter, and any other checks — *before* pushing — and where reviewers must manually re-verify all of this themselves.

CI turns that fragile, human-dependent process into something **automatic, consistent, and impossible to accidentally skip.** This is precisely the same instinct behind testing (Lesson 08), normalization (Lesson 03), and clean code (Lesson 06): **build systems and habits that make the *correct* thing the *easy*, *automatic* thing — rather than relying on everyone remembering to do the right thing, perfectly, every single time, forever.**

This is also frequently the first stage of a broader **CI/CD** pipeline — "Continuous Integration / Continuous Delivery (or Deployment)" — where validated changes can be automatically packaged (often into Docker images — Lesson 10!) and even deployed, once they've proven themselves through these automated gates.

---

## 5. Common Beginner Mistakes

1. **Thinking CI is "just for big companies."** Even a solo project benefits enormously — CI catches your *own* mistakes (e.g., "I forgot this only works because of a local file that isn't committed") before they become embarrassing surprises later.
2. **Treating a failing CI check as an annoyance to bypass**, rather than valuable, immediate information about a real problem. (Bypassing checks — e.g., merging despite a red ❌ — tends to create exactly the kind of fragile, untrustworthy `main` branch that CI exists to prevent.)
3. **Writing CI pipelines that are slow and frustrating to wait on**, which tempts people to skip or ignore them. (Fast, reliable feedback loops are what make good practices *stick* — this is the same principle behind keeping unit test suites fast, from Lesson 08.)
4. **Confusing CI with deployment.** CI is about *validating* changes (tests, linting, builds); *deployment* (getting validated changes live to users) is a related but distinct concern — often the next stage in the same pipeline, but worth being able to articulate as a separate idea.

---

## 6. Likely Interview Questions & Strong Answers

**Q: What is Continuous Integration, and why does it matter?**

> *"It's the practice of automatically building and testing code every time changes are pushed, so problems are caught immediately — rather than relying on every developer to remember to run checks manually before pushing. It matters because it catches bugs early, keeps the shared codebase trustworthy, and frees up code review to focus on design and logic instead of re-verifying things a machine can check faster and more reliably."*

**Q: Can you walk me through what a simple CI pipeline does?**

> *"On every push or pull request, it spins up a clean environment, checks out the code, installs dependencies, and runs the test suite — and often a linter too. If everything passes, the change gets a green check showing it's safe to merge; if something fails, the author gets immediate feedback with logs showing exactly what broke, while the change is still fresh in their mind — which is far more useful than discovering the same problem days later."*

**Q: How does CI change the way a team works day-to-day?**

> *"It shifts a lot of routine verification from humans to automation — so instead of every developer needing to remember to run the full suite before every push, and reviewers needing to double-check that they did, the pipeline does it consistently, every single time, without fail. That builds real trust in the `main` branch — anyone can pull the latest code and build on it confidently — and it lets human attention focus on the things machines genuinely can't evaluate well, like whether an approach actually makes sense."*

---

## 7. Tips From a Senior Interviewer

- **Connect CI back to testing and Git** (Lessons 08 and 11) in your answer — interviewers love seeing that you understand how these pieces *reinforce* each other into a coherent system, rather than being isolated trivia facts to recall independently.
- **If you've set up a CI pipeline yourself — even a tiny one for a personal project — mention it specifically.** Concrete, hands-on experience (even small-scale) outweighs textbook knowledge by a wide margin in how it's received.
- **If you haven't set one up, it's a great, low-effort thing to do *before* your interview** — even a minimal `.github/workflows/ci.yml` that runs `pytest` on push to a personal repo gives you a genuine, specific story to tell, instead of a purely theoretical answer. (This is a great companion exercise to Lesson 11 — it gives your commit history something real and current to show off.)

---

## 8. Summary

- **Continuous Integration (CI)** automatically builds and tests code on every push, catching problems immediately rather than relying on manual discipline.
- **GitHub Actions** lets you define automated workflows (in YAML) that check out code, set up the environment, install dependencies, run tests/linters, and report pass/fail status directly on pull requests.
- CI **builds trust in the shared codebase**, frees human reviewers to focus on design and logic, and is usually the foundation of a broader CI/CD pipeline that can extend to automated deployment.
- The underlying philosophy connects directly to testing, clean code, and good Git practices: **make doing the right thing automatic, not something everyone has to remember to do manually, perfectly, forever.**

**Next up →** [Lesson 13: Interview Mindset — "Why This, Not That?"](13-interview-why-this-not-that.md), where we step back from individual technical topics and focus on the meta-skill that ties every single one of them together: how to *think and talk* like an engineer under interview conditions.
