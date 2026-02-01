# Mastering Databases with PostgreSQL

A comprehensive guide to understanding relational databases, PostgreSQL, and how to design and query databases for production backend systems.

---

## Table of Contents

1. [Fundamentals](#fundamentals)
   - [Why Databases Exist](#why-databases-exist)
   - [Persistence and State](#persistence-and-state)
   - [What Is a Database](#what-is-a-database)
2. [Storage Architecture](#storage-architecture)
   - [Disk-Based vs RAM Storage](#disk-based-vs-ram-storage)
   - [The Storage Trade-off](#the-storage-trade-off)
3. [Database Management Systems](#database-management-systems)
   - [DBMS Responsibilities](#dbms-responsibilities)
   - [Why Not Just Text Files](#why-not-just-text-files)
4. [Database Types](#database-types)
   - [Relational Databases](#relational-databases)
   - [Non-Relational Databases](#non-relational-databases)
   - [Choosing Your Database](#choosing-your-database)
5. [PostgreSQL Essentials](#postgresql-essentials)
   - [Why PostgreSQL](#why-postgresql)
   - [Data Types](#data-types)
6. [Database Migrations](#database-migrations)
   - [What Are Migrations](#what-are-migrations)
   - [Up and Down Migrations](#up-and-down-migrations)
   - [Setting Up Migrations](#setting-up-migrations)
7. [Database Design](#database-design)
   - [Relationships](#relationships)
   - [Constraints and Integrity](#constraints-and-integrity)
8. [Querying and Optimization](#querying-and-optimization)
   - [Basic CRUD Operations](#basic-crud-operations)
   - [Joins and Filtering](#joins-and-filtering)
   - [Indexing](#indexing)
   - [Triggers](#triggers)
9. [Common Pitfalls & Debugging](#common-pitfalls--debugging)
10. [Quick Revision Cheat Sheet](#quick-revision-cheat-sheet)

---

## Fundamentals

### Why Databases Exist

At its core, a database solves one fundamental problem: **how do we keep data alive after our program stops running?**

Imagine you're building a to-do list app. Every time you close and reopen it, you expect your tasks to still be there. That expectation—that data survives program termination—is what we call **persistence**.

#### Real-World Analogy

Think of a database like a library. When you check out a book, the librarian doesn't rewrite the entire library catalog each time a book is checked out. Instead, they have a **permanent system** that tracks which books are checked out, who has them, and when they're due back. Even if the librarian goes home for the day, the system persists. When they return, everything is exactly as it was left.

### Persistence and State

**Persistence** means storing data in a way that survives:
- Application restarts
- System crashes
- Network disconnections
- Time passing

Without persistence, every interaction would start from scratch. Your user would have no history, no progress, no saved state.

### What Is a Database

The term "database" is surprisingly broad. In its simplest form:

> Any structured system that allows you to **create, read, update, and delete (CRUD)** data persistently

Examples of databases range from:
- **Browser Local Storage** (key-value pairs saved on your computer)
- **Contact List on Your Phone** (structured records of people)
- **Text Files with Notes** (basic persistence)
- **Enterprise PostgreSQL Systems** (complex, multi-user, industrial-strength)

However, when backend developers say "database," they typically mean **disk-based relational or non-relational database systems** like PostgreSQL or MongoDB.

---

## Storage Architecture

### Disk-Based vs RAM Storage

To understand why we need databases, we must understand the computer memory hierarchy:

```
CPU
  ↓
Primary Memory (RAM)
  ↓
Secondary Memory (Disk - HDD/SSD)
```

**RAM (Primary Memory):**
- ✅ **Extremely fast** (nanoseconds to microseconds)
- ❌ **Expensive** (roughly $5-10 per GB)
- ❌ **Limited capacity** (typical laptop: 8-32 GB)
- ❌ **Volatile** (data disappears when power is lost)

**Disk Storage (Secondary Memory):**
- ✅ **Cheap** (roughly $0.01-0.05 per GB)
- ✅ **Massive capacity** (typical laptop: 256 GB - 2 TB)
- ✅ **Persistent** (survives power loss)
- ❌ **Slower** (milliseconds for access)

### The Storage Trade-off

Systems like **Redis** (in-memory cache) use RAM because they need extreme speed. But they're typically used for temporary data—session tokens, frequently accessed data, real-time analytics.

Databases like **PostgreSQL** use disk storage because:
1. We need **large capacity** (millions/billions of records)
2. We can **tolerate slower speeds** (milliseconds vs nanoseconds)
3. We need **permanence** (data survives restarts)

This trade-off is the foundation of modern database architecture.

#### Visual Comparison

| Characteristic | RAM | Disk |
|---|---|---|
| Speed | Nanoseconds | Milliseconds |
| Capacity | 8-128 GB | 256 GB - 10 TB+ |
| Cost per GB | $5-10 | $0.01-0.05 |
| Persistence | ❌ Volatile | ✅ Persistent |
| Best For | Caching | Long-term storage |

---

## Database Management Systems

### What Is a DBMS?

Simply throwing data on a disk isn't enough. A **Database Management System (DBMS)** is specialized software that handles:

1. **Storage Organization** - Arranging data efficiently for fast retrieval
2. **Access Methods** - Providing ways to create, read, update, and delete data
3. **Data Integrity** - Ensuring data accuracy and consistency
4. **Security** - Controlling who can access what
5. **Concurrency** - Handling multiple users simultaneously
6. **Scaling** - Managing growth and load balancing

### DBMS Responsibilities

#### 1. Data Organization

The database must structure data so that operations are efficient. Without organization, every query would require scanning all data from the disk—impossibly slow for large datasets.

#### 2. Efficient Access (CRUD Operations)

```
C - Create  (INSERT)
R - Read    (SELECT)
U - Update  (UPDATE)
D - Delete  (DELETE)
```

The DBMS must perform these operations quickly, even with millions of records.

#### 3. Data Integrity

**Integrity** means the accuracy and validity of data.

**Example:** An e-commerce database stores order amounts as numbers. The DBMS ensures:
- You cannot insert the string "hello" into a price field
- You cannot insert negative prices (if that's a business rule)
- The data type matches the schema definition

**Why This Matters:** Corrupt data can cascade through your entire system. A single invalid record can cause calculations to fail, reports to be wrong, and users to see incorrect information.

#### 4. Consistency and Atomicity

Imagine transferring money: $100 from Account A to Account B.

**Without a DBMS:**
```
1. Read Account A balance: $500
2. Subtract $100: Now $400
3. Read Account B balance: $300
4. Add $100: Now $400
5. System crashes...
6. Account A: $400 (updated)
7. Account B: $300 (NOT updated)
→ Money disappears!
```

**With a DBMS:**
Either both operations complete, or neither does. This is called an **ACID transaction**.

---

### Why Not Just Text Files?

Before databases existed, developers stored data in text files. This seems simple but creates massive problems:

#### Problem 1: Parsing Overhead

**Scenario:** You have a customer database in a text file:

```
1,john@example.com,John Doe,hashed_password_123
2,jane@example.com,Jane Doe,hashed_password_456
3,bob@example.com,Bob Smith,hashed_password_789
```

To find a customer by email:

```python
# Application-level parsing (slow!)
with open('customers.txt', 'r') as file:
    for line in file:
        fields = line.split(',')
        if fields[1] == search_email:
            return fields
```

**Why It's Slow:**
- Read entire file into memory
- Parse every line (string operations are slow)
- Compare every email manually
- Repeat this for every query

If you have 1 million customers, that's 1 million comparisons **for every search**. A database would find the answer in milliseconds.

#### Problem 2: No Structure Enforcement

Text files are just strings. Nothing prevents:

```
1,invalid_email,John Doe,password  # Invalid email format
2,jane@example.com,Jane Doe,not_hashed  # Password not hashed
3,bob@example.com,Bob,hashed_password_789  # Name too short
```

The application code must enforce everything. One bug in a non-critical path can corrupt your data. A database enforces rules **at the system level**.

#### Problem 3: Concurrency is Impossible

**Scenario:** Two users update the same customer record simultaneously.

```
Customer record: age = 30

User 1 reads: age = 30
User 2 reads: age = 30
User 1 changes age to 31, writes file
User 2 changes age to 32, writes file
Final result: age = 32 (User 1's change lost!)
```

Why? The last write wins. There's no locking mechanism, no transaction coordination. With millions of users, data corruption is inevitable.

**A database prevents this** with locks, transactions, and write ordering.

---

## Database Types

### Relational Databases

**Relational databases** organize data into **tables** with **rows and columns**, and define relationships between tables.

**Key Characteristics:**
- 📋 **Structured schema** (defined in advance)
- 🔗 **Relationships** (foreign keys, joins)
- ✅ **ACID compliance** (Atomicity, Consistency, Isolation, Durability)
- 🛡️ **Strong data integrity** (constraints, validation)
- 🔍 **SQL language** (industry standard)

**Examples:** PostgreSQL, MySQL, SQL Server, Oracle

#### Real-World Analogy

A relational database is like a **well-organized office with filing cabinets**. Each drawer is a table. Each folder is a row. Each label is a column. The relationships between people in one drawer link to records in another drawer—you know exactly where to find connected information.

### Non-Relational Databases

**Non-relational (NoSQL)** databases are flexible on schema and don't enforce relationships the same way.

**Key Characteristics:**
- 📝 **Flexible/dynamic schema** (add fields on the fly)
- 🎯 **Document-oriented** (JSON-like storage)
- ⚡ **Horizontal scaling** (distribute across many servers)
- ⚠️ **Weaker consistency guarantees** (eventual consistency)
- 🔓 **No structured query language** (different APIs per system)

**Examples:** MongoDB, Cassandra, DynamoDB, Firebase

#### Real-World Analogy

A NoSQL database is like a **messy garage where you pile things however you want**. You can throw anything in, organize it however, and retrieve it however you want. Freedom, but no structure.

### Comparison Table

| Feature | Relational | Non-Relational |
|---|---|---|
| Schema | Strict, predefined | Flexible, dynamic |
| Data Consistency | Strong (ACID) | Eventual consistency |
| Relationships | Foreign keys, joins | Embedded documents |
| Scaling | Vertical (bigger server) | Horizontal (more servers) |
| Query Language | SQL (standard) | Varies (API-based) |
| Best For | Structured data, consistency | Flexible content, scalability |
| Enforcement | Database level | Application level |

### When to Use Each

#### Choose Relational If:

✅ **CRM System** - Customer, sales, contact data with many relationships
✅ **Banking** - Transactions, accounts, transfers (must be consistent)
✅ **E-commerce** - Orders, products, inventory with strict relationships
✅ **Anything with complex relationships and consistency requirements**

#### Choose Non-Relational If:

✅ **CMS (Content Management)** - Blog posts with varying structures (images, videos, code blocks)
✅ **User profiles** - Each user might have different custom fields
✅ **Real-time analytics** - High write volume, eventual consistency acceptable
✅ **Prototyping** - You're not sure of your schema yet

---

### Choosing Your Database

For most backend systems, **PostgreSQL is the best first choice**. Here's why:

#### 1. Open Source & Free

No licensing fees. Deploy anywhere. Full source code available for inspection.

#### 2. SQL Standard Compliance

Queries written for PostgreSQL work on MySQL, SQL Server, etc. Migration is easier.

#### 3. Extensibility

1400+ pages of documentation. JSON support. Custom types. Full-text search. Geo-spatial data. It covers almost every use case.

#### 4. Excellent JSON Support

PostgreSQL has native JSON and JSONB types with indexing. You get the flexibility of NoSQL with the power of relational databases.

**Example:**

```sql
-- Store flexible data in a relational table
CREATE TABLE articles (
    id UUID PRIMARY KEY,
    title TEXT NOT NULL,
    content JSONB NOT NULL,  -- Can vary per article
    created_at TIMESTAMP DEFAULT NOW()
);

-- Store dynamic content
INSERT INTO articles (id, title, content) VALUES (
    'article-1',
    'Getting Started',
    '{"sections": [{"title": "Intro"}, {"title": "Basics"}], "author": "John"}'::jsonb
);
```

#### 5. Reliability & Scalability

Trusted by startups and enterprises. Handles millions of transactions per second.

**Verdict:** Unless you have specific reasons (need distributed NoSQL scaling, already heavily invested in MongoDB), **start with PostgreSQL**. You won't outgrow it for years.

---

## PostgreSQL Essentials

### Why PostgreSQL

PostgreSQL combines the best of both worlds:

| Feature | Benefit |
|---|---|
| Strong schema + JSON support | Structured data + flexibility |
| ACID transactions | Data integrity you can trust |
| Advanced indexing | Fast queries at scale |
| SQL standard | Knowledge transfers between jobs |
| Open source | No vendor lock-in |
| Community support | Thousands of articles, tools, forums |

### Data Types

PostgreSQL offers 30+ data types. Here are the ones you'll use 90% of the time:

#### Integer Types

```sql
CREATE TABLE numbers_demo (
    small_num SMALLINT,      -- -32,768 to 32,767
    regular_num INTEGER,     -- -2.1B to 2.1B
    big_num BIGINT,          -- -9.2 quadrillion to 9.2 quadrillion
    auto_id SERIAL,          -- INTEGER with auto-increment
    auto_big_id BIGSERIAL    -- BIGINT with auto-increment
);
```

**When to Use:**
- **SERIAL/BIGSERIAL** - For ID fields (auto-incrementing primary keys)
- **BIGINT** - For production systems (more capacity than INTEGER)
- **INTEGER** - Rarely needed; BIGINT is safer and barely slower
- **SMALLINT** - Almost never; storage savings aren't worth the limits

#### Decimal Types

```sql
CREATE TABLE prices_demo (
    -- DECIMAL for money: max 10 total digits, 2 after decimal
    price DECIMAL(10, 2),      -- Example: 12345678.99
    
    -- NUMERIC is identical to DECIMAL
    amount NUMERIC(10, 2),
    
    -- FLOAT for approximate values
    percentage FLOAT,          -- Example: 3.14159265
    
    -- DOUBLE PRECISION for more precision
    scientific_value DOUBLE PRECISION
);
```

**Critical Rule:** For financial data (prices, payments, money), **always use DECIMAL or NUMERIC**, never FLOAT.

**Why?** Floating-point numbers are approximations:

```sql
-- FLOAT representation problems
SELECT 0.1 + 0.2;  -- Returns 0.30000000000000004, not 0.3!

-- DECIMAL is exact
SELECT 0.1::DECIMAL + 0.2::DECIMAL;  -- Returns 0.3 exactly
```

For scientific computing or approximate values (measurements, percentages), FLOAT is fine and faster.

#### String Types

```sql
CREATE TABLE strings_demo (
    -- TEXT: Use this 99% of the time
    email TEXT NOT NULL,
    full_name TEXT NOT NULL,
    bio TEXT,
    
    -- VARCHAR(length): Older style, length is arbitrary in PostgreSQL
    -- CHAR(length): Pads with spaces, almost never needed
    country_code CHAR(2)      -- Example: "US", "GB", "CA"
);
```

**Best Practice:** Use **TEXT** for flexibility. In PostgreSQL, there's no performance difference between TEXT and VARCHAR(255). Don't use VARCHAR with random lengths like VARCHAR(255)—it's cargo cult programming from MySQL conventions.

**When to Use CHAR(length):**
- Fixed-length codes: Country codes (2 characters), state abbreviations
- You know for certain the length never changes

**Real Example:**

```sql
-- Good: Flexible and clear
CREATE TABLE users (
    email TEXT NOT NULL UNIQUE,
    bio TEXT,
    full_name TEXT NOT NULL
);

-- Avoid: Random length, misleading
CREATE TABLE users (
    email VARCHAR(255) NOT NULL UNIQUE,  -- Why 255? Arbitrary
    bio VARCHAR(1000),                    -- Why 1000? Vague
    full_name VARCHAR(100) NOT NULL       -- Why 100? Not enforced
);
```

#### Boolean

```sql
CREATE TABLE flags_demo (
    is_active BOOLEAN DEFAULT TRUE,
    is_verified BOOLEAN DEFAULT FALSE
);

-- Insert
INSERT INTO flags_demo (is_active) VALUES (true);
INSERT INTO flags_demo (is_active) VALUES (false);
```

#### Date & Time

```sql
CREATE TABLE datetime_demo (
    -- Date only
    birth_date DATE,
    
    -- Time only (hour, minute, second)
    meeting_time TIME,
    
    -- Date and time without timezone
    created_at TIMESTAMP DEFAULT NOW(),
    
    -- Date and time with timezone (recommended)
    last_login TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

**Best Practice:** Use **TIMESTAMP WITH TIME ZONE** for international apps. It stores the time in UTC and automatically converts for different timezones.

#### JSON Types

```sql
CREATE TABLE json_demo (
    -- JSON: Stored as text, validated but slower to query
    raw_json JSON,
    
    -- JSONB: Stored in binary, indexed, much faster
    structured_data JSONB
);

-- Insert JSON
INSERT INTO json_demo VALUES (
    '{"name": "John", "age": 30}'::json,
    '{"name": "Jane", "skills": ["Python", "SQL"]}'::jsonb
);

-- Query JSON fields
SELECT structured_data->'name' AS name FROM json_demo;
SELECT structured_data->>'skills' AS skills FROM json_demo;
```

**When to Use:**
- **JSONB** - Almost always. Better performance, proper indexing support
- **JSON** - Rarely. Only if you need exact text representation

#### UUID Type

```sql
CREATE TABLE users (
    -- UUID: Universally unique identifier
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email TEXT NOT NULL UNIQUE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

**Why UUIDs Instead of Serial?**
- ✅ Globally unique (no conflicts in distributed systems)
- ✅ Can generate on client side (no need to contact database first)
- ✅ Better privacy (IDs don't leak user count)
- ❌ Slightly larger (16 bytes vs 8 bytes for BIGINT)

**For most backends: Use UUIDs for user-facing IDs, BIGSERIAL for internal IDs.**

#### Array Types

```sql
CREATE TABLE arrays_demo (
    -- Array of integers
    favorite_numbers INTEGER[],
    
    -- Array of text
    skills TEXT[],
    
    -- Array of JSON
    metadata JSONB[]
);

-- Insert arrays
INSERT INTO arrays_demo VALUES (
    ARRAY[1, 2, 3],
    ARRAY['Python', 'SQL', 'Go'],
    ARRAY['{"level": "expert"}'::jsonb]
);

-- Query arrays
SELECT skills[1] FROM arrays_demo;  -- Returns 'Python'
SELECT array_length(skills, 1) FROM arrays_demo;  -- Returns 3
```

#### Enum Types

```sql
-- Define enum type
CREATE TYPE task_status AS ENUM ('pending', 'in_progress', 'completed', 'cancelled');

-- Use in table
CREATE TABLE tasks (
    id UUID PRIMARY KEY,
    status task_status DEFAULT 'pending',
    title TEXT NOT NULL
);

-- Insert
INSERT INTO tasks (id, status, title) VALUES 
    ('task-1', 'in_progress', 'Build database');

-- This fails with database-level error:
INSERT INTO tasks (id, status, title) VALUES 
    ('task-2', 'invalid_status', 'Bad task');
    -- ERROR: invalid input value for enum task_status
```

**Why Enums?**
1. **Database enforcement** - Invalid values rejected at DB level, not application level
2. **Documentation** - Anyone reading migrations sees valid values immediately
3. **Type safety** - Can't accidentally insert typos

#### Data Type Selection Table

| Use Case | Type | Example |
|---|---|---|
| Primary key (distributed) | UUID | User IDs, resource IDs |
| Primary key (single server) | BIGSERIAL | Auto-incrementing sequences |
| Money/prices | DECIMAL(10,2) | $99.99 |
| Percentages/ratings | NUMERIC or DECIMAL | 4.5 stars |
| Measurements/approximations | FLOAT | Temperature: 98.6°F |
| Email/names/descriptions | TEXT | User emails, product names |
| Fixed codes | CHAR(2) | Country codes: "US" |
| Variable length text | TEXT | Blog posts, user bios |
| Binary yes/no | BOOLEAN | is_active, is_verified |
| Dates only | DATE | Birth dates, event dates |
| Times with timezone | TIMESTAMP WITH TIME ZONE | User login times |
| Dynamic content | JSONB | Blog post metadata, user preferences |
| Lists of values | ARRAY | Tags, skills, permissions |
| Fixed set of values | ENUM | Status (pending, approved, rejected) |

---

## Database Migrations

### What Are Migrations?

A migration is a **versioned SQL file** that describes a change to your database schema.

**Why Migrations?**

Without migrations:
- 😞 No history of database changes
- 😞 No way to roll back changes
- 😞 Different developers have different schemas
- 😞 Production and development schemas drift apart
- 😞 Deploying changes is manual and error-prone

**With migrations:**
- ✅ Every change is tracked (like Git for databases)
- ✅ Changes are reproducible
- ✅ You can roll back safely
- ✅ Schema is version-controlled with code
- ✅ New team members can build exact same schema

### Folder Structure

```
my-app/
├── src/
├── db/
│   └── migrations/
│       ├── 001_create_users_table.sql
│       ├── 002_create_projects_table.sql
│       └── 003_add_indexes.sql
├── .env
└── package.json
```

### Up and Down Migrations

Each migration has two parts:

**UP:** What to do (apply change)
**DOWN:** How to undo it (revert change)

```sql
-- 001_create_users_table.sql

-- UP MIGRATION
-- ============
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email TEXT NOT NULL UNIQUE,
    full_name TEXT NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- DOWN MIGRATION
-- ==============
DROP TABLE users;
```

**Real-World Scenario:**

```
Day 1: Run migration 001 → users table exists
Day 2: Run migration 002 → projects table exists
Day 3: Bug in production! Rollback migration 002 → users table still there, projects table gone
Day 4: Fix the bug, run migration 002 again → projects table created correctly
```

### Setting Up Migrations

Most backend frameworks have built-in migration tools:

**Node.js (using DBMate):**

```bash
# Install migration tool
npm install -g dbmate

# Set database URL
export DATABASE_URL="postgres://user:password@localhost/mydb"

# Create a new migration
dbmate new create_users_table

# Apply migrations
dbmate up

# Rollback last migration
dbmate down

# Check migration status
dbmate status
```

**Python (using Alembic):**

```bash
# Install Alembic
pip install alembic

# Initialize Alembic
alembic init alembic

# Create a migration
alembic revision --autogenerate -m "create users table"

# Apply migrations
alembic upgrade head

# Rollback last migration
alembic downgrade -1
```

**Go (using golang-migrate):**

```bash
# Install migrate
go install -tags 'postgres' github.com/golang-migrate/migrate/v4/cmd/migrate@latest

# Create migration
migrate create -ext sql -dir db/migrations -seq create_users_table

# Apply migrations
migrate -path db/migrations -database "postgres://user:pass@localhost/mydb" up

# Rollback
migrate -path db/migrations -database "postgres://user:pass@localhost/mydb" down
```

### Migration Workflow

```
1. Developer creates feature (e.g., "add user roles")
   ↓
2. Creates migration file: 003_add_user_roles.sql
   ↓
3. Writes UP migration (CREATE TYPE user_role, ADD COLUMN role)
   ↓
4. Writes DOWN migration (DROP COLUMN role, DROP TYPE user_role)
   ↓
5. Tests locally: dbmate up
   ↓
6. Tests rollback: dbmate down (everything returns to previous state)
   ↓
7. Commits migration + code to Git
   ↓
8. Deploy to staging: dbmate up
   ↓
9. Run tests against new schema
   ↓
10. Deploy to production: dbmate up
    (If something breaks, dbmate down is a safe rollback)
```

---

## Database Design

### Relationships

In relational databases, tables are connected through relationships. There are three types:

#### 1. One-to-One (1:1)

Each row in Table A corresponds to **exactly one** row in Table B.

**Example:** User ↔ UserProfile

```sql
-- Users table
CREATE TABLE users (
    id UUID PRIMARY KEY,
    email TEXT NOT NULL UNIQUE,
    full_name TEXT NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE
);

-- UserProfile table (one-to-one with users)
-- Important: Use user_id as PRIMARY KEY (not just a foreign key)
CREATE TABLE user_profiles (
    user_id UUID PRIMARY KEY,  -- Both FK and PK
    avatar_url TEXT,
    bio TEXT,
    phone_number TEXT,
    created_at TIMESTAMP WITH TIME ZONE,
    
    -- This creates the relationship
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

**Why Separate Tables?**
- User data changes frequently (name, email)
- Profile data is optional and changes less often
- This separation makes the schema flexible for future additions

**Implementation Pattern:**
- Keep the primary key of the main table as the primary key in the related table
- This prevents duplicate profiles for the same user

#### 2. One-to-Many (1:M)

One row in Table A can correspond to **many** rows in Table B.

**Example:** Project → Tasks (one project has many tasks)

```sql
CREATE TABLE projects (
    id UUID PRIMARY KEY,
    name TEXT NOT NULL,
    description TEXT,
    owner_id UUID NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE,
    
    FOREIGN KEY (owner_id) REFERENCES users(id) ON DELETE RESTRICT
);

CREATE TABLE tasks (
    id UUID PRIMARY KEY,
    project_id UUID NOT NULL,  -- Foreign key, not primary key
    title TEXT NOT NULL,
    description TEXT,
    assigned_to UUID,
    created_at TIMESTAMP WITH TIME ZONE,
    
    FOREIGN KEY (project_id) REFERENCES projects(id) ON DELETE CASCADE,
    FOREIGN KEY (assigned_to) REFERENCES users(id) ON DELETE SET NULL
);
```

**Key Differences from 1:1:**
- `project_id` in tasks is a **foreign key only**, not the primary key
- A project can have 0, 1, or many tasks
- Tasks cannot exist without a project

#### 3. Many-to-Many (M:M)

Many rows in Table A can correspond to many rows in Table B.

**Example:** Users ↔ Projects (a user can work on multiple projects, a project can have multiple users)

```sql
-- Create a "junction" or "linking" table
CREATE TABLE project_members (
    project_id UUID NOT NULL,
    user_id UUID NOT NULL,
    role membership_role DEFAULT 'member',  -- owner, admin, or member
    created_at TIMESTAMP WITH TIME ZONE,
    
    -- Composite primary key: this combination is unique
    PRIMARY KEY (project_id, user_id),
    
    -- Foreign keys
    FOREIGN KEY (project_id) REFERENCES projects(id) ON DELETE CASCADE,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

**Why a Separate Table?**

Without it:
- Users table would need a `projects` column (array of project IDs)
- Projects table would need a `members` column (array of user IDs)
- Queries become complex, updates become fragile

With the junction table:
- Clean, normalized schema
- Easy to add extra metadata (like `role`)
- Queries are straightforward

**Example Queries:**

```sql
-- Get all projects for a user
SELECT p.* FROM projects p
JOIN project_members pm ON p.id = pm.project_id
WHERE pm.user_id = 'user-123';

-- Get all members of a project
SELECT u.* FROM users u
JOIN project_members pm ON u.id = pm.user_id
WHERE pm.project_id = 'project-456';

-- Add a user to a project
INSERT INTO project_members (project_id, user_id, role)
VALUES ('project-456', 'user-123', 'member');
```

### Constraints and Integrity

Constraints are **rules** enforced by the database to keep data accurate.

#### Primary Key

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY,  -- Uniquely identifies each row
    email TEXT NOT NULL UNIQUE
);
```

**Properties:**
- ✅ Must be unique across all rows
- ✅ Cannot be NULL
- ✅ Implicitly indexed (fast lookups)

**Rule:** Every table should have a primary key.

#### Unique Constraint

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY,
    email TEXT NOT NULL UNIQUE,  -- Each user has unique email
    username TEXT UNIQUE          -- Usernames are also unique
);
```

**Prevents duplicate values** in a column.

**When to Use:**
- Email addresses (one email = one user)
- Usernames (one username = one user)
- Product SKUs (each product has unique identifier)

#### Not Null Constraint

```sql
CREATE TABLE products (
    id UUID PRIMARY KEY,
    name TEXT NOT NULL,           -- Every product must have a name
    price DECIMAL(10,2) NOT NULL, -- Every product must have a price
    description TEXT              -- Description is optional
);
```

**Rule of Thumb:** 70%+ of your columns should be NOT NULL. Only omit when there's a real reason for missing data.

**Bad Example:**
```sql
-- Avoid: Too much optionality, data consistency suffers
CREATE TABLE users (
    email TEXT,      -- Nullable! But user needs email
    age INTEGER,     -- Nullable! But we want demographics
    location TEXT    -- Nullable! We'd like location data
);
```

**Good Example:**
```sql
-- Better: Clear requirements
CREATE TABLE users (
    id UUID PRIMARY KEY,
    email TEXT NOT NULL UNIQUE,
    full_name TEXT NOT NULL,
    phone_number TEXT,           -- Optional: user may not provide
    bio TEXT,                    -- Optional: user may not fill profile
    created_at TIMESTAMP NOT NULL DEFAULT NOW()
);
```

#### Foreign Key Constraint

```sql
CREATE TABLE tasks (
    id UUID PRIMARY KEY,
    project_id UUID NOT NULL,
    title TEXT NOT NULL,
    
    -- Foreign key: project_id must exist in projects table
    FOREIGN KEY (project_id) REFERENCES projects(id)
);
```

**Referential Integrity Actions:**

When a referenced row is deleted, what happens?

```sql
-- RESTRICT: Prevent deletion if related rows exist
FOREIGN KEY (owner_id) REFERENCES users(id) ON DELETE RESTRICT
-- Cannot delete a user if they own projects

-- CASCADE: Delete related rows automatically
FOREIGN KEY (project_id) REFERENCES projects(id) ON DELETE CASCADE
-- If project deleted, all tasks deleted too

-- SET NULL: Set foreign key to NULL if referenced row deleted
FOREIGN KEY (assigned_to) REFERENCES users(id) ON DELETE SET NULL
-- If user deleted, tasks become unassigned (assigned_to = NULL)

-- SET DEFAULT: Set foreign key to default value
FOREIGN KEY (status_id) REFERENCES statuses(id) ON DELETE SET DEFAULT
```

**When to Use Each:**

| Action | When | Example |
|---|---|---|
| RESTRICT | Prevent orphan data | User deletes, but they own projects |
| CASCADE | Related data isn't useful | Project deleted, delete all tasks |
| SET NULL | Keep record but lose relationship | User deleted, task stays but unassigned |
| SET DEFAULT | Use fallback value | Status deleted, use "pending" status |

#### Check Constraint

```sql
CREATE TABLE tasks (
    id UUID PRIMARY KEY,
    title TEXT NOT NULL,
    priority INTEGER NOT NULL,
    due_date DATE,
    
    -- Priority must be 1-5
    CHECK (priority >= 1 AND priority <= 5),
    
    -- Due date must be in the future (optional business rule)
    CHECK (due_date IS NULL OR due_date > NOW())
);
```

**Prevents invalid values** at the database level.

**Examples:**

```sql
-- Valid: priority is 3
INSERT INTO tasks VALUES ('task-1', 'Build feature', 3, '2025-02-15');

-- Invalid: priority is 10
INSERT INTO tasks VALUES ('task-2', 'Bug fix', 10, '2025-02-15');
-- ERROR: new row for relation "tasks" violates check constraint "tasks_priority_check"
```

---

## Querying and Optimization

### Basic CRUD Operations

#### CREATE (Insert)

```sql
-- Simple insert
INSERT INTO users (id, email, full_name, created_at)
VALUES (
    'user-123',
    'john@example.com',
    'John Doe',
    NOW()
);

-- Insert with parameterized queries (SAFE - prevents SQL injection)
INSERT INTO users (email, full_name, password_hash)
VALUES ($1, $2, $3);
-- In your code: db.query(query, [email, fullName, passwordHash]);

-- Insert and return the created row
INSERT INTO users (email, full_name)
VALUES ('jane@example.com', 'Jane Doe')
RETURNING id, email, full_name, created_at;
```

#### READ (Select)

```sql
-- Get all users
SELECT * FROM users;

-- Get specific columns
SELECT id, email, full_name FROM users;

-- Get one user
SELECT * FROM users WHERE id = $1;

-- Filter
SELECT * FROM users WHERE email LIKE '%@example.com';

-- Multiple conditions
SELECT * FROM users
WHERE created_at > NOW() - INTERVAL '7 days'
  AND is_active = true;

-- Order results
SELECT * FROM users ORDER BY created_at DESC;

-- Limit results (pagination)
SELECT * FROM users
ORDER BY created_at DESC
LIMIT 10 OFFSET 20;  -- Skip 20, return next 10 (page 3 if page size is 10)
```

#### UPDATE (Modify)

```sql
-- Update a single row
UPDATE users
SET full_name = 'John Smith'
WHERE id = $1;

-- Update multiple columns
UPDATE users
SET full_name = $1, email = $2
WHERE id = $3;

-- Update and return changed row
UPDATE users
SET full_name = $1
WHERE id = $2
RETURNING id, email, full_name, updated_at;

-- Partial updates (only update provided fields)
-- Handle in application logic:
-- if (name) { SET full_name = name, ... }
UPDATE users
SET full_name = COALESCE($1, full_name),
    email = COALESCE($2, email)
WHERE id = $3;
```

#### DELETE (Remove)

```sql
-- Delete one record
DELETE FROM tasks WHERE id = $1;

-- Delete multiple records
DELETE FROM tasks WHERE project_id = $1;

-- Delete and return deleted row
DELETE FROM users WHERE id = $1
RETURNING id, email;
```

### Joins and Filtering

#### Inner Join

Returns rows that have matches in **both** tables.

```sql
-- Get users with their projects
SELECT 
    u.id, 
    u.email, 
    p.id as project_id,
    p.name as project_name
FROM users u
INNER JOIN projects p ON u.id = p.owner_id;

-- Returns only users who own projects
-- Users with no projects are excluded
```

#### Left Join

Returns **all** rows from the left table, and matches from the right table.

```sql
-- Get all users and their projects (if any)
SELECT 
    u.id, 
    u.email, 
    p.id as project_id,
    p.name as project_name
FROM users u
LEFT JOIN projects p ON u.id = p.owner_id;

-- Returns all users
-- If user has no projects, project_id and project_name are NULL
```

#### Real Example: Build API Response

**Requirement:** GET /users - return list of users with their profile data

```sql
-- Step 1: Start with users table
SELECT * FROM users;

-- Step 2: Add joined profile data
SELECT 
    u.*,
    p.avatar_url,
    p.bio,
    p.phone_number
FROM users u
LEFT JOIN user_profiles p ON u.id = p.user_id;

-- Step 3: Convert profile data to JSON object (optional in app code)
SELECT 
    u.*,
    row_to_json(p) as profile  -- Entire row as JSON
FROM users u
LEFT JOIN user_profiles p ON u.id = p.user_id;

-- Step 4: Add filtering
SELECT 
    u.*,
    row_to_json(p) as profile
FROM users u
LEFT JOIN user_profiles p ON u.id = p.user_id
WHERE u.email LIKE $1  -- Filter by email pattern

-- Step 5: Add sorting and pagination
SELECT 
    u.*,
    row_to_json(p) as profile
FROM users u
LEFT JOIN user_profiles p ON u.id = p.user_id
ORDER BY u.created_at DESC
LIMIT $1 OFFSET $2;
```

#### Dynamic Filtering Example

**User sends:** `/api/users?search=john&limit=20&page=1`

**Backend logic:**
```javascript
// Construct query dynamically based on input
let query = `SELECT u.*, row_to_json(p) as profile
             FROM users u
             LEFT JOIN user_profiles p ON u.id = p.user_id`;
let params = [];
let paramCount = 1;

// Add filter if provided
if (search) {
    query += ` WHERE u.full_name ILIKE $${paramCount}`;
    params.push(`%${search}%`);
    paramCount++;
}

// Add pagination
query += ` ORDER BY u.created_at DESC LIMIT $${paramCount} OFFSET $${paramCount + 1}`;
params.push(limit || 20);
params.push((page - 1) * limit);

// Execute
const result = await db.query(query, params);
```

### Indexing

#### What Is an Index?

An index is like a **lookup table** that lets the database find data instantly instead of scanning all rows.

**Analogy:** 

Imagine a book with no index. To find "chapter 4", you flip through pages one by one. An index says "Chapter 4 starts on page 43"—you jump directly there.

#### Why Indexes Matter

**Without index on email:**
```sql
SELECT * FROM users WHERE email = 'john@example.com';
-- Database scans 1 million rows checking each email
-- Time: seconds or minutes
```

**With index on email:**
```sql
SELECT * FROM users WHERE email = 'john@example.com';
-- Database looks up 'john@example.com' in index (milliseconds)
-- Index points to exact row location
-- Time: milliseconds
```

#### When to Create Indexes

Create indexes on columns used in:

1. **WHERE clauses** (filtering)
2. **JOIN conditions** (linking tables)
3. **ORDER BY clauses** (sorting)
4. **Frequently searched columns**

```sql
-- Common indexes for a typical application

-- Index users table for common lookups
CREATE INDEX idx_users_email ON users(email);        -- search by email
CREATE INDEX idx_users_created_at_desc ON users(created_at DESC);  -- sort by newest

-- Index tasks table for filtering and joining
CREATE INDEX idx_tasks_project_id ON tasks(project_id);     -- find tasks per project
CREATE INDEX idx_tasks_assigned_to ON tasks(assigned_to);   -- find tasks per user
CREATE INDEX idx_tasks_status ON tasks(status);             -- filter by status

-- Index junction table for many-to-many lookups
CREATE INDEX idx_project_members_user_id ON project_members(user_id);
CREATE INDEX idx_project_members_project_id ON project_members(project_id);

-- Composite index for sorting (important!)
CREATE INDEX idx_tasks_status_created ON tasks(status, created_at DESC);
-- Fast for: WHERE status = 'pending' ORDER BY created_at DESC
```

#### Rules for Indexing

**✅ DO Create Indexes For:**
- Foreign keys (used in joins)
- Frequently filtered columns (WHERE clauses)
- Sorting columns (ORDER BY)
- Search fields (LIKE, full-text)
- Unique identifiers (implicitly indexed)

**❌ DON'T Over-Index:**
- Every column has a cost (write operations slower)
- Indexes take disk space
- Maintenance overhead
- Index on columns in SELECT but not WHERE is usually wasted

**Index Size Trade-offs:**

| Index Count | Read Speed | Write Speed | Disk Usage |
|---|---|---|---|
| 0 (none) | 🐢 Slow | ⚡ Fast | Minimal |
| 5-10 (good) | ⚡ Fast | ✓ OK | Reasonable |
| 30+ (bad) | ⚡⚡ Fast | 🐢 Slow | Large |

#### Composite Indexes

Index multiple columns together for specific query patterns.

```sql
-- Query pattern: Filter by status, sort by date
SELECT * FROM tasks
WHERE status = 'pending'
ORDER BY created_at DESC;

-- Create composite index matching this pattern
CREATE INDEX idx_tasks_status_created_desc
ON tasks(status, created_at DESC);
-- MUCH faster than two separate indexes
```

#### Monitoring Indexes

```sql
-- Find unused indexes (eating space for no benefit)
SELECT schemaname, tablename, indexname
FROM pg_indexes
WHERE schemaname = 'public';

-- Check if index is being used
SELECT idx.relname,
       COALESCE(stat.idx_scan, 0) as scans
FROM pg_index i
JOIN pg_class idx ON idx.oid = i.indexrelid
LEFT JOIN pg_stat_user_indexes stat ON stat.indexrelid = i.indexrelid
ORDER BY stat.idx_scan DESC;

-- Drop unused index
DROP INDEX IF EXISTS idx_unused_column;
```

---

### Triggers

#### What Is a Trigger?

A trigger is a **function that automatically runs** when a specific database event occurs.

**When to Use:**
- ✅ Auto-update timestamps (created_at, updated_at)
- ✅ Validate data before insertion
- ✅ Maintain data consistency across tables
- ✅ Audit logging (track all changes)
- ❌ Complex business logic (belongs in application code)

#### Auto-Update Timestamps

**Problem:** Every UPDATE query must manually set `updated_at`:

```sql
-- Tedious: Must remember to set updated_at every time
UPDATE user_profiles
SET bio = $1, updated_at = NOW()
WHERE user_id = $2;

-- Easy to forget:
UPDATE user_profiles SET bio = $1 WHERE user_id = $2;
-- updated_at is NOT updated! ❌
```

**Solution: Trigger**

```sql
-- Create function
CREATE OR REPLACE FUNCTION update_timestamp()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Create trigger on users table
CREATE TRIGGER trigger_users_update_timestamp
BEFORE UPDATE ON users
FOR EACH ROW
EXECUTE FUNCTION update_timestamp();

-- Create trigger on projects table
CREATE TRIGGER trigger_projects_update_timestamp
BEFORE UPDATE ON projects
FOR EACH ROW
EXECUTE FUNCTION update_timestamp();

-- Now every UPDATE automatically sets updated_at
UPDATE users SET email = 'new@example.com' WHERE id = $1;
-- updated_at is automatically set! ✅
```

#### Trigger for Validation

```sql
-- Ensure task priority is valid
CREATE OR REPLACE FUNCTION validate_task_priority()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.priority < 1 OR NEW.priority > 5 THEN
        RAISE EXCEPTION 'Priority must be between 1 and 5';
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trigger_task_validate_priority
BEFORE INSERT OR UPDATE ON tasks
FOR EACH ROW
EXECUTE FUNCTION validate_task_priority();
```

#### Audit Logging with Triggers

```sql
-- Create audit table
CREATE TABLE audit_log (
    id BIGSERIAL PRIMARY KEY,
    table_name TEXT,
    operation TEXT,
    old_data JSONB,
    new_data JSONB,
    changed_at TIMESTAMP DEFAULT NOW()
);

-- Create audit function
CREATE OR REPLACE FUNCTION audit_changes()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO audit_log (table_name, operation, old_data, new_data)
    VALUES (
        TG_TABLE_NAME,
        TG_OP,
        row_to_json(OLD),
        row_to_json(NEW)
    );
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Create trigger on users table
CREATE TRIGGER trigger_users_audit
AFTER UPDATE OR DELETE ON users
FOR EACH ROW
EXECUTE FUNCTION audit_changes();

-- Now every change is logged
UPDATE users SET email = 'new@example.com' WHERE id = $1;
-- audit_log has a record of old_data and new_data
```

#### Trigger Pitfalls

⚠️ **Avoid Using Triggers For:**

- Complex business logic (hard to debug, hidden in database)
- Application-specific logic (belongs in code)
- Performance-critical operations (triggers add overhead)
- Calculations that change frequently (maintain in code instead)

**Why?** Triggers are:
- Hard to test
- Hidden (developers don't see them in code)
- Database-specific (PostgreSQL triggers don't port to MySQL)
- Difficult to debug

**Better Approach:** Keep business logic in application code.

```sql
-- ❌ Bad: Logic hidden in trigger
CREATE TRIGGER calculate_discount
BEFORE INSERT ON orders
FOR EACH ROW
EXECUTE FUNCTION apply_loyalty_discount();

-- ✅ Good: Logic in application code
app.post('/orders', async (req, res) => {
    let order = req.body;
    if (user.loyaltyPoints > 1000) {
        order.discount = 10;  // Clear, testable, auditable
    }
    await db.query('INSERT INTO orders ...', [order]);
});
```

---

## Common Pitfalls & Debugging

### Pitfall 1: Missing Indexes

**Symptom:** Queries start slow as data grows

**Detection:**
```sql
-- Check query execution plan
EXPLAIN ANALYZE SELECT * FROM users WHERE email = $1;

-- Look for "Seq Scan" (bad) vs "Index Scan" (good)
-- Seq Scan on users (cost=0.00..35.50 rows=1 width=200)
--   Filter: (email = 'john@example.com')
-- (2 rows)

-- If Seq Scan on large table → CREATE INDEX
CREATE INDEX idx_users_email ON users(email);

-- Run again with index
EXPLAIN ANALYZE SELECT * FROM users WHERE email = $1;
-- Index Scan using idx_users_email on users
-- (2 rows)
```

### Pitfall 2: N+1 Query Problem

**Problem:** Fetch 100 users, then for each user fetch their profile

```javascript
// ❌ BAD: 101 queries total (1 + 100)
const users = await db.query('SELECT * FROM users');
for (const user of users) {
    user.profile = await db.query(
        'SELECT * FROM user_profiles WHERE user_id = $1',
        [user.id]
    );
}

// ✅ GOOD: 1 query with join
const users = await db.query(`
    SELECT u.*, row_to_json(p) as profile
    FROM users u
    LEFT JOIN user_profiles p ON u.id = p.user_id
`);
```

### Pitfall 3: Missing NOT NULL Constraints

**Problem:** Data becomes inconsistent when NULLs creep in

```sql
-- ❌ Too nullable
CREATE TABLE products (
    id UUID PRIMARY KEY,
    name TEXT,        -- What if NULL?
    price DECIMAL,    -- What if NULL?
    description TEXT
);

-- Later: Some products have no name. Which name should appear?
SELECT name FROM products;
-- Returns: "Widget", NULL, "Gadget", NULL, ...

-- ✅ Explicit constraints
CREATE TABLE products (
    id UUID PRIMARY KEY,
    name TEXT NOT NULL,
    price DECIMAL NOT NULL,
    description TEXT              -- Optional: nullable OK here
);
```

### Pitfall 4: Forgetting Referential Integrity

**Problem:** Deleting a user breaks all related records

```sql
-- ❌ No constraint
CREATE TABLE tasks (
    id UUID PRIMARY KEY,
    assigned_to UUID,  -- Points to users.id but no constraint!
    title TEXT
);

-- Can delete user, leaving orphan tasks
DELETE FROM users WHERE id = 'user-123';
-- Tasks still reference non-existent user

-- ✅ With constraint
CREATE TABLE tasks (
    id UUID PRIMARY KEY,
    assigned_to UUID REFERENCES users(id) ON DELETE SET NULL,
    title TEXT
);

-- Delete user → tasks are unassigned (clean state)
```

### Pitfall 5: Not Using Parameterized Queries

**Problem:** SQL Injection vulnerability

```sql
-- ❌ DANGEROUS: String concatenation
const email = req.body.email;  // User input
const query = `SELECT * FROM users WHERE email = '${email}'`;
// If user sends: ' OR '1'='1' → Gets all users!

-- ✅ SAFE: Parameterized query
const query = `SELECT * FROM users WHERE email = $1`;
await db.query(query, [email]);
// User input is escaped, safe from injection
```

### Pitfall 6: Wrong Data Type Choices

**Problem:** Using FLOAT for money

```sql
-- ❌ Precision lost
CREATE TABLE orders (
    id UUID PRIMARY KEY,
    amount FLOAT  -- 0.1 + 0.2 = 0.30000000000000004 !
);

-- ✅ Exact values
CREATE TABLE orders (
    id UUID PRIMARY KEY,
    amount DECIMAL(10, 2)  -- Always exactly 99.99
);
```

### Pitfall 7: Missing Composite Indexes

**Problem:** Query uses multiple conditions but only single-column indexes exist

```sql
-- ❌ Two separate indexes
CREATE INDEX idx_tasks_status ON tasks(status);
CREATE INDEX idx_tasks_created ON tasks(created_at);

-- Query uses both - only one index used, other is filtered in application
SELECT * FROM tasks 
WHERE status = 'pending' 
ORDER BY created_at DESC;

-- ✅ Composite index
CREATE INDEX idx_tasks_status_created 
ON tasks(status, created_at DESC);
-- Now the index handles both filtering AND sorting
```

### Pitfall 8: Not Using Transactions

**Problem:** Partial updates fail silently

```javascript
// ❌ Not atomic
async function transferMoney(fromUser, toUser, amount) {
    await db.query('UPDATE accounts SET balance = balance - $1 WHERE user_id = $2',
        [amount, fromUser]);
    // Server crashes here... toUser never gets the money!
    await db.query('UPDATE accounts SET balance = balance + $1 WHERE user_id = $2',
        [amount, toUser]);
}

// ✅ Atomic transaction
async function transferMoney(fromUser, toUser, amount) {
    await db.query('BEGIN');
    try {
        await db.query('UPDATE accounts SET balance = balance - $1 WHERE user_id = $2',
            [amount, fromUser]);
        await db.query('UPDATE accounts SET balance = balance + $1 WHERE user_id = $2',
            [amount, toUser]);
        await db.query('COMMIT');
    } catch (error) {
        await db.query('ROLLBACK');
        throw error;
    }
}
```

### Pitfall 9: Schema Migrations Without Testing

**Problem:** Migration fails in production, hard to rollback

```sql
-- ❌ Risky: What if this fails halfway?
CREATE TABLE users AS SELECT * FROM old_users;
DROP TABLE old_users;

-- ✅ Safe: Test rollback locally first
-- 1. Write migration file
-- 2. Test locally: dbmate up (verify works)
-- 3. Test rollback: dbmate down (verify rollback works)
-- 4. Run again: dbmate up (verify reproducible)
-- 5. Commit migration
-- 6. Deploy with confidence
```

---

## Quick Revision Cheat Sheet

### Essential Commands

```sql
-- Create table
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email TEXT NOT NULL UNIQUE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Insert
INSERT INTO users (email) VALUES ('john@example.com')
RETURNING id, email, created_at;

-- Select
SELECT * FROM users WHERE email = $1;

-- Update
UPDATE users SET email = $1 WHERE id = $2
RETURNING *;

-- Delete
DELETE FROM users WHERE id = $1;

-- Join
SELECT u.*, p.* FROM users u
LEFT JOIN user_profiles p ON u.id = p.user_id;

-- Group and aggregate
SELECT status, COUNT(*) as count
FROM tasks GROUP BY status;

-- Create index
CREATE INDEX idx_users_email ON users(email);

-- Create trigger
CREATE FUNCTION update_timestamp() RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trigger_update_timestamp
BEFORE UPDATE ON users FOR EACH ROW
EXECUTE FUNCTION update_timestamp();
```

### Data Types Quick Reference

| Purpose | Type | Example |
|---|---|---|
| ID (distributed) | UUID | `gen_random_uuid()` |
| ID (sequential) | BIGSERIAL | Auto-incrementing |
| Integers | BIGINT | User ages, counts |
| Decimals | DECIMAL(10,2) | Money: $99.99 |
| Approximate | FLOAT | Ratings: 4.5 |
| Text (flexible) | TEXT | Emails, names, bios |
| Text (fixed) | CHAR(2) | Country codes: US |
| Dates | DATE | 2025-01-31 |
| Times | TIMESTAMP WITH TIME ZONE | 2025-01-31 14:30:00+00 |
| Boolean | BOOLEAN | true/false |
| JSON (flexible) | JSONB | Nested objects |
| Sets of values | ENUM | Status types |
| Lists | ARRAY | Tags, skills |

### Constraint Quick Reference

```sql
-- Enforce unique value
CREATE TABLE users (
    email TEXT UNIQUE
);

-- Prevent NULL
CREATE TABLE users (
    email TEXT NOT NULL
);

-- Reference another table
CREATE TABLE tasks (
    project_id UUID REFERENCES projects(id)
);

-- Custom validation
CREATE TABLE tasks (
    priority INTEGER CHECK (priority >= 1 AND priority <= 5)
);

-- Primary key (unique + not null)
CREATE TABLE users (
    id UUID PRIMARY KEY
);
```

### Query Patterns

```sql
-- Pagination
SELECT * FROM users 
ORDER BY created_at DESC 
LIMIT 20 OFFSET 40;  -- Page 3 (if page size = 20)

-- Filter multiple conditions
SELECT * FROM tasks 
WHERE status = $1 
  AND assigned_to = $2 
  AND priority > 2;

-- Filter with OR
SELECT * FROM tasks 
WHERE status = 'pending' 
  OR status = 'in_progress';

-- Search with LIKE (case insensitive)
SELECT * FROM users 
WHERE full_name ILIKE '%john%';

-- Count aggregates
SELECT 
    status, 
    COUNT(*) as task_count,
    AVG(priority) as avg_priority
FROM tasks 
GROUP BY status;

-- Join with grouped data
SELECT 
    u.email,
    COUNT(t.id) as task_count
FROM users u
LEFT JOIN tasks t ON u.id = t.assigned_to
GROUP BY u.id, u.email;
```

### Performance Tips

1. **Always add indexes** on foreign keys, filter columns, and sort columns
2. **Use EXPLAIN ANALYZE** to see if queries are using indexes
3. **Use `LIMIT` + `OFFSET`** for pagination, not fetching all rows
4. **Avoid SELECT \*** (be explicit about which columns you need)
5. **Use parameterized queries** (prevents SQL injection + caching)
6. **Use transactions** for multi-step operations
7. **Denormalize carefully** (only when performance testing shows bottleneck)
8. **Monitor slow queries** with PostgreSQL logs
9. **Keep triggers minimal** (business logic in code)
10. **Test migrations locally** before deploying

### Migration Workflow

```bash
# Create migration
dbmate new create_users_table

# Edit migration file (up and down sections)

# Test locally
dbmate up      # Apply
dbmate down    # Rollback
dbmate up      # Apply again (verify idempotent)

# Deploy to staging
dbmate up

# Deploy to production
dbmate up

# If something breaks
dbmate down    # Rollback safely
```

### Common Relationships

```sql
-- One-to-One: User ↔ UserProfile
CREATE TABLE user_profiles (
    user_id UUID PRIMARY KEY,
    avatar_url TEXT,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- One-to-Many: Project → Tasks
CREATE TABLE tasks (
    id UUID PRIMARY KEY,
    project_id UUID NOT NULL,
    FOREIGN KEY (project_id) REFERENCES projects(id) ON DELETE CASCADE
);

-- Many-to-Many: Users ↔ Projects
CREATE TABLE project_members (
    project_id UUID NOT NULL,
    user_id UUID NOT NULL,
    role TEXT,
    PRIMARY KEY (project_id, user_id),
    FOREIGN KEY (project_id) REFERENCES projects(id) ON DELETE CASCADE,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

---

## Conclusion

Mastering databases is critical for backend engineering. This guide covers 80% of what you'll encounter day-to-day:

1. **Design** - Create proper schemas with relationships
2. **Migrate** - Use migrations to version control schema changes
3. **Query** - Write efficient SQL with joins, filters, and pagination
4. **Optimize** - Add indexes strategically, avoid N+1 problems
5. **Maintain** - Use constraints, triggers, and transactions

**Next Steps:**

1. Practice designing schemas for different domains
2. Write migrations for your own projects
3. Profile your queries with EXPLAIN ANALYZE
4. Learn your application framework's ORM
5. Understand your production database's slow query logs

Remember: A well-designed database is the foundation of a scalable, reliable backend system.
