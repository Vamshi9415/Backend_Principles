# Complete REST API Design: A Comprehensive Guide

## Table of Contents

1. [Introduction](#introduction)
2. [The History of REST](#the-history-of-rest)
   - [The Birth of the Web](#the-birth-of-the-web)
   - [The Scalability Crisis](#the-scalability-crisis)
   - [Roy Fielding's Constraints](#roy-fieldings-constraints)
   - [What Does REST Mean?](#what-does-rest-mean)
3. [Understanding URLs and API Routes](#understanding-urls-and-api-routes)
   - [URL Structure Breakdown](#url-structure-breakdown)
   - [API URL Patterns](#api-url-patterns)
   - [Resource Naming Conventions](#resource-naming-conventions)
4. [HTTP Methods and Idempotency](#http-methods-and-idempotency)
   - [What is Idempotency?](#what-is-idempotency)
   - [GET - Idempotent](#get---idempotent)
   - [POST - Non-Idempotent](#post---non-idempotent)
   - [PUT & PATCH - Idempotent](#put--patch---idempotent)
   - [DELETE - Idempotent](#delete---idempotent)
5. [Designing CRUD APIs](#designing-crud-apis)
   - [Create (POST)](#create-post)
   - [List/Read All (GET)](#listread-all-get)
   - [Read Single (GET)](#read-single-get)
   - [Update (PATCH/PUT)](#update-patchput)
   - [Delete (DELETE)](#delete-delete)
6. [Advanced API Features](#advanced-api-features)
   - [Pagination](#pagination)
   - [Filtering](#filtering)
   - [Sorting](#sorting)
7. [Custom Actions](#custom-actions)
   - [When to Use Custom Actions](#when-to-use-custom-actions)
   - [Designing Custom Action Endpoints](#designing-custom-action-endpoints)
8. [HTTP Status Codes](#http-status-codes)
9. [Best Practices](#best-practices)
10. [Common Pitfalls & Debugging](#common-pitfalls--debugging)
11. [Quick Revision / Cheat Sheet](#quick-revision--cheat-sheet)

---

## Introduction

API design is one of the most critical skills for a backend engineer. Whether you're building internal services, public APIs, or microservices, the quality of your API design directly impacts:

- **Developer Experience**: How easy is it for others to integrate your API?
- **Maintainability**: Can you and your team understand and extend the API later?
- **Performance**: Are your APIs efficient and scalable?
- **Consistency**: Do your APIs follow predictable patterns?

This guide focuses on **REST APIs** (Representational State Transfer), one of the most widely used API standards in the industry. While other technologies exist (gRPC, GraphQL, etc.), REST remains the foundation of modern web services.

### Why Standards Matter

Even experienced backend engineers often face questions like:
- Should the URI path be singular or plural? (`/book` vs `/books`)
- When updating, should I use PATCH or PUT?
- What status code should I return for different scenarios?
- How do I handle non-CRUD operations?

The reason these questions persist is that the standards were developed when the web was very different. We've evolved from **Multi-Page Applications (MPAs)** to **Single-Page Applications (SPAs)**, but the core principles remain valuable.

**The goal of this guide** is to help you:
1. Extract clear rules from existing REST standards
2. Make consistent design decisions
3. Focus on business logic instead of worrying about conventions
4. Create delightful, intuitive APIs for consumers

> **Remember**: A good API is designed first, then implemented. Design is a separate phase that happens before writing code.

---

## The History of REST

Understanding where REST came from gives you critical context for why we follow certain patterns today.

### The Birth of the Web

In **1990**, Tim Berners-Lee started a project called the **World Wide Web** to share knowledge globally. Within about a year, he invented:

1. **URI** (Uniform Resource Identifier) - A way to identify resources
2. **HTTP** (HyperText Transfer Protocol) - The communication protocol
3. **HTML** (HyperText Markup Language) - The page structure format
4. **The first web server** - Software to serve content
5. **The first web browser** - Software to consume content
6. **The first WYSIWYG HTML editor** - Built into the browser

These technologies, in more advanced forms, are still the foundation of the web today.

### The Scalability Crisis

The World Wide Web grew exponentially fast. The problem? Tim Berners-Lee hadn't anticipated this massive scale when designing the initial system. The web was heading toward a breakdown due to:

- Exponential user growth
- Insufficient architectural planning for scale
- No standardized patterns for scalability

New techniques and standards were desperately needed.

### Roy Fielding's Constraints

Around **1993**, **Roy Fielding** (co-founder of the Apache HTTP Server project) became concerned about the web's scalability problem. To address this, he proposed **six architectural constraints** that would make the web more scalable:

#### 1. **Client-Server**
- **What it is**: Separation of concerns between client (UI/UX) and server (data/business logic)
- **Why it exists**: Allows each component to evolve independently
- **Benefit**: Better scalability - frontend and backend can scale separately

```
┌─────────────┐           ┌─────────────┐
│   CLIENT    │◄─────────►│   SERVER    │
│             │   HTTP    │             │
│  UI/UX      │           │ Data/Logic  │
└─────────────┘           └─────────────┘
```

#### 2. **Uniform Interface**
- **What it is**: Standardized way for components to communicate
- **Why it exists**: Simplifies the overall system architecture
- **Four sub-constraints**:
  - Resource identification
  - Resource manipulation through representations
  - Self-descriptive messages
  - Hypermedia as the engine of application state (HATEOAS)

#### 3. **Layered System**
- **What it is**: Architecture composed of hierarchical layers
- **How it works**: Each layer only interacts with the immediate layer below it
- **Benefit**: Can add load balancers, proxies, caches without affecting core functionality

```
Client → Load Balancer → API Gateway → Server → Database
  ↓         ↓              ↓             ↓          ↓
Layer 1   Layer 2        Layer 3      Layer 4   Layer 5
```

#### 4. **Cache**
- **What it is**: Responses must be labeled as cacheable or non-cacheable
- **Why it exists**: Reduces server load and network traffic
- **Benefit**: Faster response times and better user experience

#### 5. **Stateless**
- **What it is**: Each request must contain ALL information needed to process it
- **Why it exists**: Server doesn't remember previous requests
- **Benefit**: Any server can handle any request (horizontal scalability)

**Real-world analogy**: Imagine calling customer support. In a stateless system, every time you call, you must re-introduce yourself and explain your issue from scratch. The support agent doesn't remember your previous call. This seems inconvenient, but it means ANY agent can help you, not just the one you spoke to before.

```
Request 1: GET /user/123 (includes auth token)
Request 2: GET /user/123 (includes auth token again)
↓
Server doesn't remember Request 1 when processing Request 2
```

#### 6. **Code on Demand** (Optional)
- **What it is**: Servers can send executable code (like JavaScript) to clients
- **Why it's optional**: Provides flexibility without being mandatory

### The Impact

In 2000, after helping solve the web's scalability crisis, Roy Fielding named and described these architectural principles in his **PhD dissertation**: **"Architectural Styles and the Design of Network-based Software Architectures"**.

He called this architectural style **REST** (Representational State Transfer).

> 📖 **Recommended Reading**: Search for "Roy Fielding REST dissertation" to read the original paper. It's essential reading for serious backend engineers.

### What Does REST Mean?

Let's break down the name **Representational State Transfer**:

#### **Representational**
Resources can be represented in different formats depending on client needs:

```
User Resource (Database)
    ↓
┌───────────────┬───────────────┬──────────────┐
│   JSON        │     HTML      │     XML      │
│ (API Client)  │ (Browser)     │ (Legacy)     │
└───────────────┴───────────────┴──────────────┘
```

**Example**: A `User` object can be:
- **JSON** for API consumers (other servers)
- **HTML** for web browsers
- **XML** for legacy systems

The same resource, different representations.

#### **State**
The current condition or attributes of a resource.

**Example**: A shopping cart's state includes:
- Items in the cart
- Quantities
- Total price
- User ID

This state is transferred between client and server.

#### **Transfer**
The movement of resource representations between client and server using HTTP.

```
Client                              Server
  │                                   │
  ├─── GET /books ──────────────────► │
  │                                   │
  │ ◄── 200 OK + [{book1}, {book2}]── │
  │                                   │
  ├─── POST /books + {new book} ────► │
  │                                   │
  │ ◄── 201 Created + {new book} ────┤
  │                                   │
```

### Putting It Together

**REST** describes an architectural style where:
1. Resources are represented in different formats (JSON, HTML, XML)
2. Resource state can be transferred between client and server
3. Client and server communicate by exchanging these representations
4. The system follows specific constraints to ensure scalability

---

## Understanding URLs and API Routes

### URL Structure Breakdown

A typical URL has the following structure:

```
https://api.example.com/v1/books/123?status=active#chapter-1
│      │  │              │   │     │   │             │
│      │  │              │   │     │   │             └─ Fragment
│      │  │              │   │     │   └─ Query Parameters
│      │  │              │   │     └─ Resource ID (dynamic)
│      │  │              │   └─ Resource Path
│      │  │              └─ Version
│      │  └─ Domain
│      └─ Subdomain
└─ Scheme
```

**Component Breakdown**:

| Component | Example | Purpose |
|-----------|---------|---------|
| **Scheme** | `https` | Protocol (HTTP/HTTPS) |
| **Subdomain** | `api` | Separates API from main site |
| **Domain** | `example.com` | Your organization's domain |
| **Version** | `/v1` | API version for backward compatibility |
| **Resource Path** | `/books` | Collection or resource type |
| **Resource ID** | `/123` | Specific resource identifier |
| **Query Parameters** | `?status=active` | Filters, pagination, etc. |
| **Fragment** | `#chapter-1` | Scroll to page section (rarely used in APIs) |

### API URL Patterns

Following industry standards, a well-structured API URL looks like:

```
https://api.example.com/v1/books
│      │  │              │   │
│      │  │              │   └─ Resource (PLURAL)
│      │  │              └─ Version
│      │  └─ Domain
│      └─ Subdomain (api)
└─ Scheme (https for production)
```

### Resource Naming Conventions

#### ✅ **Rule #1: Always Use Plural Form**

Resources in the path should ALWAYS be plural, even when fetching a single resource.

```
✅ CORRECT:
GET /books          → List all books
GET /books/123      → Get one book
POST /books         → Create a book

❌ WRONG:
GET /book           → Inconsistent
GET /book/123       → Violates standard
```

**Why plural?** 
- You're accessing the "books collection"
- Even for a single book, you're accessing one item FROM the collection
- Consistency across all endpoints

#### ✅ **Rule #2: Use Lowercase**

```
✅ CORRECT:  /books
❌ WRONG:    /Books or /BOOKS
```

**Why?** URLs travel through different operating systems and environments. Some are case-sensitive, others aren't. Lowercase eliminates ambiguity.

#### ✅ **Rule #3: Use Hyphens for Multi-Word Resources**

When you need multiple words, use hyphens (kebab-case):

```
✅ CORRECT:  /book-reviews
❌ WRONG:    /book_reviews (underscores)
❌ WRONG:    /bookReviews (camelCase)
❌ WRONG:    /book reviews (spaces - invalid)
```

#### ✅ **Rule #4: Hierarchical Relationships Use Forward Slashes**

The `/` character represents a hierarchical relationship:

```
/organizations              → All organizations
/organizations/5            → Organization #5
/organizations/5/projects   → All projects IN organization #5
/organizations/5/projects/3 → Project #3 IN organization #5
```

**Read it like**: "In organizations, find organization 5, then in its projects, find project 3"

### Slugs: Human-Readable Identifiers

Sometimes you want human-readable URLs instead of numeric IDs:

```
Numeric ID:  /books/12345
Slug:        /books/harry-potter-philosophers-stone
```

**How to create a slug** from "Harry Potter and the Philosopher's Stone":

1. Convert to lowercase: `harry potter and the philosopher's stone`
2. Replace spaces with hyphens: `harry-potter-and-the-philosopher's-stone`
3. Remove special characters: `harry-potter-and-the-philosophers-stone`

**Example Implementation**:

```javascript
function createSlug(text) {
  return text
    .toLowerCase()                // Lowercase
    .trim()                       // Remove whitespace
    .replace(/[^\w\s-]/g, '')     // Remove special chars
    .replace(/\s+/g, '-')         // Replace spaces with hyphens
    .replace(/--+/g, '-');        // Replace multiple hyphens with single
}

createSlug("Harry Potter & the Philosopher's Stone")
// Output: "harry-potter-the-philosophers-stone"
```

---

## HTTP Methods and Idempotency

### What is Idempotency?

**Idempotency** is a critical concept in REST API design.

**Definition**: An operation is idempotent if performing it multiple times has the same effect as performing it once.

**Real-world analogy**: 
- **Idempotent**: Turning on a light switch. Whether you flip it once or 100 times, the light is still on.
- **Non-idempotent**: Adding $10 to your bank account. Each time you do it, you add another $10.

**In REST APIs**: An HTTP method is idempotent if making the same request multiple times produces the same server-side state (not necessarily the same response).

```
┌─────────────────────────────────────────────────┐
│  Idempotent Methods: GET, PUT, PATCH, DELETE   │
│  Non-Idempotent: POST                          │
└─────────────────────────────────────────────────┘
```

### GET - Idempotent

**Purpose**: Retrieve data from the server

**Why it's idempotent**: Reading data doesn't change server state. You can call it 1 time or 1000 times - the server state remains unchanged.

```http
GET /books/123
GET /books/123
GET /books/123
↓
Server state unchanged (only reading)
```

**Example**:

```http
GET /api/books/123 HTTP/1.1
Host: api.example.com

Response:
200 OK
{
  "id": 123,
  "title": "Clean Code",
  "author": "Robert C. Martin"
}
```

**Characteristics**:
- ✅ Safe (no side effects)
- ✅ Idempotent
- ✅ Cacheable
- ❌ No request body (use query parameters instead)

### POST - Non-Idempotent

**Purpose**: Create new resources or perform custom actions

**Why it's non-idempotent**: Each call typically creates a new resource, changing server state differently each time.

```http
POST /books + {title: "New Book"}  → Creates book with ID 1
POST /books + {title: "New Book"}  → Creates book with ID 2
POST /books + {title: "New Book"}  → Creates book with ID 3
↓
Different side effects each time!
```

**Example**:

```http
POST /api/books HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
  "title": "Clean Code",
  "author": "Robert C. Martin",
  "isbn": "978-0132350884"
}

Response:
201 Created
Location: /api/books/123
{
  "id": 123,
  "title": "Clean Code",
  "author": "Robert C. Martin",
  "isbn": "978-0132350884",
  "createdAt": "2026-01-30T10:00:00Z"
}
```

**Characteristics**:
- ❌ Not safe (has side effects)
- ❌ Not idempotent
- ❌ Not cacheable
- ✅ Has request body

**When to use POST**:
1. Creating new resources
2. Custom actions that don't fit other methods
3. Operations that change server state unpredictably

### PUT & PATCH - Idempotent

Both are used for updates, but with different semantics.

#### **PUT: Complete Replacement**

**Purpose**: Replace the entire resource

**Why it's idempotent**: 
```http
PUT /users/1 {name: "John", age: 30}  → User 1: {name: "John", age: 30}
PUT /users/1 {name: "John", age: 30}  → User 1: {name: "John", age: 30}
PUT /users/1 {name: "John", age: 30}  → User 1: {name: "John", age: 30}
↓
Same result every time
```

**Example**:

```http
PUT /api/users/123 HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
  "id": 123,
  "name": "John Doe",
  "email": "john@example.com",
  "role": "admin"
}

Response:
200 OK
{
  "id": 123,
  "name": "John Doe",
  "email": "john@example.com",
  "role": "admin",
  "updatedAt": "2026-01-30T10:30:00Z"
}
```

**Note**: You must send ALL fields, even unchanged ones. Missing fields may be set to null or default values.

#### **PATCH: Partial Update**

**Purpose**: Update specific fields of a resource

**Why it's idempotent**:
```http
PATCH /users/1 {name: "John"}  → User 1 name: "John"
PATCH /users/1 {name: "John"}  → User 1 name: "John"
PATCH /users/1 {name: "John"}  → User 1 name: "John"
↓
Same result every time
```

**Example**:

```http
PATCH /api/users/123 HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
  "email": "newemail@example.com"
}

Response:
200 OK
{
  "id": 123,
  "name": "John Doe",
  "email": "newemail@example.com",  ← Only this changed
  "role": "admin",
  "updatedAt": "2026-01-30T10:45:00Z"
}
```

**PUT vs PATCH Comparison**:

| Aspect | PUT | PATCH |
|--------|-----|-------|
| **Updates** | Entire resource | Partial fields |
| **Payload** | All fields required | Only changed fields |
| **Missing fields** | Set to null/default | Unchanged |
| **Common usage** | Full replacement | Most updates |
| **Bandwidth** | Higher (all data) | Lower (only changes) |

**Best Practice**: Use PATCH for most update operations. Use PUT only when you truly want to replace the entire resource.

### DELETE - Idempotent

**Purpose**: Remove a resource

**Why it's idempotent**:

```http
DELETE /books/123  → Book 123 deleted (resource doesn't exist)
DELETE /books/123  → 404 Not Found (already gone, no state change)
DELETE /books/123  → 404 Not Found (already gone, no state change)
↓
After first call, state doesn't change
```

**Example**:

```http
DELETE /api/books/123 HTTP/1.1
Host: api.example.com

Response (first call):
204 No Content

Response (subsequent calls):
404 Not Found
{
  "error": "Book not found",
  "message": "Book with ID 123 does not exist"
}
```

**Characteristics**:
- ❌ Not safe (has side effects on first call)
- ✅ Idempotent (repeated calls don't change state further)
- ❌ No request body needed
- ✅ Returns 204 No Content on success
- ✅ Returns 404 Not Found if resource already deleted

### Idempotency Summary Table

| Method | Idempotent | Safe | Typical Use |
|--------|-----------|------|-------------|
| **GET** | ✅ Yes | ✅ Yes | Retrieve data |
| **POST** | ❌ No | ❌ No | Create resource, custom actions |
| **PUT** | ✅ Yes | ❌ No | Full replacement |
| **PATCH** | ✅ Yes | ❌ No | Partial update |
| **DELETE** | ✅ Yes | ❌ No | Remove resource |
| **HEAD** | ✅ Yes | ✅ Yes | Get headers only |
| **OPTIONS** | ✅ Yes | ✅ Yes | Check allowed methods |

---

## Designing CRUD APIs

CRUD stands for **Create, Read, Update, Delete** - the four basic operations for any resource.

Let's design a complete set of CRUD APIs for a `books` resource.

### Create (POST)

**Endpoint**: `POST /books`

**Purpose**: Create a new book

**Request**:
```http
POST /api/v1/books HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
  "title": "The Pragmatic Programmer",
  "author": "David Thomas",
  "isbn": "978-0135957059",
  "publishedYear": 2019,
  "price": 42.99
}
```

**Success Response**:
```http
201 Created
Location: /api/v1/books/456
Content-Type: application/json

{
  "id": 456,
  "title": "The Pragmatic Programmer",
  "author": "David Thomas",
  "isbn": "978-0135957059",
  "publishedYear": 2019,
  "price": 42.99,
  "createdAt": "2026-01-30T10:00:00Z",
  "updatedAt": "2026-01-30T10:00:00Z"
}
```

**Key Points**:
- ✅ Returns **201 Created** status code
- ✅ Includes `Location` header with new resource URL
- ✅ Returns the complete created object (including generated fields like `id`, `createdAt`)
- ✅ Route uses plural form (`/books`, not `/book`)

**Error Response**:
```http
400 Bad Request
{
  "error": "Validation Error",
  "message": "Invalid request data",
  "details": [
    {
      "field": "isbn",
      "message": "ISBN already exists"
    }
  ]
}
```

### List/Read All (GET)

**Endpoint**: `GET /books`

**Purpose**: Retrieve a list of all books (with pagination, filtering, sorting)

**Request**:
```http
GET /api/v1/books?page=1&limit=10&sortBy=createdAt&sortOrder=desc&status=published HTTP/1.1
Host: api.example.com
```

**Success Response**:
```http
200 OK
Content-Type: application/json

{
  "data": [
    {
      "id": 456,
      "title": "The Pragmatic Programmer",
      "author": "David Thomas",
      "status": "published",
      "createdAt": "2026-01-30T10:00:00Z"
    },
    {
      "id": 455,
      "title": "Clean Code",
      "author": "Robert C. Martin",
      "status": "published",
      "createdAt": "2026-01-29T15:30:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 42,
    "totalPages": 5
  }
}
```

**Key Points**:
- ✅ Returns **200 OK** status code
- ✅ Wraps results in a `data` field
- ✅ Includes pagination metadata
- ✅ Even if empty, returns `{"data": [], "pagination": {...}}`, NOT 404

**Empty Result** (NOT an error):
```http
200 OK
{
  "data": [],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 0,
    "totalPages": 0
  }
}
```

### Read Single (GET)

**Endpoint**: `GET /books/{id}`

**Purpose**: Retrieve a specific book

**Request**:
```http
GET /api/v1/books/456 HTTP/1.1
Host: api.example.com
```

**Success Response**:
```http
200 OK
Content-Type: application/json

{
  "id": 456,
  "title": "The Pragmatic Programmer",
  "author": "David Thomas",
  "isbn": "978-0135957059",
  "publishedYear": 2019,
  "price": 42.99,
  "status": "published",
  "createdAt": "2026-01-30T10:00:00Z",
  "updatedAt": "2026-01-30T10:00:00Z"
}
```

**Error Response** (Not Found):
```http
404 Not Found
Content-Type: application/json

{
  "error": "Not Found",
  "message": "Book with ID 456 does not exist"
}
```

**Key Points**:
- ✅ Returns **200 OK** for success
- ✅ Returns **404 Not Found** if resource doesn't exist
- ✅ Returns the complete resource object

### Update (PATCH/PUT)

#### PATCH - Partial Update (Recommended)

**Endpoint**: `PATCH /books/{id}`

**Purpose**: Update specific fields of a book

**Request**:
```http
PATCH /api/v1/books/456 HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
  "price": 39.99,
  "status": "on-sale"
}
```

**Success Response**:
```http
200 OK
Content-Type: application/json

{
  "id": 456,
  "title": "The Pragmatic Programmer",
  "author": "David Thomas",
  "isbn": "978-0135957059",
  "publishedYear": 2019,
  "price": 39.99,           ← Updated
  "status": "on-sale",      ← Updated
  "createdAt": "2026-01-30T10:00:00Z",
  "updatedAt": "2026-01-30T11:15:00Z"  ← Updated
}
```

**Key Points**:
- ✅ Returns **200 OK** status code
- ✅ Only send fields you want to update
- ✅ Returns the complete updated object
- ✅ Server updates `updatedAt` timestamp automatically

#### PUT - Full Replacement (Less Common)

**Endpoint**: `PUT /books/{id}`

**Request** (must include ALL fields):
```http
PUT /api/v1/books/456 HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
  "title": "The Pragmatic Programmer",
  "author": "David Thomas",
  "isbn": "978-0135957059",
  "publishedYear": 2019,
  "price": 39.99,
  "status": "on-sale"
}
```

**When to use PUT vs PATCH**:
- Use **PATCH** for most updates (partial field updates)
- Use **PUT** only when replacing the entire resource representation

### Delete (DELETE)

**Endpoint**: `DELETE /books/{id}`

**Purpose**: Delete a book

**Request**:
```http
DELETE /api/v1/books/456 HTTP/1.1
Host: api.example.com
```

**Success Response**:
```http
204 No Content
```

**Key Points**:
- ✅ Returns **204 No Content** (empty body)
- ✅ No response body needed
- ✅ Subsequent DELETE calls return 404

**Error Response** (Already Deleted):
```http
404 Not Found
{
  "error": "Not Found",
  "message": "Book with ID 456 does not exist"
}
```

### Complete CRUD Endpoint Summary

| Operation | Method | Endpoint | Success Status | Returns |
|-----------|--------|----------|----------------|---------|
| **Create** | POST | `/books` | 201 Created | Created object |
| **List All** | GET | `/books` | 200 OK | Array + pagination |
| **Get One** | GET | `/books/{id}` | 200 OK | Single object |
| **Update** | PATCH | `/books/{id}` | 200 OK | Updated object |
| **Replace** | PUT | `/books/{id}` | 200 OK | Replaced object |
| **Delete** | DELETE | `/books/{id}` | 204 No Content | Empty body |

---

## Advanced API Features

### Pagination

**What it is**: A technique to return large datasets in smaller, manageable chunks.

**Why it exists**: 
- Prevents overwhelming the network with massive payloads
- Reduces memory usage on both client and server
- Improves performance (serialization/deserialization is expensive)
- Better user experience (faster initial load)

**How it works**:

```
Total: 100 books in database

Page 1 (limit=10): Books 1-10
Page 2 (limit=10): Books 11-20
Page 3 (limit=10): Books 21-30
...
Page 10 (limit=10): Books 91-100
```

**Implementation**:

**Query Parameters**:
- `page`: Which page to retrieve (default: 1)
- `limit`: Number of items per page (default: 10 or 20)

**Request**:
```http
GET /api/v1/books?page=2&limit=10 HTTP/1.1
```

**Response**:
```json
{
  "data": [
    { "id": 11, "title": "Book 11" },
    { "id": 12, "title": "Book 12" },
    // ... 8 more books
  ],
  "pagination": {
    "page": 2,          // Current page
    "limit": 10,        // Items per page
    "total": 95,        // Total items in database
    "totalPages": 10    // Total pages (Math.ceil(95/10))
  }
}
```

**Pagination Metadata Explained**:

| Field | Description | Example |
|-------|-------------|---------|
| `page` | Current page number | `2` |
| `limit` | Items per page | `10` |
| `total` | Total items matching query | `95` |
| `totalPages` | Total pages available | `10` |

**Why include metadata?**
- Client can display "Showing 11-20 of 95 results"
- Client knows when to stop pagination (page === totalPages)
- Client can build page navigation UI

**Example - Pagination Calculation**:

```javascript
// Server-side logic
function paginateResults(allItems, page = 1, limit = 10) {
  const offset = (page - 1) * limit;
  const paginatedItems = allItems.slice(offset, offset + limit);
  
  return {
    data: paginatedItems,
    pagination: {
      page: page,
      limit: limit,
      total: allItems.length,
      totalPages: Math.ceil(allItems.length / limit)
    }
  };
}

// Usage
const books = [...]; // 95 books from database
const result = paginateResults(books, 2, 10);
// Returns books 11-20
```

**Default Behavior**:
```http
GET /api/v1/books  (no params)
↓
Default: page=1, limit=10
```

**Always provide defaults** so clients don't have to specify obvious parameters.

### Filtering

**What it is**: Allowing clients to narrow down results based on field values.

**Why it exists**: Users often want specific subsets of data (e.g., "only active users", "only published books").

**How it works**: Use query parameters with field names as keys.

**Examples**:

```http
# Filter by single field
GET /api/v1/books?status=published

# Filter by multiple fields
GET /api/v1/books?status=published&author=Martin+Fowler

# Combine with pagination
GET /api/v1/books?status=published&page=1&limit=10
```

**Response**:
```json
{
  "data": [
    {
      "id": 1,
      "title": "Refactoring",
      "author": "Martin Fowler",
      "status": "published"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 1,
    "totalPages": 1
  }
}
```

**Common Filter Patterns**:

| Filter Type | Example | Description |
|-------------|---------|-------------|
| **Exact match** | `?status=active` | Exact value |
| **Multiple values** | `?status=active,pending` | OR condition |
| **Date range** | `?createdAfter=2026-01-01` | Comparison |
| **Search** | `?search=pragmatic` | Text search |

**Advanced Filtering Example**:

```http
GET /api/v1/books?
  status=published&
  publishedYear=2023&
  minPrice=20&
  maxPrice=50&
  author=Martin
```

**Implementation Tip**: Document which fields support filtering in your API documentation.

### Sorting

**What it is**: Allowing clients to order results by specific fields.

**Why it exists**: Different views need different sort orders (newest first, alphabetical, by price, etc.).

**How it works**: Use `sortBy` and `sortOrder` query parameters.

**Parameters**:
- `sortBy`: Field name to sort by
- `sortOrder`: `asc` (ascending) or `desc` (descending)

**Examples**:

```http
# Sort by creation date, newest first (default)
GET /api/v1/books?sortBy=createdAt&sortOrder=desc

# Sort by title alphabetically
GET /api/v1/books?sortBy=title&sortOrder=asc

# Sort by price, highest first
GET /api/v1/books?sortBy=price&sortOrder=desc
```

**Default Sorting Behavior**:

Even if clients don't specify sorting, **always apply a default sort** to ensure consistent results:

```http
GET /api/v1/books
↓
Default: sortBy=createdAt, sortOrder=desc
```

**Why?** Database query results are not guaranteed to return in the same order. Sorting ensures consistency.

**Complete Example** (Pagination + Filtering + Sorting):

```http
GET /api/v1/books?
  status=published&           ← Filter
  author=Martin&              ← Filter
  page=2&                     ← Pagination
  limit=10&                   ← Pagination
  sortBy=title&               ← Sort
  sortOrder=asc               ← Sort

Response:
{
  "data": [
    // 10 published books by "Martin", sorted by title A-Z, page 2
  ],
  "pagination": {
    "page": 2,
    "limit": 10,
    "total": 25,
    "totalPages": 3
  }
}
```

**Implementation Example**:

```javascript
function buildQuery(filters) {
  const {
    page = 1,
    limit = 10,
    sortBy = 'createdAt',
    sortOrder = 'desc',
    ...otherFilters
  } = filters;
  
  let query = db.select('*').from('books');
  
  // Apply filters
  Object.entries(otherFilters).forEach(([key, value]) => {
    query = query.where(key, value);
  });
  
  // Apply sorting
  query = query.orderBy(sortBy, sortOrder);
  
  // Apply pagination
  const offset = (page - 1) * limit;
  query = query.limit(limit).offset(offset);
  
  return query;
}
```

**Sorting Best Practices**:
1. ✅ Always provide default sort (usually `createdAt desc`)
2. ✅ Document which fields support sorting
3. ✅ Validate `sortBy` field exists in schema
4. ✅ Validate `sortOrder` is either `asc` or `desc`

---

## Custom Actions

### When to Use Custom Actions

Sometimes, an operation doesn't fit the standard CRUD pattern. These are called **custom actions**.

**Examples of custom actions**:
- Sending an email
- Archiving a project
- Cloning a resource
- Approving a document
- Publishing a draft
- Exporting data

**How to identify a custom action**:
1. It's not a simple create/read/update/delete
2. It triggers complex business logic
3. Multiple operations happen server-side
4. Side effects beyond just updating fields

### Real-World Example: Archiving an Organization

**Naive approach** (just update status):
```http
PATCH /organizations/123
{
  "status": "archived"
}
```

**Problem**: Archiving isn't just changing a status field. It might involve:
- Soft-deleting all projects under the organization
- Removing all tasks in those projects
- Sending notifications to all users
- Triggering audit logs
- Updating billing systems

**Correct approach** (custom action):
```http
POST /organizations/123/archive
```

### Designing Custom Action Endpoints

**Pattern**:
```
POST /{resource}/{id}/{action}
```

**Examples**:

```http
# Archive an organization
POST /organizations/123/archive

# Clone a project
POST /projects/456/clone

# Send an email
POST /emails/send

# Publish a draft article
POST /articles/789/publish

# Approve a document
POST /documents/101/approve
```

**Complete Example - Clone Project**:

**Request**:
```http
POST /api/v1/projects/456/clone HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
  "newName": "Project Copy",
  "includeTasks": true
}
```

**Success Response**:
```http
201 Created
Location: /api/v1/projects/789
{
  "id": 789,
  "name": "Project Copy",
  "status": "draft",
  "clonedFrom": 456,
  "createdAt": "2026-01-30T12:00:00Z"
}
```

**Key Points**:
- ✅ Use **POST** method (open-ended for custom actions)
- ✅ Action name at the end of URL path
- ✅ Can accept request body if needed
- ✅ Returns appropriate status code (often 200 or 201)

**Status Code Guidelines for Custom Actions**:

| Scenario | Status Code |
|----------|-------------|
| Action creates a new resource | 201 Created |
| Action succeeds, returns data | 200 OK |
| Action succeeds, no response needed | 204 No Content |
| Action is async (queued) | 202 Accepted |

**Example - Archive Organization**:

**Request**:
```http
POST /api/v1/organizations/123/archive HTTP/1.1
```

**Response**:
```http
200 OK
{
  "id": 123,
  "name": "ACME Corp",
  "status": "archived",
  "archivedAt": "2026-01-30T12:30:00Z",
  "archivedBy": "user-456"
}
```

Behind the scenes, the server:
1. Updates organization status to "archived"
2. Soft-deletes all associated projects
3. Removes all tasks
4. Sends notification emails
5. Logs the action
6. Updates billing

**All of this from one API call** - that's the power of custom actions.

### Custom Action vs PATCH - Decision Tree

```
Does the operation involve ONLY updating fields?
  ├─ YES → Use PATCH
  └─ NO → Does it trigger complex business logic?
          └─ YES → Use custom action (POST /{id}/{action})
```

**Examples**:

| Operation | Method | Why |
|-----------|--------|-----|
| Change user email | `PATCH /users/1` | Simple field update |
| Change user password | `POST /users/1/change-password` | Requires hashing, validation, email notification |
| Update book price | `PATCH /books/1` | Simple field update |
| Publish a book | `POST /books/1/publish` | Changes status + triggers notifications + updates catalog |

---

## HTTP Status Codes

Choosing the right status code is crucial for API clarity.

### Success Codes (2xx)

| Code | Name | When to Use | Example |
|------|------|-------------|---------|
| **200** | OK | Successful GET, PATCH, PUT | Fetched or updated data |
| **201** | Created | Successful POST (new resource) | Created a new book |
| **204** | No Content | Successful DELETE, or action with no response | Deleted a user |

**Examples**:

```http
# 200 OK
GET /books/123
Response: 200 OK + book data

PATCH /books/123
Response: 200 OK + updated book

# 201 Created
POST /books
Response: 201 Created + new book + Location header

# 204 No Content
DELETE /books/123
Response: 204 No Content (empty body)
```

### Client Error Codes (4xx)

| Code | Name | When to Use | Example |
|------|------|-------------|---------|
| **400** | Bad Request | Invalid request data | Validation errors |
| **401** | Unauthorized | Missing or invalid authentication | No auth token |
| **403** | Forbidden | Authenticated but no permission | User can't access admin route |
| **404** | Not Found | Resource doesn't exist | Book with ID 999 not found |
| **409** | Conflict | Resource conflict | Email already exists |
| **422** | Unprocessable Entity | Valid syntax, but semantic errors | Business rule violation |

**Examples**:

```http
# 400 Bad Request - Validation Error
POST /books
{
  "title": "",  ← Empty title
  "author": "John"
}

Response: 400 Bad Request
{
  "error": "Validation Error",
  "details": [
    {
      "field": "title",
      "message": "Title is required"
    }
  ]
}
```

```http
# 401 Unauthorized - Missing Auth
GET /admin/users

Response: 401 Unauthorized
{
  "error": "Unauthorized",
  "message": "Authentication token required"
}
```

```http
# 403 Forbidden - No Permission
DELETE /users/123
(authenticated as regular user, trying to delete another user)

Response: 403 Forbidden
{
  "error": "Forbidden",
  "message": "You do not have permission to delete this user"
}
```

```http
# 404 Not Found
GET /books/999

Response: 404 Not Found
{
  "error": "Not Found",
  "message": "Book with ID 999 does not exist"
}
```

```http
# 409 Conflict
POST /users
{
  "email": "existing@example.com"
}

Response: 409 Conflict
{
  "error": "Conflict",
  "message": "User with this email already exists"
}
```

### Server Error Codes (5xx)

| Code | Name | When to Use | Example |
|------|------|-------------|---------|
| **500** | Internal Server Error | Unexpected server error | Unhandled exception |
| **502** | Bad Gateway | Upstream service error | Database connection failed |
| **503** | Service Unavailable | Temporary outage | Server under maintenance |

**Best Practice**: Never expose internal error details in 5xx responses (security risk).

```http
# 500 Internal Server Error
GET /books

Response: 500 Internal Server Error
{
  "error": "Internal Server Error",
  "message": "An unexpected error occurred. Please try again later.",
  "requestId": "abc-123-def"  ← For support debugging
}
```

### Status Code Decision Tree

```
Is the request successful?
  ├─ YES → Was a resource created?
  │         ├─ YES → 201 Created
  │         └─ NO → Is there response data?
  │                  ├─ YES → 200 OK
  │                  └─ NO → 204 No Content
  │
  └─ NO → Is it a client error?
           ├─ YES → What kind?
           │        ├─ Invalid data → 400 Bad Request
           │        ├─ Not authenticated → 401 Unauthorized
           │        ├─ No permission → 403 Forbidden
           │        ├─ Resource not found → 404 Not Found
           │        └─ Conflict → 409 Conflict
           │
           └─ NO → It's a server error
                    └─ 500 Internal Server Error
```

### Important Rules

#### **Rule #1: List APIs Never Return 404**

```http
# Even if no results
GET /books?status=unpublished

✅ CORRECT:
200 OK
{
  "data": [],
  "pagination": { "total": 0 }
}

❌ WRONG:
404 Not Found
```

**Why?** You're requesting a list, not a specific resource. An empty list is a valid response.

#### **Rule #2: Use 404 Only for Specific Resources**

```http
✅ Use 404:
GET /books/999    ← Requesting specific book
DELETE /books/999 ← Deleting specific book

❌ Don't use 404:
GET /books        ← Listing books
GET /books?status=active ← Filtering books
```

---

## Best Practices

### 1. **Design Before Implementation**

Use tools like **Swagger**, **Postman**, or **Insomnia** to design your API interface BEFORE writing code.

**Workflow**:
```
Requirements → Design API Interface → Document → Implement → Test
```

Not:
```
❌ Requirements → Start coding → Figure out routes later
```

### 2. **Consistency is King**

Once you establish a pattern, stick to it across ALL endpoints.

**Consistent**:
```http
POST /organizations  (plural)
POST /projects       (plural)
POST /tasks          (plural)
```

**Inconsistent** (❌ Bad):
```http
POST /organizations  (plural)
POST /project        (singular)
POST /Tasks          (capitalized)
```

**Consistency checklist**:
- ✅ Field names (always camelCase in JSON)
- ✅ Route patterns (always plural)
- ✅ Error response format
- ✅ Pagination structure
- ✅ Date formats (ISO 8601)

### 3. **Provide Sane Defaults**

Don't force clients to specify obvious parameters.

**Examples**:

```javascript
// Pagination defaults
const page = req.query.page || 1;
const limit = req.query.limit || 10;

// Sorting defaults
const sortBy = req.query.sortBy || 'createdAt';
const sortOrder = req.query.sortOrder || 'desc';

// Filter defaults
const status = req.query.status || 'active';
```

This allows:
```http
GET /books  ← Works without any query params
```

Instead of:
```http
GET /books?page=1&limit=10&sortBy=createdAt&sortOrder=desc  ← Tedious
```

### 4. **Use Descriptive Field Names (No Abbreviations)**

```
✅ GOOD:
{
  "description": "...",
  "organizationId": 123
}

❌ BAD:
{
  "desc": "...",      ← Abbreviation
  "orgId": 123        ← Abbreviation
}
```

**Why?** API consumers shouldn't have to guess what fields mean.

### 5. **Follow JSON Naming Conventions**

Always use **camelCase** for JSON fields:

```json
✅ CORRECT:
{
  "userId": 123,
  "createdAt": "2026-01-30T10:00:00Z",
  "firstName": "John"
}

❌ WRONG:
{
  "user_id": 123,        ← snake_case
  "CreatedAt": "...",    ← PascalCase
  "first-name": "John"   ← kebab-case
}
```

### 6. **Use ISO 8601 for Dates**

```
✅ CORRECT:  "2026-01-30T10:00:00Z"
❌ WRONG:    "01/30/2026"
❌ WRONG:    "1738233600"  (Unix timestamp - use for internal storage only)
```

### 7. **Include Relationship Data Thoughtfully**

**Option 1: Include IDs only**
```json
{
  "id": 1,
  "title": "Project Alpha",
  "organizationId": 5
}
```

**Option 2: Embed related data**
```json
{
  "id": 1,
  "title": "Project Alpha",
  "organization": {
    "id": 5,
    "name": "ACME Corp"
  }
}
```

**Option 3: Provide expansion** (like Stripe API)
```http
GET /projects/1?expand=organization
```

Choose based on your use case. Document your approach clearly.

### 8. **Provide Interactive Documentation**

Use tools like:
- **Swagger/OpenAPI**: Auto-generated from code comments
- **Postman Collections**: Shareable API examples
- **API Blueprint**: Markdown-based documentation

**Benefits**:
- Developers can test APIs without writing code
- Self-documenting
- Reduces support questions

### 9. **Version Your APIs**

```
https://api.example.com/v1/books
https://api.example.com/v2/books
```

**When to create a new version**:
- Breaking changes (removing fields, changing response structure)
- Major functionality overhauls

**Don't version for**:
- Adding new optional fields
- Adding new endpoints
- Bug fixes

### 10. **Handle Errors Consistently**

**Standard error response format**:

```json
{
  "error": "Error Type",
  "message": "Human-readable message",
  "details": [
    {
      "field": "email",
      "message": "Invalid email format"
    }
  ],
  "requestId": "abc-123-def",
  "timestamp": "2026-01-30T10:00:00Z"
}
```

Use this format for ALL errors (400, 404, 500, etc.).

---

## Common Pitfalls & Debugging

### Pitfall #1: Inconsistent Pluralization

**Problem**:
```
GET /book/123     ← Singular
GET /books        ← Plural
```

**Solution**: Always use plural, even for single resources.
```
GET /books/123    ✅
GET /books        ✅
```

---

### Pitfall #2: Using GET for State-Changing Operations

**Problem**:
```
GET /users/123/delete  ❌
GET /users/123/activate  ❌
```

**Why it's bad**: GET should be safe (no side effects). Search engine crawlers, browser prefetching, and caching can inadvertently trigger these.

**Solution**:
```
DELETE /users/123  ✅
POST /users/123/activate  ✅
```

---

### Pitfall #3: Returning 404 for Empty Lists

**Problem**:
```http
GET /books?status=unpublished

Response: 404 Not Found  ❌
```

**Solution**:
```http
200 OK
{
  "data": [],
  "pagination": { "total": 0 }
}
```

---

### Pitfall #4: Exposing Internal Error Details

**Problem**:
```json
{
  "error": "Database connection failed: Connection refused at 192.168.1.100:5432",
  "stack": "Error at line 45 in db.js..."
}
```

**Security risk**: Exposes internal architecture.

**Solution**:
```json
{
  "error": "Internal Server Error",
  "message": "An unexpected error occurred",
  "requestId": "abc-123"
}
```

Log full details server-side, return generic message to client.

---

### Pitfall #5: Confusing PUT and PATCH

**Problem**: Using them interchangeably without understanding semantics.

**Solution**:
- **PATCH**: Partial updates (send only changed fields)
- **PUT**: Full replacement (send all fields)

Most of the time, use **PATCH**.

---

### Pitfall #6: No Pagination on List Endpoints

**Problem**:
```http
GET /books
→ Returns 10,000 books in one response
```

**Issues**:
- Slow response times
- High memory usage
- Poor user experience

**Solution**: Always implement pagination with defaults.

---

### Pitfall #7: Inconsistent Field Naming

**Problem**:
```json
# Endpoint 1
{ "user_id": 123 }

# Endpoint 2
{ "userId": 123 }

# Endpoint 3
{ "UserId": 123 }
```

**Solution**: Choose one convention (camelCase recommended) and stick to it everywhere.

---

### Pitfall #8: Using POST for Everything

**Problem**: Using POST for all operations because "it works."

**Why it's bad**: Loses semantic meaning, prevents caching, breaks RESTful conventions.

**Solution**: Use appropriate HTTP methods:
- GET for fetching
- POST for creating
- PATCH for updating
- DELETE for deleting

---

### Debugging Tips

#### **Tip #1: Use Request IDs**

Include a unique ID in every response:

```json
{
  "requestId": "abc-123-def-456",
  "data": { ... }
}
```

When debugging, you can trace the exact request in server logs.

#### **Tip #2: Log Request/Response Payloads**

In development, log full request and response bodies (sanitize sensitive data in production).

#### **Tip #3: Test with Different HTTP Clients**

Test your API with:
- cURL
- Postman
- Browser fetch()
- Language-specific HTTP libraries

Different clients may reveal edge cases.

#### **Tip #4: Validate with OpenAPI Spec**

Use tools like **Swagger Validator** to ensure your API matches your OpenAPI specification.

---

## Quick Revision / Cheat Sheet

### URL Structure

```
https://api.example.com/v1/books/123?status=active
│      │  │              │   │     │  │
│      │  │              │   │     │  └─ Query params
│      │  │              │   │     └─ Resource ID
│      │  │              │   └─ Resource (plural)
│      │  │              └─ Version
│      │  └─ Domain
│      └─ Subdomain
└─ Scheme
```

### HTTP Methods

| Method | Purpose | Idempotent | Request Body | Response Body |
|--------|---------|-----------|--------------|---------------|
| GET | Fetch data | ✅ Yes | ❌ No | ✅ Yes |
| POST | Create/Custom action | ❌ No | ✅ Yes | ✅ Yes |
| PATCH | Partial update | ✅ Yes | ✅ Yes | ✅ Yes |
| PUT | Full replacement | ✅ Yes | ✅ Yes | ✅ Yes |
| DELETE | Remove resource | ✅ Yes | ❌ No | ❌ Usually not |

### CRUD Endpoints

| Operation | Method | Endpoint | Success Code |
|-----------|--------|----------|--------------|
| Create | POST | `/books` | 201 Created |
| List | GET | `/books` | 200 OK |
| Get | GET | `/books/{id}` | 200 OK |
| Update | PATCH | `/books/{id}` | 200 OK |
| Delete | DELETE | `/books/{id}` | 204 No Content |

### Status Codes

| Code | Name | When to Use |
|------|------|-------------|
| 200 | OK | Successful GET, PATCH, PUT |
| 201 | Created | Successful POST (resource created) |
| 204 | No Content | Successful DELETE |
| 400 | Bad Request | Validation error |
| 401 | Unauthorized | Authentication required |
| 403 | Forbidden | No permission |
| 404 | Not Found | Resource doesn't exist |
| 500 | Internal Server Error | Server error |

### Pagination Response

```json
{
  "data": [ /* array of items */ ],
  "pagination": {
    "page": 2,
    "limit": 10,
    "total": 95,
    "totalPages": 10
  }
}
```

### Query Parameters

| Param | Purpose | Example |
|-------|---------|---------|
| `page` | Page number | `?page=2` |
| `limit` | Items per page | `?limit=10` |
| `sortBy` | Field to sort by | `?sortBy=createdAt` |
| `sortOrder` | Sort direction | `?sortOrder=desc` |
| `{field}` | Filter by field | `?status=active` |

### Custom Actions

**Pattern**: `POST /{resource}/{id}/{action}`

**Examples**:
```
POST /organizations/123/archive
POST /projects/456/clone
POST /articles/789/publish
```

### Best Practices Checklist

- ✅ Always use plural resource names (`/books`, not `/book`)
- ✅ Use lowercase for routes
- ✅ Use camelCase for JSON fields
- ✅ Provide sane defaults (pagination, sorting)
- ✅ Return 404 only for specific resources, not lists
- ✅ Use appropriate HTTP methods (not just POST)
- ✅ Include pagination for list endpoints
- ✅ Return 201 for created resources
- ✅ Return 204 for successful deletes
- ✅ Document your API (Swagger/OpenAPI)
- ✅ Version your API (`/v1/`, `/v2/`)
- ✅ Use ISO 8601 for dates
- ✅ Never expose internal errors to clients

### Error Response Format

```json
{
  "error": "Error Type",
  "message": "Human-readable description",
  "details": [
    {
      "field": "email",
      "message": "Invalid format"
    }
  ],
  "requestId": "abc-123",
  "timestamp": "2026-01-30T10:00:00Z"
}
```

### JSON Payload Standards

```json
{
  "userId": 123,              // camelCase
  "firstName": "John",         // Full words, no abbreviations
  "createdAt": "2026-01-30T10:00:00Z",  // ISO 8601
  "isActive": true,            // Boolean, not 0/1
  "organizationId": 5          // Not org_id or orgId
}
```

### Complete Example Flow

**Create a Book**:
```http
POST /api/v1/books
{
  "title": "Clean Code",
  "author": "Robert Martin",
  "isbn": "978-0132350884"
}

→ 201 Created
{
  "id": 123,
  "title": "Clean Code",
  "author": "Robert Martin",
  "isbn": "978-0132350884",
  "createdAt": "2026-01-30T10:00:00Z"
}
```

**List Books with Filters**:
```http
GET /api/v1/books?status=published&page=1&limit=10&sortBy=title&sortOrder=asc

→ 200 OK
{
  "data": [ /* 10 books */ ],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 42,
    "totalPages": 5
  }
}
```

**Get Single Book**:
```http
GET /api/v1/books/123

→ 200 OK
{
  "id": 123,
  "title": "Clean Code",
  ...
}
```

**Update Book**:
```http
PATCH /api/v1/books/123
{
  "price": 39.99
}

→ 200 OK
{
  "id": 123,
  "price": 39.99,
  ...
}
```

**Delete Book**:
```http
DELETE /api/v1/books/123

→ 204 No Content
(empty body)
```

**Custom Action**:
```http
POST /api/v1/books/123/publish

→ 200 OK
{
  "id": 123,
  "status": "published",
  "publishedAt": "2026-01-30T12:00:00Z"
}
```

---

## Final Thoughts

REST API design is about **consistency**, **intuitiveness**, and **adherence to standards**. By following these principles:

1. **Clients** can integrate your API faster with fewer errors
2. **Maintainers** can understand and extend the API easily
3. **Users** get a better experience (faster, more reliable)

**Remember**:
- Design your API interface BEFORE writing code
- Use tools like Swagger for documentation and testing
- Consistency across all endpoints is more important than perfection
- When in doubt, follow the standard

Good API design is a skill that separates good backend engineers from great ones. Master these principles, and you'll build APIs that are a joy to use.

---

**Further Reading**:
- [Roy Fielding's REST Dissertation](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm)
- [HTTP/1.1 Specification (RFC 7231)](https://tools.ietf.org/html/rfc7231)
- [OpenAPI Specification](https://swagger.io/specification/)
- [JSON API Specification](https://jsonapi.org/)

---

*This guide is designed for practical, interview-ready knowledge. Bookmark it, revisit it, and use it as a reference when designing your next API.*