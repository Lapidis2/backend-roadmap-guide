# Lesson 10 — Docker (High-Level Understanding)

> **Where this fits:** You've written code, modeled data, and tested it. But here's a question every backend engineer eventually faces: *"It works on my machine — why doesn't it work on the server?"* **Docker** is the industry-standard answer to that exact problem. You don't need deep DevOps expertise for a junior interview — but you do need to clearly explain *what problem it solves and why*.

---

## 1. The problem Docker solves: "But it works on my machine!"

Imagine your application runs perfectly on your laptop. You deploy it to a server, and it crashes. Why? Maybe:
- The server has a different Python version
- A required system library is missing or a different version
- An environment variable is set differently
- A configuration file path doesn't match

This is one of the most common — and most frustrating — sources of real-world bugs: **the environment your code runs in subtly differs from the environment you developed and tested it in.**

> **Docker's core promise: "Package the application *together with* everything it needs to run, so it behaves identically everywhere — your laptop, a teammate's laptop, the test server, and production."**

---

## 2. What is a container? (And how is it different from a virtual machine?)

### Virtual Machines (VMs) — the older approach
A VM emulates an *entire computer*, including its own full operating system, running on top of your actual computer's operating system (via a "hypervisor"). It's like building a complete house inside another house — thorough, fully isolated, but heavy: each VM consumes significant memory, disk space, and startup time.

### Containers — the lighter approach
A **container** packages an application *and its dependencies* (libraries, runtime, configuration) — but **shares the host machine's operating system kernel** rather than emulating a whole separate OS. It's more like a shipping container: a standardized box that holds everything needed for the cargo inside, easy to move between ships, trucks, and ports — without needing to rebuild the ship each time.

```
┌─────────────────────────────┐     ┌─────────────────────────────┐
│   Virtual Machines           │     │   Containers                 │
│  ┌────────┐  ┌────────┐     │     │  ┌────────┐  ┌────────┐     │
│  │  App A │  │  App B │     │     │  │  App A │  │  App B │     │
│  │ Bins/  │  │ Bins/  │     │     │  │ Bins/  │  │ Bins/  │     │
│  │ Libs   │  │ Libs   │     │     │  │ Libs   │  │ Libs   │     │
│  │ Guest  │  │ Guest  │     │     │  └────────┘  └────────┘     │
│  │  OS    │  │  OS    │     │     │  ────────────────────────   │
│  └────────┘  └────────┘     │     │     Container Engine        │
│  ────────────────────────   │     │  ────────────────────────   │
│       Hypervisor             │     │       Host OS                │
│  ────────────────────────   │     │  ────────────────────────   │
│       Host OS                │     │       Hardware               │
│  ────────────────────────   │     └─────────────────────────────┘
│       Hardware               │
└─────────────────────────────┘
```

**The key practical difference:**
| | Virtual Machines | Containers |
|---|------------------|------------|
| What's included | Full guest OS + app + dependencies | App + dependencies only (shares host OS kernel) |
| Startup time | Minutes | Seconds (often less than one) |
| Resource usage | Heavy (each VM duplicates an OS) | Lightweight |
| Isolation | Very strong (separate OS per VM) | Strong, but shares the host kernel |
| Typical use | Running fully different OSes, strong isolation needs | Packaging and running applications consistently and efficiently |

> 💡 **Interview-ready one-liner:** *"VMs virtualize the hardware and run a full OS per instance; containers virtualize at the OS level, sharing the host's kernel — making them dramatically lighter and faster to start, while still giving each app its own isolated filesystem and dependencies."*

---

## 3. The Dockerfile — explained line by line

A **Dockerfile** is a plain-text recipe describing how to build a container image for your application.

```dockerfile
# Start from an official, minimal Python base image
FROM python:3.12-slim

# Set the working directory inside the container
WORKDIR /app

# Copy dependency list first (more on why, below)
COPY requirements.txt .

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy the rest of the application code
COPY . .

# Document which port the app listens on
EXPOSE 8000

# The command that runs when the container starts
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

**Line-by-line explanation:**
- `FROM python:3.12-slim` — every Docker image starts from a **base image**. Here, we start from an official, minimal image that already has Python 3.12 installed — saving us from setting up Python from scratch. ("slim" means a smaller, stripped-down version — fewer extras, smaller image, faster downloads.)
- `WORKDIR /app` — sets `/app` as the working directory inside the container; subsequent commands run relative to this path.
- `COPY requirements.txt .` — copies just the dependency list into the container *first*.
- `RUN pip install --no-cache-dir -r requirements.txt` — installs the dependencies. **Why copy `requirements.txt` before the rest of the code?** Docker caches each step ("layer"). If your code changes but your dependencies don't, Docker can reuse the cached "install dependencies" layer instead of re-running it — making rebuilds *much* faster. This ordering trick is a small detail that signals real, hands-on Docker experience.
- `COPY . .` — copies the rest of the application code into the container.
- `EXPOSE 8000` — documents that the application listens on port 8000 (a hint for humans and orchestration tools — it doesn't actually publish the port by itself).
- `CMD [...]` — the default command that runs when a container starts from this image — here, launching the FastAPI app via Uvicorn (an ASGI server, the thing that actually runs your FastAPI application and handles incoming HTTP requests).

### The basic workflow
```bash
docker build -t my-backend-app .      # Build an image from the Dockerfile
docker run -p 8000:8000 my-backend-app  # Run a container from that image
```
- `docker build -t my-backend-app .` — builds an **image** (a packaged blueprint) from the Dockerfile in the current directory (`.`), tagging it with a name.
- `docker run -p 8000:8000 my-backend-app` — starts a **container** (a running instance of that image), mapping port 8000 on your machine to port 8000 inside the container.

> 📝 **Image vs. Container — a useful analogy:** An *image* is like a class definition (a blueprint); a *container* is like an instance of that class (a running thing). You can run many containers from the same image, just like you can create many objects from the same class (Lesson 06).

---

## 4. Why companies use Docker

- **Consistency**: "Works on my machine" becomes "works everywhere," because the *exact same packaged environment* runs in development, testing, and production.
- **Faster onboarding**: A new engineer can get a complex multi-service application running locally with a single command, instead of a multi-page setup guide that's perpetually out of date.
- **Simplified deployments**: Deploying becomes "ship this container image" rather than "carefully replicate this server's configuration by hand."
- **Scalability**: Container orchestration tools (like Kubernetes) can start, stop, and replicate containers automatically based on demand — a foundation of modern cloud infrastructure.
- **Isolation**: Multiple applications with conflicting dependencies (e.g., one needs Python 3.9, another needs 3.12) can run side-by-side on the same machine without interfering with each other.

---

## 5. Common Beginner Mistakes

1. **Confusing images and containers.** An image is the static, reusable blueprint; a container is a live, running instance of that blueprint.
2. **Copying the entire codebase before installing dependencies**, breaking Docker's layer caching and making every rebuild painfully slow — even when only application code (not dependencies) changed.
3. **Hardcoding secrets (API keys, passwords) directly into a Dockerfile or image.** These get baked into the image and can leak. Secrets should be injected at runtime via environment variables or a secrets manager — never committed into an image (or a git repo, for that matter).
4. **Assuming Docker automatically makes an application "production-ready."** Docker solves *consistency and packaging* — it doesn't replace good architecture, monitoring, security practices, or thoughtful scaling decisions.
5. **Thinking you need to be a "DevOps expert" to discuss Docker intelligently in an interview.** A clear, high-level grasp of *what problem it solves and why* is exactly what's expected of a backend engineer — deep orchestration expertise is a separate specialty.

---

## 6. Likely Interview Questions & Strong Answers

**Q: What problem does Docker solve?**

> *"It solves the 'works on my machine' problem — where an application behaves differently across environments because of subtle differences in installed versions, configuration, or system libraries. Docker packages an application together with everything it needs to run into a single, portable image, so it behaves consistently whether it's running on a developer's laptop, a test server, or in production."*

**Q: What's the difference between a container and a virtual machine?**

> *"A virtual machine virtualizes hardware and runs a complete guest operating system on top of a hypervisor — which gives strong isolation but is heavy and slow to start. A container virtualizes at the operating-system level, sharing the host machine's kernel while keeping the application and its dependencies isolated — making containers dramatically lighter, faster to start, and more efficient to run many of side by side."*

**Q: Walk me through what a Dockerfile does, at a high level.**

> *"It's a recipe for building an image. Typically you start from a base image with the right runtime already installed, set a working directory, copy in dependency files and install them — ideally before copying the rest of the code, so Docker can cache that layer and speed up rebuilds — then copy the application code and define the command that runs when a container starts. The result is a self-contained, portable package that runs the same way anywhere Docker is installed."*

---

## 7. Tips From a Senior Interviewer

- **You don't need to recite every Docker CLI flag.** What matters is being able to clearly explain the *problem* Docker solves and the *shape* of how it solves it — that's what a backend engineering interview is actually probing for (DevOps-specialist roles go far deeper).
- **Anchor your answer in a relatable scenario**: "Imagine onboarding a new teammate — instead of a three-page setup guide that's always slightly out of date, they run one command and have the exact same environment as everyone else." Concrete scenarios like this land far better than abstract definitions.
- **If you haven't used Docker hands-on, say so honestly — and pivot to what you *do* understand.** *"I haven't deployed with Docker myself yet, but I understand the core idea: packaging an app with its dependencies into a portable, consistent unit, and that containers are lighter than VMs because they share the host OS kernel."* This is a complete, honest, and strong answer — see Lesson 13 for why honesty plus understanding consistently beats bluffing.

---

## 8. Summary

- **Docker solves the "works on my machine" problem** by packaging an application with everything it needs into a portable, consistent unit.
- **Containers** are lighter than **virtual machines** because they share the host OS's kernel rather than running a full guest OS each — faster to start, less resource-hungry.
- A **Dockerfile** is a recipe for building an **image**; a **container** is a running instance of that image — same relationship as a class and an object (Lesson 06).
- Companies use Docker for **consistency, faster onboarding, simpler deployments, scalability, and isolation** — not as a replacement for good architecture and practices, but as a foundation that makes them easier to deliver reliably.

**Next up →** [Lesson 11: Git & Commit Messages](11-git-and-commit-messages.md), where we shift from *running* code to *collaborating* on it as a team.
