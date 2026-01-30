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

```javascript
// Everything in one function - NOT RECOMMENDED
app.post('/api/books', async (req, res) => {
  // HTTP parsing
  const { title, author, isbn } = req.body;
  
  // Validation
  if (!title || title.length < 3) {
    return res.status(400).json({ error: 'Invalid title' });
  }
  
  // Business logic
  const book = {
    id: generateId(),
    title: title.trim(),
    author: author.trim(),
    isbn,
    createdAt: new Date()
  };
  
  // Database operation
  const sql = 'INSERT INTO books (id, title, author, isbn, created_at) VALUES (?, ?, ?, ?, ?)';
  await db.query(sql, [book.id, book.title, book.author, book.isbn, book.createdAt]);
  
  // Another database operation
  const userSql = 'UPDATE users SET book_count = book_count + 1 WHERE id = ?';
  await db.query(userSql, [req.user.id]);
  
  // Email sending
  await sendEmail(req.user.email, 'Book created!');
  
  // HTTP response
  res.status(201).json(book);
});
```

**Problems:**
- Hard to test (everything is coupled)
- Cannot reuse logic (tied to HTTP)
- Difficult to maintain
- Mixed concerns
- Cannot easily change database or framework

#### ✅ With the Pattern

```javascript
// HANDLER - handles HTTP concerns
async function createBookHandler(req, res) {
  const { title, author, isbn } = req.body;
  const userId = req.context.userId; // from auth middleware
  
  const book = await bookService.createBook({ title, author, isbn, userId });
  
  res.status(201).json(book);
}

// SERVICE - business logic
async function createBook({ title, author, isbn, userId }) {
  // Business logic
  const book = {
    id: generateId(),
    title: title.trim(),
    author: author.trim(),
    isbn,
    createdAt: new Date()
  };
  
  // Orchestrate multiple operations
  await bookRepository.insert(book);
  await userRepository.incrementBookCount(userId);
  await emailService.sendBookCreatedEmail(userId);
  
  return book;
}

// REPOSITORY - database operations
async function insertBook(book) {
  const sql = 'INSERT INTO books (id, title, author, isbn, created_at) VALUES (?, ?, ?, ?, ?)';
  return db.query(sql, [book.id, book.title, book.author, book.isbn, book.createdAt]);
}
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

Every handler typically receives two (or three) objects:

```javascript
// Node.js / Express
function handler(req, res, next) {
  // req: request object
  // res: response object
  // next: pass to next middleware (optional)
}

// Go (using Gin framework)
func handler(c *gin.Context) {
  // c contains both request and response
}

// Python (FastAPI)
def handler(request: Request):
  # request object
  return response
```

#### Handler Examples

**Example 1: GET Request - Fetch Books**

```javascript
// GET /api/books?sort=name&limit=10
async function getBooksHandler(req, res) {
  try {
    // 1. Extract data from request
    const { sort = 'date', limit = 10, offset = 0 } = req.query;
    
    // 2. Validation (basic - more complex validation can use libraries)
    if (limit < 1 || limit > 100) {
      return res.status(400).json({ error: 'Limit must be between 1 and 100' });
    }
    
    // 3. Transformation - convert strings to numbers
    const parsedLimit = parseInt(limit);
    const parsedOffset = parseInt(offset);
    
    // 4. Call service layer
    const books = await bookService.getBooks({
      sort,
      limit: parsedLimit,
      offset: parsedOffset
    });
    
    // 5. Send response
    res.status(200).json({
      data: books,
      pagination: {
        limit: parsedLimit,
        offset: parsedOffset
      }
    });
    
  } catch (error) {
    // 6. Error handling
    console.error('Error in getBooksHandler:', error);
    res.status(500).json({ error: 'Internal server error' });
  }
}
```

**Example 2: POST Request - Create Book**

```javascript
// POST /api/books
async function createBookHandler(req, res) {
  try {
    // 1. Extract data from request body
    const { title, author, isbn, publishedYear } = req.body;
    
    // 2. Get authenticated user from context (set by auth middleware)
    const userId = req.context.userId;
    
    // 3. Validation
    if (!title || title.trim().length < 3) {
      return res.status(400).json({ 
        error: 'Title is required and must be at least 3 characters' 
      });
    }
    
    if (!author || author.trim().length < 2) {
      return res.status(400).json({ 
        error: 'Author is required and must be at least 2 characters' 
      });
    }
    
    // 4. Transformation
    const bookData = {
      title: title.trim(),
      author: author.trim(),
      isbn: isbn?.trim(),
      publishedYear: publishedYear ? parseInt(publishedYear) : null,
      userId
    };
    
    // 5. Call service layer
    const book = await bookService.createBook(bookData);
    
    // 6. Send response (201 Created for successful POST)
    res.status(201).json(book);
    
  } catch (error) {
    console.error('Error in createBookHandler:', error);
    
    // More specific error handling
    if (error.code === 'DUPLICATE_ISBN') {
      return res.status(409).json({ error: 'Book with this ISBN already exists' });
    }
    
    res.status(500).json({ error: 'Internal server error' });
  }
}
```

**Example 3: DELETE Request**

```javascript
// DELETE /api/books/:id
async function deleteBookHandler(req, res) {
  try {
    // 1. Extract path parameter
    const { id } = req.params;
    
    // 2. Get user from context
    const userId = req.context.userId;
    const userRole = req.context.role;
    
    // 3. Validation
    if (!id || !isValidUUID(id)) {
      return res.status(400).json({ error: 'Invalid book ID' });
    }
    
    // 4. Call service layer (service handles permission checks)
    await bookService.deleteBook({ id, userId, userRole });
    
    // 5. Send response (204 No Content for successful DELETE)
    res.status(204).send();
    
  } catch (error) {
    if (error.code === 'NOT_FOUND') {
      return res.status(404).json({ error: 'Book not found' });
    }
    
    if (error.code === 'FORBIDDEN') {
      return res.status(403).json({ error: 'You do not have permission to delete this book' });
    }
    
    res.status(500).json({ error: 'Internal server error' });
  }
}
```

#### Deserialization in Different Languages

**JavaScript/Node.js:**
```javascript
// Usually handled by middleware (express.json())
app.use(express.json());

// In handler, you get parsed object directly
function handler(req, res) {
  const { name, age } = req.body; // Already a JS object
}
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

**Python (FastAPI):**
```python
from pydantic import BaseModel

class CreateBookRequest(BaseModel):
    title: str
    author: str
    isbn: Optional[str] = None

@app.post("/api/books")
async def create_book(book: CreateBookRequest):
    # FastAPI automatically deserializes and validates
    # book is a Python object with type checking
    pass
```

#### When to Use Handlers

**Always** - Every API endpoint needs a handler. The handler is the entry point for your business logic.

#### Common Mistakes in Handlers

❌ **Putting business logic in handlers**
```javascript
// WRONG - business logic in handler
async function createBookHandler(req, res) {
  const book = { ...req.body, id: generateId() };
  
  // This business logic should be in service layer
  if (book.publishedYear > new Date().getFullYear()) {
    return res.status(400).json({ error: 'Future publication date not allowed' });
  }
  
  await db.query('INSERT INTO books ...', book);
  res.status(201).json(book);
}
```

✅ **Correct - delegate to service layer**
```javascript
async function createBookHandler(req, res) {
  const book = await bookService.createBook(req.body);
  res.status(201).json(book);
}

// Business logic in service
async function createBook(data) {
  if (data.publishedYear > new Date().getFullYear()) {
    throw new ValidationError('Future publication date not allowed');
  }
  // ... rest of logic
}
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

```javascript
// bookService.js

async function getBooks({ sort = 'date', limit = 10, offset = 0 }) {
  // 1. Apply defaults (can also be done in handler)
  const validSort = ['date', 'name'].includes(sort) ? sort : 'date';
  
  // 2. Call repository
  const books = await bookRepository.findAll({
    sortBy: validSort,
    limit,
    offset
  });
  
  // 3. Business logic - maybe filter or transform
  // For example, hide books that are not published yet
  const publishedBooks = books.filter(book => {
    return book.publishedDate <= new Date();
  });
  
  return publishedBooks;
}
```

**Example 2: Complex Service - Create Book**

```javascript
async function createBook({ title, author, isbn, publishedYear, userId }) {
  // 1. Business validation
  if (publishedYear > new Date().getFullYear()) {
    throw new BusinessError('Cannot create book with future publication year', 'INVALID_YEAR');
  }
  
  // 2. Check for duplicates
  if (isbn) {
    const existing = await bookRepository.findByISBN(isbn);
    if (existing) {
      throw new BusinessError('Book with this ISBN already exists', 'DUPLICATE_ISBN');
    }
  }
  
  // 3. Prepare data
  const book = {
    id: generateUUID(),
    title: title.trim(),
    author: author.trim(),
    isbn,
    publishedYear,
    userId,
    createdAt: new Date(),
    updatedAt: new Date()
  };
  
  // 4. Orchestrate multiple operations
  // Insert book
  await bookRepository.insert(book);
  
  // Update user's book count
  await userRepository.incrementBookCount(userId);
  
  // Add to search index (external service)
  await searchService.indexBook(book);
  
  // Send notification email
  await emailService.sendBookCreatedNotification(userId, book);
  
  // 5. Return data
  return book;
}
```

**Example 3: Service with Transaction**

```javascript
async function transferBookOwnership({ bookId, fromUserId, toUserId, requesterId }) {
  // 1. Permission check
  if (requesterId !== fromUserId) {
    throw new ForbiddenError('You can only transfer your own books');
  }
  
  // 2. Verify book exists and belongs to user
  const book = await bookRepository.findById(bookId);
  if (!book) {
    throw new NotFoundError('Book not found');
  }
  
  if (book.userId !== fromUserId) {
    throw new ForbiddenError('This book does not belong to you');
  }
  
  // 3. Verify recipient exists
  const recipient = await userRepository.findById(toUserId);
  if (!recipient) {
    throw new NotFoundError('Recipient user not found');
  }
  
  // 4. Perform transfer in transaction (all or nothing)
  await database.transaction(async (trx) => {
    // Update book ownership
    await bookRepository.updateOwner(bookId, toUserId, trx);
    
    // Decrement old owner's count
    await userRepository.decrementBookCount(fromUserId, trx);
    
    // Increment new owner's count
    await userRepository.incrementBookCount(toUserId, trx);
    
    // Create transfer log
    await transferLogRepository.create({
      bookId,
      fromUserId,
      toUserId,
      timestamp: new Date()
    }, trx);
  });
  
  // 5. Send notifications
  await emailService.sendTransferNotification(fromUserId, toUserId, book);
  
  return { success: true };
}
```

**Example 4: Service Without Database Calls**

```javascript
// Not all services need to call repositories
async function sendBookRecommendations(userId) {
  // 1. Get user preferences
  const user = await userRepository.findById(userId);
  
  // 2. Call external recommendation API
  const recommendations = await externalAPI.getRecommendations({
    genres: user.favoriteGenres,
    authors: user.favoriteAuthors
  });
  
  // 3. Send email (no database operation)
  await emailService.sendRecommendationEmail(user.email, recommendations);
  
  return { sent: true, count: recommendations.length };
}
```

#### Service Pattern Guidelines

**Single Responsibility:**
```javascript
// GOOD - Each service method does one thing
bookService.createBook(data)
bookService.updateBook(id, data)
bookService.deleteBook(id)
bookService.getBook(id)
bookService.getAllBooks(filters)

// BAD - Too much in one method
bookService.handleBookOperation(action, data)
```

**No HTTP Awareness:**
```javascript
// WRONG - Service should not know about HTTP
async function createBook(req, res) {
  // ...
  res.status(201).json(book); // ❌ Service shouldn't handle responses
}

// CORRECT - Service just returns data or throws errors
async function createBook(data) {
  // ...
  return book; // ✅ Handler decides what HTTP response to send
}
```

**Meaningful Errors:**
```javascript
class BusinessError extends Error {
  constructor(message, code) {
    super(message);
    this.code = code;
    this.isBusinessError = true;
  }
}

async function createBook(data) {
  if (data.isbn) {
    const existing = await bookRepository.findByISBN(data.isbn);
    if (existing) {
      // Throw business error - handler will convert to HTTP error
      throw new BusinessError('Duplicate ISBN', 'DUPLICATE_ISBN');
    }
  }
  // ...
}
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

```javascript
// bookRepository.js

// Find all books with filtering and sorting
async function findAll({ sortBy = 'date', limit = 10, offset = 0 }) {
  let query = 'SELECT * FROM books WHERE deleted_at IS NULL';
  const params = [];
  
  // Add sorting
  if (sortBy === 'name') {
    query += ' ORDER BY title ASC';
  } else {
    query += ' ORDER BY created_at DESC';
  }
  
  // Add pagination
  query += ' LIMIT ? OFFSET ?';
  params.push(limit, offset);
  
  const rows = await db.query(query, params);
  return rows.map(mapRowToBook);
}

// Find one book by ID
async function findById(id) {
  const query = 'SELECT * FROM books WHERE id = ? AND deleted_at IS NULL';
  const rows = await db.query(query, [id]);
  
  if (rows.length === 0) {
    return null;
  }
  
  return mapRowToBook(rows[0]);
}

// Find by ISBN
async function findByISBN(isbn) {
  const query = 'SELECT * FROM books WHERE isbn = ? AND deleted_at IS NULL';
  const rows = await db.query(query, [isbn]);
  
  if (rows.length === 0) {
    return null;
  }
  
  return mapRowToBook(rows[0]);
}

// Insert new book
async function insert(book) {
  const query = `
    INSERT INTO books (id, title, author, isbn, published_year, user_id, created_at, updated_at)
    VALUES (?, ?, ?, ?, ?, ?, ?, ?)
  `;
  
  const params = [
    book.id,
    book.title,
    book.author,
    book.isbn,
    book.publishedYear,
    book.userId,
    book.createdAt,
    book.updatedAt
  ];
  
  await db.query(query, params);
  return book;
}

// Update book
async function update(id, updates) {
  const query = `
    UPDATE books 
    SET title = ?, author = ?, isbn = ?, published_year = ?, updated_at = ?
    WHERE id = ? AND deleted_at IS NULL
  `;
  
  const params = [
    updates.title,
    updates.author,
    updates.isbn,
    updates.publishedYear,
    new Date(),
    id
  ];
  
  const result = await db.query(query, params);
  return result.affectedRows > 0;
}

// Soft delete
async function softDelete(id) {
  const query = 'UPDATE books SET deleted_at = ? WHERE id = ?';
  const result = await db.query(query, [new Date(), id]);
  return result.affectedRows > 0;
}

// Helper function to map database row to object
function mapRowToBook(row) {
  return {
    id: row.id,
    title: row.title,
    author: row.author,
    isbn: row.isbn,
    publishedYear: row.published_year,
    userId: row.user_id,
    createdAt: row.created_at,
    updatedAt: row.updated_at
  };
}

module.exports = {
  findAll,
  findById,
  findByISBN,
  insert,
  update,
  softDelete
};
```

**Example 2: User Repository**

```javascript
// userRepository.js

async function findById(id) {
  const query = 'SELECT * FROM users WHERE id = ?';
  const rows = await db.query(query, [id]);
  return rows.length > 0 ? mapRowToUser(rows[0]) : null;
}

async function incrementBookCount(userId) {
  const query = 'UPDATE users SET book_count = book_count + 1 WHERE id = ?';
  await db.query(query, [userId]);
}

async function decrementBookCount(userId) {
  const query = 'UPDATE users SET book_count = book_count - 1 WHERE id = ?';
  await db.query(query, [userId]);
}

// With transaction support
async function incrementBookCount(userId, transaction = null) {
  const db = transaction || database;
  const query = 'UPDATE users SET book_count = book_count + 1 WHERE id = ?';
  await db.query(query, [userId]);
}

function mapRowToUser(row) {
  return {
    id: row.id,
    email: row.email,
    name: row.name,
    bookCount: row.book_count,
    createdAt: row.created_at
  };
}

module.exports = {
  findById,
  incrementBookCount,
  decrementBookCount
};
```

**Example 3: Advanced Queries**

```javascript
// Find books with complex filtering
async function findByFilters({ genre, minYear, maxYear, authorName, limit, offset }) {
  let query = 'SELECT * FROM books WHERE deleted_at IS NULL';
  const params = [];
  
  if (genre) {
    query += ' AND genre = ?';
    params.push(genre);
  }
  
  if (minYear) {
    query += ' AND published_year >= ?';
    params.push(minYear);
  }
  
  if (maxYear) {
    query += ' AND published_year <= ?';
    params.push(maxYear);
  }
  
  if (authorName) {
    query += ' AND author LIKE ?';
    params.push(`%${authorName}%`);
  }
  
  query += ' ORDER BY created_at DESC LIMIT ? OFFSET ?';
  params.push(limit, offset);
  
  const rows = await db.query(query, params);
  return rows.map(mapRowToBook);
}

// Join query
async function findBooksWithUserDetails(bookId) {
  const query = `
    SELECT 
      b.*,
      u.name as user_name,
      u.email as user_email
    FROM books b
    JOIN users u ON b.user_id = u.id
    WHERE b.id = ? AND b.deleted_at IS NULL
  `;
  
  const rows = await db.query(query, [bookId]);
  
  if (rows.length === 0) {
    return null;
  }
  
  const row = rows[0];
  return {
    book: mapRowToBook(row),
    user: {
      name: row.user_name,
      email: row.user_email
    }
  };
}
```

#### Repository Best Practices

**Single Purpose per Method:**
```javascript
// GOOD - Separate methods for different queries
bookRepository.findById(id)
bookRepository.findByISBN(isbn)
bookRepository.findAll(filters)

// BAD - One method trying to do everything
bookRepository.find({ id, isbn, filters }) // Confusing and hard to maintain
```

**No Business Logic:**
```javascript
// WRONG - Business logic in repository
async function insert(book) {
  // ❌ This check belongs in service layer
  if (book.publishedYear > new Date().getFullYear()) {
    throw new Error('Future year not allowed');
  }
  
  await db.query('INSERT INTO books ...', book);
}

// CORRECT - Repository just does database operations
async function insert(book) {
  await db.query('INSERT INTO books ...', book);
}
```

**Always Return Consistent Types:**
```javascript
// GOOD - Always return same type
async function findById(id) {
  const rows = await db.query('SELECT * FROM books WHERE id = ?', [id]);
  return rows.length > 0 ? mapRowToBook(rows[0]) : null; // Always Book | null
}

// BAD - Inconsistent return types
async function findById(id) {
  const rows = await db.query('SELECT * FROM books WHERE id = ?', [id]);
  if (rows.length === 0) {
    throw new Error('Not found'); // Sometimes throws
  }
  return mapRowToBook(rows[0]); // Sometimes returns book
}
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

```javascript
// ============================================
// HANDLER (bookHandler.js)
// ============================================
async function createBookHandler(req, res) {
  try {
    // 1. Extract and validate
    const { title, author, isbn } = req.body;
    const userId = req.context.userId; // From auth middleware
    
    if (!title || !author) {
      return res.status(400).json({ error: 'Title and author are required' });
    }
    
    // 2. Call service
    const book = await bookService.createBook({ title, author, isbn, userId });
    
    // 3. Send response
    res.status(201).json(book);
    
  } catch (error) {
    // 4. Handle errors
    if (error.code === 'DUPLICATE_ISBN') {
      return res.status(409).json({ error: error.message });
    }
    
    console.error('Handler error:', error);
    res.status(500).json({ error: 'Internal server error' });
  }
}

// ============================================
// SERVICE (bookService.js)
// ============================================
async function createBook({ title, author, isbn, userId }) {
  // 1. Business validation
  if (isbn && !isValidISBN(isbn)) {
    throw new BusinessError('Invalid ISBN format', 'INVALID_ISBN');
  }
  
  // 2. Check for duplicates
  if (isbn) {
    const existing = await bookRepository.findByISBN(isbn);
    if (existing) {
      throw new BusinessError('Book with this ISBN already exists', 'DUPLICATE_ISBN');
    }
  }
  
  // 3. Create book object
  const book = {
    id: generateUUID(),
    title: title.trim(),
    author: author.trim(),
    isbn,
    userId,
    createdAt: new Date(),
    updatedAt: new Date()
  };
  
  // 4. Persist to database
  await bookRepository.insert(book);
  
  // 5. Update user's book count
  await userRepository.incrementBookCount(userId);
  
  // 6. Send notification (fire and forget - don't wait)
  emailService.sendBookCreatedEmail(userId, book).catch(err => {
    console.error('Failed to send email:', err);
  });
  
  // 7. Return result
  return book;
}

// ============================================
// REPOSITORY (bookRepository.js)
// ============================================
async function findByISBN(isbn) {
  const query = 'SELECT * FROM books WHERE isbn = ? AND deleted_at IS NULL';
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

function mapRowToBook(row) {
  return {
    id: row.id,
    title: row.title,
    author: row.author,
    isbn: row.isbn,
    userId: row.user_id,
    createdAt: row.created_at,
    updatedAt: row.updated_at
  };
}

// ============================================
// USER REPOSITORY (userRepository.js)
// ============================================
async function incrementBookCount(userId) {
  const query = 'UPDATE users SET book_count = book_count + 1 WHERE id = ?';
  await db.query(query, [userId]);
}
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

```javascript
// Every handler has duplicated code
app.get('/api/books', (req, res) => {
  // Check authentication - DUPLICATED
  if (!req.headers.authorization) {
    return res.status(401).json({ error: 'Unauthorized' });
  }
  
  // Log request - DUPLICATED
  console.log(`${req.method} ${req.path}`);
  
  // CORS headers - DUPLICATED
  res.setHeader('Access-Control-Allow-Origin', 'https://example.com');
  
  // Actual logic
  const books = await getBooks();
  res.json(books);
});

app.post('/api/books', (req, res) => {
  // Same authentication - DUPLICATED
  if (!req.headers.authorization) {
    return res.status(401).json({ error: 'Unauthorized' });
  }
  
  // Same logging - DUPLICATED
  console.log(`${req.method} ${req.path}`);
  
  // Same CORS - DUPLICATED
  res.setHeader('Access-Control-Allow-Origin', 'https://example.com');
  
  // Actual logic
  const book = await createBook(req.body);
  res.json(book);
});

// ... repeated for 100s of endpoints
```

#### Solution With Middleware

```javascript
// Define once, use everywhere
app.use(corsMiddleware);
app.use(loggingMiddleware);
app.use(authMiddleware);

// Handlers focus only on their specific logic
app.get('/api/books', async (req, res) => {
  const books = await getBooks();
  res.json(books);
});

app.post('/api/books', async (req, res) => {
  const book = await createBook(req.body);
  res.json(book);
});
```

### How Middleware Works

Every middleware receives three parameters:

```javascript
function middleware(req, res, next) {
  // req: Request object (can read and modify)
  // res: Response object (can send response)
  // next: Function to pass control to next middleware
  
  // Do something...
  
  next(); // Pass to next middleware
}
```

#### The `next()` Function

The `next()` function is crucial - it controls the flow:

```javascript
function middleware1(req, res, next) {
  console.log('MW1: Before next()');
  next(); // Pass control to next middleware
  console.log('MW1: After next()'); // Executes after downstream completes
}

function middleware2(req, res, next) {
  console.log('MW2: Before next()');
  next();
  console.log('MW2: After next()');
}

function handler(req, res) {
  console.log('Handler executing');
  res.send('Done');
}

// Order of execution:
// MW1: Before next()
// MW2: Before next()
// Handler executing
// MW2: After next()
// MW1: After next()
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

```javascript
function authMiddleware(req, res, next) {
  const token = req.headers.authorization;
  
  if (!token) {
    // Don't call next() - stop here and send response
    return res.status(401).json({ error: 'No token provided' });
  }
  
  // Token exists, continue to next middleware
  next();
}
```

### Common Middleware Types

#### 1. CORS Middleware

**What it does:** Adds headers to allow cross-origin requests

**Why:** Browsers block requests from different origins by default (security)

**When:** First middleware (before routing)

```javascript
function corsMiddleware(req, res, next) {
  const allowedOrigins = ['https://example.com', 'https://app.example.com'];
  const origin = req.headers.origin;
  
  if (allowedOrigins.includes(origin)) {
    res.setHeader('Access-Control-Allow-Origin', origin);
    res.setHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, OPTIONS');
    res.setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization');
    res.setHeader('Access-Control-Allow-Credentials', 'true');
  }
  
  // Handle preflight requests
  if (req.method === 'OPTIONS') {
    return res.status(204).send();
  }
  
  next();
}

// Usage
app.use(corsMiddleware);
```

#### 2. Logging Middleware

**What it does:** Logs request details for debugging and monitoring

**Why:** Track all incoming requests, debug issues, audit trail

**When:** Early in the chain (after CORS)

```javascript
function loggingMiddleware(req, res, next) {
  const start = Date.now();
  
  // Log request
  console.log(`[${new Date().toISOString()}] ${req.method} ${req.path}`);
  console.log('Query params:', req.query);
  console.log('Body:', req.body);
  
  // Capture response time after response is sent
  res.on('finish', () => {
    const duration = Date.now() - start;
    console.log(`[${new Date().toISOString()}] ${req.method} ${req.path} - ${res.statusCode} - ${duration}ms`);
  });
  
  next();
}

// Usage
app.use(loggingMiddleware);
```

**Advanced Logging Example:**

```javascript
const winston = require('winston');

function advancedLoggingMiddleware(req, res, next) {
  const requestId = req.context.requestId; // From earlier middleware
  const start = Date.now();
  
  const logData = {
    requestId,
    method: req.method,
    path: req.path,
    ip: req.ip,
    userAgent: req.headers['user-agent'],
    userId: req.context?.userId || 'anonymous'
  };
  
  winston.info('Incoming request', logData);
  
  res.on('finish', () => {
    winston.info('Request completed', {
      ...logData,
      statusCode: res.statusCode,
      duration: Date.now() - start
    });
  });
  
  next();
}
```

#### 3. Authentication Middleware

**What it does:** Verifies user credentials/tokens

**Why:** Protect endpoints from unauthorized access

**When:** After logging, before handlers

```javascript
const jwt = require('jsonwebtoken');

function authMiddleware(req, res, next) {
  // 1. Extract token
  const authHeader = req.headers.authorization;
  
  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'No token provided' });
  }
  
  const token = authHeader.substring(7); // Remove 'Bearer '
  
  try {
    // 2. Verify token
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    
    // 3. Add user info to request context
    req.context = req.context || {};
    req.context.userId = decoded.userId;
    req.context.userRole = decoded.role;
    req.context.permissions = decoded.permissions;
    
    // 4. Continue to next middleware
    next();
    
  } catch (error) {
    // Token invalid or expired
    return res.status(401).json({ error: 'Invalid token' });
  }
}

// Usage - apply to specific routes
app.use('/api/protected/*', authMiddleware);

// Or apply to specific endpoints
app.post('/api/books', authMiddleware, createBookHandler);
```

**Session-based Authentication:**

```javascript
function sessionAuthMiddleware(req, res, next) {
  const sessionId = req.cookies.sessionId;
  
  if (!sessionId) {
    return res.status(401).json({ error: 'Not authenticated' });
  }
  
  // Verify session in database or Redis
  const session = await sessionStore.get(sessionId);
  
  if (!session || session.expiresAt < Date.now()) {
    return res.status(401).json({ error: 'Session expired' });
  }
  
  // Add user info to context
  req.context = req.context || {};
  req.context.userId = session.userId;
  req.context.sessionId = sessionId;
  
  next();
}
```

#### 4. Rate Limiting Middleware

**What it does:** Limits number of requests from a client

**Why:** Prevent abuse, DDoS attacks, ensure fair usage

**When:** Early in the chain

```javascript
const rateLimit = require('express-rate-limit');

// Simple rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Max 100 requests per window
  message: 'Too many requests, please try again later',
  standardHeaders: true, // Return rate limit info in `RateLimit-*` headers
  legacyHeaders: false
});

app.use('/api/', limiter);
```

**Custom Rate Limiting:**

```javascript
const requestCounts = new Map(); // In production, use Redis

function customRateLimitMiddleware(req, res, next) {
  const clientIp = req.ip;
  const now = Date.now();
  const windowMs = 60 * 1000; // 1 minute
  const maxRequests = 30;
  
  // Get or create client record
  let clientData = requestCounts.get(clientIp) || { count: 0, resetTime: now + windowMs };
  
  // Reset if window expired
  if (now > clientData.resetTime) {
    clientData = { count: 0, resetTime: now + windowMs };
  }
  
  // Increment count
  clientData.count++;
  requestCounts.set(clientIp, clientData);
  
  // Check limit
  if (clientData.count > maxRequests) {
    return res.status(429).json({
      error: 'Too many requests',
      retryAfter: Math.ceil((clientData.resetTime - now) / 1000)
    });
  }
  
  // Add rate limit headers
  res.setHeader('X-RateLimit-Limit', maxRequests);
  res.setHeader('X-RateLimit-Remaining', maxRequests - clientData.count);
  res.setHeader('X-RateLimit-Reset', new Date(clientData.resetTime).toISOString());
  
  next();
}
```

#### 5. Request ID Middleware

**What it does:** Assigns unique ID to each request

**Why:** Track requests across services, debugging, logging correlation

**When:** Very early (first or second)

```javascript
const { v4: uuidv4 } = require('uuid');

function requestIdMiddleware(req, res, next) {
  // Generate or use existing request ID
  const requestId = req.headers['x-request-id'] || uuidv4();
  
  // Store in context
  req.context = req.context || {};
  req.context.requestId = requestId;
  
  // Add to response headers
  res.setHeader('X-Request-ID', requestId);
  
  next();
}

// Usage
app.use(requestIdMiddleware);

// Now all logs and errors can include this ID
function loggingMiddleware(req, res, next) {
  console.log(`[${req.context.requestId}] ${req.method} ${req.path}`);
  next();
}
```

#### 6. Security Headers Middleware

**What it does:** Adds security-related HTTP headers

**Why:** Protect against common web vulnerabilities

**When:** Early in the chain

```javascript
function securityHeadersMiddleware(req, res, next) {
  // Prevent clickjacking
  res.setHeader('X-Frame-Options', 'DENY');
  
  // Prevent MIME type sniffing
  res.setHeader('X-Content-Type-Options', 'nosniff');
  
  // Enable XSS protection
  res.setHeader('X-XSS-Protection', '1; mode=block');
  
  // Content Security Policy
  res.setHeader('Content-Security-Policy', "default-src 'self'");
  
  // Strict Transport Security (HTTPS only)
  res.setHeader('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
  
  // Referrer Policy
  res.setHeader('Referrer-Policy', 'strict-origin-when-cross-origin');
  
  next();
}

// Or use helmet library
const helmet = require('helmet');
app.use(helmet());
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

**When:** LAST middleware (catches all errors from upstream)

```javascript
// Error handling middleware has 4 parameters (err, req, res, next)
function errorHandlerMiddleware(err, req, res, next) {
  // Log error
  console.error('Error:', {
    requestId: req.context?.requestId,
    error: err.message,
    stack: err.stack,
    path: req.path,
    method: req.method
  });
  
  // Client errors (4xx)
  if (err.isBusinessError || err.name === 'ValidationError') {
    return res.status(err.statusCode || 400).json({
      error: err.message,
      code: err.code,
      requestId: req.context?.requestId
    });
  }
  
  // Authentication errors
  if (err.name === 'UnauthorizedError' || err.statusCode === 401) {
    return res.status(401).json({
      error: 'Unauthorized',
      requestId: req.context?.requestId
    });
  }
  
  // Permission errors
  if (err.name === 'ForbiddenError' || err.statusCode === 403) {
    return res.status(403).json({
      error: 'Forbidden',
      requestId: req.context?.requestId
    });
  }
  
  // Not found errors
  if (err.name === 'NotFoundError' || err.statusCode === 404) {
    return res.status(404).json({
      error: 'Resource not found',
      requestId: req.context?.requestId
    });
  }
  
  // Server errors (5xx) - don't expose internal details
  res.status(500).json({
    error: 'Internal server error',
    requestId: req.context?.requestId
    // Don't send stack trace or detailed error message in production
  });
}

// Usage - MUST be last
app.use(errorHandlerMiddleware);
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

```javascript
const Joi = require('joi');

// Create validation middleware factory
function validate(schema) {
  return (req, res, next) => {
    const { error, value } = schema.validate(req.body, {
      abortEarly: false, // Collect all errors
      stripUnknown: true // Remove unknown fields
    });
    
    if (error) {
      const validationErrors = error.details.map(detail => ({
        field: detail.path.join('.'),
        message: detail.message
      }));
      
      return res.status(400).json({
        error: 'Validation failed',
        validationErrors
      });
    }
    
    // Replace req.body with validated and transformed data
    req.body = value;
    next();
  };
}

// Define schemas
const createBookSchema = Joi.object({
  title: Joi.string().min(3).max(200).required(),
  author: Joi.string().min(2).max(100).required(),
  isbn: Joi.string().pattern(/^978-\d{10}$/).optional(),
  publishedYear: Joi.number().integer().min(1000).max(new Date().getFullYear()).optional()
});

// Use in route
app.post('/api/books', validate(createBookSchema), createBookHandler);
```

### Middleware Ordering

**The order matters!** Middleware execute in the order they are defined.

#### ❌ Wrong Order

```javascript
// WRONG - handler won't have access to parsed body
app.post('/api/books', createBookHandler);
app.use(express.json()); // Too late!

// WRONG - authentication happens after handler
app.get('/api/protected', protectedHandler);
app.use(authMiddleware); // Too late!
```

#### ✅ Correct Order

```javascript
// 1. Request ID (first - needed for logging)
app.use(requestIdMiddleware);

// 2. CORS (early - might terminate request)
app.use(corsMiddleware);

// 3. Security headers
app.use(helmet());

// 4. Logging (after request ID and CORS)
app.use(loggingMiddleware);

// 5. Body parsing (before handlers need req.body)
app.use(express.json());

// 6. Rate limiting
app.use(rateLimitMiddleware);

// 7. Compression (can be early or late)
app.use(compression());

// 8. Routes and handlers
app.use('/api/public', publicRoutes); // No auth needed

app.use('/api/protected', authMiddleware); // Auth for protected routes
app.use('/api/protected', protectedRoutes);

// 9. 404 handler (after all routes)
app.use((req, res) => {
  res.status(404).json({ error: 'Route not found' });
});

// 10. Error handler (LAST - catches all errors)
app.use(errorHandlerMiddleware);
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

```javascript
// Authentication middleware extracts user ID
function authMiddleware(req, res, next) {
  const userId = verifyToken(req.headers.authorization);
  // How do we pass this to the handler?
  next();
}

// Handler needs user ID - but how to get it?
function createBookHandler(req, res) {
  const userId = ???; // Where does this come from?
  // ...
}
```

**Bad Solutions:**

```javascript
// ❌ Global variable - breaks with concurrent requests
let currentUserId;

function authMiddleware(req, res, next) {
  currentUserId = verifyToken(req.headers.authorization);
  next();
}

// ❌ Attach to req directly - can conflict with Express properties
function authMiddleware(req, res, next) {
  req.userId = verifyToken(req.headers.authorization);
  next();
}

// ❌ Pass as parameter - tightly couples components
function authMiddleware(req, res, next) {
  const userId = verifyToken(req.headers.authorization);
  next(userId); // next() doesn't work this way
}
```

#### ✅ Solution: Request Context

```javascript
// Authentication middleware stores user in context
function authMiddleware(req, res, next) {
  const userId = verifyToken(req.headers.authorization);
  
  // Create or use existing context
  req.context = req.context || {};
  req.context.userId = userId;
  req.context.userRole = 'user';
  
  next();
}

// Handler retrieves user from context
function createBookHandler(req, res) {
  const userId = req.context.userId; // Available!
  const userRole = req.context.userRole; // Available!
  // ...
}
```

### How It Works

Request context is typically implemented as a **key-value store** attached to the request object.

#### Basic Implementation (Node.js)

```javascript
// Context initialization middleware
function initContextMiddleware(req, res, next) {
  req.context = {
    requestId: generateRequestId(),
    startTime: Date.now()
  };
  next();
}

// Use context in other middleware
function authMiddleware(req, res, next) {
  const token = req.headers.authorization;
  const decoded = jwt.verify(token, SECRET);
  
  // Add to context
  req.context.userId = decoded.userId;
  req.context.userRole = decoded.role;
  req.context.permissions = decoded.permissions;
  
  next();
}

// Use context in handlers
function createBookHandler(req, res) {
  const { userId, userRole } = req.context;
  // Use the data
}
```

#### Implementation in Different Languages

**Node.js (Express):**
```javascript
// Simple object attached to req
req.context = {
  requestId: 'abc-123',
  userId: 'user-456'
};
```

**Go (using context package):**
```go
import "context"

// Create context with value
ctx := context.WithValue(r.Context(), "userId", userID)

// Retrieve value
userId := ctx.Value("userId").(string)
```

**Python (FastAPI):**
```python
from starlette.middleware.base import BaseHTTPMiddleware

class ContextMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request, call_next):
        request.state.user_id = "user-123"
        response = await call_next(request)
        return response

# Access in handler
@app.get("/books")
async def get_books(request: Request):
    user_id = request.state.user_id
```

### Common Use Cases

#### 1. User Authentication Data

Store authenticated user information for use throughout the request.

```javascript
// Auth middleware
function authMiddleware(req, res, next) {
  const token = req.headers.authorization?.replace('Bearer ', '');
  
  if (!token) {
    return res.status(401).json({ error: 'No token' });
  }
  
  try {
    const decoded = jwt.verify(token, SECRET);
    
    // Store in context
    req.context = req.context || {};
    req.context.userId = decoded.userId;
    req.context.userEmail = decoded.email;
    req.context.userRole = decoded.role;
    req.context.permissions = decoded.permissions || [];
    req.context.isAuthenticated = true;
    
    next();
  } catch (error) {
    res.status(401).json({ error: 'Invalid token' });
  }
}

// Handler uses auth data
async function createBookHandler(req, res) {
  const { userId, userRole } = req.context;
  
  const book = {
    ...req.body,
    userId, // From context, not from client (security!)
    createdAt: new Date()
  };
  
  await bookRepository.insert(book);
  res.status(201).json(book);
}

// Another handler checks permissions
async function deleteBookHandler(req, res) {
  const { userId, userRole, permissions } = req.context;
  
  if (userRole !== 'admin' && !permissions.includes('delete:books')) {
    return res.status(403).json({ error: 'Forbidden' });
  }
  
  await bookRepository.delete(req.params.id);
  res.status(204).send();
}
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

```javascript
// ============================================
// 1. REQUEST ARRIVES
// ============================================

// 2. Request ID Middleware
function requestIdMiddleware(req, res, next) {
  req.context = { requestId: uuidv4() };
  res.setHeader('X-Request-ID', req.context.requestId);
  next(); // → Continue to next middleware
}

// 3. CORS Middleware
function corsMiddleware(req, res, next) {
  const origin = req.headers.origin;
  if (origin === 'https://example.com') {
    res.setHeader('Access-Control-Allow-Origin', origin);
  }
  next(); // → Continue
}

// 4. Logging Middleware
function loggingMiddleware(req, res, next) {
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
```javascript
async function createBookHandler(req, res) {
  const { title, author, isbn } = req.body;
  
  // Business logic in handler - WRONG
  if (isbn && isbn.length !== 13) {
    return res.status(400).json({ error: 'Invalid ISBN' });
  }
  
  // Database operation in handler - WRONG
  await db.query('INSERT INTO books ...', [title, author, isbn]);
  
  res.status(201).json({ message: 'Created' });
}
```

✅ **Correct:**
```javascript
async function createBookHandler(req, res) {
  const book = await bookService.createBook(req.body);
  res.status(201).json(book);
}

async function createBook(data) {
  // Business logic in service - CORRECT
  if (data.isbn && !isValidISBN(data.isbn)) {
    throw new ValidationError('Invalid ISBN');
  }
  
  // Database via repository - CORRECT
  return await bookRepository.insert(data);
}
```

### 2. Service Layer Knowing About HTTP

❌ **Wrong:**
```javascript
async function createBook(req, res) {
  // Service should not have req, res parameters
  const book = { ...req.body };
  
  await bookRepository.insert(book);
  
  // Service should not send responses
  res.status(201).json(book);
}
```

✅ **Correct:**
```javascript
async function createBook(data) {
  // Just returns data, no HTTP
  const book = { ...data, id: generateId() };
  await bookRepository.insert(book);
  return book;
}
```

### 3. Repository Containing Business Logic

❌ **Wrong:**
```javascript
async function insert(book) {
  // Business validation in repository - WRONG
  if (book.publishedYear > new Date().getFullYear()) {
    throw new Error('Future year not allowed');
  }
  
  // Complex business logic - WRONG
  if (book.isbn) {
    const existing = await this.findByISBN(book.isbn);
    if (existing) {
      throw new Error('Duplicate');
    }
  }
  
  await db.query('INSERT ...');
}
```

✅ **Correct:**
```javascript
// Repository just does database operations
async function insert(book) {
  const query = 'INSERT INTO books (id, title, author) VALUES (?, ?, ?)';
  await db.query(query, [book.id, book.title, book.author]);
}

// Business logic in service
async function createBook(data) {
  if (data.publishedYear > new Date().getFullYear()) {
    throw new ValidationError('Future year not allowed');
  }
  
  if (data.isbn) {
    const existing = await bookRepository.findByISBN(data.isbn);
    if (existing) {
      throw new DuplicateError('ISBN already exists');
    }
  }
  
  await bookRepository.insert(data);
}
```

### 4. Wrong Middleware Order

❌ **Wrong:**
```javascript
// Auth before body parser - won't work
app.use(authMiddleware);
app.use(express.json());

// Error handler not last - won't catch errors
app.use(errorHandler);
app.use('/api/books', bookRoutes);
```

✅ **Correct:**
```javascript
app.use(express.json()); // Parse body first
app.use(authMiddleware); // Then auth
// ... routes
app.use(errorHandler); // Error handler LAST
```

### 5. Not Handling Errors Properly

❌ **Wrong:**
```javascript
async function handler(req, res) {
  const book = await bookService.createBook(req.body);
  res.json(book);
  // No try-catch - unhandled promise rejection
}
```

✅ **Correct:**
```javascript
async function handler(req, res, next) {
  try {
    const book = await bookService.createBook(req.body);
    res.json(book);
  } catch (error) {
    next(error); // Pass to error handler
  }
}
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
2. **Always handle errors** - use try-catch or .catch()
3. **Validate early** - reject bad data immediately
4. **Use appropriate HTTP status codes**
5. **Don't trust client data** - always validate
6. **Extract user info from context** - never from request body

```javascript
async function createBookHandler(req, res, next) {
  try {
    // 1. Extract data
    const data = req.body;
    const userId = req.context.userId; // From auth, not from client
    
    // 2. Basic validation
    if (!data.title) {
      return res.status(400).json({ error: 'Title required' });
    }
    
    // 3. Call service
    const book = await bookService.createBook({ ...data, userId });
    
    // 4. Send appropriate response
    res.status(201).json(book);
    
  } catch (error) {
    next(error); // Let error handler deal with it
  }
}
```

### Service Layer Best Practices

1. **No HTTP dependencies** - should work in CLI, cron jobs, etc.
2. **Single responsibility** - one method does one thing
3. **Throw meaningful errors** - with error codes
4. **Orchestrate operations** - coordinate repositories
5. **Handle transactions** - ensure data consistency

```javascript
class BookService {
  async createBook({ title, author, isbn, userId }) {
    // Business validation
    this.validateBookData({ title, author, isbn });
    
    // Check duplicates
    await this.ensureNoDuplicate(isbn);
    
    // Create book
    const book = {
      id: generateId(),
      title: title.trim(),
      author: author.trim(),
      isbn,
      userId,
      createdAt: new Date()
    };
    
    // Persist
    await bookRepository.insert(book);
    
    // Side effects
    await this.updateUserStats(userId);
    
    return book;
  }
  
  validateBookData(data) {
    if (!data.title || data.title.length < 3) {
      throw new ValidationError('Title must be at least 3 characters');
    }
    // ...
  }
  
  async ensureNoDuplicate(isbn) {
    if (!isbn) return;
    
    const existing = await bookRepository.findByISBN(isbn);
    if (existing) {
      throw new DuplicateError('Book with this ISBN already exists');
    }
  }
}
```

### Repository Best Practices

1. **Single purpose** - one method = one query
2. **No business logic** - just database operations
3. **Consistent return types** - always return same type
4. **Use parameterized queries** - prevent SQL injection
5. **Map database rows to objects** - encapsulate mapping logic

```javascript
class BookRepository {
  async findById(id) {
    const query = 'SELECT * FROM books WHERE id = ? AND deleted_at IS NULL';
    const rows = await db.query(query, [id]);
    return rows.length > 0 ? this.mapRow(rows[0]) : null;
  }
  
  async findByISBN(isbn) {
    const query = 'SELECT * FROM books WHERE isbn = ? AND deleted_at IS NULL';
    const rows = await db.query(query, [isbn]);
    return rows.length > 0 ? this.mapRow(rows[0]) : null;
  }
  
  async insert(book) {
    const query = `
      INSERT INTO books (id, title, author, isbn, user_id, created_at)
      VALUES (?, ?, ?, ?, ?, ?)
    `;
    
    await db.query(query, [
      book.id,
      book.title,
      book.author,
      book.isbn,
      book.userId,
      book.createdAt
    ]);
    
    return book;
  }
  
  mapRow(row) {
    return {
      id: row.id,
      title: row.title,
      author: row.author,
      isbn: row.isbn,
      userId: row.user_id,
      createdAt: row.created_at
    };
  }
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

```javascript
// ===== MIDDLEWARE =====
app.use(initContext);
app.use(cors);
app.use(express.json());
app.use(logging);

// ===== ROUTES =====
app.post('/api/books', 
  authMiddleware,
  validate(createBookSchema),
  createBookHandler
);

// ===== HANDLER =====
async function createBookHandler(req, res, next) {
  try {
    const { userId } = req.context;
    const book = await bookService.createBook({ ...req.body, userId });
    res.status(201).json(book);
  } catch (error) {
    next(error);
  }
}

// ===== SERVICE =====
async function createBook(data) {
  if (data.isbn) {
    const exists = await bookRepository.findByISBN(data.isbn);
    if (exists) throw new DuplicateError('ISBN exists');
  }
  
  const book = { id: generateId(), ...data, createdAt: new Date() };
  await bookRepository.insert(book);
  return book;
}

// ===== REPOSITORY =====
async function insert(book) {
  const query = 'INSERT INTO books (id, title, author) VALUES (?, ?, ?)';
  await db.query(query, [book.id, book.title, book.author]);
  return book;
}

// ===== ERROR HANDLER =====
app.use((err, req, res, next) => {
  const status = err.statusCode || 500;
  res.status(status).json({ 
    error: err.message,
    requestId: req.context.requestId 
  });
});
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
