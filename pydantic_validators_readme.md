# Pydantic Validators (Pre, Post, Root) --- Short Guide

In **Pydantic**, validators are used to **validate or modify data before
or after model fields are processed**.

There are two main types:

-   **Field validators** (validate individual fields)\
-   **Root validators** (validate the whole model)

Below is a **short and clear explanation** of **pre validator, post
validator, and root validator**.

------------------------------------------------------------------------

# 1. Pre Validator (`pre=True`)

A **pre validator runs before Pydantic performs type validation or
parsing**.

Meaning:

-   It receives the **raw input value**
-   Runs **before Pydantic converts types**

### Example

``` python
from pydantic import BaseModel, validator

class User(BaseModel):
    age: int

    @validator("age", pre=True)
    def convert_age(cls, value):
        return int(value)

u = User(age="25")
print(u)
```

### What happens

Input

    age = "25"   (string)

Pre validator converts it to

    25 (int)

Output

    age=25

### When to use

Use **pre validator** when you want to:

-   Clean raw input
-   Convert formats
-   Handle messy data

Examples

-   `" 25 "` → `25`
-   `"True"` → `True`

------------------------------------------------------------------------

# 2. Post Validator (default)

A **post validator runs after Pydantic validates and converts the field
type**.

Meaning:

-   Value is already **validated and converted**

### Example

``` python
from pydantic import BaseModel, validator

class User(BaseModel):
    age: int

    @validator("age")
    def check_age(cls, value):
        if value < 18:
            raise ValueError("Age must be >= 18")
        return value
```

### What happens

1.  Pydantic converts input → `int`\
2.  Post validator checks rule

Example input

    age = 15

Error

    Age must be >= 18

### When to use

Use **post validator** when you want to:

-   Apply business rules
-   Check ranges
-   Validate final values

------------------------------------------------------------------------

# 3. Root Validator

A **root validator validates the entire model instead of individual
fields**.

Used when **multiple fields depend on each other**.

Examples

-   `password = confirm_password`
-   `start_date < end_date`

------------------------------------------------------------------------

### Example

``` python
from pydantic import BaseModel, root_validator

class User(BaseModel):
    password: str
    confirm_password: str

    @root_validator
    def check_passwords(cls, values):
        if values["password"] != values["confirm_password"]:
            raise ValueError("Passwords do not match")
        return values
```

Input

    password = "abc"
    confirm_password = "xyz"

Error

    Passwords do not match

------------------------------------------------------------------------

# Root Validator with `pre=True`

This runs **before field validation**.

``` python
@root_validator(pre=True)
```

Use when you want to **validate raw input data of the entire model**.

------------------------------------------------------------------------

# Quick Summary

  Validator            Runs When                Used For
  -------------------- ------------------------ --------------------
  **pre validator**    Before type conversion   Clean raw input
  **post validator**   After validation         Business rules
  **root validator**   Whole model validation   Cross-field checks

------------------------------------------------------------------------

# Simple Memory Trick

**Pre → Before parsing**

**Post → After parsing**

**Root → Entire model**
