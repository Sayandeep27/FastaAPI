# 🧩 Pydantic `Field()` and Validators – Complete & Simple Guide

A **professional, GitHub-ready README.md** explaining **`Field()`** and **different types of validators in Pydantic**, written in **simple language**, with **clear rules, examples, and mental models**.

---

## 🎯 Why This Topic Matters

In real applications:

* Data is messy
* Users make mistakes
* APIs send wrong values

Pydantic protects your code using:

1. **`Field()`** → rules for individual fields
2. **Validators** → custom logic when rules are not enough

---

## 🧠 First: What is `Field()`?

`Field()` is used to **add extra information and constraints** to a model field.

Think of it as:

> **Rules + metadata attached to a variable**

---

## 🧱 Basic Example of `Field()`

```python
from pydantic import BaseModel, Field

class User(BaseModel):
    username: str = Field(..., min_length=3, max_length=20)
    age: int = Field(..., ge=18, le=60)
```

### What `Field()` Does Here

| Field      | Rule                      |
| ---------- | ------------------------- |
| `username` | Must be 3–20 characters   |
| `age`      | Must be between 18 and 60 |
| `...`      | Field is required         |

---

## 🔍 Meaning of `...` (Ellipsis)

```python
Field(...)
```

Means:

> **This field MUST be provided**

Without it, Pydantic may treat the field as optional.

---

## 🎛️ Common `Field()` Constraints

| Constraint   | Applies To | Meaning               |
| ------------ | ---------- | --------------------- |
| `min_length` | str        | Minimum characters    |
| `max_length` | str        | Maximum characters    |
| `ge`         | int, float | Greater than or equal |
| `le`         | int, float | Less than or equal    |
| `gt`         | int, float | Greater than          |
| `lt`         | int, float | Less than             |
| `regex`      | str        | Pattern matching      |
| `default`    | all        | Default value         |

---

## 📌 Field Metadata (for APIs & Docs)

```python
class User(BaseModel):
    username: str = Field(
        ...,
        description="Unique username",
        example="john_doe"
    )
```

Used heavily in **FastAPI docs (Swagger UI)**.

---

## 🚨 When `Field()` Is NOT Enough

`Field()` can only enforce **simple rules**.

It cannot:

* Check email format deeply
* Compare two fields
* Apply business logic

That’s where **validators** come in.

---

# 🧪 Validators in Pydantic

Validators allow you to write **custom Python logic** to validate data.

---

## 🧩 Types of Validators

Pydantic provides **three main types**:

1. **Field Validators**
2. **Root Validators**
3. **Pre vs Post Validators**

---

## 1️⃣ Field Validators (Most Common)

Used to validate **a single field**.

### Example

```python
from pydantic import BaseModel, validator

class User(BaseModel):
    email: str

    @validator("email")
    def validate_email(cls, value):
        if "@" not in value:
            raise ValueError("Invalid email")
        return value
```

### What Happens

* Value enters
* Validator runs
* Either accepts or raises error

---

## 🧠 Important Rule

> **Validators must always return the value**

Otherwise, the field becomes `None`.

---

## 2️⃣ Multiple Field Validators

```python
class User(BaseModel):
    username: str
    email: str

    @validator("username")
    def no_spaces(cls, v):
        if " " in v:
            raise ValueError("No spaces allowed")
        return v
```

---

## 3️⃣ Root Validators (Whole Model Validation)

Used when validation depends on **multiple fields**.

### Example

```python
from pydantic import root_validator

class Payment(BaseModel):
    amount: float
    currency: str

    @root_validator
    def check_payment(cls, values):
        if values["amount"] <= 0:
            raise ValueError("Amount must be positive")
        return values
```

### Use Cases

* Password + confirm password
* Start date < end date
* Amount vs currency logic

---

## 4️⃣ Pre Validators vs Post Validators

### Pre Validator

Runs **before type conversion**.

```python
class User(BaseModel):
    age: int

    @validator("age", pre=True)
    def clean_age(cls, v):
        return int(v)
```

Used when:

* Input is messy
* You want to clean raw data

---

### Post Validator (Default)

Runs **after type conversion**.

```python
@validator("age")
def check_age(cls, v):
    if v < 18:
        raise ValueError("Too young")
    return v
```

---

## 5️⃣ Always Validators

Runs even if the value is missing.

```python
@validator("age", always=True)
def default_age(cls, v):
    return v or 18
```

---

## 🆚 Field vs Validators (Very Important Table)

| Feature            | Field() | Validator |
| ------------------ | ------- | --------- |
| Simple constraints | ✅       | ❌         |
| Complex logic      | ❌       | ✅         |
| Multiple fields    | ❌       | ✅ (root)  |
| Regex / length     | ✅       | ❌         |
| Business rules     | ❌       | ✅         |

---

## 🧠 Mental Model (Lock This In)

* **`Field()`** → Static rules
* **Validators** → Dynamic logic

Together, they create **bulletproof validation**.

---

## 🏗️ Real-World Usage Pattern

```python
class User(BaseModel):
    username: str = Field(..., min_length=3)
    age: int = Field(..., ge=18)
    email: str

    @validator("email")
    def validate_email(cls, v):
        if "@" not in v:
            raise ValueError("Invalid email")
        return v
```

This is **production-grade validation**.

---

## ✅ Final Takeaway

* Use **`Field()`** for simple constraints
* Use **validators** for complex logic
* Use **root validators** for cross-field checks

If you master these, you master **Pydantic validation**.

---

📌 **This README is ready to copy, edit, and download for GitHub use.**
