# Lesson 07 — FastAPI

> **Where this fits:** We now know how to model data (Lessons 03–05) and write clean Python (Lesson 06). **FastAPI** is where it all comes together — a modern Python web framework for building APIs, and a very common choice for backend interview take-home tasks and live coding rounds.

---

## 1. What is an API, and why FastAPI?

Quick refresher: an **API** (Application Programming Interface) is how different pieces of software talk to each other. A **REST API** exposes resources (like `users`, `orders`) over HTTP, using standard verbs:

| HTTP Method | Typical meaning | Example |
|-------------|----------------|---------|
| `GET` | Retrieve data | `GET /users/1` → fetch user with ID 1 |
| `POST` | Create new data | `POST /users` → create a new user |
| `PUT` / `PATCH` | Update existing data | `PATCH /users/1` → update user 1 |
| `DELETE` | Remove data | `DELETE /users/1` → delete user 1 |

### Why FastAPI specifically?
- **Speed of development**: minimal boilerplate to get a working API.
- **Automatic data validation** via **Pydantic** (more below) — catches bad input before it ever reaches your business logic.
- **Automatic interactive documentation** (Swagger UI / ReDoc), generated from your code — a huge productivity win for teams.
- **Built on modern Python**: uses type hints (which you now know from Lesson 06!) as the foundation for validation *and* documentation — one source of truth, less duplication.
- **Async support out of the box** — important for I/O-heavy workloads like backend APIs (more below).

---

## 2. Request Validation with Pydantic

**Pydantic** is a data-validation library. In FastAPI, you define the *shape* of expected request/response data as Python classes — and FastAPI automatically validates incoming data against them, returning a clear error if it doesn't match.

```python
from pydantic import BaseModel, EmailStr, Field

class UserCreate(BaseModel):
    username: str = Field(min_length=3, max_length=30)
    email: EmailStr
    age: int = Field(gt=0, lt=120)
```

**Line-by-line explanation:**
- `class UserCreate(BaseModel):` — defines a schema describing what a "create user" request must look like, by inheriting from Pydantic's `BaseModel`.
- `username: str = Field(min_length=3, max_length=30)` — `username` must be a string between 3 and 30 characters.
- `email: EmailStr` — must be a syntactically valid email address (Pydantic validates the format for you).
- `age: int = Field(gt=0, lt=120)` — must be an integer greater than 0 and less than 120.

If a client sends `{"username": "ab", "email": "not-an-email", "age": 200}`, FastAPI/Pydantic automatically rejects it with a structured `422 Unprocessable Entity` response explaining *exactly* what's wrong with each field — **before your code even runs**. This is a massive win: your business logic never has to defensively check "is this even a valid email?" — that's already guaranteed by the time your function is called.

---

## 3. CRUD API Example

Let's build a minimal but complete CRUD (Create, Read, Update, Delete) API for managing books.

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

app = FastAPI()

# --- Schemas (the "shape" of data going in and out) ---
class BookCreate(BaseModel):
    title: str
    author: str
    price: float

class Book(BookCreate):
    id: int

# --- Fake "database" for this example (a simple in-memory list) ---
fake_db: list[Book] = []
next_id = 1

# --- CREATE ---
@app.post("/books", response_model=Book, status_code=201)
def create_book(book: BookCreate):
    global next_id
    new_book = Book(id=next_id, **book.model_dump())
    fake_db.append(new_book)
    next_id += 1
    return new_book

# --- READ (list) ---
@app.get("/books", response_model=list[Book])
def list_books():
    return fake_db

# --- READ (single) ---
@app.get("/books/{book_id}", response_model=Book)
def get_book(book_id: int):
    for book in fake_db:
        if book.id == book_id:
            return book
    raise HTTPException(status_code=404, detail="Book not found")

# --- UPDATE ---
@app.put("/books/{book_id}", response_model=Book)
def update_book(book_id: int, updated: BookCreate):
    for index, book in enumerate(fake_db):
        if book.id == book_id:
            fake_db[index] = Book(id=book_id, **updated.model_dump())
            return fake_db[index]
    raise HTTPException(status_code=404, detail="Book not found")

# --- DELETE ---
@app.delete("/books/{book_id}", status_code=204)
def delete_book(book_id: int):
    for index, book in enumerate(fake_db):
        if book.id == book_id:
            fake_db.pop(index)
            return
    raise HTTPException(status_code=404, detail="Book not found")
```

**Line-by-line explanation of the key parts:**
- `app = FastAPI()` — creates the application instance; this object is what wires everything together.
- `class BookCreate(BaseModel)` / `class Book(BookCreate)` — `Book` *inherits* from `BookCreate` and adds an `id` field. This avoids repeating `title`, `author`, `price` — the response model is "everything in `BookCreate`, plus an ID."
- `@app.post("/books", response_model=Book, status_code=201)` — a **decorator** registering this function to handle `POST` requests to `/books`. `response_model=Book` tells FastAPI to validate *and document* the shape of the response; `status_code=201` is the conventional "Created" status.
- `def create_book(book: BookCreate):` — FastAPI sees the `book: BookCreate` type hint and automatically: (1) parses the incoming JSON body, (2) validates it against `BookCreate`, (3) hands you a ready-to-use Python object. No manual parsing required.
- `**book.model_dump()` — unpacks the validated data's fields as keyword arguments when constructing the full `Book` object (this is standard Python dictionary-unpacking syntax).
- `@app.get("/books/{book_id}")` — `{book_id}` is a **path parameter**; FastAPI extracts it from the URL and, because the function parameter is typed `book_id: int`, automatically converts and validates it as an integer.
- `raise HTTPException(status_code=404, detail="Book not found")` — FastAPI's standard way to return a proper HTTP error response with a clear message — much better than returning `None` and leaving the client to guess what went wrong.

> 💡 In a real project, `fake_db` would be replaced with actual database queries via an ORM (Lesson 05) — but the *shape* of the endpoint logic (validate → look up → act → respond) stays the same. This is intentionally simplified so you can focus on how FastAPI wires requests to functions.

---

## 4. Error Handling

Good error handling means **clients always get a clear, predictable response — never a confusing crash.**

```python
from fastapi import HTTPException

@app.get("/books/{book_id}")
def get_book(book_id: int):
    book = find_book_by_id(book_id)
    if book is None:
        raise HTTPException(
            status_code=404,
            detail=f"Book with id {book_id} was not found"
        )
    return book
```

**Why this matters:** Without this check, trying to access an attribute on `None` would cause an unhandled `AttributeError`, returning a generic, unhelpful `500 Internal Server Error` — telling the client (and you, when debugging logs later) nothing useful. The explicit check turns a confusing crash into a clear, actionable `404 Not Found` with a helpful message.

A well-designed API uses HTTP status codes *meaningfully*:
- `200 OK` — success
- `201 Created` — resource successfully created
- `204 No Content` — success, nothing to return (common for `DELETE`)
- `400 Bad Request` — the client sent invalid data
- `401 Unauthorized` / `403 Forbidden` — authentication/permission issues
- `404 Not Found` — the resource doesn't exist
- `422 Unprocessable Entity` — validation failed (FastAPI returns this automatically for Pydantic errors)
- `500 Internal Server Error` — something broke on the server's side (you generally want to *avoid* triggering these through unhandled bugs — they should be rare and investigated, not routine)

---

## 5. Async Basics

You'll often see FastAPI route functions defined with `async def`:

```python
@app.get("/books/{book_id}")
async def get_book(book_id: int):
    book = await fetch_book_from_database(book_id)
    return book
```

### What `async`/`await` actually means (in plain terms)
Backend servers spend a *lot* of time **waiting** — waiting for a database query to return, waiting for an external API call to respond, waiting for a file to be read. Traditional ("synchronous") code blocks entirely during that wait — the server can't do anything else until the operation finishes.

**`async`/`await`** lets the server say: *"While I'm waiting on this slow operation, let me go handle other incoming requests — and come back to this one the moment its data is ready."* This allows a single process to juggle many in-flight requests efficiently, which is hugely valuable for I/O-heavy workloads like web APIs (lots of waiting on databases and external services, relatively little raw computation).

- `async def` marks a function as asynchronous — it *can* be paused and resumed.
- `await` marks a point where the function pauses, handing control back, until the awaited operation completes.

> ⚠️ **Important nuance for interviews:** `async` doesn't make a *single* operation faster — it makes the *server* better at handling *many* operations concurrently by not sitting idle during waits. And mixing `async def` with a *blocking* (synchronous) call inside it can actually make things *worse*, because that blocking call freezes the entire event loop — defeating the purpose. If you're not 100% sure whether a library call is async-compatible, it's fine to say so plainly: *"I'd check whether the database driver supports async — if not, I'd either use a sync route function or run the blocking call in a thread pool, since mixing them carelessly can hurt performance rather than help it."* That answer demonstrates real understanding, not just keyword familiarity.

---

## 6. Common Beginner Mistakes

1. **Skipping Pydantic validation and manually parsing request bodies** — reinventing a wheel that FastAPI already gives you for free, and almost certainly worse.
2. **Returning raw database/ORM objects directly** instead of defining response schemas — leaking internal fields (like password hashes!) that clients should never see. Always define explicit response models.
3. **Using `async def` "because it looks modern"** without understanding whether the operations inside are actually async-compatible — see the warning above.
4. **Returning `None` or a bare string on error** instead of raising a proper `HTTPException` with a meaningful status code — leaving API clients to guess what went wrong.
5. **Putting business logic directly inside route functions.** As an app grows, route functions should mostly *coordinate* — call validation, call a service/business-logic function, call the database layer, return a response — rather than containing all the logic themselves (this connects back to "single responsibility" from Lesson 06).

---

## 7. Likely Interview Questions & Strong Answers

**Q: Why might you choose FastAPI over another framework?**

> *"FastAPI gives you automatic request validation and interactive documentation directly from Python type hints — which means less boilerplate and fewer chances for the code and the docs to drift out of sync. It also has first-class async support, which fits well with how much backend work involves waiting on databases and external services rather than heavy computation."*

**Q: How does FastAPI validate incoming request data?**

> *"Through Pydantic models — you define the expected shape and constraints of the data as a class with typed fields, and FastAPI automatically parses incoming JSON against that model. If the data doesn't match — wrong type, missing field, value out of range — it returns a structured 422 error explaining exactly what's wrong, before your route function ever runs. That keeps validation logic out of your business code entirely."*

**Q: What's the difference between `def` and `async def` route functions in FastAPI, and when would you use each?**

> *"`async def` lets a function pause at `await` points — like waiting on a database query — so the server can handle other requests in the meantime, which is great for I/O-heavy workloads. I'd use it when the operations inside are async-compatible, like an async database driver. If I'm calling something synchronous/blocking, I'd either keep the route as a regular `def` — FastAPI runs those in a thread pool automatically — or be careful not to block the async event loop, since that can hurt performance rather than help it."*

---

## 8. Tips From a Senior Interviewer

- **Walk through the request lifecycle out loud** when explaining an endpoint: "the request comes in, FastAPI validates it against this schema, then we look up the resource, handle the not-found case, and return a typed response." This narrative structure is exactly what interviewers are listening for.
- **Mention validation and error handling unprompted.** Many juniors write the "happy path" only; proactively discussing what happens when things go wrong is a strong differentiator.
- **If asked to design an endpoint live, start by defining the Pydantic schema first.** This shows you think about the *contract* (what data goes in, what comes out) before the implementation — exactly how senior engineers approach API design.

---

## 9. Summary

- **FastAPI** is a modern Python web framework that uses type hints to provide automatic request validation (via **Pydantic**) and interactive documentation.
- **CRUD endpoints** map HTTP verbs (`GET`, `POST`, `PUT`, `DELETE`) to operations on resources — define schemas, write route functions, raise `HTTPException` for errors.
- **Error handling** means returning clear, meaningful HTTP status codes and messages — never letting clients receive confusing crashes.
- **`async`/`await`** lets a server handle many in-flight requests efficiently by not blocking on slow I/O — but only helps when the operations inside are genuinely async-compatible.

**Next up →** [Lesson 08: Testing — Unit, Integration, E2E](08-testing-unit-integration-e2e.md), where we learn how to *prove* that the code we just wrote actually works — and keeps working as it changes.
