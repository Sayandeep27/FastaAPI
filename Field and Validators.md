# Pydantic Field and Validators Guide

---

# What is Pydantic

Pydantic is a Python library used for **data validation and parsing using Python type hints**.

It is widely used in:

* FastAPI
* Machine Learning APIs
* Data pipelines
* Backend applications

Pydantic automatically:

* Validates input data
* Converts data types

Example:

```python
from pydantic import BaseModel

class User(BaseModel):
    name: str
    age: int

u = User(name="John", age="25")
print(u)
```

Output:

```
name='John' age=25
```

Even though age was provided as a string, Pydantic converted it to integer.

---

# What is Field in Pydantic

`Field()` is used to **add validation rules and metadata to model fields**.

Without Field, Pydantic only validates the **data type**.

With Field, you can add:

* Required fields
* Default values
* Numeric constraints
* String constraints
* Regex validation
* Metadata

---

# Basic Example of Field

```python
from pydantic import BaseModel, Field

class User(BaseModel):
    name: str = Field(...)
    age: int = Field(...)

u = User(name="Alice", age=25)
print(u)
```

Explanation

```
...  → means required field
```

If you run:

```python
User(age=25)
```

Error:

```
ValidationError: name field required
```

---

# Why Field is Needed

Without Field:

```python
class User(BaseModel):
    age: int
```

Only **type validation** happens.

But with Field we can enforce constraints:

```python
class User(BaseModel):
    age: int = Field(gt=18, lt=60)
```

Now age must be between 18 and 60.

---

# Common Field Parameters

| Parameter  | Meaning               |
| ---------- | --------------------- |
| ...        | Required field        |
| default    | Default value         |
| gt         | Greater than          |
| ge         | Greater than or equal |
| lt         | Less than             |
| le         | Less than or equal    |
| min_length | Minimum string length |
| max_length | Maximum string length |
| regex      | Pattern validation    |

---

# Required Field Example

```python
from pydantic import BaseModel, Field

class User(BaseModel):
    name: str = Field(...)
```

Valid

```
User(name="John")
```

Invalid

```
User()
```

---

# Default Value Example

```python
class User(BaseModel):
    age: int = Field(default=18)
```

Usage

```python
User()
```

Output

```
age=18
```

---

# gt (Greater Than)

```python
class Product(BaseModel):
    price: float = Field(gt=0)
```

Valid

```
price = 100
```

Invalid

```
price = -5
```

Error

```
value must be greater than 0
```

---

# ge (Greater Than or Equal)

```python
class Product(BaseModel):
    quantity: int = Field(ge=1)
```

Valid

```
quantity = 1
quantity = 5
```

Invalid

```
quantity = 0
```

---

# lt (Less Than)

```python
class Student(BaseModel):
    age: int = Field(lt=60)
```

Invalid

```
age = 70
```

---

# le (Less Than or Equal)

```python
class Student(BaseModel):
    marks: int = Field(le=100)
```

Invalid

```
marks = 110
```

---

# min_length

```python
class User(BaseModel):
    username: str = Field(min_length=3)
```

Valid

```
abc
```

Invalid

```
ab
```

---

# max_length

```python
class User(BaseModel):
    username: str = Field(max_length=10)
```

Invalid

```
verylongusername
```

---

# regex

Used for pattern validation.

Example email validation:

```python
class User(BaseModel):
    email: str = Field(regex=r'^\S+@\S+\.\S+$')
```

Valid

```
abc@gmail.com
```

Invalid

```
abc.com
```

---

# Example Combining Multiple Field Rules

```python
from pydantic import BaseModel, Field

class Product(BaseModel):

    name: str = Field(
        ...,
        min_length=3,
        max_length=20
    )

    price: float = Field(
        gt=0
    )

    quantity: int = Field(
        ge=1
    )

p = Product(
    name="Laptop",
    price=50000,
    quantity=2
)

print(p)
```

---

# Validators in Pydantic

Validators are used when **Field constraints are not enough**.

They allow **custom validation logic**.

Example

```
password must contain uppercase letter
```

Field cannot easily enforce that rule.

Validators are required.

---

# Types of Validators

| Validator       | Purpose                   |
| --------------- | ------------------------- |
| Field Validator | Validates a single field  |
| Pre Validator   | Runs before validation    |
| Post Validator  | Runs after validation     |
| Root Validator  | Validates multiple fields |

---

# Field Validator

Used to validate a specific field.

Example: username cannot contain numbers.

```python
from pydantic import BaseModel, validator

class User(BaseModel):
    username: str

    @validator('username')
    def check_username(cls, v):
        if any(char.isdigit() for char in v):
            raise ValueError("username cannot contain numbers")
        return v
```

Usage

```python
User(username="john123")
```

Error

```
username cannot contain numbers
```

---

# Pre Validator

Runs before type validation.

Used for cleaning or transforming data.

```python
class User(BaseModel):
    age: int

    @validator('age', pre=True)
    def convert_age(cls, v):
        return int(v)
```

Usage

```python
User(age="25")
```

---

# Post Validator

Runs after validation.

```python
class User(BaseModel):
    age: int

    @validator('age')
    def check_age(cls, v):
        if v < 18:
            raise ValueError("Age must be >=18")
        return v
```

---

# Root Validator

Used when validation depends on multiple fields.

Example

```
password and confirm_password must match
```

```python
from pydantic import BaseModel, root_validator

class User(BaseModel):

    password: str
    confirm_password: str

    @root_validator
    def check_passwords(cls, values):

        p1 = values.get("password")
        p2 = values.get("confirm_password")

        if p1 != p2:
            raise ValueError("Passwords do not match")

        return values
```

Usage

```python
User(
    password="abc123",
    confirm_password="abc123"
)
```

Invalid

```python
User(
    password="abc123",
    confirm_password="xyz"
)
```

Error

```
Passwords do not match
```

---

# Field vs Validator

| Feature                 | Field | Validator |
| ----------------------- | ----- | --------- |
| Basic validation        | Yes   | Yes       |
| Numeric constraints     | Yes   | Yes       |
| String constraints      | Yes   | Yes       |
| Custom validation logic | No    | Yes       |
| Multi field validation  | No    | Yes       |

---

# Real FastAPI Style Example

```python
from pydantic import BaseModel, Field, validator

class User(BaseModel):

    username: str = Field(
        ...,
        min_length=3,
        max_length=20
    )

    age: int = Field(
        ge=18,
        le=60
    )

    email: str = Field(
        regex=r'^\S+@\S+\.\S+$'
    )

    @validator('username')
    def no_spaces(cls, v):
        if " " in v:
            raise ValueError("username cannot contain spaces")
        return v
```

---

# Quick Summary

Field is used for:

* Required fields
* Default values
* Numeric validation
* String validation
* Regex pattern validation

Common Field parameters

```
...
default
gt
ge
lt
le
min_length
max_length
regex
```

Validators are used for **custom validation logic**.

Validator types

```
Field validator
Pre validator
Post validator
Root validator
```

---

# End of Guide
