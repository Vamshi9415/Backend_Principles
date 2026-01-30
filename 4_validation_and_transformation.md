# Validation and Transformation in Backend APIs

> *The gatekeepers of data integrity and security in your API architecture*

## Table of Contents

1. [Introduction](#introduction)
2. [Architecture Overview](#architecture-overview)
3. [Why Validation & Transformation Matter](#why-validation--transformation-matter)
4. [Where Validations Fit](#where-validations-fit)
5. [Core Concepts](#core-concepts)
   - [Validation](#validation)
   - [Transformation](#transformation)
6. [Types of Validation](#types-of-validation)
   - [Type Validation](#type-validation)
   - [Syntactic Validation](#syntactic-validation)
   - [Semantic Validation](#semantic-validation)
   - [Complex/Cross-field Validation](#complexcross-field-validation)
7. [Transformation Strategies](#transformation-strategies)
8. [Frontend vs Backend Validation](#frontend-vs-backend-validation)
9. [Implementation Examples](#implementation-examples)
10. [Common Pitfalls & Debugging](#common-pitfalls--debugging)
11. [Best Practices](#best-practices)
12. [Quick Revision & Cheat Sheet](#quick-revision--cheat-sheet)

---

## Introduction

Validation and transformation are fundamental practices in API design that protect your system's data integrity and security. Think of them as bouncers at a nightclub—their job is to check that everyone entering meets the requirements before they can go inside.

This guide covers:
- What validation and transformation are
- Why they're essential in backend architecture
- How to implement them correctly
- Common mistakes to avoid
- Interview-ready patterns and best practices

---

## Architecture Overview

To understand where validation fits, we need to understand the typical backend architecture. Most well-designed APIs follow a layered approach:

```
┌─────────────────────────────────────────────────────────────┐
│                        CLIENT REQUEST                       │
│                      (JSON, Query Params,                   │
│                       Path Params, Headers)                 │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
        ┌──────────────────────────────────────┐
        │   ROUTE MATCHING & HANDLER SELECTION │
        └──────────────────────┬───────────────┘
                               │
                               ▼
        ┌──────────────────────────────────────┐
        │  VALIDATION & TRANSFORMATION LAYER   │  ◄─── Focus
        │  (Middleware / Input Pipeline)      │      Here
        └──────────────────────┬───────────────┘
                               │
                               ▼
        ┌──────────────────────────────────────┐
        │    CONTROLLER LAYER                  │
        │    • HTTP logic                      │
        │    • Orchestration                   │
        └──────────────────────┬───────────────┘
                               │
                               ▼
        ┌──────────────────────────────────────┐
        │    SERVICE LAYER                     │
        │    • Business logic                  │
        │    • Orchestration                   │
        └──────────────────────┬───────────────┘
                               │
                               ▼
        ┌──────────────────────────────────────┐
        │    REPOSITORY LAYER                  │
        │    • Database operations             │
        │    • Data persistence                │
        └──────────────────────┬───────────────┘
                               │
                               ▼
        ┌──────────────────────────────────────┐
        │    DATABASE / PERSISTENT STORAGE     │
        └──────────────────────────────────────┘
```

### Layer Responsibilities

**Repository Layer** (Bottom)
- Handles all database connections and operations
- Pure data persistence (CRUD operations)
- Works with relational databases, caches (Redis), NoSQL, etc.

**Service Layer** (Middle)
- Contains all business logic
- Orchestrates repository calls
- Handles side effects (emails, notifications, webhooks)
- Independent of HTTP protocol

**Controller Layer** (Top)
- Handles HTTP-specific concerns
- Maps HTTP requests to service methods
- Manages response formats and status codes
- **Entry point for validation and transformation**

The separation exists because:
- Services can be reused by multiple controllers
- Services are protocol-agnostic (could be called from CLI, message queue, etc.)
- Validation needs to happen once, at the entry point

---

## Why Validation & Transformation Matter

### The Real-World Problem

Imagine a request arrives at your API with invalid data:

```
POST /api/books
{
  "name": 0,
  "author": null,
  "pages": "five"
}
```

**Without validation:**
1. Controller receives bad data
2. Controller calls service
3. Service calls repository
4. Repository tries to insert into database
5. Database rejects the data (type mismatch)
6. User gets a 500 Internal Server Error
7. User has no idea what went wrong

**Result:** Poor UX, exposed internal errors, unpredictable behavior

**With validation:**
1. Validation pipeline checks data at entry
2. Immediately returns 400 Bad Request with clear errors
3. User knows exactly what's wrong
4. Never wastes time processing invalid data

### Security & Data Integrity

Bad data can:
- **Corrupt your database** - Null values where strings expected
- **Break business logic** - Age 500 shouldn't pass
- **Enable attacks** - SQL injection if not properly validated
- **Cause crashes** - Unexpected types cause exceptions
- **Create compliance issues** - Invalid data violates regulations

Validation is your first line of defense.

---

## Where Validations Fit

Validations happen **immediately after route matching**, before any business logic executes:

```
Request arrives
    ↓
Route matches (/api/users POST)
    ↓
Validation & Transformation runs  ◄─── HERE
    ↓
(If valid, continue)
    ↓
Controller method called
    ↓
Service method called
    ↓
Repository method called
    ↓
Response returned
```

This strategic placement ensures:
- ✅ Invalid data never reaches your business logic
- ✅ System always in a known, valid state
- ✅ Early error feedback to clients
- ✅ Consistent validation across all endpoints
- ✅ Centralized input processing

---

## Core Concepts

### Validation

**What it is:** The process of checking that incoming data matches expected requirements.

**Why it exists:** To ensure data integrity and prevent invalid data from corrupting your system.

**How it works:**
1. Define a schema (expected structure, types, constraints)
2. Receive data from client
3. Check data against schema
4. Return errors if mismatches found
5. Allow data through if valid

**Simple example:**

```python
from pydantic import BaseModel, EmailStr, Field, validator

# Define what we expect
class UserSchema(BaseModel):
    email: EmailStr
    age: int = Field(ge=0, le=150)
    name: str = Field(min_length=2, max_length=100)

# Incoming request
{
  "email": "user@example.com",
  "age": 25,
  "name": "John"
}

# Result: ✓ Valid, proceed

# Another request
{
  "email": "invalid-email",
  "age": 500,
  "name": ""
}

# Result: ✗ Invalid
# Errors:
# - email: Invalid email format
# - age: Must be between 0 and 150
# - name: Minimum length is 2
```

### Transformation

**What it is:** Converting or modifying data into the format required by your system.

**Why it exists:** Clients send data in various formats; your system needs consistency.

**How it works:**
1. Receive data in one format
2. Apply conversion/normalization
3. Output in expected format

**Real-world examples:**

```python
from datetime import date

# Example 1: Query parameter casting
# Client sends: GET /api/posts?page=2&limit=10
# Query params are always strings
page: "2"    → needs to be → page: 2
limit: "10"  → needs to be → limit: 10

# Example 2: Email normalization
# Client sends: "JohnDoe@EXAMPLE.COM"
# Transform to: "johndoe@example.com" (lowercase)

# Example 3: Phone number formatting
# Client sends: "9876543210"
# Transform to: "+1-987-654-3210" (formatted)

# Example 4: Date parsing
# Client sends: "2025-01-28"
# Transform to: date(2025, 1, 28) (date object)

# Example 5: Trimming whitespace
# Client sends: "  John Doe  "
# Transform to: "John Doe"
```

**Key insight:** Validation and transformation often work together:
```
Raw input → Transform → Validate → Clean data for business logic
```

---

## Types of Validation

### Type Validation

**What it is:** Checking that data types match expectations.

**Why it exists:** Your code assumes certain types; mismatches cause crashes.

**How it works:** Check the data type of each field.

**Example:**

```python
from pydantic import BaseModel, ValidationError
from typing import List

# Schema definition
class DataSchema(BaseModel):
    age: int
    name: str
    active: bool
    tags: List[str]

# Invalid data
{
  "age": "25",        # ✗ String, not int
  "name": 123,        # ✗ Number, not string
  "active": "true",   # ✗ String, not boolean
  "tags": "a,b,c"     # ✗ String, not list
}

# Errors:
# age: value is not a valid integer
# name: str type expected
# active: value is not a valid boolean
# tags: value is not a valid list
```

**When to use it:** Always. Type validation is the foundation.

**Common mistake:** Assuming `"123"` and `123` are interchangeable. They're not.

---

### Syntactic Validation

**What it is:** Checking that data follows a specific pattern or structure.

**Why it exists:** Some data (emails, phone numbers, dates) must match specific formats.

**How it works:** Use patterns, regex, or specialized validators.

**Common patterns:**

```python
from pydantic import BaseModel, EmailStr, HttpUrl, UUID4, Field, validator
from datetime import date
import re

# Email format
class EmailSchema(BaseModel):
    email: EmailStr

Valid:   "user@example.com"
Invalid: "user@", "@example.com", "user example.com"

# Phone number (E.164 format)
class PhoneSchema(BaseModel):
    phone: str
    
    @validator('phone')
    def validate_phone(cls, v):
        if not re.match(r'^\+?[1-9]\d{1,14}$', v):
            raise ValueError('Invalid phone format')
        return v

Valid:   "+1234567890", "+442071838750"
Invalid: "1234567890", "phone: 123"

# ISO 8601 Date
class DateSchema(BaseModel):
    date: date

Valid:   "2025-01-28"
Invalid: "01/28/2025", "28-01-2025"

# URL
class UrlSchema(BaseModel):
    website: HttpUrl

Valid:   "https://example.com", "http://example.co.uk/path"
Invalid: "example.com", "not a url"

# UUID
class UuidSchema(BaseModel):
    id: UUID4

Valid:   "550e8400-e29b-41d4-a716-446655440000"
Invalid: "550e8400-e29b-41d4", "not-a-uuid"
```

**Real-world example:**

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, EmailStr, HttpUrl, validator
import re

app = FastAPI()

# Syntactic validation example
class ContactRequest(BaseModel):
    name: str
    email: EmailStr
    phone: str
    website: HttpUrl
    
    @validator('phone')
    def validate_phone(cls, v):
        if not re.match(r'^\+?[1-9]\d{1,14}$', v):
            raise ValueError('Invalid phone format')
        return v

@app.post("/api/contacts")
async def create_contact(contact: ContactRequest):
    # Validation results
    # ✓ email: Matches email pattern
    # ✓ phone: Matches phone pattern
    # ✓ website: Matches URL pattern
    return {"status": "success", "data": contact.dict()}

# Example request
{
  "name": "John Doe",
  "email": "john@example.com",
  "phone": "+12025551234",
  "website": "https://johndoe.com"
}
```

**When to use it:** When specific formats are required.

---

### Semantic Validation

**What it is:** Checking that data makes logical sense in the real world.

**Why it exists:** Syntactically correct data can still be nonsensical (age 500, future birth date).

**How it works:** Apply business logic rules to field values.

**Examples:**

```python
from pydantic import BaseModel, Field, validator, root_validator
from datetime import date, datetime
from typing import Optional, Literal

# Age validation
class AgeSchema(BaseModel):
    age: int = Field(ge=1, le=150)

✓ 25    # Reasonable age
✗ 500   # Syntactically valid number, semantically invalid
✗ -5    # Negative age doesn't make sense

# Date of birth validation
class BirthDateSchema(BaseModel):
    date_of_birth: date
    
    @validator('date_of_birth')
    def validate_dob(cls, v):
        if v > date.today():
            raise ValueError('Date of birth cannot be in the future')
        return v

✓ 1995-06-15     # Valid past date
✗ 2026-01-28     # Future date (today is 2025-01-28)

# Password confirmation
class PasswordSchema(BaseModel):
    password: str = Field(min_length=8)
    password_confirmation: str
    
    @root_validator
    def passwords_match(cls, values):
        pw = values.get('password')
        pw_conf = values.get('password_confirmation')
        if pw != pw_conf:
            raise ValueError('Passwords do not match')
        return values

✓ password: "secure123", password_confirmation: "secure123"
✗ password: "secure123", password_confirmation: "different"

# Conditional requirements
class MarriageSchema(BaseModel):
    is_married: bool
    spouse_name: Optional[str] = None
    
    @root_validator
    def validate_spouse(cls, values):
        if values.get('is_married') and not values.get('spouse_name'):
            raise ValueError('Spouse name is required when married')
        return values

✓ is_married: False (spouse_name not provided)
✓ is_married: True, spouse_name: "Jane"
✗ is_married: True (spouse_name missing)

# Range validation
class DiscountSchema(BaseModel):
    discount: float = Field(ge=0, le=100)

✓ 15       # Valid discount percentage
✗ 150      # Over 100%
✗ -10      # Negative discount

# Enum validation
class StatusSchema(BaseModel):
    status: Literal['active', 'inactive', 'pending']

✓ 'active'      # Valid option
✗ 'unknown'     # Not in allowed list
```

**When to use it:** When values must satisfy real-world business rules.

---

### Complex/Cross-field Validation

**What it is:** Validating relationships between multiple fields.

**Why it exists:** Some constraints depend on the values of multiple fields together.

**Examples:**

```python
from pydantic import BaseModel, Field, root_validator, validator
from datetime import date
from typing import Optional, Literal

# Example 1: Matching fields
class PasswordMatchSchema(BaseModel):
    password: str = Field(min_length=8)
    confirm_password: str
    
    @root_validator
    def passwords_match(cls, values):
        pw = values.get('password')
        confirm = values.get('confirm_password')
        if pw != confirm:
            raise ValueError('Passwords must match')
        return values

# Example 2: Conditional requirements
class ShippingSchema(BaseModel):
    method: Literal['pickup', 'delivery']
    shipping_address: Optional[str] = None
    
    @root_validator
    def validate_address(cls, values):
        if values.get('method') != 'pickup' and not values.get('shipping_address'):
            raise ValueError('Shipping address required for delivery')
        return values

# Example 3: Date ranges
class DateRangeSchema(BaseModel):
    start_date: date
    end_date: date
    
    @root_validator
    def validate_date_range(cls, values):
        start = values.get('start_date')
        end = values.get('end_date')
        if start and end and end <= start:
            raise ValueError('End date must be after start date')
        return values

# Example 4: Dependent fields
class LocationSchema(BaseModel):
    country: Literal['US', 'CA', 'MX']
    state: str
    
    @root_validator
    def validate_state(cls, values):
        country = values.get('country')
        state = values.get('state')
        valid_states = {
            'US': ['CA', 'NY', 'TX'],
            'CA': ['ON', 'BC', 'AB'],
            'MX': ['CDMX', 'JAL', 'BC']
        }
        if state and state not in valid_states.get(country, []):
            raise ValueError(f'Invalid state for country {country}')
        return values

# Example 5: Either/or validation
class ContactSchema(BaseModel):
    email: Optional[str] = None
    phone: Optional[str] = None
    
    @root_validator
    def validate_contact(cls, values):
        if not values.get('email') and not values.get('phone'):
            raise ValueError('Either email or phone is required')
        return values
```

**When to use it:** When multiple fields interact or depend on each other.

---

## Transformation Strategies

### Why Transformation Happens

Data from clients arrives in various formats:
- **Query parameters:** Always strings (`"10"` instead of `10`)
- **JSON payloads:** May have inconsistent casing
- **Dates:** Various formats (ISO, Unix timestamps, etc.)
- **Phone numbers:** Different formatting conventions
- **User input:** Extra whitespace

**Goal:** Convert to a standardized format before business logic.

### Common Transformations

```python
# 1. Type casting
"123" → 123                    # String to int
"true" → True                  # String to boolean

# 2. String normalization
"  John  " → "John"            # Trim whitespace
"JohnDoe@EXAMPLE.COM" → "johndoe@example.com"  # Lowercase
"john doe" → "John Doe"        # Capitalize

# 3. Format standardization
"9876543210" → "+1-987-654-3210"  # Phone
"01/28/25" → "2025-01-28"          # Date
"$100.50" → 100.50                 # Currency

# 4. Default values
{"name": "John"} → {"name": "John", "status": "active"}

# 5. Filtering sensitive data
{"password": "secret", "name": "John"} → {"name": "John"}

# 6. Data restructuring
{"first_name": "John", "last_name": "Doe"} → {"full_name": "John Doe"}

# 7. Flattening nested data
{"user": {"name": "John"}} → {"user_name": "John"}

# 8. Extracting specific fields
{"email": ..., "password": ..., ...} → {"email": ...}  # Keep only email
```

### Practical Example

```python
from fastapi import FastAPI, Query
from pydantic import BaseModel, Field, validator
from typing import Literal, Optional
from enum import Enum

app = FastAPI()

# Raw incoming query: GET /api/posts?page=2&limit=10&sort=-createdAt

class SortOrder(str, Enum):
    ASC = "ASC"
    DESC = "DESC"

class PostsQuery(BaseModel):
    page: int = Field(ge=1, le=500)
    limit: int = Field(ge=1, le=100)
    sort_field: str
    sort_direction: SortOrder
    
    class Config:
        use_enum_values = True

# Step 1: Transformation (handled by FastAPI/Pydantic automatically)
# page: "2" → page: 2              # Cast to int
# limit: "10" → limit: 10          # Cast to int
# sort: "-createdAt" → 
#   {"field": "createdAt", "direction": "DESC"}  # Parse sort param

@app.get("/api/posts")
async def get_posts(
    page: int = Query(ge=1, le=500, default=1),
    limit: int = Query(ge=1, le=100, default=10),
    sort: str = Query(default="-createdAt")
):
    # Parse sort parameter
    if sort.startswith('-'):
        sort_field = sort[1:]
        sort_direction = "DESC"
    else:
        sort_field = sort
        sort_direction = "ASC"
    
    # Step 2: Validation
    # page: ✓ Valid (1-500)
    # limit: ✓ Valid (1-100)
    # sort: ✓ Valid
    
    # Step 3: Ready for business logic
    posts = await db.posts.find(
        page=page,
        limit=limit,
        sort={"field": sort_field, "direction": sort_direction}
    )
    
    return posts
```

### Transformation Pipeline Order

```
Raw Input
    ↓
1. Type Casting (string "10" → number 10)
    ↓
2. Formatting (normalize phone, email, date)
    ↓
3. Normalization (trim, lowercase)
    ↓
4. Default Values (add missing fields)
    ↓
5. Filtering (remove sensitive fields)
    ↓
Validation Occurs
    ↓
6. Restructuring (group fields, rename)
    ↓
Clean Data Ready for Business Logic
```

---

## Frontend vs Backend Validation

### The Critical Distinction

This is one of the most important concepts in API design. **Never rely on frontend validation for security.**

| Aspect | Frontend Validation | Backend Validation |
|--------|-------------------|-------------------|
| **Purpose** | User experience | Security & data integrity |
| **When** | Before API call | Every API call |
| **Effect** | Gives immediate feedback | Protects system |
| **Trustworthiness** | ❌ Can be bypassed | ✅ Enforced always |
| **Tools** | HTML5, JavaScript | Server code |
| **Requirement** | Optional | Mandatory |

### Why You Can't Trust Frontend Validation

```python
# Scenario: Web form with email validation
# Frontend: User types invalid email, form shows error
# Frontend: User types valid email, form allows submit
# Frontend: API call is made

# BUT... A malicious user can:
# 1. Disable JavaScript
# 2. Use browser dev tools to remove validation
# 3. Use curl/Postman to bypass frontend entirely
# 4. Write a bot that hits your API directly
# 5. Modify the HTML to remove validation

# Result: Invalid data reaches your API anyway
```

### Example: Two Different Clients

```python
# Client 1: Web Application
# Has form validation, user can't submit invalid data
GET https://api.example.com/posts?page=2&limit=10

# Client 2: Insomnia/Postman (API testing tool)
# No form, no validation
GET https://api.example.com/posts?page=abc&limit=xyz

# Client 3: Mobile App
# Different validation rules or might be outdated
GET https://api.example.com/posts?page=-5&limit=99999

# Your API receives all three requests
# Backend validation must handle all cases
```

### Correct Architecture

```
┌─────────────────────┐
│   Web Frontend      │
│  (Form validation)  │
│  ↓ (User experience)│
│  POST /api/users    │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────────────────┐
│   Backend API                   │
│  MANDATORY VALIDATION HERE      │
│  (Security + data integrity)    │
└──────────┬──────────────────────┘
           │
           ▼
        Database

┌─────────────────────┐
│   Mobile App        │
│  (May have different│
│   validation rules) │
│  ↓                  │
│  POST /api/users    │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────────────────┐
│   Backend API                   │
│  MANDATORY VALIDATION HERE      │
│  (Security + data integrity)    │
└──────────┬──────────────────────┘
           │
           ▼
        Database

┌─────────────────────┐
│   Insomnia/curl     │
│  (No validation)    │
│  ↓                  │
│  POST /api/users    │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────────────────┐
│   Backend API                   │
│  MANDATORY VALIDATION HERE      │
│  (Security + data integrity)    │
└──────────┬──────────────────────┘
           │
           ▼
        Database
```

### The Golden Rule

> **Treat the backend as if no frontend validation exists. Every API must validate every input.**

---

## Implementation Examples

### Example 1: Email Validation (Syntactic)

**Scenario:** User registration API

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, EmailStr, Field, ValidationError

app = FastAPI()

# Validation Schema
class RegisterRequest(BaseModel):
    email: EmailStr
    password: str = Field(min_length=8)

# Invalid request
@app.post("/api/auth/register")
async def register(user: RegisterRequest):
    # If validation fails, FastAPI automatically returns 422
    return {"status": "success", "user_id": "usr_123"}

# Request example:
# POST /api/auth/register
# {
#   "email": "invalid-email",
#   "password": "secure123"
# }

# Automatic validation error response:
# HTTP 422 Unprocessable Entity
# {
#   "detail": [
#     {
#       "loc": ["body", "email"],
#       "msg": "value is not a valid email address",
#       "type": "value_error.email"
#     }
#   ]
# }

# Or with custom error handling:
from fastapi.exceptions import RequestValidationError
from fastapi.responses import JSONResponse

@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request, exc):
    errors = {}
    for error in exc.errors():
        field = error['loc'][-1]
        errors[field] = error['msg']
    
    return JSONResponse(
        status_code=400,
        content={
            "status": "error",
            "message": "Validation failed",
            "errors": errors
        }
    )

# Response with custom handler:
# HTTP 400 Bad Request
# {
#   "status": "error",
#   "message": "Validation failed",
#   "errors": {
#     "email": "value is not a valid email address"
#   }
# }
```

### Example 2: Age Validation (Semantic)

**Scenario:** User profile API

```python
from fastapi import FastAPI
from pydantic import BaseModel, Field, validator
from datetime import date

app = FastAPI()

# Validation Schema
class ProfileRequest(BaseModel):
    name: str
    date_of_birth: date
    age: int = Field(ge=1, le=150)
    
    @validator('date_of_birth')
    def validate_dob(cls, v):
        if v > date.today():
            raise ValueError('Date of birth cannot be in the future')
        return v

# Invalid request
@app.post("/api/users/profile")
async def update_profile(profile: ProfileRequest):
    return {"status": "success", "data": profile.dict()}

# Request example:
# POST /api/users/profile
# {
#   "name": "John Doe",
#   "date_of_birth": "2026-01-28",  # Future date!
#   "age": 500                      # Unrealistic age!
# }

# Validation errors:
# HTTP 400 Bad Request
# {
#   "status": "error",
#   "message": "Validation failed",
#   "errors": {
#     "date_of_birth": "Date of birth cannot be in the future",
#     "age": "ensure this value is less than or equal to 150"
#   }
# }
```

### Example 3: Password Matching (Cross-field)

**Scenario:** User registration with confirmation

```python
from fastapi import FastAPI
from pydantic import BaseModel, EmailStr, Field, root_validator

app = FastAPI()

# Validation Schema
class RegisterRequest(BaseModel):
    email: EmailStr
    password: str = Field(min_length=8)
    password_confirmation: str
    
    @root_validator
    def passwords_match(cls, values):
        pw = values.get('password')
        pw_conf = values.get('password_confirmation')
        if pw != pw_conf:
            raise ValueError('Passwords do not match')
        return values

# Invalid request
@app.post("/api/auth/register")
async def register(user: RegisterRequest):
    return {
        "status": "success",
        "data": {
            "user_id": "usr_123",
            "email": user.email
        }
    }

# Request example:
# POST /api/auth/register
# {
#   "email": "user@example.com",
#   "password": "secure123456",
#   "password_confirmation": "different123"  # Mismatch!
# }

# Validation error:
# {
#   "status": "error",
#   "message": "Validation failed",
#   "errors": {
#     "password_confirmation": "Passwords do not match"
#   }
# }

# Correct request:
# POST /api/auth/register
# {
#   "email": "user@example.com",
#   "password": "secure123456",
#   "password_confirmation": "secure123456"  # Match!
# }

# Response:
# HTTP 201 Created
# {
#   "status": "success",
#   "data": {
#     "user_id": "usr_123",
#     "email": "user@example.com"
#   }
# }
```

### Example 4: Query Parameter Transformation

**Scenario:** Paginated list API with sorting

```python
from fastapi import FastAPI, Query, HTTPException
from pydantic import BaseModel, Field, validator
from typing import List, Literal
from enum import Enum

app = FastAPI()

class SortDirection(str, Enum):
    ASC = "ASC"
    DESC = "DESC"

class Post(BaseModel):
    id: int
    title: str
    created_at: str

# Raw request: GET /api/posts?page=2&limit=10&sort=-createdAt

# Query parameters arrive as strings
# {
#   "page": "2",           # String, needs int
#   "limit": "10",         # String, needs int
#   "sort": "-createdAt"   # String, needs parsing
# }

@app.get("/api/posts")
async def get_posts(
    page: int = Query(ge=1, le=500, default=1),
    limit: int = Query(ge=1, le=100, default=10),
    sort: str = Query(default="-createdAt")
):
    # Transformation Steps:
    # 1. Cast page "2" → 2 (handled by FastAPI)
    # 2. Cast limit "10" → 10 (handled by FastAPI)
    # 3. Parse sort "-createdAt" → {field: "createdAt", direction: "DESC"}
    
    # Parse sort parameter
    if sort.startswith('-'):
        sort_field = sort[1:]
        sort_direction = SortDirection.DESC
    else:
        sort_field = sort
        sort_direction = SortDirection.ASC
    
    # Validate sort field
    valid_fields = ['createdAt', 'updatedAt', 'title', 'author']
    if sort_field not in valid_fields:
        raise HTTPException(
            status_code=400,
            detail={
                "status": "error",
                "message": "Validation failed",
                "errors": {"sort": "Invalid sort field"}
            }
        )
    
    # Transformed data ready for use
    # page: 2, limit: 10, sort: {field: "createdAt", direction: "DESC"}
    
    # Business logic
    posts = [
        Post(id=1, title="Post 1", created_at="2025-01-28"),
        Post(id=2, title="Post 2", created_at="2025-01-27")
    ]
    
    return {
        "status": "success",
        "data": posts,
        "pagination": {
            "page": page,
            "limit": limit,
            "total": 245
        }
    }

# If invalid (e.g., page=999):
# HTTP 422 Unprocessable Entity
# {
#   "detail": [
#     {
#       "loc": ["query", "page"],
#       "msg": "ensure this value is less than or equal to 500",
#       "type": "value_error.number.not_le"
#     }
#   ]
# }

# If valid:
# HTTP 200 OK
# {
#   "status": "success",
#   "data": [
#     {"id": 1, "title": "Post 1", "created_at": "2025-01-28"},
#     {"id": 2, "title": "Post 2", "created_at": "2025-01-27"}
#   ],
#   "pagination": {
#     "page": 2,
#     "limit": 10,
#     "total": 245
#   }
# }
```

### Example 5: Email Normalization (Transformation)

**Scenario:** User might send email in various formats

```python
from fastapi import FastAPI
from pydantic import BaseModel, EmailStr, validator

app = FastAPI()

class RegisterRequest(BaseModel):
    email: EmailStr
    name: str
    
    @validator('email')
    def normalize_email(cls, v):
        # EmailStr automatically validates format
        # Additional normalization
        return v.lower().strip()
    
    @validator('name')
    def normalize_name(cls, v):
        return v.strip()

# Incoming request
@app.post("/api/auth/register")
async def register(user: RegisterRequest):
    # Pydantic automatically applies transformations
    return {
        "status": "success",
        "data": {
            "email": user.email,
            "name": user.name
        }
    }

# Request example:
# POST /api/auth/register
# {
#   "email": "  JohnDoe@EXAMPLE.COM  ",
#   "name": "John Doe"
# }

# Transformation:
# 1. Trim whitespace: "  JohnDoe@EXAMPLE.COM  " → "JohnDoe@EXAMPLE.COM"
# 2. Lowercase: "JohnDoe@EXAMPLE.COM" → "johndoe@example.com"

# Transformed data:
# {
#   "email": "johndoe@example.com",
#   "name": "John Doe"
# }

# Validation:
# email: ✓ Valid (EmailStr validates format)

# Result:
# ✓ Valid, proceeds to service layer with normalized email

# Later when user tries to login with "JOHNDOE@EXAMPLE.COM"
# It gets transformed to "johndoe@example.com" again
# Matches the registered email ✓
```

### Example 6: Conditional Requirements

**Scenario:** Shipping address only needed if shipping method is "deliver"

```python
from fastapi import FastAPI
from pydantic import BaseModel, Field, root_validator
from typing import Optional, Literal, List

app = FastAPI()

class OrderRequest(BaseModel):
    items: List[dict]
    method: Literal['pickup', 'deliver']
    shipping_address: Optional[str] = None
    
    @root_validator
    def validate_shipping(cls, values):
        method = values.get('method')
        address = values.get('shipping_address')
        
        if method == 'deliver':
            if not address:
                raise ValueError('Shipping address is required for delivery')
            if len(address) < 10:
                raise ValueError('Shipping address must be at least 10 characters')
        
        return values

# Request 1: Pickup method (no shipping address)
@app.post("/api/orders")
async def create_order(order: OrderRequest):
    return {
        "status": "success",
        "data": order.dict()
    }

# Example Request 1:
# POST /api/orders
# {
#   "items": [...],
#   "method": "pickup"
#   # shipping_address not provided
# }

# Result of Request 1:
# ✓ Valid (pickup doesn't require address)

# Example Request 2 (invalid):
# POST /api/orders
# {
#   "items": [...],
#   "method": "deliver"
#   # shipping_address missing!
# }

# Result of Request 2:
# ✗ Invalid
# {
#   "errors": {
#     "shipping_address": "Shipping address is required for delivery"
#   }
# }

# Correct Request 2:
# POST /api/orders
# {
#   "items": [...],
#   "method": "deliver",
#   "shipping_address": "123 Main St, New York, NY 10001"
# }

# Result:
# ✓ Valid, order processed
```

---

## Common Pitfalls & Debugging

### Pitfall 1: Relying Only on Frontend Validation

**Problem:**
```python
# Frontend has validation, assume backend is safe
# Backend: No validation, trusts frontend

# Attacker sends bad data directly:
# curl -X POST https://api.example.com/users \
#   -H "Content-Type: application/json" \
#   -d '{"age": -5, "email": "invalid"}'

# Result: Database corrupted
```

**Solution:**
```python
# Always validate on backend
# Assume frontend doesn't exist
# Treat all input as potentially malicious

from pydantic import BaseModel, EmailStr, Field

class UserRequest(BaseModel):
    email: EmailStr
    age: int = Field(ge=0, le=150)
```

### Pitfall 2: Validating After Processing

**Problem:**
```python
# Bad: Process first, validate later
from fastapi import FastAPI, Request

app = FastAPI()

@app.post("/users")
async def create_user(request: Request):
    data = await request.json()
    await db.user.create(data)  # May fail!
    if data.get('age', 0) < 0:
        return {"error": "Invalid age"}
```

**Solution:**
```python
# Good: Validate first, then process
from fastapi import FastAPI
from pydantic import BaseModel, Field

app = FastAPI()

class UserRequest(BaseModel):
    age: int = Field(ge=0)
    name: str

@app.post("/users")
async def create_user(user: UserRequest):
    # Validation already happened via Pydantic
    await db.user.create(user.dict())  # Safe
```

### Pitfall 3: Inconsistent Error Messages

**Problem:**
```python
# Different errors for same issue
# POST /api/users (Frontend validation): "Email is required"
# POST /api/users (Backend validation): "email must be defined"
# POST /api/users (From script): "Missing field: email"

# Confuses developers
```

**Solution:**
```python
# Consistent error format everywhere
from fastapi import FastAPI
from fastapi.exceptions import RequestValidationError
from fastapi.responses import JSONResponse

app = FastAPI()

@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request, exc):
    errors = {}
    for error in exc.errors():
        field = error['loc'][-1]
        errors[field] = error['msg']
    
    return JSONResponse(
        status_code=400,
        content={
            "status": "error",
            "errors": errors
        }
    )
```

### Pitfall 4: Not Sanitizing Input

**Problem:**
```python
# Accept and save raw input
from fastapi import FastAPI, Request

app = FastAPI()

@app.post("/comments")
async def create_comment(request: Request):
    data = await request.json()
    comment = data.get('comment')
    await db.comments.insert(comment)  # XSS vulnerability!

# Attacker sends: "<script>alert('hacked')</script>"
# Saved to database, executed in browsers
```

**Solution:**
```python
# Sanitize HTML/dangerous content
from fastapi import FastAPI
from pydantic import BaseModel, validator
import bleach

app = FastAPI()

class CommentRequest(BaseModel):
    comment: str
    
    @validator('comment')
    def sanitize_comment(cls, v):
        # Remove dangerous HTML tags
        return bleach.clean(v)

@app.post("/comments")
async def create_comment(comment_data: CommentRequest):
    await db.comments.insert(comment_data.comment)
```

### Pitfall 5: Unclear Validation Error

**Problem:**
```python
# Vague error message
{
  "status": "error",
  "message": "Validation failed"
}

# User has no idea what's wrong
```

**Solution:**
```python
# Specific error messages with field details
from fastapi import FastAPI
from fastapi.exceptions import RequestValidationError
from fastapi.responses import JSONResponse

app = FastAPI()

@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request, exc):
    errors = {}
    for error in exc.errors():
        field = error['loc'][-1]
        # Customize messages
        if error['type'] == 'value_error.email':
            errors[field] = 'Invalid email format'
        elif error['type'] == 'value_error.number.not_ge':
            errors[field] = f"Must be at least {error['ctx']['limit_value']}"
        else:
            errors[field] = error['msg']
    
    return JSONResponse(
        status_code=400,
        content={
            "status": "error",
            "message": "Validation failed",
            "errors": errors
        }
    )

# Example response:
# {
#   "status": "error",
#   "message": "Validation failed",
#   "errors": {
#     "email": "Invalid email format",
#     "age": "Must be at least 18",
#     "password": "Must be at least 8 characters"
#   }
# }
```

### Pitfall 6: Over-Validating

**Problem:**
```python
# Too strict validation defeats purpose
import re
from pydantic import BaseModel, validator

class UserRequest(BaseModel):
    username: str
    
    @validator('username')
    def validate_username(cls, v):
        if len(v) < 10:  # Too long
            raise ValueError('Username too short')
        if len(v) > 15:  # Too restrictive
            raise ValueError('Username too long')
        if not re.match(r'^[A-Z][a-z]+$', v):  # Only capitalized single names?
            raise ValueError('Invalid format')
        return v

# Result: 80% of valid users rejected
```

**Solution:**
```python
# Reasonable constraints
import re
from pydantic import BaseModel, Field, validator

class UserRequest(BaseModel):
    username: str = Field(min_length=3, max_length=50)
    
    @validator('username')
    def validate_username(cls, v):
        if not re.match(r'^[a-zA-Z0-9_-]+$', v):  # Flexible pattern
            raise ValueError('Username can only contain letters, numbers, hyphens, and underscores')
        return v
```

### Debugging Validation Issues

**When validation fails unexpectedly:**

```python
# 1. Check transformation happened
from fastapi import FastAPI, Request
import logging

logger = logging.getLogger(__name__)

app = FastAPI()

@app.post("/users")
async def create_user(request: Request):
    body = await request.json()
    logger.info(f"Before: {body}")
    # After Pydantic validation, data is transformed
    logger.info(f"After transformation: {user.dict()}")

# 2. Check schema
from pydantic import BaseModel

class UserRequest(BaseModel):
    email: str
    age: int

logger.info(f"Schema: {UserRequest.schema()}")

# 3. Test schema against data
try:
    user = UserRequest(**data)
    logger.info("Validation passed")
except ValidationError as e:
    logger.error(f"Errors: {e.errors()}")

# 4. Check order of operations
# Is validation before or after transformation?
# Is transformation applied correctly?

# 5. Test with curl
# curl -X POST http://localhost:8000/api/users \
#   -H "Content-Type: application/json" \
#   -d '{"email":"test@example.com","age":25}'

# 6. Check error messages
# Are they helpful?
# Do they match frontend?
```

---

## Best Practices

### 1. Validate at the Entry Point

```python
# ✓ Good: Validate using Pydantic models
from fastapi import FastAPI
from pydantic import BaseModel, EmailStr

app = FastAPI()

class UserRequest(BaseModel):
    email: EmailStr
    name: str

@app.post('/users')
async def create_user(user: UserRequest):
    # Validation automatic via Pydantic
    # Safe to proceed
    await user_service.create_user(user.dict())
    return {"status": "success"}

# ✗ Bad: Validate deep in service layer
class UserService:
    async def create_user(self, data):
        # Validation scattered throughout
        if not data.get('email'):
            raise ValueError('email required')
        # ... lots of other code
```

### 2. Use a Validation Library

```python
# Don't reinvent the wheel
# Python FastAPI uses Pydantic (built-in)
# Other popular libraries:
# - Marshmallow
# - Cerberus
# - Django REST Framework serializers

# Example with Pydantic (recommended for FastAPI)
from pydantic import BaseModel, EmailStr, Field, validator

class UserRequest(BaseModel):
    email: EmailStr
    age: int = Field(ge=1, le=150)
    password: str = Field(min_length=8)
    
    @validator('password')
    def password_strength(cls, v):
        if not any(char.isdigit() for char in v):
            raise ValueError('Password must contain at least one number')
        return v

# Usage in FastAPI
from fastapi import FastAPI

app = FastAPI()

@app.post('/users')
async def create_user(user: UserRequest):
    # Validation happens automatically
    return {"status": "success", "data": user.dict()}
```

### 3. Define Schemas Separately

```python
# ✓ Good: Reusable schemas
from pydantic import BaseModel, EmailStr

class UserBase(BaseModel):
    email: EmailStr
    name: str

class UserCreate(UserBase):
    password: str

class UserUpdate(UserBase):
    # All fields optional for updates
    email: EmailStr | None = None
    name: str | None = None

# Use in endpoints
from fastapi import FastAPI

app = FastAPI()

@app.post('/users')
async def create_user(user: UserCreate):
    return {"status": "success"}

@app.put('/users/{user_id}')
async def update_user(user_id: int, user: UserUpdate):
    return {"status": "success"}

# ✗ Bad: Inline validation everywhere
@app.post('/users')
async def create_user(request: Request):
    data = await request.json()
    if not data.get('email'):
        raise HTTPException(400, "email required")
    if not data.get('name'):
        raise HTTPException(400, "name required")
    # ...
```

### 4. Return Clear Error Responses

```python
# ✓ Good: Structured errors
from fastapi import FastAPI
from fastapi.exceptions import RequestValidationError
from fastapi.responses import JSONResponse

app = FastAPI()

@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request, exc):
    errors = {}
    for error in exc.errors():
        field = error['loc'][-1]
        errors[field] = error['msg']
    
    return JSONResponse(
        status_code=400,
        content={
            "status": "error",
            "message": "Validation failed",
            "errors": errors
        }
    )

# Example response:
# {
#   "status": "error",
#   "message": "Validation failed",
#   "errors": {
#     "email": "value is not a valid email address",
#     "age": "ensure this value is less than or equal to 150"
#   }
# }

# ✗ Bad: Unclear errors
# {"error": "Invalid input"}
```

### 5. Validate ALL Input Sources

```python
# Validate all these sources:
from fastapi import FastAPI, Query, Path, Header
from pydantic import BaseModel, Field
from typing import Optional

app = FastAPI()

class PostBody(BaseModel):
    title: str = Field(min_length=1)
    content: str

@app.get('/posts/{post_id}')
async def get_post(
    post_id: int = Path(ge=1),  # Path parameter validation
    page: int = Query(ge=1, le=100, default=1),  # Query parameter
    limit: int = Query(ge=1, le=100, default=10),
    api_key: str = Header(..., min_length=32)  # Header validation
):
    # All inputs validated automatically
    return {"post_id": post_id, "page": page, "limit": limit}

@app.post('/posts/{post_id}')
async def update_post(
    post_id: int = Path(ge=1),
    post: PostBody = ...,  # Body validation
    api_key: str = Header(..., min_length=32)
):
    return {"status": "success"}
```

### 6. Transform Before Validating

```python
# Transformation pipeline
from pydantic import BaseModel, EmailStr, validator
import re

class UserRequest(BaseModel):
    email: EmailStr
    age: int
    phone: str
    
    @validator('email', pre=True)
    def normalize_email(cls, v):
        if isinstance(v, str):
            return v.lower().strip()
        return v
    
    @validator('phone', pre=True)
    def normalize_phone(cls, v):
        if isinstance(v, str):
            # Remove non-digits
            return re.sub(r'\D', '', v)
        return v

# Usage
from fastapi import FastAPI

app = FastAPI()

@app.post('/users')
async def create_user(user: UserRequest):
    # Data is transformed and validated
    return {"status": "success", "data": user.dict()}
```

### 7. Document Requirements Clearly

```python
from pydantic import BaseModel, EmailStr, Field
from typing import Dict, Any

class UserCreateRequest(BaseModel):
    """
    Create a new user
    
    Attributes:
        email: Email address (required, must be valid)
        age: Age in years (required, 1-150)
        name: Full name (required, 2-100 chars)
        password: Password (required, min 8 chars)
    """
    email: EmailStr = Field(..., description="Valid email address")
    age: int = Field(..., ge=1, le=150, description="Age in years")
    name: str = Field(..., min_length=2, max_length=100, description="Full name")
    password: str = Field(..., min_length=8, description="Password (minimum 8 characters)")
    
    class Config:
        schema_extra = {
            "example": {
                "email": "user@example.com",
                "age": 25,
                "name": "John Doe",
                "password": "securepass123"
            }
        }

# FastAPI automatically generates OpenAPI docs from this
```

### 8. Log Validation Failures

```python
# Helpful for debugging
from fastapi import FastAPI, Request
from fastapi.exceptions import RequestValidationError
from fastapi.responses import JSONResponse
import logging
from datetime import datetime

logger = logging.getLogger(__name__)
app = FastAPI()

@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request: Request, exc: RequestValidationError):
    errors = {}
    for error in exc.errors():
        field = error['loc'][-1]
        errors[field] = error['msg']
    
    # Log validation failure
    logger.warning(
        "Validation failed",
        extra={
            "endpoint": request.url.path,
            "method": request.method,
            "received_data": await request.json(),
            "errors": errors,
            "ip": request.client.host,
            "timestamp": datetime.now().isoformat()
        }
    )
    
    return JSONResponse(
        status_code=400,
        content={
            "status": "error",
            "message": "Validation failed",
            "errors": errors
        }
    )
```

### 9. Use Type-Safe Validation (Python with type hints)

```python
# Define types with Pydantic
from pydantic import BaseModel, EmailStr, Field
from typing import Optional

class CreateUserRequest(BaseModel):
    email: EmailStr
    name: str = Field(min_length=2, max_length=100)
    age: int = Field(ge=1, le=150)
    password: str = Field(min_length=8)

class UserResponse(BaseModel):
    id: int
    email: str
    name: str
    age: int

# FastAPI provides full type safety
from fastapi import FastAPI

app = FastAPI()

@app.post('/users', response_model=UserResponse)
async def create_user(user: CreateUserRequest) -> UserResponse:
    # Type checker (mypy) validates types at development time
    # Pydantic validates at runtime
    created_user = await user_service.create(user)
    return UserResponse(**created_user)

# Type checkers like mypy will catch type errors before runtime
```

### 10. Test Your Validation

```python
# Unit test validation schemas
import pytest
from pydantic import ValidationError
from your_app.schemas import UserCreateRequest

def test_accepts_valid_user():
    """Test that valid user data is accepted"""
    valid_data = {
        "email": "test@example.com",
        "age": 25,
        "name": "John Doe",
        "password": "secure123"
    }
    user = UserCreateRequest(**valid_data)
    assert user.email == "test@example.com"
    assert user.age == 25

def test_rejects_invalid_email():
    """Test that invalid email is rejected"""
    invalid_data = {
        "email": "invalid-email",
        "age": 25,
        "name": "John Doe",
        "password": "secure123"
    }
    with pytest.raises(ValidationError) as exc_info:
        UserCreateRequest(**invalid_data)
    
    errors = exc_info.value.errors()
    assert any(e['loc'] == ('email',) for e in errors)

def test_rejects_invalid_age():
    """Test that age over 150 is rejected"""
    invalid_data = {
        "email": "test@example.com",
        "age": 500,
        "name": "John Doe",
        "password": "secure123"
    }
    with pytest.raises(ValidationError) as exc_info:
        UserCreateRequest(**invalid_data)
    
    errors = exc_info.value.errors()
    assert any(e['loc'] == ('age',) for e in errors)

# Integration test with FastAPI TestClient
from fastapi.testclient import TestClient
from your_app.main import app

client = TestClient(app)

def test_api_validates_user():
    """Test that API endpoint validates user input"""
    response = client.post(
        "/users",
        json={"email": "invalid", "age": 500, "name": "John"}
    )
    assert response.status_code == 422
    assert "email" in str(response.json())
```

---

## Quick Revision & Cheat Sheet

### Key Concepts Recap

| Concept | Definition | Example |
|---------|-----------|---------|
| **Validation** | Checking if data matches requirements | Email format, age range |
| **Transformation** | Converting data to expected format | String "10" → Number 10 |
| **Type validation** | Check data types match | "age": 25 not "age": "25" |
| **Syntactic validation** | Check format/pattern | Email, phone, date |
| **Semantic validation** | Check real-world logic | Age 0-150, birthdate in past |
| **Schema** | Definition of expected data structure | { email: string, age: number } |
| **Middleware** | Code that runs for every request | Validation layer |

### Validation Checklist

- [ ] Define schema for every endpoint input
- [ ] Validate request body
- [ ] Validate query parameters
- [ ] Validate path parameters
- [ ] Validate headers if needed
- [ ] Transform data before validation
- [ ] Return 400 for validation errors
- [ ] Include specific error messages
- [ ] Test validation with curl/Postman
- [ ] Don't trust frontend validation
- [ ] Use validation library (don't roll your own)
- [ ] Document requirements
- [ ] Log validation failures

### Common Validation Patterns

```python
from pydantic import BaseModel, Field, validator
from typing import Literal
import re

# Required field
class Example1(BaseModel):
    name: str  # Required by default

# Type checking
class Example2(BaseModel):
    text: str
    count: int
    active: bool
    items: list

# String length
class Example3(BaseModel):
    username: str = Field(min_length=3, max_length=100)

# Number range
class Example4(BaseModel):
    percentage: int = Field(ge=0, le=100)

# Pattern matching
class Example5(BaseModel):
    code: str
    
    @validator('code')
    def validate_code(cls, v):
        if not re.match(r'^[a-zA-Z0-9]+$', v):
            raise ValueError('Invalid format')
        return v

# Predefined values (enum)
class Example6(BaseModel):
    status: Literal['active', 'inactive', 'pending']

# Email format
from pydantic import EmailStr

class Example7(BaseModel):
    email: EmailStr

# Custom validation
class Example8(BaseModel):
    age: int
    
    @validator('age')
    def validate_age(cls, v):
        if v <= 0:
            raise ValueError('Age must be positive')
        return v

# Cross-field validation
from pydantic import root_validator

class Example9(BaseModel):
    password: str
    confirm_password: str
    
    @root_validator
    def passwords_match(cls, values):
        if values.get('password') != values.get('confirm_password'):
            raise ValueError('Passwords must match')
        return values

# Conditional requirement
from typing import Optional

class Example10(BaseModel):
    method: Literal['pickup', 'deliver']
    address: Optional[str] = None
    
    @root_validator
    def validate_address(cls, values):
        if values.get('method') == 'deliver' and not values.get('address'):
            raise ValueError('Address required for delivery')
        return values
```

### Error Response Template

```python
# Always use this format
from fastapi import FastAPI
from fastapi.exceptions import RequestValidationError
from fastapi.responses import JSONResponse
from starlette import status

app = FastAPI()

@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request, exc):
    errors = {}
    for error in exc.errors():
        field = error['loc'][-1]
        errors[field] = error['msg']
    
    return JSONResponse(
        status_code=400,
        content={
            "status": "error",
            "message": "Validation failed",
            "errors": errors
        }
    )

# Example response:
# {
#   "status": "error",
#   "message": "Validation failed",
#   "errors": {
#     "fieldName": "Human-readable error message",
#     "anotherField": "Another error message"
#   }
# }

# HTTP status codes
# 400  # Bad Request (validation failed)
# 422  # Unprocessable Entity (validation failed) - FastAPI default
# 401  # Unauthorized (auth validation failed)
# 403  # Forbidden (permission validation failed)
```

### Quick Decision Tree

```
Is the input from a client? 
├─ YES → MUST VALIDATE
└─ NO → Still validate if possible

Does the field represent real-world data?
├─ YES → Semantic validation needed
└─ NO → Type validation sufficient

Should multiple fields interact?
├─ YES → Cross-field validation needed
└─ NO → Individual field validation OK

Is the format strictly defined?
├─ YES → Syntactic validation needed
└─ NO → Type validation sufficient

Can frontend guarantee validation?
└─ NO → Backend must validate (always false)
```

### Interview Tips

**When asked about validation:**

1. **Emphasize security**: "Validation is critical for security and data integrity"

2. **Mention frontend vs backend**: "Always validate on the backend regardless of frontend validation"

3. **Give examples**: "Type validation ensures `age` is a number, semantic validation ensures age is between 0-150"

4. **Explain the entry point**: "Validation happens immediately after route matching, before business logic"

5. **Show awareness of pitfalls**: "Don't validate after processing, validate before passing to service layer"

6. **Mention libraries**: "Use established libraries like Joi, Yup, or Zod rather than custom validation"

7. **Explain error handling**: "Always return 400 Bad Request with specific error messages"

8. **Show complete picture**: "Validation + transformation work together: transform the data, then validate it"

### Code Template

```python
# Complete validation example
from fastapi import FastAPI
from pydantic import BaseModel, EmailStr, Field
from fastapi.exceptions import RequestValidationError
from fastapi.responses import JSONResponse

app = FastAPI()

# Define validation schema
class UserCreateRequest(BaseModel):
    email: EmailStr
    age: int = Field(ge=1, le=150)
    password: str = Field(min_length=8)

# Custom error handler (optional)
@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request, exc):
    errors = {}
    for error in exc.errors():
        field = error['loc'][-1]
        errors[field] = error['msg']
    
    return JSONResponse(
        status_code=400,
        content={
            "status": "error",
            "message": "Validation failed",
            "errors": errors
        }
    )

@app.post('/users', status_code=201)
async def create_user(user: UserCreateRequest):
    # Validation happens automatically via Pydantic
    # If validation fails, custom error handler is called
    
    # Safe to proceed - data is valid
    created_user = await user_service.create_user(user.dict())
    
    return {
        "status": "success",
        "data": created_user
    }
```

---

## Summary

Validation and transformation are **not optional nice-to-haves**—they're **essential security and data integrity measures**.

### Key Takeaways

1. **Always validate at the entry point** (controller/middleware), before any business logic
2. **Never trust frontend validation** for security—it's only for UX
3. **Validate all input sources**: body, query, params, headers
4. **Transform data first**, then validate
5. **Use established libraries** rather than rolling custom validation
6. **Return clear, specific error messages** (status 400)
7. **Validate multiple ways**:
   - Type validation (is it a number?)
   - Syntactic validation (does it match the pattern?)
   - Semantic validation (does it make logical sense?)
   - Cross-field validation (do related fields make sense together?)
8. **Document requirements** in code and API docs
9. **Test your validation** with unit and integration tests
10. **Remember**: A system is only as secure as its input validation

Good validation prevents 90% of data-related bugs before they happen.

---

