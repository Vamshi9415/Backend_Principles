

---

# **Backend Routing**

> A complete, exhaustive, from-zero-to-advanced explanation of routing in backend systems.

---

# Table of Contents

1. What is Routing?
2. HTTP Methods vs Routing: The “What” vs The “Where”
3. The Big Picture: How a Request is Handled in a Server
4. Static Routes
5. Dynamic Routes (Path Parameters)
6. Query Parameters
7. Path Parameters vs Query Parameters (Comparison Table)
8. Nested Routes
9. Route Versioning and Deprecation
10. Catch-All Routes (404 Handling)
11. How Route Matching Works Internally
12. Complete Example Backend (Express.js)
13. Common Mistakes and How to Avoid Them
14. Debugging and Troubleshooting Routing Issues
15. Best Practices for Designing Routes
16. RESTful Semantics and Why Routes Are Designed This Way
17. Quick Reference / Cheat Sheet

---

# 1. What is Routing?

> **Routing is basically mapping URL parameters to server-side logic.**

In the transcript, the core idea is:

> Routing expresses **WHERE** your request should go.

If HTTP methods express **WHAT you want to do**, then routing expresses:

> **On which resource do you want to do it?**

---

## 🏙️ Real-World Analogy: City + Addresses

Imagine:

* HTTP Method = What you want to do

  * GET = Look at something
  * POST = Add something
  * PUT = Replace something
  * DELETE = Remove something

* Route (URL) = Address of the building

> “I want to GET something from **Building No. 42, Books Street**”

The server is like a **city administration system** that:

1. Reads what you want to do (method)
2. Reads where you want to go (route)
3. Sends you to the correct office (handler function)

---

# 2. HTTP Methods vs Routing: The “What” vs The “Where”

From your transcript:

> HTTP methods describe the **WHAT** of a request.
> Routing describes the **WHERE** of a request.

Example:

```
GET /api/books
```

* `GET` → I want to **fetch** data
* `/api/books` → From **books resource**

---

## 🧠 Mental Model

| Part          | Meaning               |
| ------------- | --------------------- |
| Method        | What action           |
| Route         | On which resource     |
| Both together | Which code should run |

---

# 3. The Big Picture: How a Request is Handled

From the transcript:

> The server takes the method and the route and maps it to a particular handler and performs business logic, database operations, and returns data.

---

## 🔁 Step-by-step Flow

```
Client → HTTP Request → Server
                  ↓
           Method + Route
                  ↓
           Route Matching
                  ↓
              Handler
                  ↓
        Business Logic + DB
                  ↓
              Response
```

---

# 4. Static Routes

From transcript:

> These routes are called static routes because they don’t have any variable parameters inside them.

Example:

```
GET /api/books
POST /api/books
```

They **never change**.

---

## 🏠 Real-World Analogy

A **fixed shop**:

> “Go to Shop No. 10 every time”

Not:

> “Go to Shop No. {X}”

---

## 🧪 FastAPI Example

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/api/books")
async def get_books():
    return ["Book 1", "Book 2"]

@app.post("/api/books")
async def create_book():
    return {"message": "Book created"}

# Run with: uvicorn main:app --reload --port 3000
```

---

## ✅ Characteristics

* Fixed string
* No variables
* Always same structure
* Same type of response

---

# 5. Dynamic Routes (Path Parameters)

From transcript:

> This part is called a route parameter or path parameter.

Example:

```
GET /api/users/123
```

Here:

* `123` is **dynamic**
* Server extracts it from URL

---

## 🧠 Server-side Matching

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/api/users/{id}")
async def get_user(id: str):
    return {"userId": id}
```

---

## 🧱 Visual Breakdown

```
/api/users/:id
/api/users/123  ← id = "123"
/api/users/999  ← id = "999"
```

---

## 🧍 Real-World Analogy

Apartment building:

> “Go to Building ‘Users’, Flat Number 123”

---

## 🧠 Why Path Parameters Exist?

Because:

* They represent **identity**
* They express **semantic meaning**
* They are part of the **resource path**

> `/users/123` means: *The user whose id is 123*

---

# 6. Query Parameters

From transcript:

> In GET requests, we don’t have a body, so we send data using query parameters.

Example:

```
GET /api/search?query=some+value
```

---

## 🧱 Visual

```
/api/books?page=2&limit=20
```

---

## 🧪 FastAPI Example

```python
from fastapi import FastAPI
from typing import Optional

app = FastAPI()

@app.get("/api/books")
async def get_books(page: Optional[int] = None, limit: Optional[int] = None):
    return {"page": page, "limit": limit}
```

---

## 📦 Real-World Analogy

You go to a shop and say:

> “Show me books, **but sorted by price**, **and only 20 items**”

The shop = `/api/books`
Your instructions = `?sort=price&limit=20`

---

## 🧠 Why Not Use Path Parameters For This?

From transcript:

> It defeats the purpose of REST semantics.

This is BAD:

```
/api/search/someRandomUserInput
```

Because:

* That value is not identity
* It’s not hierarchical
* It’s not a resource

---

# 7. Path vs Query Parameters

| Feature           | Path Param   | Query Param                  |
| ----------------- | ------------ | ---------------------------- |
| Part of route     | Yes          | No                           |
| Semantic identity | Yes          | No                           |
| Used for          | Resource ID  | Filters, pagination, sorting |
| Example           | `/users/123` | `?page=2`                    |

---

# 8. Nested Routes

From transcript:

> We nest resources to express semantic meaning.

Example:

```
/api/users/123/posts/456
```

---

## 🧠 Meaning

> Get post **456** of user **123**

---

## 🧪 FastAPI Example

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/api/users/{userId}/posts/{postId}")
async def get_user_post(userId: str, postId: str):
    return {"userId": userId, "postId": postId}
```

---

## 🏢 Real-World Analogy

Country → State → City → Street → House

Each level adds **context**.

---

# 9. Route Versioning and Deprecation

From transcript:

```
/api/v1/products
/api/v2/products
```

---

## 🧠 Why It Exists?

Because:

* APIs change
* Clients depend on old format
* You cannot break them suddenly

---

## 🧪 FastAPI Example

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/api/v1/products")
async def get_products_v1():
    return {"id": 1, "name": "Phone", "price": 100}

@app.get("/api/v2/products")
async def get_products_v2():
    return {"id": 1, "title": "Phone", "price": 100}
```

---

## 🏗️ Real-World Analogy

Like:

> “Road Version 1” and “Road Version 2” while people slowly migrate

---

# 10. Catch-All Routes (404 Handling)

From transcript:

> After serving all routes, we add a final route to catch everything else.

---

## 🧪 FastAPI Example

```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

app = FastAPI()

# Define all your routes first

# Catch-all route (must be defined last)
@app.api_route("/{full_path:path}", methods=["GET", "POST", "PUT", "DELETE", "PATCH"])
async def catch_all(request: Request, full_path: str):
    return JSONResponse(
        status_code=404,
        content={"message": "Route not found"}
    )
```

---

## 🧱 Why It Exists?

Otherwise:

* Server returns nothing
* Client gets confusing error

---

# 11. How Route Matching Works

From transcript:

> Server checks method, then route, then maps to handler.

---

## 🧠 Matching Priority

Order matters:

```python
@app.get("/users/{id}")
async def get_user(id: str):
    ...

@app.get("/users/profile")  # This might never be reached!
async def get_profile():
    ...

# Better: Define specific routes BEFORE dynamic ones
@app.get("/users/profile")
async def get_profile():
    ...

@app.get("/users/{id}")
async def get_user(id: str):
    ...
```

---

# 12. Complete Example Backend

```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse
from typing import Optional

app = FastAPI()

# Static
@app.get("/api/books")
async def get_books():
    return ["Book A", "Book B"]

# Dynamic
@app.get("/api/users/{id}")
async def get_user(id: str):
    return {"userId": id}

# Query
@app.get("/api/search")
async def search(q: Optional[str] = None):
    return {"query": q}

# Nested
@app.get("/api/users/{userId}/posts/{postId}")
async def get_user_post(userId: str, postId: str):
    return {"userId": userId, "postId": postId}

# Versioning
@app.get("/api/v1/products")
async def get_products_v1():
    return {"id": 1, "name": "Phone"}

@app.get("/api/v2/products")
async def get_products_v2():
    return {"id": 1, "title": "Phone"}

# Catch all (must be last)
@app.api_route("/{full_path:path}", methods=["GET", "POST", "PUT", "DELETE", "PATCH"])
async def catch_all(request: Request, full_path: str):
    return JSONResponse(
        status_code=404,
        content={"message": "Not Found"}
    )

# Run with: uvicorn main:app --reload --port 3000
```

---

# 13. Common Mistakes

* Using path params for filters
* Breaking APIs without versioning
* Wrong route order
* Not handling 404
* Inconsistent naming

---

# 14. Debugging Routing

* Log incoming method + URL
* Print matched handler
* Use Postman / curl
* Check order of routes

---

# 15. Best Practices

* Use nouns, not verbs
* Use versioning
* Keep routes semantic
* Use nesting carefully
* Be consistent

---

# 16. REST Semantics Summary

> Routes describe **resources**, not actions.

Bad:

```
/getUser
```

Good:

```
GET /users/123
```

---

# 17. Quick Cheat Sheet

```
GET    /users          → list users
GET    /users/123      → get user
POST   /users          → create user
PUT    /users/123      → replace user
PATCH  /users/123      → update user
DELETE /users/123      → delete user
```

---

# Final Mental Model

> Routing = **Mapping (Method + URL) → Code**

---

