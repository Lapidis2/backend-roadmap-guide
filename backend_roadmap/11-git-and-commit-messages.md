# Lesson 11 — Git & Commit Messages

> **Where this fits:** Everything we've built so far — code, tests, Docker configs — needs to be tracked, shared, and collaborated on safely. **Git** is the tool that makes that possible, and how you *use* it (especially how you write commit messages) is one of the clearest, most easily observable signals of professionalism in any codebase — including yours, the moment an interviewer opens your GitHub profile.

---

## 1. What is Git, and why does it matter?

**Git** is a **version control system** — it tracks every change made to a codebase over time, who made it, and why, and lets multiple people work on the same project without overwriting each other's work.

Without it, collaboration looks like emailing files back and forth named `final_v2_ACTUALLY_FINAL.py` — chaotic, error-prone, and impossible to safely undo. With it, every change is tracked, attributable, reversible, and mergeable.

### The basic Git workflow

```bash
git checkout -b feature/add-user-search    # 1. Create a new branch for your work
# ... make changes to files ...
git add src/search.py                       # 2. Stage the specific changes you want to commit
git commit -m "Add case-insensitive user search by username"  # 3. Commit with a clear message
git push origin feature/add-user-search     # 4. Push the branch to the shared remote repository
# ... open a Pull Request (PR) for review ...
git checkout main                           # 5. After the PR is approved and merged...
git pull origin main                         #    ...sync your local main branch
```

**Line-by-line explanation:**
- `git checkout -b feature/add-user-search` — creates and switches to a new **branch**, an isolated line of development. This means your work-in-progress doesn't affect the stable `main` branch until it's reviewed and merged — letting multiple people work on different features simultaneously without stepping on each other.
- `git add src/search.py` — **stages** a specific file's changes, marking them to be included in the next commit. (You can stage multiple files, or selectively stage *parts* of a file — letting you build focused, logical commits rather than one giant "everything I did today" blob.)
- `git commit -m "..."` — creates a **commit**: a permanent, named snapshot of the staged changes, with a message explaining *what* changed and (ideally) *why*.
- `git push origin feature/add-user-search` — uploads your branch and its commits to the shared remote repository (e.g., GitHub), making it visible to teammates.
- A **Pull Request (PR)** is a request to merge your branch into another (usually `main`) — the central place where teammates review your code, leave comments, and discuss changes before they become part of the shared codebase.
- `git pull origin main` — downloads and integrates the latest changes from the remote `main` branch into your local copy.

---

## 2. Why clean commits matter in teams

A commit isn't just "a save point" — it's a **piece of communication** with your future teammates (and your future self). Good commit history answers questions that come up constantly during real engineering work:

- *"Why was this line of code written this way?"* → `git blame` shows you the commit that introduced it — and a clear message explains the reasoning.
- *"Something broke between last week and today — which change caused it?"* → A clean history of small, focused commits makes it possible to pinpoint the exact change (often using `git bisect`).
- *"Can we safely revert just this one feature without losing everything else done since?"* → Only possible if that feature lives in its own clean, focused commit(s).

> **A messy commit history doesn't just look unprofessional — it actively costs the team time and makes debugging production incidents harder, slower, and more stressful.** This is *exactly* why interviewers (and hiring managers browsing your GitHub) pay attention to it.

---

## 3. Examples of Good vs. Bad Commits

### Bad commit messages
```
fix
update stuff
asdf
final fix for real this time
WIP
changes
fixed the bug
more changes to the thing we discussed
```
**What's wrong with these?** They tell a future reader *nothing*. "Fix" — fix *what*? "Update stuff" — *what* stuff, and *why*? Six months from now (or even six hours from now), nobody — including the author — will remember what these commits actually did without opening each one and reading every line of the diff.

### Good commit messages
```
Fix off-by-one error in pagination that skipped the last result
Add input validation for negative order quantities
Refactor user authentication to use dependency injection for testability
Remove deprecated /v1/legacy-search endpoint (replaced by /v2/search)
Add integration test for N+1 query regression in book listing endpoint
```
**What makes these good?**
- They describe **what changed** *and*, often, **why** — in clear, specific language.
- They're written in the **imperative mood** ("Fix," "Add," "Refactor," "Remove") — as if completing the sentence "If applied, this commit will... [fix off-by-one error in pagination...]". This is a widely-adopted convention (and matches the auto-generated language Git itself uses, e.g., "Merge branch...").
- Each one represents **one logical change** — not "fixed three unrelated bugs and also reformatted the whole file."

### A well-structured commit message (for more complex changes)
```
Fix N+1 query in GET /authors endpoint

Author listings were triggering one query per author to fetch their
books, due to lazy-loaded relationship access inside a loop. This
caused ~500ms response times with 200+ authors in staging.

Switched to eager loading via selectinload(), reducing the endpoint
to two queries total regardless of result size. Response time in
staging dropped to ~40ms.
```
**Anatomy of this message:**
- **First line**: a short (ideally <50-72 characters), imperative summary — this is what shows up in compact log views and PR lists.
- **Blank line**, then a **body**: explains *why* the change was needed (the problem) and *what* the fix does (the solution) — context that the diff alone can't convey. This connects directly back to the N+1 problem from Lesson 05 — notice how a good commit message *teaches* the reader something, the same way this roadmap tries to.

---

## 4. Common Beginner Mistakes

1. **Committing huge, unrelated changes together** ("fixed the bug AND reformatted the whole file AND added a new feature") — making it impossible to review, revert, or understand any single change in isolation.
2. **Vague messages** ("fix," "update," "changes") that provide zero context for future readers — including the author, a surprisingly short time later.
3. **Committing commented-out code, debug print statements, or secrets** (API keys, passwords) — these become a permanent part of history (yes — even if you delete them in a *later* commit, they remain visible in the repository's history unless you take special, disruptive steps to purge them).
4. **Working directly on `main`/`master`** instead of feature branches — risking destabilizing the shared codebase and making collaborative review impossible.
5. **Force-pushing over shared branch history** without team awareness — this can silently destroy teammates' work and is one of the fastest ways to erode trust on a team. (If you're not sure whether an operation is "safe" to run on a shared branch, that uncertainty is itself the answer: ask first.)

---

## 5. Why this especially matters for *your* interview

> Here's something many candidates don't realize: **interviewers very often look at your GitHub profile before — or during — your interview.** Your commit history isn't just practice for "real" work; it's a live, public portfolio of how you think, communicate, and collaborate.

A profile full of clear, well-scoped commits with thoughtful messages quietly signals: *"This person already works the way our team works."* A profile full of "fix," "asdf," and "final final v3" signals the opposite — regardless of how good the underlying code actually is.

---

## 6. Likely Interview Questions & Strong Answers

**Q: What makes a good commit message?**

> *"It clearly communicates what changed and, ideally, why — written in the imperative mood, like 'Fix pagination off-by-one error' rather than 'fixed it.' For more complex changes, I'll add a body explaining the problem and the reasoning behind the solution, since the diff alone often can't convey *why* a particular approach was chosen. The goal is that someone — including me, months later — can understand the change without having to reverse-engineer it from the code alone."*

**Q: Why do clean commits matter on a team, beyond just 'looking nice'?**

> *"They make the codebase's history actually useful — for debugging ('which change caused this regression?'), for context ('why was this written this way?'), and for safety ('can we revert just this one change cleanly?'). A history of large, vague commits turns all of these into slow, frustrating archaeology projects. Clean commits are an investment that pays off every time something needs to be understood, debugged, or undone — which, on any real project, is constantly."*

**Q: How do you approach branching and pull requests when working on a team?**

> *"I'd create a feature branch for each piece of work, keeping `main` stable and deployable at all times. I'd commit in small, logical chunks as I go — each one representing one coherent change — then open a pull request for review before merging. This gives teammates a clean, reviewable unit of work, creates a natural point for feedback and discussion, and keeps the shared history easy to follow and trust."*

---

## 7. Tips From a Senior Interviewer

- **If your own GitHub history has messy commits from when you were learning — that's completely normal, and not something to be defensive about.** What stands out is whether you can clearly articulate *what* good practice looks like and *why* it matters — that's the actual signal being assessed, not a flawless personal history (everyone's includes some "fix typo" commits from learning days).
- **Use this lesson's vocabulary deliberately in your answers**: "imperative mood," "logical commit," "feature branch," "PR review." These aren't jargon for jargon's sake — they're the shared vocabulary of professional teams, and using them naturally signals you'll fit smoothly into one.
- **If asked to walk through your workflow, narrate it as a *story with reasoning*** — "I'd branch off main so my work-in-progress doesn't destabilize the shared codebase, then commit in focused chunks so each change is reviewable and revertible independently..." — rather than a flat list of commands. The reasoning is what's being evaluated, not your command-line recall.

---

## 8. Summary

- **Git** tracks changes over time, enabling safe collaboration — branches isolate work, commits create reviewable snapshots, and PRs provide a structured space for review and discussion.
- **Clean commits** — small, focused, written in the imperative mood, explaining *what* and *why* — make a codebase's history a genuinely useful tool for debugging, understanding, and safely reverting changes.
- **Bad commit habits cost teams real time and trust**; good ones quietly compound into a codebase that's a pleasure (rather than a nightmare) to work in.
- Your **own GitHub history is a live portfolio** — interviewers notice, and the habits in this lesson are exactly what they're looking for.

**Next up →** [Lesson 12: GitHub Actions & CI](12-github-actions-ci.md), where we automate the process of checking that every change — yours and your teammates' — actually works before it's merged.
