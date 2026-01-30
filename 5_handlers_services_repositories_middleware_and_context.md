# Handlers, Services, Repositories, Middleware, and Request Context

A comprehensive guide to backend architecture patterns and request lifecycle management.

---

## Table of Contents

1. [Introduction](#introduction)
2. [Request Lifecycle Overview](#request-lifecycle-overview)
3. [Handlers, Services, and Repository Pattern](#handlers-services-and-repository-pattern)
   - [What is the Pattern](#what-is-the-pattern)
   - [Why Use This Pattern](#why-use-this-pattern)
   - [Handler/Controller Layer](#handlercontroller-layer)
   - [Service Layer](#service-layer)
   - [Repository Layer](#repository-layer)
   - [How They Work Together](#how-they-work-together)
4. [Middleware](#middleware)
   - [What is Middleware](#what-is-middleware)
   - [Why Middleware Exists](#why-middleware-exists)
   - [How Middleware Works](#how-middleware-works)
   - [Common Middleware Types](#common-middleware-types)
   - [Middleware Ordering](#middleware-ordering)
5. [Request Context](#request-context)
   - [What is Request Context](#what-is-request-context)
   - [Why We Need It](#why-we-need-it)
   - [How It Works](#how-it-works)
   - [Common Use Cases](#common-use-cases)
6. [Complete Request Flow Example](#complete-request-flow-example)
7. [Real-World Analogies](#real-world-analogies)
8. [Common Mistakes and Pitfalls](#common-mistakes-and-pitfalls)
9. [Best Practices](#best-practices)
10. [Quick Revision Cheat Sheet](#quick-revision-cheat-sheet)

---

## Introduction

When building backend applications, understanding how a request flows through your server is critical. From the moment a client sends an HTTP request until it receives a response, multiple layers of your application work together to:

- Receive and validate data
- Process business logic
- Interact with databases
- Handle errors
- Send responses

This guide covers three fundamental concepts that form the backbone of scalable, maintainable backend architectures:

1. **Handler-Service-Repository Pattern**: A layered architecture that separates concerns
2. **Middleware**: Reusable functions that intercept requests before they reach handlers
3. **Request Context**: Shared state that flows through the entire request lifecycle

These patterns are language-agnostic and appear in Node.js, Go, Python, Java, Rust, and virtually every modern backend framework.

---

## Request Lifecycle Overview

### What Happens When a Client Sends a Request?

```
┌─────────┐                                    ┌─────────┐
│ Client  │────── HTTP Request ───────────────>│ Server  │
└─────────┘                                    └─────────┘
                                                    │
                                                    ▼
                                            ┌───────────────┐
                                            │  Entry Point  │
                                            │ (Port 3000)   │
                                            └───────────────┘
                                                    │
                                                    ▼
                                            ┌───────────────┐
                                            │   Middleware  │
                                            │   Pipeline    │
                                            └───────────────┘
                                                    │
                                                    ▼
                                            ┌───────────────┐
                                            │    Routing    │
                                            │   Algorithm   │
                                            └───────────────┘
                                                    │
                                                    ▼
                                            ┌───────────────┐
                                            │   Middleware  │
                                            │ (Route-level) │
                                            └───────────────┘
                                                    │
                                                    ▼
                                            ┌───────────────┐
                                            │    Handler    │
                                            │  (Controller) │
                                            └───────────────┘
                                                    │
                                                    ▼
                                            ┌───────────────┐
                                            │    Service    │
                                            │     Layer     │
                                            └───────────────┘
                                                    │
                                                    ▼
                                            ┌───────────────┐
                                            │  Repository   │
                                            │   (Database)  │
                                            └───────────────┘
                                                    │
                                                    ▼
                                            ┌───────────────┐
                                            │   Response    │
                                            │  Sent Back    │
                                            └───────────────┘
```

### The Journey of a Request

1. **Entry Point**: OS forwards the request to your server's port
2. **Global Middleware**: Security, logging, CORS, etc.
3. **Routing**: Matches URL pattern to a specific handler
4. **Route-Specific Middleware**: Authentication, validation, etc.
5. **Handler**: Extracts data, validates, calls service
6. **Service**: Business logic execution
7. **Repository**: Database operations
8. **Response**: Back through the layers to the client

---

## Handlers, Services, and Repository Pattern

### What is the Pattern?

The **Handler-Service-Repository (HSR)** pattern is a layered architecture that separates your backend code into three distinct responsibilities:

- **Handler (Controller)**: Handles HTTP concerns (request/response)
- **Service**: Contains business logic
- **Repository**: Manages data persistence

This is also known as the **three-tier architecture** or **layered architecture**.

### Why Use This Pattern?

#### ❌ Without the Pattern

```python
# Everything in one function - NOT RECOMMENDED
from fastapi import FastAPI, HTTPException, Request
from datetime import datetime
import uuid

app = FastAPI()

@app.post("/api/books")
async def create_book(request: Request):
    # HTTP parsing
    data = await request.json()
    title = data.get('title')
    author = data.get('author')
    isbn = data.get('isbn')
    
    # Validation
    if not title or len(title) < 3:
        raise HTTPException(status_code=400, detail="Invalid title")
    
    # Business logic
    book = {
        "id": str(uuid.uuid4()),
        "title": title.strip(),
        "author": author.strip(),
        "isbn": isbn,
        "created_at": datetime.now()
    }
    
    # Database operation
    sql = 'INSERT INTO books (id, title, author, isbn, created_at) VALUES (?, ?, ?, ?, ?)'
    await db.execute(sql, [book["id"], book["title"], book["author"], book["isbn"], book["created_at"]])
    
    # Another database operation
    user_sql = 'UPDATE users SET book_count = book_count + 1 WHERE id = ?'
    await db.execute(user_sql, [request.state.user_id])
    
    # Email sending
    await send_email(request.state.user_email, 'Book created!')
    
    # HTTP response
    return book
```

**Problems:**
- Hard to test (everything is coupled)
- Cannot reuse logic (tied to HTTP)
- Difficult to maintain
- Mixed concerns
- Cannot easily change database or framework

#### ✅ With the Pattern

```python
# HANDLER - handles HTTP concerns
from fastapi import APIRouter, Request, status
from pydantic import BaseModel

router = APIRouter()

class CreateBookRequest(BaseModel):
    title: str
    author: str
    isbn: str | None = None

@router.post("/api/books", status_code=status.HTTP_201_CREATED)
async def create_book_handler(book_data: CreateBookRequest, request: Request):
    user_id = request.state.user_id  # from auth middleware
    
    book = await book_service.create_book(
        title=book_data.title,
        author=book_data.author,
        isbn=book_data.isbn,
        user_id=user_id
    )
    
    return book

# SERVICE - business logic
from datetime import datetime
import uuid

class BookService:
    async def create_book(self, title: str, author: str, isbn: str | None, user_id: str):
        # Business logic
        book = {
            "id": str(uuid.uuid4()),
            "title": title.strip(),
            "author": author.strip(),
            "isbn": isbn,
            "created_at": datetime.now()
        }
        
        # Orchestrate multiple operations
        await book_repository.insert(book)
        await user_repository.increment_book_count(user_id)
        await email_service.send_book_created_email(user_id)
        
        return book

# REPOSITORY - database operations
class BookRepository:
    async def insert(self, book: dict):
        sql = 'INSERT INTO books (id, title, author, isbn, created_at) VALUES (?, ?, ?, ?, ?)'
        return await db.execute(sql, [book["id"], book["title"], book["author"], book["isbn"], book["created_at"]])
```

**Benefits:**
- ✅ Each layer has a single responsibility
- ✅ Easy to test each layer independently
- ✅ Can reuse service logic (in CLI, cron jobs, different APIs)
- ✅ Can swap databases without changing business logic
- ✅ Can change HTTP framework without changing core logic
- ✅ Easier to maintain and debug

---

### Handler/Controller Layer

#### What is a Handler?

A handler (also called controller) is a function that:
- Receives HTTP requests
- Extracts data from requests
- Validates and transforms data
- Calls appropriate service methods
- Sends HTTP responses

#### Responsibilities

1. **Data Extraction**: Get data from request (body, query params, path params, headers)
2. **Deserialization**: Convert JSON to native language objects
3. **Validation**: Ensure data meets requirements
4. **Transformation**: Apply defaults, normalize data
5. **Service Invocation**: Call business logic layer
6. **Response Formation**: Send appropriate HTTP response codes and data
7. **Error Handling**: Convert errors to HTTP error responses

#### What a Handler Receives

Every handler typically receives request object and any dependencies:

```python
# Python (FastAPI)
from fastapi import Request, Depends
from pydantic import BaseModel

class BookRequest(BaseModel):
    title: str
    author: str

@app.post("/api/books")
async def handler(book: BookRequest, request: Request):
    # book: validated request data (Pydantic model)
    # request: request object with headers, state, etc.
    return {"message": "Book created"}

# Go (using Gin framework)
func handler(c *gin.Context) {
  // c contains both request and response
}

# Node.js / Express
function handler(req, res, next) {
  // req: request object
  // res: response object
  // next: pass to next middleware (optional)
}
```

#### Handler Examples

**Example 1: GET Request - Fetch Books**

```python
# GET /api/books?sort=name&limit=10
from fastapi import APIRouter, HTTPException, Query
from typing import Optional

router = APIRouter()

@router.get("/api/books")
async def get_books_handler(
    sort: str = Query("date", description="Sort by field"),
    limit: int = Query(10, ge=1, le=100, description="Number of books to return"),
    offset: int = Query(0, ge=0, description="Offset for pagination")
):
    try:
        # 1. Extract data from query params (done by FastAPI)
        
        # 2. Validation (FastAPI validates via Query parameters)
        # Limit is already validated to be between 1 and 100
        
        # 3. Transformation is automatic with type hints
        # FastAPI converts strings to integers automatically
        
        # 4. Call service layer
        books = await book_service.get_books(
            sort=sort,
            limit=limit,
            offset=offset
        )
        
        # 5. Send response
        return {
            "data": books,
            "pagination": {
                "limit": limit,
                "offset": offset
            }
        }
        
    except Exception as error:
        # 6. Error handling
        print(f"Error in get_books_handler: {error}")
        raise HTTPException(status_code=500, detail="Internal server error")
```

**Example 2: POST Request - Create Book**

```python
# POST /api/books
from fastapi import APIRouter, HTTPException, Request, status
from pydantic import BaseModel, Field, validator
from typing import Optional

router = APIRouter()

class CreateBookRequest(BaseModel):
    title: str = Field(..., min_length=3)
    author: str = Field(..., min_length=2)
    isbn: Optional[str] = None
    published_year: Optional[int] = None
    
    @validator('title', 'author')
    def strip_whitespace(cls, v):
        return v.strip() if v else v

@router.post("/api/books", status_code=status.HTTP_201_CREATED)
async def create_book_handler(book_data: CreateBookRequest, request: Request):
    try:
        # 1. Extract data from request body (done by Pydantic)
        # 2. Get authenticated user from state (set by auth middleware)
        user_id = request.state.user_id
        
        # 3. Validation (done by Pydantic model)
        # Title and author are already validated
        
        # 4. Transformation (done by Pydantic validators)
        # Data is already stripped and transformed
        
        # 5. Call service layer
        book = await book_service.create_book(
            title=book_data.title,
            author=book_data.author,
            isbn=book_data.isbn,
            published_year=book_data.published_year,
            user_id=user_id
        )
        
        # 6. Send response (201 Created for successful POST)
        return book
        
    except ValueError as error:
        print(f"Error in create_book_handler: {error}")
        
        # More specific error handling
        if "DUPLICATE_ISBN" in str(error):
            raise HTTPException(
                status_code=status.HTTP_409_CONFLICT,
                detail="Book with this ISBN already exists"
            )
        
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail="Internal server error"
        )
```

**Example 3: DELETE Request**

```python
# DELETE /api/books/:id
from fastapi import APIRouter, HTTPException, Request, status, Path
import uuid

router = APIRouter()

def is_valid_uuid(value: str) -> bool:
    try:
        uuid.UUID(value)
        return True
    except ValueError:
        return False

@router.delete("/api/books/{book_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_book_handler(book_id: str = Path(...), request: Request = None):
    try:
        # 1. Extract path parameter (done by FastAPI)
        # 2. Get user from state
        user_id = request.state.user_id
        user_role = request.state.role
        
        # 3. Validation
        if not book_id or not is_valid_uuid(book_id):
            raise HTTPException(
                status_code=status.HTTP_400_BAD_REQUEST,
                detail="Invalid book ID"
            )
        
        # 4. Call service layer (service handles permission checks)
        await book_service.delete_book(
            book_id=book_id,
            user_id=user_id,
            user_role=user_role
        )
        
        # 5. Send response (204 No Content for successful DELETE)
        return None  # FastAPI returns 204 automatically
        
    except ValueError as error:
        error_msg = str(error)
        if "NOT_FOUND" in error_msg:
            raise HTTPException(
                status_code=status.HTTP_404_NOT_FOUND,
                detail="Book not found"
            )
        
        if "FORBIDDEN" in error_msg:
            raise HTTPException(
                status_code=status.HTTP_403_FORBIDDEN,
                detail="You do not have permission to delete this book"
            )
        
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail="Internal server error"
        )
```

#### Deserialization in Different Languages

**Python (FastAPI):**
```python
from pydantic import BaseModel
from fastapi import FastAPI

app = FastAPI()

class CreateBookRequest(BaseModel):
    title: str
    author: str
    isbn: str | None = None

@app.post("/api/books")
async def create_book(book: CreateBookRequest):
    # FastAPI automatically deserializes and validates
    # book is a Python object with type checking
    # Access fields like: book.title, book.author
    pass
```

**Go:**
```go
// Must explicitly bind JSON to struct
type CreateBookRequest struct {
    Title  string `json:"title" binding:"required"`
    Author string `json:"author" binding:"required"`
}

func createBookHandler(c *gin.Context) {
    var req CreateBookRequest
    
    // Bind and validate
    if err := c.ShouldBindJSON(&req); err != nil {
        c.JSON(400, gin.H{"error": err.Error()})
        return
    }
    
    // Now req is a Go struct with validated data
}
```

**JavaScript/Node.js:**
```javascript
// Usually handled by middleware (express.json())
app.use(express.json());

// In handler, you get parsed object directly
function handler(req, res) {
  const { name, age } = req.body; // Already a JS object
}
```

#### When to Use Handlers

**Always** - Every API endpoint needs a handler. The handler is the entry point for your business logic.

#### Common Mistakes in Handlers

❌ **Putting business logic in handlers**
```python
# WRONG - business logic in handler
from datetime import datetime

@app.post("/api/books")
async def create_book_handler(book_data: CreateBookRequest):
    book = {**book_data.dict(), "id": str(uuid.uuid4())}
    
    # This business logic should be in service layer
    if book.get("published_year", 0) > datetime.now().year:
        raise HTTPException(
            status_code=400,
            detail="Future publication date not allowed"
        )
    
    await db.execute('INSERT INTO books ...', book)
    return book
```

✅ **Correct - delegate to service layer**
```python
@app.post("/api/books")
async def create_book_handler(book_data: CreateBookRequest):
    book = await book_service.create_book(book_data.dict())
    return book

# Business logic in service
class BookService:
    async def create_book(self, data: dict):
        if data.get("published_year", 0) > datetime.now().year:
            raise ValueError("Future publication date not allowed")
        # ... rest of logic
```

---

### Service Layer

#### What is a Service?

A service is a module/class/set of functions that contains your **business logic**. It's isolated from HTTP concerns and database implementation details.

#### Key Characteristics

- **No HTTP dependencies**: Should not know about `req`, `res`, or HTTP status codes
- **Framework-agnostic**: Can be used in API, CLI, cron jobs, message queues
- **Orchestrates operations**: Coordinates multiple repository calls, external APIs, etc.
- **Returns data or throws errors**: Doesn't decide on HTTP response codes

#### Responsibilities

1. **Business Logic**: Core rules and requirements of your application
2. **Orchestration**: Coordinate multiple repository operations
3. **Data Processing**: Transform data according to business rules
4. **External Service Calls**: Call third-party APIs, send emails, etc.
5. **Transaction Management**: Ensure data consistency across operations
6. **Error Handling**: Throw meaningful business errors

#### Service Examples

**Example 1: Simple Service - Get Books**

```python
# book_service.py
from datetime import datetime
from typing import List, Dict

class BookService:
    def __init__(self, book_repository):
        self.book_repository = book_repository
    
    async def get_books(
        self,
        sort: str = "date",
        limit: int = 10,
        offset: int = 0
    ) -> List[Dict]:
        # 1. Apply defaults and validation
        valid_sort = sort if sort in ["date", "name"] else "date"
        
        # 2. Call repository
        books = await self.book_repository.find_all(
            sort_by=valid_sort,
            limit=limit,
            offset=offset
        )
        
        # 3. Business logic - maybe filter or transform
        # For example, hide books that are not published yet
        published_books = [
            book for book in books
            if book.get("published_date") and book["published_date"] <= datetime.now()
        ]
        
        return published_books
```

**Example 2: Complex Service - Create Book**

```python
from datetime import datetime
import uuid
from typing import Optional

class BusinessError(Exception):
    def __init__(self, message: str, code: str):
        self.message = message
        self.code = code
        super().__init__(self.message)

class BookService:
    def __init__(self, book_repository, user_repository, search_service, email_service):
        self.book_repository = book_repository
        self.user_repository = user_repository
        self.search_service = search_service
        self.email_service = email_service
    
    async def create_book(
        self,
        title: str,
        author: str,
        isbn: Optional[str],
        published_year: Optional[int],
        user_id: str
    ) -> dict:
        # 1. Business validation
        if published_year and published_year > datetime.now().year:
            raise BusinessError(
                "Cannot create book with future publication year",
                "INVALID_YEAR"
            )
        
        # 2. Check for duplicates
        if isbn:
            existing = await self.book_repository.find_by_isbn(isbn)
            if existing:
                raise BusinessError(
                    "Book with this ISBN already exists",
                    "DUPLICATE_ISBN"
                )
        
        # 3. Prepare data
        book = {
            "id": str(uuid.uuid4()),
            "title": title.strip(),
            "author": author.strip(),
            "isbn": isbn,
            "published_year": published_year,
            "user_id": user_id,
            "created_at": datetime.now(),
            "updated_at": datetime.now()
        }
        
        # 4. Orchestrate multiple operations
        # Insert book
        await self.book_repository.insert(book)
        
        # Update user's book count
        await self.user_repository.increment_book_count(user_id)
        
        # Add to search index (external service)
        await self.search_service.index_book(book)
        
        # Send notification email
        await self.email_service.send_book_created_notification(user_id, book)
        
        # 5. Return data
        return book
```

**Example 3: Service with Transaction**

```python
from typing import Dict

class ForbiddenError(Exception):
    pass

class NotFoundError(Exception):
    pass

class BookService:
    def __init__(self, database, book_repository, user_repository, transfer_log_repository, email_service):
        self.database = database
        self.book_repository = book_repository
        self.user_repository = user_repository
        self.transfer_log_repository = transfer_log_repository
        self.email_service = email_service
    
    async def transfer_book_ownership(
        self,
        book_id: str,
        from_user_id: str,
        to_user_id: str,
        requester_id: str
    ) -> Dict[str, bool]:
        # 1. Permission check
        if requester_id != from_user_id:
            raise ForbiddenError('You can only transfer your own books')
        
        # 2. Verify book exists and belongs to user
        book = await self.book_repository.find_by_id(book_id)
        if not book:
            raise NotFoundError('Book not found')
        
        if book["user_id"] != from_user_id:
            raise ForbiddenError('This book does not belong to you')
        
        # 3. Verify recipient exists
        recipient = await self.user_repository.find_by_id(to_user_id)
        if not recipient:
            raise NotFoundError('Recipient user not found')
        
        # 4. Perform transfer in transaction (all or nothing)
        async with self.database.transaction() as trx:
            # Update book ownership
            await self.book_repository.update_owner(book_id, to_user_id, trx)
            
            # Decrement old owner's count
            await self.user_repository.decrement_book_count(from_user_id, trx)
            
            # Increment new owner's count
            await self.user_repository.increment_book_count(to_user_id, trx)
            
            # Create transfer log
            await self.transfer_log_repository.create({
                "book_id": book_id,
                "from_user_id": from_user_id,
                "to_user_id": to_user_id,
                "timestamp": datetime.now()
            }, trx)
        
        # 5. Send notifications
        await self.email_service.send_transfer_notification(from_user_id, to_user_id, book)
        
        return {"success": True}
```

**Example 4: Service Without Database Calls**

```python
# Not all services need to call repositories
class RecommendationService:
    def __init__(self, user_repository, external_api, email_service):
        self.user_repository = user_repository
        self.external_api = external_api
        self.email_service = email_service
    
    async def send_book_recommendations(self, user_id: str) -> Dict[str, any]:
        # 1. Get user preferences
        user = await self.user_repository.find_by_id(user_id)
        
        # 2. Call external recommendation API
        recommendations = await self.external_api.get_recommendations(
            genres=user["favorite_genres"],
            authors=user["favorite_authors"]
        )
        
        # 3. Send email (no database operation)
        await self.email_service.send_recommendation_email(
            user["email"],
            recommendations
        )
        
        return {"sent": True, "count": len(recommendations)}
```

#### Service Pattern Guidelines

**Single Responsibility:**
```python
# GOOD - Each service method does one thing
class BookService:
    async def create_book(self, data): ...
    async def update_book(self, book_id, data): ...
    async def delete_book(self, book_id): ...
    async def get_book(self, book_id): ...
    async def get_all_books(self, filters): ...

# BAD - Too much in one method
class BookService:
    async def handle_book_operation(self, action, data): ...
```

**No HTTP Awareness:**
```python
# WRONG - Service should not know about HTTP
from fastapi import Response

async def create_book(self, response: Response, data: dict):
    # ...
    response.status_code = 201  # ❌ Service shouldn't handle responses
    return book

# CORRECT - Service just returns data or raises errors
async def create_book(self, data: dict):
    # ...
    return book  # ✅ Handler decides what HTTP response to send
```

**Meaningful Errors:**
```python
class BusinessError(Exception):
    def __init__(self, message: str, code: str):
        self.message = message
        self.code = code
        super().__init__(self.message)

class BookService:
    async def create_book(self, data: dict):
        if data.get("isbn"):
            existing = await self.book_repository.find_by_isbn(data["isbn"])
            if existing:
                # Raise business error - handler will convert to HTTP error
                raise BusinessError("Duplicate ISBN", "DUPLICATE_ISBN")
        # ...
```

---

### Repository Layer

#### What is a Repository?

A repository is a module that encapsulates **all database operations** for a specific entity. It's the only layer that should directly interact with the database.

#### Responsibilities

1. **Database Queries**: Construct and execute SQL/NoSQL queries
2. **Data Mapping**: Convert database rows to application objects
3. **Single Responsibility**: One repository method = one database operation
4. **No Business Logic**: Just CRUD operations and queries

#### Repository Examples

**Example 1: Book Repository**

```python
# book_repository.py
from typing import List, Optional, Dict
from datetime import datetime

class BookRepository:
    def __init__(self, db):
        self.db = db
    
    # Find all books with filtering and sorting
    async def find_all(
        self,
        sort_by: str = "date",
        limit: int = 10,
        offset: int = 0
    ) -> List[Dict]:
        query = 'SELECT * FROM books WHERE deleted_at IS NULL'
        params = []
        
        # Add sorting
        if sort_by == 'name':
            query += ' ORDER BY title ASC'
        else:
            query += ' ORDER BY created_at DESC'
        
        # Add pagination
        query += ' LIMIT ? OFFSET ?'
        params.extend([limit, offset])
        
        rows = await self.db.fetch_all(query, params)
        return [self._map_row_to_book(row) for row in rows]
    
    # Find one book by ID
    async def find_by_id(self, book_id: str) -> Optional[Dict]:
        query = 'SELECT * FROM books WHERE id = ? AND deleted_at IS NULL'
        rows = await self.db.fetch_all(query, [book_id])
        
        if not rows:
            return None
        
        return self._map_row_to_book(rows[0])
    
    # Find by ISBN
    async def find_by_isbn(self, isbn: str) -> Optional[Dict]:
        query = 'SELECT * FROM books WHERE isbn = ? AND deleted_at IS NULL'
        rows = await self.db.fetch_all(query, [isbn])
        
        if not rows:
            return None
        
        return self._map_row_to_book(rows[0])
    
    # Insert new book
    async def insert(self, book: Dict) -> Dict:
        query = '''
            INSERT INTO books (id, title, author, isbn, published_year, user_id, created_at, updated_at)
            VALUES (?, ?, ?, ?, ?, ?, ?, ?)
        '''
        
        params = [
            book["id"],
            book["title"],
            book["author"],
            book.get("isbn"),
            book.get("published_year"),
            book["user_id"],
            book["created_at"],
            book["updated_at"]
        ]
        
        await self.db.execute(query, params)
        return book
    
    # Update book
    async def update(self, book_id: str, updates: Dict) -> bool:
        query = '''
            UPDATE books 
            SET title = ?, author = ?, isbn = ?, published_year = ?, updated_at = ?
            WHERE id = ? AND deleted_at IS NULL
        '''
        
        params = [
            updates["title"],
            updates["author"],
            updates.get("isbn"),
            updates.get("published_year"),
            datetime.now(),
            book_id
        ]
        
        result = await self.db.execute(query, params)
        return result.rowcount > 0
    
    # Soft delete
    async def soft_delete(self, book_id: str) -> bool:
        query = 'UPDATE books SET deleted_at = ? WHERE id = ?'
        result = await self.db.execute(query, [datetime.now(), book_id])
        return result.rowcount > 0
    
    # Helper function to map database row to object
    def _map_row_to_book(self, row) -> Dict:
        return {
            "id": row["id"],
            "title": row["title"],
            "author": row["author"],
            "isbn": row["isbn"],
            "published_year": row["published_year"],
            "user_id": row["user_id"],
            "created_at": row["created_at"],
            "updated_at": row["updated_at"]
        }
```

**Example 2: User Repository**

```python
# user_repository.py
from typing import Optional, Dict

class UserRepository:
    def __init__(self, db):
        self.db = db
    
    async def find_by_id(self, user_id: str) -> Optional[Dict]:
        query = 'SELECT * FROM users WHERE id = ?'
        rows = await self.db.fetch_all(query, [user_id])
        return self._map_row_to_user(rows[0]) if rows else None
    
    async def increment_book_count(self, user_id: str, transaction=None):
        db = transaction or self.db
        query = 'UPDATE users SET book_count = book_count + 1 WHERE id = ?'
        await db.execute(query, [user_id])
    
    async def decrement_book_count(self, user_id: str, transaction=None):
        db = transaction or self.db
        query = 'UPDATE users SET book_count = book_count - 1 WHERE id = ?'
        await db.execute(query, [user_id])
    
    def _map_row_to_user(self, row) -> Dict:
        return {
            "id": row["id"],
            "email": row["email"],
            "name": row["name"],
            "book_count": row["book_count"],
            "created_at": row["created_at"]
        }
```

**Example 3: Advanced Queries**

```python
from typing import List, Optional, Dict

class BookRepository:
    def __init__(self, db):
        self.db = db
    
    # Find books with complex filtering
    async def find_by_filters(
        self,
        genre: Optional[str] = None,
        min_year: Optional[int] = None,
        max_year: Optional[int] = None,
        author_name: Optional[str] = None,
        limit: int = 10,
        offset: int = 0
    ) -> List[Dict]:
        query = 'SELECT * FROM books WHERE deleted_at IS NULL'
        params = []
        
        if genre:
            query += ' AND genre = ?'
            params.append(genre)
        
        if min_year:
            query += ' AND published_year >= ?'
            params.append(min_year)
        
        if max_year:
            query += ' AND published_year <= ?'
            params.append(max_year)
        
        if author_name:
            query += ' AND author LIKE ?'
            params.append(f'%{author_name}%')
        
        query += ' ORDER BY created_at DESC LIMIT ? OFFSET ?'
        params.extend([limit, offset])
        
        rows = await self.db.fetch_all(query, params)
        return [self._map_row_to_book(row) for row in rows]
    
    # Join query
    async def find_books_with_user_details(self, book_id: str) -> Optional[Dict]:
        query = '''
            SELECT 
                b.*,
                u.name as user_name,
                u.email as user_email
            FROM books b
            JOIN users u ON b.user_id = u.id
            WHERE b.id = ? AND b.deleted_at IS NULL
        '''
        
        rows = await self.db.fetch_all(query, [book_id])
        
        if not rows:
            return None
        
        row = rows[0]
        return {
            "book": self._map_row_to_book(row),
            "user": {
                "name": row["user_name"],
                "email": row["user_email"]
            }
        }
    
    def _map_row_to_book(self, row) -> Dict:
        return {
            "id": row["id"],
            "title": row["title"],
            "author": row["author"],
            "isbn": row["isbn"],
            "user_id": row["user_id"],
            "created_at": row["created_at"]
        }
```

#### Repository Best Practices

**Single Purpose per Method:**
```python
# GOOD - Separate methods for different queries
class BookRepository:
    async def find_by_id(self, book_id): ...
    async def find_by_isbn(self, isbn): ...
    async def find_all(self, filters): ...

# BAD - One method trying to do everything
class BookRepository:
    async def find(self, id=None, isbn=None, filters=None): ...  # Confusing and hard to maintain
```

**No Business Logic:**
```python
# WRONG - Business logic in repository
async def insert(self, book: dict):
    # ❌ This check belongs in service layer
    if book.get("published_year", 0) > datetime.now().year:
        raise ValueError('Future year not allowed')
    
    await self.db.execute('INSERT INTO books ...', book)

# CORRECT - Repository just does database operations
async def insert(self, book: dict):
    await self.db.execute('INSERT INTO books ...', book)
```

**Always Return Consistent Types:**
```python
# GOOD - Always return same type
async def find_by_id(self, book_id: str) -> Optional[Dict]:
    rows = await self.db.fetch_all('SELECT * FROM books WHERE id = ?', [book_id])
    return self._map_row_to_book(rows[0]) if rows else None  # Always Dict | None

# BAD - Inconsistent return types
async def find_by_id(self, book_id: str):
    rows = await self.db.fetch_all('SELECT * FROM books WHERE id = ?', [book_id])
    if not rows:
        raise ValueError('Not found')  # Sometimes raises
    return self._map_row_to_book(rows[0])  # Sometimes returns book
```

---

### How They Work Together

Let's see a complete flow from HTTP request to database and back:

```
CLIENT REQUEST
     │
     ├─ POST /api/books
     │  Body: { "title": "Clean Code", "author": "Robert Martin", "isbn": "978-0132350884" }
     │
     ▼
┌─────────────────────────────────────────────────────────────┐
│                      HANDLER LAYER                          │
│  • Extract data from req.body                               │
│  • Validate basic structure                                 │
│  • Get userId from req.context (set by auth middleware)     │
│  • Call service layer                                       │
│  • Send HTTP response based on result                       │
└─────────────────────────────────────────────────────────────┘
     │
     │ bookService.createBook({ title, author, isbn, userId })
     ▼
┌─────────────────────────────────────────────────────────────┐
│                      SERVICE LAYER                          │
│  • Apply business rules (e.g., validate ISBN format)        │
│  • Check for duplicates via repository                      │
│  • Create book object with generated ID                     │
│  • Call repository to insert book                           │
│  • Update user's book count via repository                  │
│  • Send email notification                                  │
│  • Return book object                                       │
└─────────────────────────────────────────────────────────────┘
     │                              │
     │ findByISBN(isbn)             │ insert(book)
     ▼                              ▼
┌───────────────────────────────────────────────────────────────┐
│                    REPOSITORY LAYER                           │
│  • Construct SQL queries                                      │
│  • Execute database operations                                │
│  • Map database rows to objects                               │
│  • Return results                                             │
└───────────────────────────────────────────────────────────────┘
     │
     │ Database operations complete
     ▼
   DATABASE
```

**Complete Code Example:**

```python
# ============================================
# HANDLER (book_handler.py)
# ============================================
from fastapi import APIRouter, HTTPException, Request, status
from pydantic import BaseModel

router = APIRouter()

class CreateBookRequest(BaseModel):
    title: str
    author: str
    isbn: str | None = None

@router.post("/api/books", status_code=status.HTTP_201_CREATED)
async def create_book_handler(book_data: CreateBookRequest, request: Request):
    try:
        # 1. Extract and validate (done by Pydantic)
        user_id = request.state.user_id  # From auth middleware
        
        # 2. Call service
        book = await book_service.create_book(
            title=book_data.title,
            author=book_data.author,
            isbn=book_data.isbn,
            user_id=user_id
        )
        
        # 3. Send response
        return book
        
    except ValueError as error:
        # 4. Handle errors
        if "DUPLICATE_ISBN" in str(error):
            raise HTTPException(
                status_code=status.HTTP_409_CONFLICT,
                detail=str(error)
            )
        
        print(f"Handler error: {error}")
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail="Internal server error"
        )

# ============================================
# SERVICE (book_service.py)
# ============================================
from datetime import datetime
import uuid
from typing import Optional

class BusinessError(Exception):
    def __init__(self, message: str, code: str):
        self.message = message
        self.code = code
        super().__init__(self.message)

def is_valid_isbn(isbn: str) -> bool:
    # ISBN validation logic here
    return len(isbn) in [10, 13]

class BookService:
    def __init__(self, book_repository, user_repository, email_service):
        self.book_repository = book_repository
        self.user_repository = user_repository
        self.email_service = email_service
    
    async def create_book(
        self,
        title: str,
        author: str,
        isbn: Optional[str],
        user_id: str
    ) -> dict:
        # 1. Business validation
        if isbn and not is_valid_isbn(isbn):
            raise BusinessError('Invalid ISBN format', 'INVALID_ISBN')
        
        # 2. Check for duplicates
        if isbn:
            existing = await self.book_repository.find_by_isbn(isbn)
            if existing:
                raise BusinessError(
                    'Book with this ISBN already exists',
                    'DUPLICATE_ISBN'
                )
        
        # 3. Create book object
        book = {
            "id": str(uuid.uuid4()),
            "title": title.strip(),
            "author": author.strip(),
            "isbn": isbn,
            "user_id": user_id,
            "created_at": datetime.now(),
            "updated_at": datetime.now()
        }
        
        # 4. Persist to database
        await self.book_repository.insert(book)
        
        # 5. Update user's book count
        await self.user_repository.increment_book_count(user_id)
        
        # 6. Send notification (fire and forget - don't wait)
        try:
            await self.email_service.send_book_created_email(user_id, book)
        except Exception as err:
            print(f"Failed to send email: {err}")
        
        # 7. Return result
        return book

# ============================================
# REPOSITORY (book_repository.py)
# ============================================
from typing import Optional, Dict

class BookRepository:
    def __init__(self, db):
        self.db = db
    
    async def find_by_isbn(self, isbn: str) -> Optional[Dict]:
        query = 'SELECT * FROM books WHERE isbn = ? AND deleted_at IS NULL'
        rows = await self.db.fetch_all(query, [isbn])
        return self._map_row_to_book(rows[0]) if rows else None
    
    async def insert(self, book: Dict) -> Dict:
        query = '''
            INSERT INTO books (id, title, author, isbn, user_id, created_at, updated_at)
            VALUES (?, ?, ?, ?, ?, ?, ?)
        '''
        
        await self.db.execute(query, [
            book["id"],
            book["title"],
            book["author"],
            book["isbn"],
            book["user_id"],
            book["created_at"],
            book["updated_at"]
        ])
        
        return book
    
    def _map_row_to_book(self, row) -> Dict:
        return {
            "id": row["id"],
            "title": row["title"],
            "author": row["author"],
            "isbn": row["isbn"],
            "user_id": row["user_id"],
            "created_at": row["created_at"],
            "updated_at": row["updated_at"]
        }

# ============================================
# USER REPOSITORY (user_repository.py)
# ============================================
class UserRepository:
    def __init__(self, db):
        self.db = db
    
    async def increment_book_count(self, user_id: str):
        query = 'UPDATE users SET book_count = book_count + 1 WHERE id = ?'
        await self.db.execute(query, [user_id])
```

---

## Middleware

### What is Middleware?

Middleware are functions that execute **in the middle** of the request lifecycle - after the request arrives but before (or after) it reaches your handler.

**Visual Representation:**

```
Request → [MW1] → [MW2] → [Routing] → [MW3] → [Handler] → [MW4] → Response
          CORS    Logger              Auth    Controller  Error
```

### Why Middleware Exists

#### Problem Without Middleware

```python
# Every handler has duplicated code
from fastapi import FastAPI, HTTPException, Header
import logging

app = FastAPI()

@app.get('/api/books')
async def get_books(authorization: str = Header(None)):
    # Check authentication - DUPLICATED
    if not authorization:
        raise HTTPException(status_code=401, detail="Unauthorized")
    
    # Log request - DUPLICATED
    logging.info("GET /api/books")
    
    # CORS headers would be set via response - DUPLICATED
    # (In FastAPI, CORS is typically handled by middleware)
    
    # Actual logic
    books = await get_books_data()
    return books

@app.post('/api/books')
async def create_book(authorization: str = Header(None), book_data: dict = None):
    # Same authentication - DUPLICATED
    if not authorization:
        raise HTTPException(status_code=401, detail="Unauthorized")
    
    # Same logging - DUPLICATED
    logging.info("POST /api/books")
    
    # Actual logic
    book = await create_book_data(book_data)
    return book

# ... repeated for 100s of endpoints
```

#### Solution With Middleware

```python
# Define once, use everywhere
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

# Add middleware
app.add_middleware(CORSMiddleware, allow_origins=["https://example.com"])
app.middleware("http")(logging_middleware)
app.middleware("http")(auth_middleware)

# Handlers focus only on their specific logic
@app.get('/api/books')
async def get_books():
    books = await get_books_data()
    return books

@app.post('/api/books')
async def create_book(book_data: dict):
    book = await create_book_data(book_data)
    return book
```

### How Middleware Works

Every middleware in FastAPI is a function that receives the request and a call_next function:

```python
from fastapi import Request
from starlette.middleware.base import BaseHTTPMiddleware

class CustomMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        # request: Request object (can read and modify)
        # call_next: Function to pass control to next middleware/handler
        
        # Do something...
        
        response = await call_next(request)  # Pass to next middleware
        
        # Can also modify response here
        return response
```

#### The `call_next()` Function

The `call_next()` function is crucial - it controls the flow:

```python
class Middleware1(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        print('MW1: Before call_next()')
        response = await call_next(request)  # Pass control to next middleware
        print('MW1: After call_next()')  # Executes after downstream completes
        return response

class Middleware2(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        print('MW2: Before call_next()')
        response = await call_next(request)
        print('MW2: After call_next()')
        return response

@app.get("/test")
async def handler():
    print('Handler executing')
    return {"message": "Done"}

# Order of execution:
# MW1: Before call_next()
# MW2: Before call_next()
# Handler executing
# MW2: After call_next()
# MW1: After call_next()
```

**Flow Diagram:**

```
┌─────────────┐
│   Request   │
└──────┬──────┘
       │
       ▼
┌─────────────────┐
│  Middleware 1   │
│  (before next)  │
└──────┬──────────┘
       │ next()
       ▼
┌─────────────────┐
│  Middleware 2   │
│  (before next)  │
└──────┬──────────┘
       │ next()
       ▼
┌─────────────────┐
│    Handler      │
└──────┬──────────┘
       │
       ▼
┌─────────────────┐
│  Middleware 2   │
│  (after next)   │
└──────┬──────────┘
       │
       ▼
┌─────────────────┐
│  Middleware 1   │
│  (after next)   │
└──────┬──────────┘
       │
       ▼
┌─────────────────┐
│    Response     │
└─────────────────┘
```

#### Middleware Can Terminate the Request

```python
from fastapi import Request, HTTPException
from fastapi.responses import JSONResponse
from starlette.middleware.base import BaseHTTPMiddleware

class AuthMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        token = request.headers.get('authorization')
        
        if not token:
            # Don't call call_next() - stop here and return response
            return JSONResponse(
                status_code=401,
                content={"error": "No token provided"}
            )
        
        # Token exists, continue to next middleware
        response = await call_next(request)
        return response
```

### Common Middleware Types

#### 1. CORS Middleware

**What it does:** Adds headers to allow cross-origin requests

**Why:** Browsers block requests from different origins by default (security)

**When:** First middleware (before routing)

```python
from fastapi import FastAPI, Request
from fastapi.middleware.cors import CORSMiddleware
from fastapi.responses import Response

app = FastAPI()

# Using built-in CORS middleware (recommended)
app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://example.com", "https://app.example.com"],
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "DELETE", "OPTIONS"],
    allow_headers=["Content-Type", "Authorization"],
)

# Or custom CORS middleware
from starlette.middleware.base import BaseHTTPMiddleware

class CustomCORSMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        allowed_origins = ['https://example.com', 'https://app.example.com']
        origin = request.headers.get('origin')
        
        # Handle preflight requests
        if request.method == 'OPTIONS':
            response = Response(status_code=204)
            if origin in allowed_origins:
                response.headers['Access-Control-Allow-Origin'] = origin
                response.headers['Access-Control-Allow-Methods'] = 'GET, POST, PUT, DELETE, OPTIONS'
                response.headers['Access-Control-Allow-Headers'] = 'Content-Type, Authorization'
                response.headers['Access-Control-Allow-Credentials'] = 'true'
            return response
        
        response = await call_next(request)
        
        if origin in allowed_origins:
            response.headers['Access-Control-Allow-Origin'] = origin
            response.headers['Access-Control-Allow-Credentials'] = 'true'
        
        return response
```

#### 2. Logging Middleware

**What it does:** Logs request details for debugging and monitoring

**Why:** Track all incoming requests, debug issues, audit trail

**When:** Early in the chain (after CORS)

```python
import logging
import time
from fastapi import Request
from starlette.middleware.base import BaseHTTPMiddleware
from datetime import datetime

logger = logging.getLogger(__name__)

class LoggingMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        start_time = time.time()
        
        # Log request
        logger.info(f"[{datetime.now().isoformat()}] {request.method} {request.url.path}")
        logger.info(f"Query params: {dict(request.query_params)}")
        
        # Process request
        response = await call_next(request)
        
        # Log response time
        duration = (time.time() - start_time) * 1000  # Convert to milliseconds
        logger.info(
            f"[{datetime.now().isoformat()}] {request.method} {request.url.path} - "
            f"{response.status_code} - {duration:.2f}ms"
        )
        
        return response

# Usage
app.add_middleware(LoggingMiddleware)
```

**Advanced Logging Example:**

```python
import logging
import time
from fastapi import Request
from starlette.middleware.base import BaseHTTPMiddleware

logger = logging.getLogger(__name__)

class AdvancedLoggingMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        request_id = getattr(request.state, 'request_id', 'unknown')
        start_time = time.time()
        
        log_data = {
            "request_id": request_id,
            "method": request.method,
            "path": request.url.path,
            "ip": request.client.host if request.client else None,
            "user_agent": request.headers.get("user-agent"),
            "user_id": getattr(request.state, 'user_id', 'anonymous')
        }
        
        logger.info(f"Incoming request: {log_data}")
        
        response = await call_next(request)
        
        duration = (time.time() - start_time) * 1000
        logger.info(f"Request completed: {log_data} - Status: {response.status_code} - Duration: {duration:.2f}ms")
        
        return response
```

#### 3. Authentication Middleware

**What it does:** Verifies user credentials/tokens

**Why:** Protect endpoints from unauthorized access

**When:** After logging, before handlers

```python
from fastapi import Request, HTTPException, status
from starlette.middleware.base import BaseHTTPMiddleware
from jose import JWTError, jwt
import os

class AuthMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        # 1. Extract token
        auth_header = request.headers.get('authorization')
        
        if not auth_header or not auth_header.startswith('Bearer '):
            return JSONResponse(
                status_code=status.HTTP_401_UNAUTHORIZED,
                content={"error": "No token provided"}
            )
        
        token = auth_header[7:]  # Remove 'Bearer '
        
        try:
            # 2. Verify token
            payload = jwt.decode(
                token,
                os.getenv('JWT_SECRET'),
                algorithms=["HS256"]
            )
            
            # 3. Add user info to request state
            request.state.user_id = payload.get('user_id')
            request.state.user_role = payload.get('role')
            request.state.permissions = payload.get('permissions', [])
            
            # 4. Continue to next middleware
            response = await call_next(request)
            return response
            
        except JWTError:
            # Token invalid or expired
            return JSONResponse(
                status_code=status.HTTP_401_UNAUTHORIZED,
                content={"error": "Invalid token"}
            )

# Usage - apply to specific routes
# In FastAPI, middleware applies globally, so use dependencies for route-specific auth
from fastapi import Depends, Header

async def verify_token(authorization: str = Header(None)):
    if not authorization or not authorization.startswith('Bearer '):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="No token provided"
        )
    
    token = authorization[7:]
    try:
        payload = jwt.decode(token, os.getenv('JWT_SECRET'), algorithms=["HS256"])
        return payload
    except JWTError:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid token"
        )

# Use as dependency
@app.post("/api/books")
async def create_book(book_data: dict, user_data: dict = Depends(verify_token)):
    # user_data contains decoded token payload
    return {"message": "Book created"}
```

#### 4. Rate Limiting Middleware

**What it does:** Limits number of requests from a client

**Why:** Prevent abuse, DDoS attacks, ensure fair usage

**When:** Early in the chain

```python
# Using slowapi library (recommended)
from slowapi import Limiter, _rate_limit_exceeded_handler
from slowapi.util import get_remote_address
from slowapi.errors import RateLimitExceeded
from fastapi import FastAPI, Request

app = FastAPI()
limiter = Limiter(key_func=get_remote_address)
app.state.limiter = limiter
app.add_exception_handler(RateLimitExceeded, _rate_limit_exceeded_handler)

@app.get("/api/books")
@limiter.limit("100/15minutes")  # 100 requests per 15 minutes
async def get_books(request: Request):
    return {"books": []}
```

**Custom Rate Limiting:**

```python
import time
from collections import defaultdict
from fastapi import Request
from fastapi.responses import JSONResponse
from starlette.middleware.base import BaseHTTPMiddleware

# In production, use Redis instead of in-memory dict
request_counts = defaultdict(lambda: {"count": 0, "reset_time": 0})

class CustomRateLimitMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        client_ip = request.client.host
        now = time.time()
        window_ms = 60  # 1 minute
        max_requests = 30
        
        # Get or create client record
        client_data = request_counts[client_ip]
        
        # Reset if window expired
        if now > client_data["reset_time"]:
            client_data = {"count": 0, "reset_time": now + window_ms}
            request_counts[client_ip] = client_data
        
        # Increment count
        client_data["count"] += 1
        
        # Check limit
        if client_data["count"] > max_requests:
            return JSONResponse(
                status_code=429,
                content={
                    "error": "Too many requests",
                    "retry_after": int(client_data["reset_time"] - now)
                },
                headers={
                    "X-RateLimit-Limit": str(max_requests),
                    "X-RateLimit-Remaining": "0",
                    "X-RateLimit-Reset": str(int(client_data["reset_time"]))
                }
            )
        
        # Add rate limit headers to response
        response = await call_next(request)
        response.headers["X-RateLimit-Limit"] = str(max_requests)
        response.headers["X-RateLimit-Remaining"] = str(max_requests - client_data["count"])
        response.headers["X-RateLimit-Reset"] = str(int(client_data["reset_time"]))
        
        return response
```

#### 5. Request ID Middleware

**What it does:** Assigns unique ID to each request

**Why:** Track requests across services, debugging, logging correlation

**When:** Very early (first or second)

```python
import uuid
from fastapi import Request
from starlette.middleware.base import BaseHTTPMiddleware

class RequestIDMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        # Generate or use existing request ID
        request_id = request.headers.get('x-request-id') or str(uuid.uuid4())
        
        # Store in state
        request.state.request_id = request_id
        
        # Process request
        response = await call_next(request)
        
        # Add to response headers
        response.headers['X-Request-ID'] = request_id
        
        return response

# Usage
app.add_middleware(RequestIDMiddleware)

# Now all logs and errors can include this ID
class LoggingMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        request_id = getattr(request.state, 'request_id', 'unknown')
        logger.info(f"[{request_id}] {request.method} {request.url.path}")
        response = await call_next(request)
        return response
```

#### 6. Security Headers Middleware

**What it does:** Adds security-related HTTP headers

**Why:** Protect against common web vulnerabilities

**When:** Early in the chain

```python
from fastapi import Request
from starlette.middleware.base import BaseHTTPMiddleware

class SecurityHeadersMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        response = await call_next(request)
        
        # Prevent clickjacking
        response.headers['X-Frame-Options'] = 'DENY'
        
        # Prevent MIME type sniffing
        response.headers['X-Content-Type-Options'] = 'nosniff'
        
        # Enable XSS protection
        response.headers['X-XSS-Protection'] = '1; mode=block'
        
        # Content Security Policy
        response.headers['Content-Security-Policy'] = "default-src 'self'"
        
        # Strict Transport Security (HTTPS only)
        response.headers['Strict-Transport-Security'] = 'max-age=31536000; includeSubDomains'
        
        # Referrer Policy
        response.headers['Referrer-Policy'] = 'strict-origin-when-cross-origin'
        
        return response

# Usage
app.add_middleware(SecurityHeadersMiddleware)
```

#### 7. Body Parser Middleware

**What it does:** Parses request body (JSON, form data, etc.)

**Why:** Convert raw request data to JavaScript objects

**When:** Early, before handlers need to access req.body

```javascript
// Built-in Express middleware
app.use(express.json()); // Parse JSON bodies
app.use(express.urlencoded({ extended: true })); // Parse form data

// Custom JSON parser with size limit
function jsonParserMiddleware(req, res, next) {
  if (req.headers['content-type'] !== 'application/json') {
    return next();
  }
  
  let body = '';
  const maxSize = 1024 * 1024; // 1MB
  let size = 0;
  
  req.on('data', chunk => {
    size += chunk.length;
    
    if (size > maxSize) {
      return res.status(413).json({ error: 'Request body too large' });
    }
    
    body += chunk.toString();
  });
  
  req.on('end', () => {
    try {
      req.body = JSON.parse(body);
      next();
    } catch (error) {
      res.status(400).json({ error: 'Invalid JSON' });
    }
  });
}
```

#### 8. Compression Middleware

**What it does:** Compresses response data

**Why:** Reduce bandwidth, faster response times

**When:** After handlers, before sending response

```javascript
const compression = require('compression');

app.use(compression({
  level: 6, // Compression level (0-9)
  threshold: 1024, // Only compress responses larger than 1KB
  filter: (req, res) => {
    // Don't compress if client doesn't support it
    if (req.headers['x-no-compression']) {
      return false;
    }
    return compression.filter(req, res);
  }
}));
```

#### 9. Error Handling Middleware

**What it does:** Catches all errors and sends formatted responses

**Why:** Centralized error handling, consistent error format

**When:** Applied via exception handlers in FastAPI

```python
from fastapi import FastAPI, Request, status
from fastapi.responses import JSONResponse
from fastapi.exceptions import RequestValidationError
import logging

app = FastAPI()
logger = logging.getLogger(__name__)

# Custom error classes
class BusinessError(Exception):
    def __init__(self, message: str, code: str, status_code: int = 400):
        self.message = message
        self.code = code
        self.status_code = status_code
        super().__init__(self.message)

class UnauthorizedError(Exception):
    pass

class ForbiddenError(Exception):
    pass

class NotFoundError(Exception):
    pass

# Exception handlers
@app.exception_handler(BusinessError)
async def business_error_handler(request: Request, exc: BusinessError):
    request_id = getattr(request.state, 'request_id', 'unknown')
    logger.warning(f"[{request_id}] Business error: {exc.message}")
    
    return JSONResponse(
        status_code=exc.status_code,
        content={
            "error": exc.message,
            "code": exc.code,
            "request_id": request_id
        }
    )

@app.exception_handler(UnauthorizedError)
async def unauthorized_error_handler(request: Request, exc: UnauthorizedError):
    request_id = getattr(request.state, 'request_id', 'unknown')
    return JSONResponse(
        status_code=status.HTTP_401_UNAUTHORIZED,
        content={"error": "Unauthorized", "request_id": request_id}
    )

@app.exception_handler(ForbiddenError)
async def forbidden_error_handler(request: Request, exc: ForbiddenError):
    request_id = getattr(request.state, 'request_id', 'unknown')
    return JSONResponse(
        status_code=status.HTTP_403_FORBIDDEN,
        content={"error": "Forbidden", "request_id": request_id}
    )

@app.exception_handler(NotFoundError)
async def not_found_error_handler(request: Request, exc: NotFoundError):
    request_id = getattr(request.state, 'request_id', 'unknown')
    return JSONResponse(
        status_code=status.HTTP_404_NOT_FOUND,
        content={"error": "Resource not found", "request_id": request_id}
    )

@app.exception_handler(RequestValidationError)
async def validation_error_handler(request: Request, exc: RequestValidationError):
    request_id = getattr(request.state, 'request_id', 'unknown')
    return JSONResponse(
        status_code=status.HTTP_422_UNPROCESSABLE_ENTITY,
        content={
            "error": "Validation failed",
            "details": exc.errors(),
            "request_id": request_id
        }
    )

@app.exception_handler(Exception)
async def general_exception_handler(request: Request, exc: Exception):
    request_id = getattr(request.state, 'request_id', 'unknown')
    
    # Log server errors
    logger.error(f"[{request_id}] Internal error: {str(exc)}", exc_info=True)
    
    # Don't expose internal details in production
    return JSONResponse(
        status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
        content={
            "error": "Internal server error",
            "request_id": request_id
        }
    )
```

**Advanced Error Handler:**

```javascript
class AppError extends Error {
  constructor(message, statusCode, code) {
    super(message);
    this.statusCode = statusCode;
    this.code = code;
    this.isOperational = true;
  }
}

function errorHandlerMiddleware(err, req, res, next) {
  const requestId = req.context?.requestId;
  
  // Log based on severity
  if (err.statusCode >= 500) {
    logger.error('Server error', {
      requestId,
      error: err.message,
      stack: err.stack,
      path: req.path
    });
  } else {
    logger.warn('Client error', {
      requestId,
      error: err.message,
      path: req.path
    });
  }
  
  // Determine status code
  const statusCode = err.statusCode || 500;
  
  // Prepare response
  const response = {
    error: err.message || 'Something went wrong',
    requestId
  };
  
  // Add error code if available
  if (err.code) {
    response.code = err.code;
  }
  
  // Add validation details if available
  if (err.validationErrors) {
    response.validationErrors = err.validationErrors;
  }
  
  // In development, include stack trace
  if (process.env.NODE_ENV === 'development') {
    response.stack = err.stack;
  }
  
  res.status(statusCode).json(response);
}
```

#### 10. Data Validation Middleware

**What it does:** Validates request data before reaching handler

**Why:** Centralized validation, reduce handler code

**When:** After parsing, before handler

```python
# In FastAPI, validation is typically done via Pydantic models
from pydantic import BaseModel, Field, validator
from typing import Optional
from datetime import datetime

class CreateBookSchema(BaseModel):
    title: str = Field(..., min_length=3, max_length=200)
    author: str = Field(..., min_length=2, max_length=100)
    isbn: Optional[str] = Field(None, regex=r'^978-\d{10}$')
    published_year: Optional[int] = Field(None, ge=1000)
    
    @validator('published_year')
    def validate_published_year(cls, v):
        if v and v > datetime.now().year:
            raise ValueError(f'Year cannot be in the future')
        return v
    
    @validator('title', 'author')
    def strip_strings(cls, v):
        return v.strip() if v else v

# Use in route - FastAPI validates automatically
@app.post("/api/books")
async def create_book(book: CreateBookSchema):
    # book is already validated and transformed
    return {"message": "Book created", "data": book.dict()}

# For custom validation with better error messages
from fastapi import HTTPException

class CreateBookRequest(BaseModel):
    title: str
    author: str
    isbn: Optional[str] = None
    published_year: Optional[int] = None
    
    class Config:
        # Automatically strip whitespace
        anystr_strip_whitespace = True
    
    @validator('title')
    def validate_title(cls, v):
        if len(v) < 3 or len(v) > 200:
            raise ValueError('Title must be between 3 and 200 characters')
        return v
    
    @validator('author')
    def validate_author(cls, v):
        if len(v) < 2 or len(v) > 100:
            raise ValueError('Author must be between 2 and 100 characters')
        return v
```

### Middleware Ordering

**The order matters!** Middleware execute in the order they are defined.

#### ❌ Wrong Order

```python
# WRONG - Validators cannot access parsed body if it's added after
@app.post('/api/books')
async def create_book(book: CreateBookRequest):
    pass

# Too late to add JSON parsing - FastAPI does this automatically

# WRONG - authentication middleware added after route
# In FastAPI, use dependencies for route-specific auth
```

#### ✅ Correct Order

```python
from fastapi import FastAPI, Depends
from fastapi.middleware.cors import CORSMiddleware
from starlette.middleware.gzip import GZipMiddleware

app = FastAPI()

# 1. Request ID (first - needed for logging)
app.add_middleware(RequestIDMiddleware)

# 2. CORS (early - might terminate request)
app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://example.com"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# 3. Security headers
app.add_middleware(SecurityHeadersMiddleware)

# 4. Logging (after request ID and CORS)
app.add_middleware(LoggingMiddleware)

# 5. Rate limiting
app.add_middleware(CustomRateLimitMiddleware)

# 6. Compression
app.add_middleware(GZipMiddleware, minimum_size=1000)

# 7. Routes with dependencies for auth
@app.get("/api/public/books")
async def get_public_books():
    return {"books": []}

# Use dependency for auth (applied per route)
@app.get("/api/protected/books")
async def get_protected_books(user: dict = Depends(verify_token)):
    return {"books": [], "user_id": user['user_id']}

# 8. Exception handlers (registered, not middleware)
@app.exception_handler(Exception)
async def general_exception_handler(request, exc):
    return JSONResponse(
        status_code=500,
        content={"error": "Internal server error"}
    )
```

**Visual Flow:**

```
Request
  │
  ├─ [Request ID] ────────────── Generate/extract unique ID
  │
  ├─ [CORS] ──────────────────── Check origin, add headers (may terminate)
  │
  ├─ [Security Headers] ──────── Add security headers
  │
  ├─ [Logging] ───────────────── Log request details
  │
  ├─ [Body Parser] ───────────── Parse JSON/form data
  │
  ├─ [Rate Limit] ────────────── Check request count (may terminate)
  │
  ├─ [Compression] ───────────── Set up compression
  │
  ├─ [Routing] ───────────────── Match URL to handler
  │
  ├─ [Auth] (if protected) ───── Verify credentials (may terminate)
  │
  ├─ [Validation] ────────────── Validate request data (may terminate)
  │
  ├─ [Handler] ───────────────── Execute business logic
  │
  ├─ [Error Handler] ─────────── Catch any errors from above
  │
Response
```

### Global vs Route-Specific Middleware

```javascript
// GLOBAL - applies to all routes
app.use(loggingMiddleware);
app.use(corsMiddleware);

// ROUTE-SPECIFIC - applies only to specific routes
app.use('/api/admin/*', adminAuthMiddleware);
app.post('/api/books', authMiddleware, createBookHandler);

// Multiple middleware for one route
app.post('/api/books',
  authMiddleware,
  validate(createBookSchema),
  checkPermissionsMiddleware,
  createBookHandler
);
```

---

## Request Context

### What is Request Context?

Request context is a **storage mechanism** that allows you to attach data to a specific HTTP request and make that data accessible throughout the entire request lifecycle - across all middleware and handlers.

**Think of it as:** A backpack that travels with each request, where middleware and handlers can put items in or take items out.

### Why We Need It

#### Problem Without Context

```python
# Authentication middleware extracts user ID
from fastapi import Request
from starlette.middleware.base import BaseHTTPMiddleware

class AuthMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        user_id = verify_token(request.headers.get('authorization'))
        # How do we pass this to the handler?
        response = await call_next(request)
        return response

# Handler needs user ID - but how to get it?
@app.post("/api/books")
async def create_book_handler(book_data: dict):
    user_id = ???  # Where does this come from?
    # ...
```

**Bad Solutions:**

```python
# ❌ Global variable - breaks with concurrent requests
current_user_id = None

class AuthMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        global current_user_id
        current_user_id = verify_token(request.headers.get('authorization'))
        response = await call_next(request)
        return response

# ❌ Modifying request directly without state - not recommended
class AuthMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        request.user_id = verify_token(request.headers.get('authorization'))
        response = await call_next(request)
        return response
```

#### ✅ Solution: Request Context

```python
# Authentication middleware stores user in state
from fastapi import Request
from starlette.middleware.base import BaseHTTPMiddleware

class AuthMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        user_id = verify_token(request.headers.get('authorization'))
        
        # Store in request state (FastAPI's context mechanism)
        request.state.user_id = user_id
        request.state.user_role = 'user'
        
        response = await call_next(request)
        return response

# Handler retrieves user from state
@app.post("/api/books")
async def create_book_handler(book_data: dict, request: Request):
    user_id = request.state.user_id  # Available!
    user_role = request.state.user_role  # Available!
    # ...
```

### How It Works

Request context is typically implemented using FastAPI's **request.state** object.

#### Basic Implementation (FastAPI)

```python
from fastapi import Request
from starlette.middleware.base import BaseHTTPMiddleware
import time
import uuid

# Context initialization middleware
class InitContextMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        request.state.request_id = str(uuid.uuid4())
        request.state.start_time = time.time()
        
        response = await call_next(request)
        return response

# Use context in other middleware
class AuthMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        token = request.headers.get('authorization')
        decoded = verify_jwt(token, SECRET)
        
        # Add to state
        request.state.user_id = decoded['user_id']
        request.state.user_role = decoded['role']
        request.state.permissions = decoded.get('permissions', [])
        
        response = await call_next(request)
        return response

# Use context in handlers
@app.post("/api/books")
async def create_book_handler(book_data: dict, request: Request):
    user_id = request.state.user_id
    user_role = request.state.user_role
    # Use the data
    pass
```

#### Implementation in Different Languages

**Python (FastAPI):**
```python
# FastAPI uses request.state for request-scoped storage
from fastapi import Request

@app.middleware("http")
async def add_context(request: Request, call_next):
    request.state.request_id = 'abc-123'
    request.state.user_id = 'user-456'
    response = await call_next(request)
    return response

# Access in handler
@app.get("/books")
async def get_books(request: Request):
    user_id = request.state.user_id
    return {"user_id": user_id}
```

**Go (using context package):**
```go
import "context"

// Create context with value
ctx := context.WithValue(r.Context(), "userId", userID)

// Retrieve value
userId := ctx.Value("userId").(string)
```

**Node.js (Express):**
```javascript
// Simple object attached to req
req.context = {
  requestId: 'abc-123',
  userId: 'user-456'
};
```

### Common Use Cases

#### 1. User Authentication Data

Store authenticated user information for use throughout the request.

```python
from fastapi import Request, HTTPException, status
from starlette.middleware.base import BaseHTTPMiddleware
from jose import jwt, JWTError
import os

# Auth middleware
class AuthMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        token = request.headers.get('authorization', '').replace('Bearer ', '')
        
        if not token:
            return JSONResponse(
                status_code=status.HTTP_401_UNAUTHORIZED,
                content={"error": "No token"}
            )
        
        try:
            decoded = jwt.decode(token, os.getenv('JWT_SECRET'), algorithms=["HS256"])
            
            # Store in state
            request.state.user_id = decoded.get('user_id')
            request.state.user_email = decoded.get('email')
            request.state.user_role = decoded.get('role')
            request.state.permissions = decoded.get('permissions', [])
            request.state.is_authenticated = True
            
            response = await call_next(request)
            return response
        except JWTError:
            return JSONResponse(
                status_code=status.HTTP_401_UNAUTHORIZED,
                content={"error": "Invalid token"}
            )

# Handler uses auth data
@app.post("/api/books")
async def create_book_handler(book_data: dict, request: Request):
    user_id = request.state.user_id
    user_role = request.state.user_role
    
    book = {
        **book_data,
        "user_id": user_id,  # From state, not from client (security!)
        "created_at": datetime.now()
    }
    
    await book_repository.insert(book)
    return book

# Another handler checks permissions
@app.delete("/api/books/{book_id}")
async def delete_book_handler(book_id: str, request: Request):
    user_id = request.state.user_id
    user_role = request.state.user_role
    permissions = request.state.permissions
    
    if user_role != 'admin' and 'delete:books' not in permissions:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Forbidden"
        )
    
    await book_repository.delete(book_id)
    return {"message": "Deleted"}
```

#### 2. Request Tracking

Store request ID for logging and distributed tracing.

```javascript
const { v4: uuidv4 } = require('uuid');

// Request ID middleware
function requestIdMiddleware(req, res, next) {
  const requestId = req.headers['x-request-id'] || uuidv4();
  
  req.context = req.context || {};
  req.context.requestId = requestId;
  
  // Also add to response headers
  res.setHeader('X-Request-ID', requestId);
  
  next();
}

// Logging middleware uses request ID
function loggingMiddleware(req, res, next) {
  const { requestId } = req.context;
  
  console.log(`[${requestId}] ${req.method} ${req.path}`);
  
  next();
}

// Handler logs with request ID
async function createBookHandler(req, res) {
  const { requestId, userId } = req.context;
  
  try {
    logger.info(`[${requestId}] Creating book for user ${userId}`);
    const book = await bookService.createBook(req.body);
    logger.info(`[${requestId}] Book created: ${book.id}`);
    res.status(201).json(book);
  } catch (error) {
    logger.error(`[${requestId}] Error creating book:`, error);
    throw error;
  }
}

// Microservices: Pass request ID to other services
async function callExternalService() {
  const { requestId } = req.context;
  
  const response = await fetch('https://api.other-service.com/data', {
    headers: {
      'X-Request-ID': requestId // Propagate for distributed tracing
    }
  });
  
  return response.json();
}
```

#### 3. Database Transaction Context

Store database transaction for use across multiple repository calls.

```javascript
async function transferBookHandler(req, res) {
  const { userId } = req.context;
  const { bookId, toUserId } = req.body;
  
  // Start transaction
  const transaction = await database.beginTransaction();
  
  try {
    // Store transaction in context
    req.context.transaction = transaction;
    
    // Service layer uses transaction from context
    await bookService.transferBook({ bookId, fromUserId: userId, toUserId, req });
    
    // Commit transaction
    await transaction.commit();
    res.status(200).json({ success: true });
    
  } catch (error) {
    // Rollback on error
    await transaction.rollback();
    throw error;
  }
}

// Repository uses transaction from context
async function updateBookOwner(bookId, newUserId, req) {
  const transaction = req.context.transaction;
  const db = transaction || database; // Use transaction if available
  
  await db.query('UPDATE books SET user_id = ? WHERE id = ?', [newUserId, bookId]);
}
```

#### 4. Request Metadata

Store timing, performance, and other metadata.

```javascript
function timingMiddleware(req, res, next) {
  req.context = req.context || {};
  req.context.startTime = Date.now();
  
  // Measure response time
  res.on('finish', () => {
    const duration = Date.now() - req.context.startTime;
    console.log(`Request took ${duration}ms`);
    
    // Log slow requests
    if (duration > 1000) {
      logger.warn(`Slow request: ${req.method} ${req.path} took ${duration}ms`);
    }
  });
  
  next();
}

// Store various metadata
function metadataMiddleware(req, res, next) {
  req.context = req.context || {};
  req.context.ip = req.ip;
  req.context.userAgent = req.headers['user-agent'];
  req.context.referer = req.headers['referer'];
  req.context.acceptLanguage = req.headers['accept-language'];
  
  next();
}
```

#### 5. Feature Flags

Store feature flags for A/B testing or gradual rollouts.

```javascript
async function featureFlagMiddleware(req, res, next) {
  const { userId } = req.context;
  
  // Fetch feature flags for user
  const flags = await featureFlagService.getFlags(userId);
  
  req.context.featureFlags = flags;
  next();
}

// Handler checks feature flags
async function getBooksHandler(req, res) {
  const { featureFlags } = req.context;
  
  let books = await bookRepository.findAll();
  
  // Apply new recommendation algorithm if enabled
  if (featureFlags.newRecommendationAlgorithm) {
    books = await recommendationService.rank(books);
  }
  
  res.json(books);
}
```

#### 6. Cancellation and Timeouts

Store cancellation signals to abort long-running operations.

```javascript
// Go-style context with cancellation
function timeoutMiddleware(timeoutMs) {
  return (req, res, next) => {
    const controller = new AbortController();
    
    req.context = req.context || {};
    req.context.signal = controller.signal;
    
    // Set timeout
    const timeout = setTimeout(() => {
      controller.abort();
    }, timeoutMs);
    
    // Clear timeout if request completes
    res.on('finish', () => clearTimeout(timeout));
    
    next();
  };
}

// Handler respects cancellation
async function expensiveOperationHandler(req, res) {
  const { signal } = req.context;
  
  try {
    const result = await fetch('https://slow-api.com/data', { signal });
    res.json(await result.json());
  } catch (error) {
    if (error.name === 'AbortError') {
      res.status(408).json({ error: 'Request timeout' });
    } else {
      throw error;
    }
  }
}
```

### Context Best Practices

#### ✅ Do:

```javascript
// 1. Initialize context early
function initContextMiddleware(req, res, next) {
  req.context = {};
  next();
}

// 2. Use consistent naming
req.context.userId; // ✅ Good
req.context.user_id; // ❌ Inconsistent

// 3. Store only necessary data
req.context.userId = user.id; // ✅ Good
req.context.user = entireUserObject; // ❌ Too much data

// 4. Use TypeScript for type safety
interface RequestContext {
  requestId: string;
  userId?: string;
  userRole?: string;
  startTime: number;
}

declare global {
  namespace Express {
    interface Request {
      context: RequestContext;
    }
  }
}
```

#### ❌ Don't:

```javascript
// Don't store mutable objects that could be modified
req.context.user = userObject; // ❌ Could be modified

// Don't store sensitive data unnecessarily
req.context.userPassword = password; // ❌ Security risk

// Don't store large objects
req.context.allBooks = await getAllBooks(); // ❌ Memory waste

// Don't rely on context in async operations without binding
setTimeout(() => {
  console.log(req.context.userId); // ❌ Might not be available
}, 1000);
```

---

## Complete Request Flow Example

Let's trace a complete request through all layers:

**Scenario:** User creates a new book

```
POST /api/books
Authorization: Bearer eyJhbGc...
Content-Type: application/json

{
  "title": "Clean Code",
  "author": "Robert Martin",
  "isbn": "978-0132350884"
}
```

### Step-by-Step Flow

```python
# ============================================
# 1. REQUEST ARRIVES
# ============================================

# 2. Request ID Middleware
class RequestIDMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        request.state.request_id = str(uuid.uuid4())
        response = await call_next(request)
        response.headers['X-Request-ID'] = request.state.request_id
        return response  # → Continue to next middleware

# 3. CORS Middleware (FastAPI built-in)
app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://example.com"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# 4. Logging Middleware
class LoggingMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        request_id = request.state.request_id
        logger.info(f"[{request_id}] POST /api/books")
        response = await call_next(request)
        return response  # → Continue

# 5. Body Parser (automatic in FastAPI via Pydantic)
# req.body is automatically parsed to Python dict

# 6. Rate Limit Middleware
class RateLimitMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        ip = request.client.host
        count = get_rate_limit_count(ip)
        
        if count > 100:
            return JSONResponse(
                status_code=429,
                content={"error": "Too many requests"}
            )
            # ❌ Request TERMINATED here
        
        increment_rate_limit_count(ip)
        response = await call_next(request)
        return response  # → Continue

# 7. ROUTING
# FastAPI routes POST /api/books to handler

# 8. Authentication Middleware (as dependency)
from fastapi import Depends

async def verify_token(request: Request, authorization: str = Header(None)):
    token = authorization.replace('Bearer ', '') if authorization else None
    
    if not token:
        raise HTTPException(status_code=401, detail="No token")
        # ❌ Would TERMINATE here if no token
    
    try:
        decoded = jwt.decode(token, SECRET, algorithms=["HS256"])
        request.state.user_id = decoded['user_id']
        request.state.user_role = decoded['role']
        return decoded  # → Continue
    except JWTError:
        raise HTTPException(status_code=401, detail="Invalid token")
        # ❌ Would TERMINATE here if invalid

# 9. Validation (automatic via Pydantic)
class CreateBookRequest(BaseModel):
    title: str = Field(..., min_length=1)
    author: str = Field(..., min_length=1)
    isbn: str
    
    @validator('*')
    def validate_fields(cls, v):
        if not v:
            raise ValueError('Field is required')
        return v

# ============================================
# 10. HANDLER
# ============================================
@app.post("/api/books", status_code=status.HTTP_201_CREATED)
async def create_book_handler(
    book_data: CreateBookRequest,
    request: Request,
    user_data: dict = Depends(verify_token)
):
    try:
        # Extract data (done by Pydantic)
        user_id = request.state.user_id
        user_role = request.state.user_role
        
        # Call service layer
        book = await book_service.create_book(
            title=book_data.title,
            author=book_data.author,
            isbn=book_data.isbn,
            user_id=user_id
        )
        
        # Send response
        return book
        
    except ValueError as error:
        # Pass error to exception handler
        raise HTTPException(status_code=400, detail=str(error))

# ============================================
# 11. SERVICE LAYER
# ============================================
class BookService:
    async def create_book(self, title: str, author: str, isbn: str, user_id: str):
        # Business validation
        if isbn and not is_valid_isbn(isbn):
            raise ValueError("Invalid ISBN format")
        
        # Check for duplicates
        existing = await self.book_repository.find_by_isbn(isbn)
        if existing:
            raise ValueError("Duplicate ISBN")
        
        # Create book object
        book = {
            "id": str(uuid.uuid4()),
            "title": title.strip(),
            "author": author.strip(),
            "isbn": isbn,
            "user_id": user_id,
            "created_at": datetime.now(),
            "updated_at": datetime.now()
        }
        
        # Database operation
        await self.book_repository.insert(book)
        
        # Update user's book count
        await self.user_repository.increment_book_count(user_id)
        
        # Send email (async - don't wait)
        try:
            await self.email_service.send_book_created(user_id, book)
        except Exception as e:
            logger.error(f"Failed to send email: {e}")
        
        # Return result
        return book

# ============================================
# 12. REPOSITORY LAYER
# ============================================
class BookRepository:
    async def find_by_isbn(self, isbn: str):
        query = 'SELECT * FROM books WHERE isbn = ?'
        rows = await self.db.fetch_all(query, [isbn])
        return self._map_row_to_book(rows[0]) if rows else None
    
    async def insert(self, book: dict):
        query = '''
            INSERT INTO books (id, title, author, isbn, user_id, created_at, updated_at)
            VALUES (?, ?, ?, ?, ?, ?, ?)
        '''
        
        await self.db.execute(query, [
            book["id"],
            book["title"],
            book["author"],
            book["isbn"],
            book["user_id"],
            book["created_at"],
            book["updated_at"]
        ])
        
        return book

# ============================================
# 13. RESPONSE SENT
# ============================================
# Handler returns:
{
    "id": "abc-123",
    "title": "Clean Code",
    "author": "Robert Martin",
    "isbn": "978-0132350884",
    "user_id": "user-456",
    "created_at": "2026-01-30T10:00:00.000Z",
    "updated_at": "2026-01-30T10:00:00.000Z"
}

# With headers:
# HTTP/1.1 201 Created
# X-Request-ID: abc-123-def-456
# Content-Type: application/json

# ============================================
# 14. IF ERROR OCCURRED
# ============================================
# Exception handler catches it
@app.exception_handler(ValueError)
async def value_error_handler(request: Request, exc: ValueError):
    request_id = getattr(request.state, 'request_id', 'unknown')
    logger.error(f"[{request_id}] Error: {exc}")
    
    if "Duplicate ISBN" in str(exc):
        return JSONResponse(
            status_code=409,
            content={
                "error": str(exc),
                "request_id": request_id
            }
        )
    
    return JSONResponse(
        status_code=500,
        content={
            "error": "Internal server error",
            "request_id": request_id
        }
    )
```
  console.log(`[${req.context.requestId}] POST /api/books`);
  next(); // → Continue
}

// 5. Body Parser (built-in)
// Converts JSON string to JavaScript object
// req.body = { title: "Clean Code", author: "Robert Martin", isbn: "978-0132350884" }
next(); // → Continue

// 6. Rate Limit Middleware
function rateLimitMiddleware(req, res, next) {
  const ip = req.ip;
  const count = getRateLimitCount(ip);
  
  if (count > 100) {
    return res.status(429).json({ error: 'Too many requests' });
    // ❌ Request TERMINATED here
  }
  
  incrementRateLimitCount(ip);
  next(); // → Continue
}

// 7. ROUTING
// Express matches POST /api/books to handler

// 8. Authentication Middleware (route-specific)
function authMiddleware(req, res, next) {
  const token = req.headers.authorization?.replace('Bearer ', '');
  
  if (!token) {
    return res.status(401).json({ error: 'No token' });
    // ❌ Would TERMINATE here if no token
  }
  
  try {
    const decoded = jwt.verify(token, SECRET);
    req.context.userId = decoded.userId;
    req.context.userRole = decoded.role;
    next(); // → Continue
  } catch {
    return res.status(401).json({ error: 'Invalid token' });
    // ❌ Would TERMINATE here if invalid
  }
}

// 9. Validation Middleware
function validateCreateBook(req, res, next) {
  const { error } = createBookSchema.validate(req.body);
  
  if (error) {
    return res.status(400).json({ error: error.message });
    // ❌ Would TERMINATE here if validation fails
  }
  
  next(); // → Continue to handler
}

// ============================================
// 10. HANDLER
// ============================================
async function createBookHandler(req, res, next) {
  try {
    // Extract data
    const { title, author, isbn } = req.body;
    const { userId, userRole } = req.context;
    
    // Basic validation
    if (!title || !author) {
      return res.status(400).json({ error: 'Missing required fields' });
    }
    
    // Call service layer
    const book = await bookService.createBook({
      title,
      author,
      isbn,
      userId
    });
    
    // Send response
    res.status(201).json(book);
    
  } catch (error) {
    next(error); // → Pass error to error handler
  }
}

// ============================================
// 11. SERVICE LAYER
// ============================================
async function createBook({ title, author, isbn, userId }) {
  // Business validation
  if (isbn && !isValidISBN(isbn)) {
    throw new BusinessError('Invalid ISBN format', 'INVALID_ISBN');
  }
  
  // Check for duplicates
  const existing = await bookRepository.findByISBN(isbn);
  if (existing) {
    throw new BusinessError('Duplicate ISBN', 'DUPLICATE_ISBN');
  }
  
  // Create book object
  const book = {
    id: uuidv4(),
    title: title.trim(),
    author: author.trim(),
    isbn,
    userId,
    createdAt: new Date(),
    updatedAt: new Date()
  };
  
  // Database operation
  await bookRepository.insert(book);
  
  // Update user's book count
  await userRepository.incrementBookCount(userId);
  
  // Send email (async - don't wait)
  emailService.sendBookCreated(userId, book).catch(console.error);
  
  // Return result
  return book;
}

// ============================================
// 12. REPOSITORY LAYER
// ============================================
async function findByISBN(isbn) {
  const query = 'SELECT * FROM books WHERE isbn = ?';
  const rows = await db.query(query, [isbn]);
  return rows.length > 0 ? mapRowToBook(rows[0]) : null;
}

async function insert(book) {
  const query = `
    INSERT INTO books (id, title, author, isbn, user_id, created_at, updated_at)
    VALUES (?, ?, ?, ?, ?, ?, ?)
  `;
  
  await db.query(query, [
    book.id,
    book.title,
    book.author,
    book.isbn,
    book.userId,
    book.createdAt,
    book.updatedAt
  ]);
  
  return book;
}

// ============================================
// 13. RESPONSE SENT
// ============================================
// Handler sends:
{
  "id": "abc-123",
  "title": "Clean Code",
  "author": "Robert Martin",
  "isbn": "978-0132350884",
  "userId": "user-456",
  "createdAt": "2026-01-30T10:00:00.000Z",
  "updatedAt": "2026-01-30T10:00:00.000Z"
}

// With headers:
// HTTP/1.1 201 Created
// X-Request-ID: abc-123-def-456
// Content-Type: application/json

// ============================================
// 14. IF ERROR OCCURRED
// ============================================
// Error handler middleware catches it
function errorHandler(err, req, res, next) {
  console.error(`[${req.context.requestId}] Error:`, err);
  
  if (err.code === 'DUPLICATE_ISBN') {
    return res.status(409).json({
      error: err.message,
      requestId: req.context.requestId
    });
  }
  
  res.status(500).json({
    error: 'Internal server error',
    requestId: req.context.requestId
  });
}
```

### Visual Flow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                         CLIENT SENDS REQUEST                    │
│  POST /api/books                                                │
│  Authorization: Bearer token...                                 │
│  Body: { title, author, isbn }                                  │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
         ╔═══════════════════════════════╗
         ║   MIDDLEWARE PIPELINE         ║
         ╚═══════════════════════════════╝
                         │
      ┌──────────────────┼──────────────────┐
      │                  │                  │
      ▼                  ▼                  ▼
┌───────────┐   ┌──────────────┐   ┌──────────────┐
│ Request   │   │    CORS      │   │   Logging    │
│    ID     │   │  Middleware  │   │  Middleware  │
└───────────┘   └──────────────┘   └──────────────┘
      │                  │                  │
      └──────────────────┼──────────────────┘
                         │
                         ▼
      ┌──────────────────┼──────────────────┐
      │                  │                  │
      ▼                  ▼                  ▼
┌───────────┐   ┌──────────────┐   ┌──────────────┐
│   Body    │   │  Rate Limit  │   │   Routing    │
│  Parser   │   │  Middleware  │   │   Engine     │
└───────────┘   └──────────────┘   └──────────────┘
                         │
                         ▼
      ┌──────────────────┼──────────────────┐
      │                  │                  │
      ▼                  ▼                  ▼
┌───────────┐   ┌──────────────┐   ┌──────────────┐
│   Auth    │   │  Validation  │   │   Handler    │
│ Middleware│   │  Middleware  │   │  (Controller)│
└───────────┘   └──────────────┘   └──────────────┘
                                            │
                                            ▼
                         ╔═══════════════════════════════╗
                         ║      SERVICE LAYER            ║
                         ║  - Business validation        ║
                         ║  - Check duplicates           ║
                         ║  - Orchestrate operations     ║
                         ╚═══════════════════════════════╝
                                            │
                    ┌───────────────────────┼───────────────────────┐
                    │                       │                       │
                    ▼                       ▼                       ▼
          ┌──────────────────┐   ┌──────────────────┐   ┌──────────────────┐
          │   Book Repo      │   │   User Repo      │   │  Email Service   │
          │  - findByISBN()  │   │  - increment()   │   │  - sendEmail()   │
          │  - insert()      │   │                  │   │                  │
          └──────────────────┘   └──────────────────┘   └──────────────────┘
                    │                       │
                    └───────────────────────┘
                                            │
                                            ▼
                         ╔═══════════════════════════════╗
                         ║         DATABASE              ║
                         ║  - Execute queries            ║
                         ║  - Return results             ║
                         ╚═══════════════════════════════╝
                                            │
                                            │ Results flow back up
                                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                     RESPONSE SENT TO CLIENT                     │
│  Status: 201 Created                                            │
│  Headers: X-Request-ID, Content-Type                            │
│  Body: { id, title, author, isbn, userId, createdAt }           │
└─────────────────────────────────────────────────────────────────┘
```

---

## Real-World Analogies

### Restaurant Analogy

Think of your backend like a restaurant:

| Backend Component | Restaurant Equivalent | What They Do |
|-------------------|----------------------|--------------|
| **Client** | Customer | Places an order |
| **Entry Point** | Restaurant door | Customer enters |
| **Middleware (CORS)** | Dress code checker | "Are you properly dressed?" |
| **Middleware (Auth)** | Host checking reservation | "Do you have a reservation?" |
| **Middleware (Logging)** | Hostess marking clipboard | Records who arrived and when |
| **Router** | Host showing you to table | Directs you to the right area |
| **Handler** | Waiter taking order | Takes your order, validates it |
| **Service** | Kitchen manager | Coordinates cooking, checks recipes |
| **Repository** | Individual chef stations | Each station handles specific dishes |
| **Database** | Food storage/pantry | Where ingredients are stored |
| **Response** | Waiter bringing food | Delivers the meal to you |
| **Request Context** | Order ticket | Travels with order, has table number, special requests |

**Flow:**

1. **Customer enters** (request arrives)
2. **Dress code check** (CORS middleware) - properly dressed? Let them in
3. **Reservation check** (auth middleware) - valid reservation? Show them to table
4. **Clipboard note** (logging) - "Table 5, party of 2, 7 PM"
5. **Shown to table** (routing) - "You're in section B"
6. **Waiter takes order** (handler) - validates menu items, notes allergies
7. **Order to kitchen** (service layer) - kitchen manager coordinates
8. **Chefs cook** (repository) - each station makes their part
9. **Ingredients from storage** (database) - get what's needed
10. **Food delivered** (response) - enjoy your meal!

The **order ticket** (context) travels from waiter → kitchen → chefs and back, containing:
- Table number (request ID)
- Customer allergies (user preferences)
- Special instructions (metadata)

### Post Office Analogy

| Backend Component | Post Office Equivalent | What They Do |
|-------------------|----------------------|--------------|
| **Middleware Pipeline** | Postal sorting facility | Mail goes through multiple stations |
| **Request ID** | Tracking number | Unique ID for each package |
| **CORS Middleware** | International customs | Checks if package can cross border |
| **Auth Middleware** | Signature verification | "Is this really from who it says?" |
| **Rate Limit** | Spam filter | "This sender is sending too much!" |
| **Handler** | Delivery person | Takes package to final destination |
| **Context** | Package metadata | Sender, recipient, tracking, priority |

---

## Common Mistakes and Pitfalls

### 1. Putting Business Logic in Handlers

❌ **Wrong:**
```python
@app.post("/api/books")
async def create_book_handler(request: Request):
    data = await request.json()
    title = data.get('title')
    author = data.get('author')
    isbn = data.get('isbn')
    
    # Business logic in handler - WRONG
    if isbn and len(isbn) != 13:
        raise HTTPException(status_code=400, detail="Invalid ISBN")
    
    # Database operation in handler - WRONG
    await db.execute('INSERT INTO books ...', [title, author, isbn])
    
    return {"message": "Created"}
```

✅ **Correct:**
```python
@app.post("/api/books")
async def create_book_handler(book_data: CreateBookRequest):
    book = await book_service.create_book(book_data.dict())
    return book

class BookService:
    async def create_book(self, data: dict):
        # Business logic in service - CORRECT
        if data.get('isbn') and not is_valid_isbn(data['isbn']):
            raise ValidationError('Invalid ISBN')
        
        # Database via repository - CORRECT
        return await self.book_repository.insert(data)
```

### 2. Service Layer Knowing About HTTP

❌ **Wrong:**
```python
from fastapi.responses import JSONResponse

class BookService:
    # Service should not have response parameters
    async def create_book(self, data: dict, response: JSONResponse):
        book = {**data, "id": str(uuid.uuid4())}
        
        await self.book_repository.insert(book)
        
        # Service shouldn't handle responses
        response.status_code = 201  # ❌
        return book
```

✅ **Correct:**
```python
class BookService:
    # Just returns data, no HTTP
    async def create_book(self, data: dict):
        book = {**data, "id": str(uuid.uuid4())}
        await self.book_repository.insert(book)
        return book  # ✅ Handler decides what HTTP response to send
```

### 3. Repository Containing Business Logic

❌ **Wrong:**
```python
class BookRepository:
    async def insert(self, book: dict):
        # Business validation in repository - WRONG
        if book.get("published_year", 0) > datetime.now().year:
            raise ValueError('Future year not allowed')
        
        # Complex business logic - WRONG
        if book.get("isbn"):
            existing = await self.find_by_isbn(book["isbn"])
            if existing:
                raise ValueError('Duplicate')
        
        await self.db.execute('INSERT ...')
```

✅ **Correct:**
```python
# Repository just does database operations
class BookRepository:
    async def insert(self, book: dict):
        query = 'INSERT INTO books (id, title, author) VALUES (?, ?, ?)'
        await self.db.execute(query, [book["id"], book["title"], book["author"]])

# Business logic in service
class BookService:
    async def create_book(self, data: dict):
        if data.get("published_year", 0) > datetime.now().year:
            raise ValidationError('Future year not allowed')
        
        if data.get("isbn"):
            existing = await self.book_repository.find_by_isbn(data["isbn"])
            if existing:
                raise DuplicateError('ISBN already exists')
        
        await self.book_repository.insert(data)
```

### 4. Wrong Middleware Order

❌ **Wrong:**
```python
# In FastAPI, middleware is applied in reverse order
# This example shows conceptual wrong order

# Auth before initialization - handlers won't have state
app.add_middleware(AuthMiddleware)
app.add_middleware(InitContextMiddleware)

# Error handler not configured properly
# (In FastAPI, use exception_handler decorator instead)
```

✅ **Correct:**
```python
from fastapi import FastAPI

app = FastAPI()

# Correct order (last added = first executed in FastAPI)
app.add_middleware(LoggingMiddleware)  # Executes 3rd
app.add_middleware(AuthMiddleware)  # Executes 2nd  
app.add_middleware(InitContextMiddleware)  # Executes 1st

# Exception handlers registered separately
@app.exception_handler(Exception)
async def exception_handler(request, exc):
    return JSONResponse(status_code=500, content={"error": str(exc)})
```

### 5. Not Handling Errors Properly

❌ **Wrong:**
```python
@app.post("/api/books")
async def handler(book_data: CreateBookRequest):
    book = await book_service.create_book(book_data.dict())
    return book
    # No try-except - unhandled exceptions
```

✅ **Correct:**
```python
@app.post("/api/books")
async def handler(book_data: CreateBookRequest):
    try:
        book = await book_service.create_book(book_data.dict())
        return book
    except ValueError as error:
        # Handle specific errors
        raise HTTPException(status_code=400, detail=str(error))
    except Exception as error:
        # Handle unexpected errors
        logger.error(f"Unexpected error: {error}")
        raise HTTPException(status_code=500, detail="Internal server error")
```

### 6. Storing Too Much in Context

❌ **Wrong:**
```javascript
req.context.user = await userRepository.findById(userId);
req.context.allBooks = await bookRepository.findAll();
req.context.settings = await settingsRepository.findAll();
// Too much data - memory waste
```

✅ **Correct:**
```javascript
req.context.userId = user.id;
req.context.userRole = user.role;
// Only store what's needed
```

### 7. Not Calling next() in Middleware

❌ **Wrong:**
```javascript
function myMiddleware(req, res, next) {
  console.log('Request received');
  // Forgot to call next() - request hangs forever
}
```

✅ **Correct:**
```javascript
function myMiddleware(req, res, next) {
  console.log('Request received');
  next(); // Must call next()
}
```

### 8. Using Global Variables

❌ **Wrong:**
```javascript
let currentUser; // Global variable

function authMiddleware(req, res, next) {
  currentUser = verifyToken(req.headers.authorization);
  next();
}

// Breaks with concurrent requests - different users overwrite each other
```

✅ **Correct:**
```javascript
function authMiddleware(req, res, next) {
  req.context.userId = verifyToken(req.headers.authorization);
  next();
}
```

### 9. Repository Methods Doing Too Much

❌ **Wrong:**
```javascript
// One method trying to do everything
async function findBooks({ id, isbn, title, author, sort, limit }) {
  if (id) {
    // Find by ID logic
  } else if (isbn) {
    // Find by ISBN logic
  } else {
    // Find all logic
  }
  // Too complex - hard to maintain
}
```

✅ **Correct:**
```javascript
// Separate methods for different queries
async function findById(id) { }
async function findByISBN(isbn) { }
async function findAll({ sort, limit }) { }
```

---

## Best Practices

### Handler/Controller Best Practices

1. **Keep handlers thin** - delegate to services
2. **Always handle errors** - use try-except
3. **Validate early** - reject bad data immediately
4. **Use appropriate HTTP status codes**
5. **Don't trust client data** - always validate
6. **Extract user info from state** - never from request body

```python
from fastapi import APIRouter, HTTPException, Request, status
from pydantic import BaseModel

router = APIRouter()

class CreateBookRequest(BaseModel):
    title: str
    author: str

@router.post("/api/books", status_code=status.HTTP_201_CREATED)
async def create_book_handler(book_data: CreateBookRequest, request: Request):
    try:
        # 1. Extract data (done by Pydantic)
        user_id = request.state.user_id  # From auth, not from client
        
        # 2. Basic validation (done by Pydantic)
        if not book_data.title:
            raise HTTPException(
                status_code=status.HTTP_400_BAD_REQUEST,
                detail="Title required"
            )
        
        # 3. Call service
        book = await book_service.create_book(
            **book_data.dict(),
            user_id=user_id
        )
        
        # 4. Send appropriate response
        return book
        
    except ValueError as error:
        # Let exception handler deal with it
        raise HTTPException(status_code=400, detail=str(error))
```

### Service Layer Best Practices

1. **No HTTP dependencies** - should work in CLI, cron jobs, etc.
2. **Single responsibility** - one method does one thing
3. **Raise meaningful errors** - with error codes
4. **Orchestrate operations** - coordinate repositories
5. **Handle transactions** - ensure data consistency

```python
from typing import Dict
import uuid
from datetime import datetime

class ValidationError(Exception):
    def __init__(self, message: str):
        self.message = message
        super().__init__(self.message)

class DuplicateError(Exception):
    def __init__(self, message: str):
        self.message = message
        super().__init__(self.message)

class BookService:
    def __init__(self, book_repository, user_repository):
        self.book_repository = book_repository
        self.user_repository = user_repository
    
    async def create_book(self, title: str, author: str, isbn: str, user_id: str) -> Dict:
        # Business validation
        await self._validate_book_data(title=title, author=author, isbn=isbn)
        
        # Check duplicates
        await self._ensure_no_duplicate(isbn)
        
        # Create book
        book = {
            "id": str(uuid.uuid4()),
            "title": title.strip(),
            "author": author.strip(),
            "isbn": isbn,
            "user_id": user_id,
            "created_at": datetime.now()
        }
        
        # Persist
        await self.book_repository.insert(book)
        
        # Side effects
        await self._update_user_stats(user_id)
        
        return book
    
    async def _validate_book_data(self, title: str, author: str, isbn: str):
        if not title or len(title) < 3:
            raise ValidationError('Title must be at least 3 characters')
        # ...
    
    async def _ensure_no_duplicate(self, isbn: str):
        if not isbn:
            return
        
        existing = await self.book_repository.find_by_isbn(isbn)
        if existing:
            raise DuplicateError('Book with this ISBN already exists')
    
    async def _update_user_stats(self, user_id: str):
        await self.user_repository.increment_book_count(user_id)
```

### Repository Best Practices

1. **Single purpose** - one method = one query
2. **No business logic** - just database operations
3. **Consistent return types** - always return same type
4. **Use parameterized queries** - prevent SQL injection
5. **Map database rows to objects** - encapsulate mapping logic

```python
from typing import Optional, Dict, List

class BookRepository:
    def __init__(self, db):
        self.db = db
    
    async def find_by_id(self, book_id: str) -> Optional[Dict]:
        query = 'SELECT * FROM books WHERE id = ? AND deleted_at IS NULL'
        rows = await self.db.fetch_all(query, [book_id])
        return self._map_row(rows[0]) if rows else None
    
    async def find_by_isbn(self, isbn: str) -> Optional[Dict]:
        query = 'SELECT * FROM books WHERE isbn = ? AND deleted_at IS NULL'
        rows = await self.db.fetch_all(query, [isbn])
        return self._map_row(rows[0]) if rows else None
    
    async def insert(self, book: Dict) -> Dict:
        query = '''
            INSERT INTO books (id, title, author, isbn, user_id, created_at)
            VALUES (?, ?, ?, ?, ?, ?)
        '''
        
        await self.db.execute(query, [
            book["id"],
            book["title"],
            book["author"],
            book.get("isbn"),
            book["user_id"],
            book["created_at"]
        ])
        
        return book
    
    def _map_row(self, row) -> Dict:
        return {
            "id": row["id"],
            "title": row["title"],
            "author": row["author"],
            "isbn": row["isbn"],
            "user_id": row["user_id"],
            "created_at": row["created_at"]
        }
```

### Middleware Best Practices

1. **Order matters** - think about execution flow
2. **Always call next()** - unless terminating request
3. **Keep middleware focused** - one responsibility
4. **Use route-specific middleware** - don't apply globally if not needed
5. **Handle errors properly** - pass to error handler

```javascript
// Good middleware structure
app.use(requestIdMiddleware);        // 1. First
app.use(corsMiddleware);              // 2. Early (might terminate)
app.use(loggingMiddleware);           // 3. After request ID
app.use(express.json());              // 4. Parse body
app.use(rateLimitMiddleware);         // 5. Security

// Public routes
app.use('/api/public', publicRoutes);

// Protected routes
app.use('/api/protected', authMiddleware); // Route-specific
app.use('/api/protected', protectedRoutes);

// 404 handler
app.use(notFoundHandler);

// Error handler - LAST
app.use(errorHandlerMiddleware);
```

### Request Context Best Practices

1. **Initialize early** - in first middleware
2. **Store only essentials** - avoid large objects
3. **Use consistent naming** - camelCase or snake_case
4. **Don't store sensitive data** - only IDs and roles
5. **TypeScript types** - define context interface

```javascript
// Initialize context
function initContext(req, res, next) {
  req.context = {
    requestId: req.headers['x-request-id'] || generateId(),
    startTime: Date.now()
  };
  next();
}

// Add to context
function authMiddleware(req, res, next) {
  const decoded = verifyToken(req.headers.authorization);
  req.context.userId = decoded.userId;
  req.context.userRole = decoded.role;
  next();
}

// Use context
function handler(req, res) {
  const { userId, userRole, requestId } = req.context;
  // Use the data
}
```

---

## Quick Revision Cheat Sheet

### Request Lifecycle Flow

```
Client → Entry Point → Middleware Pipeline → Routing → Handler → Service → Repository → Database
                                                                                             ↓
Client ← Response ← Middleware ← Handler ← Service ← Repository ← Database Results ←────────┘
```

### Three-Layer Architecture

| Layer | Responsibility | Example |
|-------|---------------|---------|
| **Handler** | HTTP concerns: extract data, validate, send response | `createBookHandler(req, res)` |
| **Service** | Business logic: rules, orchestration, processing | `bookService.createBook(data)` |
| **Repository** | Database operations: queries, CRUD | `bookRepository.insert(book)` |

### Handler Checklist

- [ ] Extract data from request
- [ ] Deserialize JSON to native format
- [ ] Validate data
- [ ] Transform/normalize data
- [ ] Call service layer
- [ ] Send appropriate HTTP response
- [ ] Handle errors

### Service Checklist

- [ ] No HTTP dependencies (no req/res)
- [ ] Apply business rules
- [ ] Orchestrate multiple operations
- [ ] Call repositories for data
- [ ] Return data or throw errors
- [ ] Single responsibility per method

### Repository Checklist

- [ ] One method = one database operation
- [ ] No business logic
- [ ] Parameterized queries (prevent SQL injection)
- [ ] Consistent return types
- [ ] Map database rows to objects

### Middleware Types & Order

```javascript
1.  requestIdMiddleware     // Generate tracking ID
2.  corsMiddleware          // Handle cross-origin
3.  securityHeadersMiddleware // Security headers
4.  loggingMiddleware       // Log requests
5.  bodyParserMiddleware    // Parse JSON
6.  rateLimitMiddleware     // Prevent abuse
7.  compressionMiddleware   // Compress responses
// ... routing
8.  authMiddleware          // Verify credentials (route-specific)
9.  validationMiddleware    // Validate data (route-specific)
// ... handler
10. errorHandlerMiddleware  // LAST - catch all errors
```

### Middleware Signature

```javascript
function middleware(req, res, next) {
  // Do something
  next(); // Pass to next middleware
  
  // Or terminate:
  // res.status(400).json({ error: 'Bad request' });
}

// Error handler has 4 parameters
function errorHandler(err, req, res, next) {
  // Handle error
}
```

### Request Context

**What:** Shared storage for request-scoped data

**Why:** Pass data between middleware/handlers without tight coupling

**Common Uses:**
- Authentication data (`userId`, `userRole`)
- Request tracking (`requestId`)
- Timing (`startTime`)
- Feature flags
- Transaction handles

```javascript
// Initialize
req.context = { requestId: generateId() };

// Add data
req.context.userId = decoded.userId;

// Use data
const { userId } = req.context;
```

### HTTP Status Codes Quick Reference

| Code | Meaning | When to Use |
|------|---------|-------------|
| 200 | OK | Successful GET, PUT, PATCH |
| 201 | Created | Successful POST (resource created) |
| 204 | No Content | Successful DELETE |
| 400 | Bad Request | Validation error, malformed request |
| 401 | Unauthorized | Not authenticated |
| 403 | Forbidden | Authenticated but no permission |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Duplicate resource |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Unhandled server error |

### Error Handling Pattern

```javascript
// Define custom errors
class BusinessError extends Error {
  constructor(message, code) {
    super(message);
    this.code = code;
    this.statusCode = 400;
  }
}

// Throw in service
throw new BusinessError('Invalid ISBN', 'INVALID_ISBN');

// Catch in handler
try {
  await service.method();
} catch (error) {
  next(error); // Pass to error handler
}

// Global error handler
function errorHandler(err, req, res, next) {
  if (err.statusCode) {
    return res.status(err.statusCode).json({ 
      error: err.message,
      code: err.code 
    });
  }
  
  res.status(500).json({ error: 'Internal server error' });
}
```

### Complete Example (All Together)

```python
# ===== MIDDLEWARE =====
app.add_middleware(InitContextMiddleware)
app.add_middleware(CORSMiddleware, allow_origins=["*"])
app.add_middleware(LoggingMiddleware)

# ===== ROUTES =====
@app.post("/api/books", status_code=status.HTTP_201_CREATED)
async def create_book_route(
    book_data: CreateBookRequest,
    request: Request,
    user: dict = Depends(verify_token)
):
    return await create_book_handler(book_data, request)

# ===== HANDLER =====
async def create_book_handler(book_data: CreateBookRequest, request: Request):
    try:
        user_id = request.state.user_id
        book = await book_service.create_book(**book_data.dict(), user_id=user_id)
        return book
    except ValueError as error:
        raise HTTPException(status_code=400, detail=str(error))

# ===== SERVICE =====
class BookService:
    async def create_book(self, title: str, author: str, isbn: str, user_id: str):
        if isbn:
            exists = await self.book_repository.find_by_isbn(isbn)
            if exists:
                raise ValueError('ISBN exists')
        
        book = {
            "id": str(uuid.uuid4()),
            "title": title.strip(),
            "author": author.strip(),
            "isbn": isbn,
            "user_id": user_id,
            "created_at": datetime.now()
        }
        await self.book_repository.insert(book)
        return book

# ===== REPOSITORY =====
class BookRepository:
    async def insert(self, book: dict):
        query = 'INSERT INTO books (id, title, author) VALUES (?, ?, ?)'
        await self.db.execute(query, [book["id"], book["title"], book["author"]])
        return book

# ===== ERROR HANDLER =====
@app.exception_handler(ValueError)
async def value_error_handler(request: Request, exc: ValueError):
    request_id = getattr(request.state, 'request_id', 'unknown')
    return JSONResponse(
        status_code=400,
        content={
            "error": str(exc),
            "request_id": request_id
        }
    )
```

### Key Takeaways

✅ **Separation of Concerns**: Each layer has one job
✅ **Middleware for Cross-Cutting**: Reusable logic applied to many routes
✅ **Context for Sharing**: Pass data without tight coupling
✅ **Always Handle Errors**: Try-catch and error middleware
✅ **Order Matters**: Middleware execute sequentially
✅ **Keep Handlers Thin**: Business logic in services
✅ **One Query Per Repo Method**: Single responsibility
✅ **No HTTP in Services**: Make them reusable

---

**End of Guide**

This architecture pattern enables you to build scalable, maintainable, and testable backend applications. Master these concepts and you'll be ready for any backend interview or project! 🚀
