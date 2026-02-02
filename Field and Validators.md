# 📘 Pydantic Fields & Validators – Complete Guide

A **professional, GitHub‑ready README** explaining **Field**, **validators**, and **all types of validators in Pydantic** with **clear examples, tables, and real‑world code**.

---

## 📌 Table of Contents

1. What is Pydantic?
2. What is `Field`?
3. Why `Field` is Needed
4. Common `Field` Parameters
5. What are Validators?
6. Types of Validators

   * Field Validator
   * Pre Validator
   * Post Validator
   * Root Validator
   * Pre Root Validator
7. Field vs Validator (Comparison)
8. Validation Execution Order
9. Complete Real‑World Example
10. Key Takeaways

---

## 1️⃣ What is Pydantic?

**Pydantic** is a Python library used for **data validation, parsing, and settings management** using Python type hints.

It ensures that:

* Input data is **correctly typed**
* Invalid data is **rejected early**
* Data structures are **safe and predictable**

```python
from pydantic import BaseModel

class User(BaseModel):
    name: str
    age: int
```

---

## 2️⃣ What is `Field`?

### Definition

`Field` is used to **add constraints, metadata, and configuration** to model attributes.

Without `Field`, a field only has:

* a type
* an optional default value

With `Field`, you can define:

* validation rules
* required fields
* string constraints
* numeric ranges
* documentation metadata

---

## 3️⃣ Why `Field` is Needed

### ❌ Without Field

```python
class User(BaseModel):
    age: int
```

Problems:

* Negative values allowed
* No range checking

### ✅ With Field

```python
from pydantic import BaseModel, Field

class User(BaseModel):
    age: int = Field(..., ge=18, le=60)
```

✔ Age must be between **18 and 60**

---

## 4️⃣ Common `Field` Parameters

| Parameter         | Description           |
| ----------------- | --------------------- |
| `...`             | Required field        |
| `default`         | Default value         |
| `ge`              | Greater than or equal |
| `gt`              | Greater than          |
| `le`              | Less than or equal    |
| `lt`              | Less than             |
| `min_length`      | Minimum string length |
| `max_length`      | Maximum string length |
| `regex`           | Regex validation      |
| `description`     | Schema documentation  |
| `example`         | Example value         |
| `alias`           | Alternate field name  |
| `default_factory` | Dynamic default       |

---

### String Validation Example

```python
class User(BaseModel):
    username: str = Field(
        ...,
        min_length=3,
        max_length=20,
        regex="^[a-zA-Z0-9_]+$"
    )
```

---

## 5️⃣ What are Validators?

### Definition

Validators are **custom functions** that automatically validate or transform data.

Use validators when:

* `Field` is not enough
* You need custom business logic
* Validation depends on multiple fields
* Input needs transformation

---

## 6️⃣ Types of Validators

---

### 🔹 1. Field Validator

Used to validate **a single field** after type conversion.

```python
from pydantic import BaseModel, validator

class User(BaseModel):
    email: str

    @validator("email")
    def validate_email(cls, value):
        if not value.endswith("@company.com"):
            raise ValueError("Email must be a company email")
        return value
```

---

### 🔹 2. Pre Validator (`pre=True`)

Runs **before type conversion**.

Used for:

* Cleaning raw input
* Converting strings

```python
class User(BaseModel):
    age: int

    @validator("age", pre=True)
    def convert_age(cls, value):
        if isinstance(value, str):
            if not value.isdigit():
                raise ValueError("Age must be numeric")
            return int(value)
        return value
```

---

### 🔹 3. Post Validator (Default)

Runs **after type conversion**.

```python
class Product(BaseModel):
    price: float

    @validator("price")
    def check_price(cls, value):
        if value <= 0:
            raise ValueError("Price must be positive")
        return value
```

---

### 🔹 4. Root Validator

Used for **cross‑field validation**.

```python
from pydantic import BaseModel, root_validator

class User(BaseModel):
    password: str
    confirm_password: str

    @root_validator
    def passwords_match(cls, values):
        if values.get("password") != values.get("confirm_password"):
            raise ValueError("Passwords do not match")
        return values
```

---

### 🔹 5. Pre Root Validator (`pre=True`)

Runs **before individual field validation**.

```python
@root_validator(pre=True)
def inspect_raw_input(cls, values):
    print(values)
    return values
```

---

## 7️⃣ Field vs Validator – Comparison

| Feature                | Field | Validator |
| ---------------------- | ----- | --------- |
| Simple constraints     | ✅     | ❌         |
| Complex logic          | ❌     | ✅         |
| Cross‑field validation | ❌     | ✅         |
| Data transformation    | ❌     | ✅         |
| Input cleaning         | ❌     | ✅         |

---

## 8️⃣ Validation Execution Order

Validation happens in this order:

1. Pre validators
2. Type conversion
3. Field validators
4. Root validators
5. Model creation

If any step fails → `ValidationError`

---

## 9️⃣ Complete Real‑World Example

```python
from pydantic import BaseModel, Field, validator, root_validator

class User(BaseModel):
    username: str = Field(..., min_length=3)
    age: int = Field(..., ge=18)
    email: str
    password: str
    confirm_password: str

    @validator("email")
    def validate_email(cls, v):
        if not v.endswith("@company.com"):
            raise ValueError("Invalid company email")
        return v

    @validator("age", pre=True)
    def parse_age(cls, v):
        return int(v)

    @root_validator
    def check_passwords(cls, values):
        if values.get("password") != values.get("confirm_password"):
            raise ValueError("Passwords do not match")
        return values
```

---

## 🔑 Key Takeaways

* Use **Field** for simple constraints
* Use **validators** for custom logic
* Use **root validators** for cross‑field rules
* `pre=True` runs **before type casting**
* Validators must always **return values**

---

## ⭐ Recommended Usage

* APIs (FastAPI)
* Configuration validation
* Data ingestion pipelines
* Request/response schemas

---

📌 **This README is production‑ready and suitable for direct GitHub upload.**

Happy building 🚀
