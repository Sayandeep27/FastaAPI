# FastAPI – Complete Guide (From Basics to Production)

> **A professional, GitHub‑ready README covering FastAPI from fundamentals to advanced concepts, interview explanations, and real‑world patterns.**
>
> This document is designed to **clear all conceptual confusion** and help you **understand FastAPI deeply**, not just memorize syntax.

---

## Table of Contents

1. What is FastAPI?
2. What is FastAPI Built On?
3. Why FastAPI is Fast
4. WSGI vs ASGI (Core Foundation)
5. Request–Response Lifecycle
6. Route Definitions
7. Types of Request Parameters
8. Path vs Query Parameters
9. Pydantic in FastAPI (Deep Dive)
10. Validation & Error Handling
11. Response Models & Serialization
12. Dependency Injection (The Secret Sauce)
13. DB Connection Handling Pattern
14. Middleware in FastAPI
15. Dependency vs Middleware
16. Async def vs def (Very Important)
17. Background Tasks
18. WebSockets in FastAPI
19. Authentication & Security Basics
20. CORS Handling
21. Testing FastAPI Applications
22. Docker with FastAPI
23. Deployment Overview
24. Best Practices
25. Scalability & Performance Tips
26. Complete Mental Model (Final Summary)

---

## 1. What is FastAPI?

FastAPI is a **modern Python web framework** used to build APIs quickly, safely, and with high performance.

### Key Characteristics

* Built for **API‑first development**
* Uses **Python type hints** as executable instructions
* Automatic validation, serialization, and documentation
* Native async support

FastAPI turns Python code into:

* A working API
* Validated input pipeline
* OpenAPI documentation

All at the same time.

---

## 2. What is FastAPI Built On?

FastAPI is **not a single framework**. It is a **carefully composed system** built on two major pillars:

### Starlette

Responsible for:

* ASGI request/response handling
* Routing
* Middleware
* WebSockets
* Background tasks

### Pydantic

Responsible for:

* Data validation
* Type conversion
* Serialization & deserialization
* Schema generation

### Why This Matters

FastAPI focuses only on **developer experience** and **API design**, while delegating core responsibilities to proven libraries.

---

## 3. Why FastAPI is Fast

FastAPI is fast because:

| Layer     | Responsibility           |
| --------- | ------------------------ |
| Uvicorn   | ASGI server + event loop |
| Starlette | Networking & middleware  |
| FastAPI   | Validation + DI + docs   |
| Pydantic  | Optimized parsing        |

FastAPI itself is thin — that is intentional.

---

## 4. WSGI vs ASGI

### WSGI (Old)

* Synchronous
* One request blocks one worker
* No WebSockets

### ASGI (Modern)

* Asynchronous
* Event‑driven
* Supports WebSockets, SSE

FastAPI is **ASGI‑native**, which enables high concurrency.

---

## 5. Request–Response Lifecycle

Step‑by‑step flow:

1. Client sends HTTP request
2. Uvicorn receives it
3. Starlette routes request
4. Middleware runs
5. FastAPI validates input
6. Dependencies resolved
7. Endpoint function executes
8. Response serialized
9. Client receives JSON

If validation fails → request never reaches your function.

---

## 6. Route Definitions

```python
from fastapi import FastAPI
app = FastAPI()

@app.get("/")
def root():
    return {"message": "Hello World"}
```

FastAPI reads:

* HTTP method
* Path
* Function signature
* Type hints

And builds the API automatically.

---

## 7. Types of Request Parameters

FastAPI determines parameter source by **context**, not magic.

### Path Parameters

```python
@app.get("/users/{user_id}")
def get_user(user_id: int):
    return user_id
```

### Query Parameters

```python
@app.get("/items")
def items(skip: int = 0, limit: int = 10):
    return {"skip": skip, "limit": limit}
```

### Request Body

```python
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    price: float

@app.post("/items")
def create_item(item: Item):
    return item
```

### Headers

```python
from fastapi import Header

@app.get("/info")
def info(user_agent: str = Header(None)):
    return user_agent
```

### Cookies

```python
from fastapi import Cookie

@app.get("/session")
def session(session_id: str = Cookie(None)):
    return session_id
```

---

## 8. Path vs Query Parameters

| Feature  | Path              | Query                 |
| -------- | ----------------- | --------------------- |
| Required | Yes               | Optional by default   |
| Location | URL path          | After ?               |
| Use case | Resource identity | Filtering, pagination |

---

## 9. Pydantic in FastAPI (Deep Dive)

Pydantic powers:

* Input validation
* Type coercion
* Output formatting

### Example

```python
class User(BaseModel):
    name: str
    age: int
```

Input:

```json
{ "name": "John", "age": "25" }
```

Converted to:

```python
User(name="John", age=25)
```

Invalid input automatically raises 422 error.

---

## 10. Validation & Error Handling

### HTTPException

```python
from fastapi import HTTPException

@app.get("/items/{item_id}")
def get_item(item_id: int):
    if item_id != 1:
        raise HTTPException(status_code=404, detail="Item not found")
    return {"item": item_id}
```

### Custom Validation

```python
from pydantic import validator

class User(BaseModel):
    email: str

    @validator("email")
    def check_email(cls, v):
        if "@" not in v:
            raise ValueError("Invalid email")
        return v
```

---

## 11. Response Models & Serialization

```python
class Item(BaseModel):
    name: str
    price: float

@app.post("/items", response_model=Item)
def create_item(item: Item):
    return item
```

Ensures:

* Output format consistency
* Data hiding
* Clean contracts

---

## 12. Dependency Injection (The Secret Sauce)

Dependency Injection allows FastAPI to:

* Inject reusable logic
* Manage resources safely
* Keep endpoints clean

### Simple Dependency

```python
from fastapi import Depends

def common(q: str = None):
    return q

@app.get("/items")
def items(q=Depends(common)):
    return q
```

---

## 13. DB Connection Handling Pattern

```python
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

@app.get("/users")
def users(db=Depends(get_db)):
    return db.query(User).all()
```

Each request:

* Gets fresh DB session
* Automatically closed

---

## 14. Middleware in FastAPI

Middleware runs **for every request**.

```python
@app.middleware("http")
async def log(request, call_next):
    response = await call_next(request)
    return response
```

Use cases:

* Logging
* CORS
* Metrics

---

## 15. Dependency vs Middleware

| Dependency     | Middleware          |
| -------------- | ------------------- |
| Opt‑in         | Global              |
| Business logic | Cross‑cutting logic |
| Type‑safe      | No param access     |

---

## 16. Async def vs def

### async def

Use for:

* DB calls
* External APIs
* File I/O

```python
@app.get("/async")
async def async_call():
    data = await fetch_data()
    return data
```

### def

Use for CPU‑bound logic.

---

## 17. Background Tasks

```python
from fastapi import BackgroundTasks

def log(msg):
    print(msg)

@app.post("/notify")
def notify(bg: BackgroundTasks):
    bg.add_task(log, "done")
    return "sent"
```

Runs after response.
Not a job queue.

---

## 18. WebSockets in FastAPI

```python
from fastapi import WebSocket

@app.websocket("/ws")
async def ws(ws: WebSocket):
    await ws.accept()
    while True:
        msg = await ws.receive_text()
        await ws.send_text(msg)
```

Enabled by ASGI.

---

## 19. Authentication & Security Basics

Common methods:

* JWT
* OAuth2
* API Keys

Security is implemented using dependencies.

---

## 20. CORS Handling

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_methods=["*"],
    allow_headers=["*"],
)
```

---

## 21. Testing FastAPI Applications

```python
from fastapi.testclient import TestClient

client = TestClient(app)

def test_root():
    res = client.get("/")
    assert res.status_code == 200
```

---

## 22. Docker with FastAPI

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

---

## 23. Deployment Overview

Options:

* EC2 / VM
* Docker + ECS / Kubernetes
* Serverless (Lambda)

---

## 24. Best Practices

* Use type hints everywhere
* Use routers
* Validate inputs strictly
* Use async correctly
* Log properly

---

## 25. Scalability & Performance Tips

* Horizontal scaling
* Connection pooling
* Caching (Redis)
* Background tasks

---

## 26. Complete Mental Model (Final Summary)

FastAPI =

**ASGI + Starlette + Pydantic + Type Hints + Dependency Injection**

Once this clicks, everything else makes sense.

---

**End of Document**
