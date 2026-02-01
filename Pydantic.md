# 🧩 Pydantic – Complete Beginner to Advanced Guide

A **professional, GitHub‑ready README.md** explaining **Pydantic from scratch to advanced level** with clear concepts, examples, and real‑world usage.

---

## 📌 What is Pydantic?

**Pydantic** is a Python library used for **data validation, parsing, and settings management** using Python **type hints**.

In simple terms:

> Pydantic ensures that the data your program receives is **correct, clean, typed, and safe** before your logic runs.

It is widely used in:

* APIs (FastAPI)
* Machine Learning pipelines
* Configuration management
* CI/CD systems
* Data engineering workflows

---

## ❓ Why Do We Need Pydantic?

### 🔴 Problem Without Pydantic

```python
def create_user(name, age):
    print(name.upper())
    print(age + 1)

create_user("Alice", "20")
```

❌ This crashes at runtime because `age` is a string.

---

### 🟢 Solution With Pydantic

```python
from pydantic import BaseModel

class User(BaseModel):
    name: str
    age: int

user = User(name="Alice", age="20")
print(user)
```

✅ Output:

```
name='Alice' age=20
```

**Pydantic automatically validates and converts types.**

---

## 🧠 Core Concept: BaseModel

All Pydantic models inherit from **BaseModel**.

```python
from pydantic import BaseModel

class Person(BaseModel):
    name: str
    age: int
```

### What BaseModel Does Internally

* Reads type hints
* Validates input data
* Converts compatible types
* Stores validated data as attributes

---

## 🔄 Type Validation & Type Coercion

### Automatic Type Conversion

```python
class Product(BaseModel):
    price: float
    quantity: int

p = Product(price="99.5", quantity="2")
print(p)
```

Output:

```
price=99.5 quantity=2
```

---

### Invalid Data Example

```python
Product(price="abc", quantity=2)
```

❌ Error:

```
value is not a valid float
```

---

## 🧱 Common Field Types

```python
from typing import List, Optional, Dict
from pydantic import BaseModel

class Order(BaseModel):
    id: int
    items: List[str]
    discount: Optional[float] = None
    metadata: Dict[str, str]
```

### Supported Types

| Category    | Examples               |
| ----------- | ---------------------- |
| Primitive   | int, float, str, bool  |
| Collections | List, Set, Tuple, Dict |
| Optional    | Optional[T]            |
| Date/Time   | datetime, date, time   |
| Custom      | Nested models          |

---

## 🧩 Required vs Optional Fields

```python
class User(BaseModel):
    name: str              # Required
    age: Optional[int]     # Optional
```

```python
User(name="Bob")      # ✅ Valid
User(age=30)           # ❌ Error (name missing)
```

---

## 🎯 Default Values

```python
class User(BaseModel):
    username: str
    is_active: bool = True
```

If `is_active` is not provided, it defaults to `True`.

---

## 🎛️ Field Customization with Field()

```python
from pydantic import BaseModel, Field

class User(BaseModel):
    username: str = Field(..., min_length=3, max_length=20)
    age: int = Field(..., ge=18, le=60)
```

### Common Constraints

| Constraint | Meaning                   |
| ---------- | ------------------------- |
| min_length | Minimum string length     |
| max_length | Maximum string length     |
| ge / le    | Greater / Less than equal |
| gt / lt    | Greater / Less than       |
| regex      | Pattern validation        |

---

## 🚨 Validation Errors (Structured)

```python
try:
    User(username="ab", age=15)
except Exception as e:
    print(e)
```

Pydantic provides:

* Field name
* Error type
* Clear explanation

Perfect for APIs and debugging.

---

## 🧪 Custom Field Validators

Used when built‑in validation is insufficient.

```python
from pydantic import validator

class User(BaseModel):
    email: str

    @validator("email")
    def validate_email(cls, v):
        if "@" not in v:
            raise ValueError("Invalid email")
        return v
```

---

## 🔗 Root Validators (Multiple Fields)

```python
from pydantic import root_validator

class Payment(BaseModel):
    amount: float
    currency: str

    @root_validator
    def check_amount(cls, values):
        if values["amount"] <= 0:
            raise ValueError("Amount must be positive")
        return values
```

---

## 🧬 Nested Models

```python
class Address(BaseModel):
    city: str
    pincode: int

class User(BaseModel):
    name: str
    address: Address
```

```python
User(
    name="Alice",
    address={"city": "Delhi", "pincode": 110001}
)
```

---

## 📚 Lists of Nested Models

```python
class Item(BaseModel):
    name: str
    price: float

class Order(BaseModel):
    items: list[Item]
```

---

## 🔁 Serialization (dict & json)

```python
user.dict()
user.json()
```

### Excluding Fields

```python
user.dict(exclude={"age"})
```

---

## 🏷️ Field Aliases (API Compatibility)

```python
class User(BaseModel):
    user_id: int = Field(..., alias="userId")

User(userId=101)
```

---

## ⚙️ Model Configuration (Config)

```python
class User(BaseModel):
    name: str

    class Config:
        allow_population_by_field_name = True
        anystr_strip_whitespace = True
```

### Useful Config Options

| Option              | Purpose                      |
| ------------------- | ---------------------------- |
| extra="forbid"      | Reject unknown fields        |
| validate_assignment | Validate on attribute update |
| orm_mode            | ORM compatibility            |

---

## 🗄️ ORM Mode (SQLAlchemy Support)

```python
class User(BaseModel):
    id: int
    name: str

    class Config:
        orm_mode = True
```

```python
User.from_orm(sqlalchemy_user)
```

---

## 🔐 Settings Management (BaseSettings)

```python
from pydantic import BaseSettings

class Settings(BaseSettings):
    database_url: str
    debug: bool = False

settings = Settings()
```

Automatically reads:

* Environment variables
* `.env` file

---

## 🚫 Strict Types

```python
from pydantic import StrictInt

class Data(BaseModel):
    value: StrictInt
```

```python
Data(value="10")  # ❌ Rejected
```

---

## 🧠 Error Handling Best Practice

```python
from pydantic import ValidationError

try:
    User(username="a", age=10)
except ValidationError as e:
    print(e.errors())
```

Returns structured, machine‑readable errors.

---

## 🏗️ Real‑World Use Cases

| Domain           | Usage                     |
| ---------------- | ------------------------- |
| FastAPI          | Request & response models |
| ML               | Input validation          |
| CI/CD            | Env config validation     |
| Data Engineering | Schema enforcement        |
| LangChain        | Prompt & tool schemas     |

---

## 🆚 Pydantic vs Dataclasses

| Feature        | Dataclass | Pydantic |
| -------------- | --------- | -------- |
| Validation     | ❌         | ✅        |
| Type coercion  | ❌         | ✅        |
| Error messages | ❌         | ✅        |
| API Ready      | ❌         | ✅        |

---

## 🧠 Mental Model (Very Important)

> Think of Pydantic as a **gatekeeper** that cleans and validates data **before** your code touches it.

---

## ✅ When You Should Use Pydantic

Use Pydantic when:

* Data comes from users, APIs, files, or env vars
* You want strict correctness
* You build APIs, ML pipelines, or automation tools

---

## 🎯 Final Takeaway

Pydantic brings:

* Safety
* Clarity
* Reliability
* Professional‑grade data handling

It is **mandatory knowledge** for modern Python developers.

---

📌 **You can now download this README.md directly and use it in your GitHub repositories.**
