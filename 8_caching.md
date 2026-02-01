# Mastering Caching for Backend Systems

A comprehensive guide to understanding caching mechanisms, strategies, and implementation patterns for high-performance backend applications.

---

## Table of Contents

1. [What Is Caching?](#what-is-caching)
   - [The One-Line Definition](#the-one-line-definition)
   - [Technical Definition](#technical-definition)
   - [Why Caching Matters](#why-caching-matters)
2. [Real-World Examples](#real-world-examples)
   - [Google Search](#google-search)
   - [Netflix and CDNs](#netflix-and-cdns)
   - [Twitter Trending Topics](#twitter-trending-topics)
3. [Levels of Caching](#levels-of-caching)
   - [Network Level Caching](#network-level-caching)
   - [Hardware Level Caching](#hardware-level-caching)
   - [Software/Application Level Caching](#softwareapplication-level-caching)
4. [Content Delivery Networks (CDN)](#content-delivery-networks-cdn)
   - [How CDNs Work](#how-cdns-work)
   - [CDN Architecture](#cdn-architecture)
5. [DNS Caching](#dns-caching)
   - [DNS Resolution Process](#dns-resolution-process)
   - [Caching Layers in DNS](#caching-layers-in-dns)
6. [In-Memory Databases](#in-memory-databases)
   - [Why RAM Over Disk?](#why-ram-over-disk)
   - [Redis and Memcached](#redis-and-memcached)
7. [Caching Strategies](#caching-strategies)
   - [Cache-Aside (Lazy Loading)](#cache-aside-lazy-loading)
   - [Write-Through](#write-through)
   - [Write-Behind (Write-Back)](#write-behind-write-back)
   - [Read-Through](#read-through)
8. [Cache Eviction Policies](#cache-eviction-policies)
   - [No Eviction](#no-eviction)
   - [LRU (Least Recently Used)](#lru-least-recently-used)
   - [LFU (Least Frequently Used)](#lfu-least-frequently-used)
   - [TTL (Time To Live)](#ttl-time-to-live)
9. [Practical Use Cases](#practical-use-cases)
   - [Database Query Caching](#database-query-caching)
   - [Session Storage](#session-storage)
   - [API Response Caching](#api-response-caching)
   - [Rate Limiting](#rate-limiting)
10. [Implementation with FastAPI and Redis](#implementation-with-fastapi-and-redis)
11. [Common Pitfalls & Debugging](#common-pitfalls--debugging)
12. [Quick Revision Cheat Sheet](#quick-revision-cheat-sheet)

---

## What Is Caching?

### The One-Line Definition

> **Caching is a mechanism to decrease the time and effort required to perform repetitive work.**

That's it. Every caching implementation, from CPU registers to global CDNs, follows this principle.

### Technical Definition

Caching is keeping a **subset of data** from a primary storage in a **faster, more accessible location** based on:

- **Frequency of access** - How often is this data requested?
- **Recency of access** - Was this data accessed recently?
- **Probability of future access** - Will this data likely be needed soon?
- **Cost of recomputation** - How expensive is it to regenerate this data?

### Why Caching Matters

For high-performance applications tracking latency in **milliseconds or microseconds**, caching is not optional—it's essential.

| Without Caching | With Caching |
|-----------------|--------------|
| Every request hits the database | Most requests served from memory |
| High latency (100-500ms) | Low latency (1-10ms) |
| Database overloaded | Database load reduced by 80-99% |
| High compute costs | Reduced infrastructure costs |
| Poor user experience | Instant responses |

#### Real-World Analogy

Think of caching like a **desk organizer**:

- Your **filing cabinet** (database) stores all documents, but retrieving a file takes time—you have to walk there, find the drawer, locate the folder.
- Your **desk** (cache) keeps frequently-used documents within arm's reach—instant access.
- You don't keep everything on your desk (limited space), only what you use often.
- When you finish a project, you return documents to the cabinet and bring new ones to your desk.

---

## Real-World Examples

### Google Search

**The Problem:**

When you search "what is the weather today," Google must:
1. Crawl billions of web pages
2. Run complex ranking algorithms
3. Apply personalization
4. Return results

This process is **computationally expensive**. Millions of users search weather-related queries daily.

**The Solution:**

Google uses a **distributed in-memory caching system**:

```
User searches "weather in New York"
        ↓
┌─────────────────────────────────────┐
│         Check Cache                 │
│  Key: "weather_new_york_2025-01-31" │
└─────────────────────────────────────┘
        ↓
   ┌────┴────┐
   │         │
Cache Hit  Cache Miss
   ↓         ↓
Return    Run algorithms
instantly  → Cache result
           → Return result
```

**Result:** Sub-100ms response times for billions of queries.

---

### Netflix and CDNs

**The Problem:**

Netflix streams **terabytes of video data** to millions of users worldwide. Streaming from a single data center in the US would mean:
- Users in India experience 200-500ms latency
- Massive bandwidth costs
- Poor video quality and buffering

**The Solution:**

Netflix uses **Content Delivery Networks (CDNs)** with edge servers worldwide:

```
                    ┌─────────────────┐
                    │  Origin Server  │
                    │   (US Data      │
                    │    Center)      │
                    └────────┬────────┘
                             │
         ┌───────────────────┼───────────────────┐
         │                   │                   │
    ┌────┴────┐        ┌────┴────┐        ┌────┴────┐
    │  Edge   │        │  Edge   │        │  Edge   │
    │ Server  │        │ Server  │        │ Server  │
    │ (Tokyo) │        │ (London)│        │ (Mumbai)│
    └────┬────┘        └────┬────┘        └────┬────┘
         │                   │                   │
    Users in Asia      Users in EU        Users in India
```

**How It Works:**
1. Popular content is **cached at edge servers** close to users
2. User in Mumbai requests a movie
3. Edge server in Mumbai serves the content (not the US origin)
4. **Latency reduced from 200ms to 20ms**

---

### Twitter Trending Topics

**The Problem:**

Calculating trending topics requires:
- Analyzing millions of tweets in real-time
- Running ML algorithms for trend detection
- Processing terabytes of data
- GPU-intensive computation

If every user's request triggered this computation, servers would crash instantly.

**The Solution:**

Twitter **pre-computes and caches** trending topics:

```
Every 5 minutes:
┌──────────────────────────────────────────┐
│ 1. Collect tweets from last hour         │
│ 2. Run trend detection algorithms        │
│ 3. Calculate regional trends             │
│ 4. Store results in Redis cache          │
│ 5. Set TTL = 5 minutes                   │
└──────────────────────────────────────────┘

When user requests trending:
┌──────────────────────────────────────────┐
│ 1. Check Redis cache                     │
│ 2. Return pre-computed trends instantly  │
│ 3. No computation needed                 │
└──────────────────────────────────────────┘
```

**Why This Works:**
- Trends don't change every second
- A 5-minute cache is acceptable for this use case
- Millions of users get instant responses

---

## Levels of Caching

As a backend engineer, you'll encounter caching at three primary levels:

```
┌─────────────────────────────────────────────────────────────┐
│                    CACHING LEVELS                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. NETWORK LEVEL                                           │
│     ├── CDN (Content Delivery Networks)                     │
│     └── DNS Caching                                         │
│                                                             │
│  2. HARDWARE LEVEL                                          │
│     ├── CPU Cache (L1, L2, L3)                              │
│     └── RAM (Primary Memory)                                │
│                                                             │
│  3. SOFTWARE/APPLICATION LEVEL                              │
│     ├── In-Memory Databases (Redis, Memcached)              │
│     ├── Application-Level Cache                             │
│     └── ORM/Query Cache                                     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Network Level Caching

Caches data at network infrastructure level:
- **CDN**: Caches static assets (images, videos, HTML) at edge locations
- **DNS**: Caches domain-to-IP mappings to speed up resolution

### Hardware Level Caching

Built into computer architecture:
- **CPU Caches (L1, L2, L3)**: Fastest, smallest, closest to CPU
- **RAM**: Faster than disk, stores actively-used data

### Software/Application Level Caching

What you'll implement as a backend engineer:
- **Redis/Memcached**: In-memory key-value stores
- **Application cache**: In-process memory caching
- **Query cache**: Database query result caching

---

## Content Delivery Networks (CDN)

### What Is a CDN?

A CDN is a network of geographically distributed servers that cache content close to end users.

### How CDNs Work

```
Step-by-Step CDN Request Flow:
═══════════════════════════════════════════════════════════════

1. USER REQUEST
   User in Tokyo enters: https://example.com/video.mp4
                    ↓
2. DNS RESOLUTION
   CDN's DNS routes request to nearest Point of Presence (PoP)
   → Selects Tokyo PoP based on:
     • Geographic location
     • Network conditions
     • Server load
                    ↓
3. EDGE SERVER CHECK
   Tokyo edge server receives request
   → Checks local cache for video.mp4
                    ↓
           ┌───────┴───────┐
           │               │
      CACHE HIT       CACHE MISS
           ↓               ↓
   Return video      Fetch from origin
   immediately       (US data center)
                          ↓
                     Cache locally
                          ↓
                     Return video
                          ↓
                     Future requests
                     served from cache
```

### CDN Architecture

```
                         ┌─────────────────────┐
                         │   ORIGIN SERVER     │
                         │   (Primary Data)    │
                         └──────────┬──────────┘
                                    │
            ┌───────────────────────┼───────────────────────┐
            │                       │                       │
   ┌────────┴────────┐    ┌────────┴────────┐    ┌────────┴────────┐
   │    PoP: US      │    │   PoP: Europe   │    │   PoP: Asia     │
   │ ┌─────────────┐ │    │ ┌─────────────┐ │    │ ┌─────────────┐ │
   │ │Edge Server 1│ │    │ │Edge Server 1│ │    │ │Edge Server 1│ │
   │ │Edge Server 2│ │    │ │Edge Server 2│ │    │ │Edge Server 2│ │
   │ │Edge Server 3│ │    │ │Edge Server 3│ │    │ │Edge Server 3│ │
   │ └─────────────┘ │    │ └─────────────┘ │    │ └─────────────┘ │
   └────────┬────────┘    └────────┬────────┘    └────────┬────────┘
            │                      │                      │
       US Users              European Users          Asian Users
```

### Key CDN Concepts

| Concept | Description |
|---------|-------------|
| **Origin Server** | Primary server with original content |
| **Edge Server** | Cache server close to users |
| **PoP (Point of Presence)** | Collection of edge servers in a region |
| **TTL (Time To Live)** | How long content stays cached |
| **Cache Hit** | Content found in cache |
| **Cache Miss** | Content not in cache, fetched from origin |

---

## DNS Caching

### What Is DNS?

DNS (Domain Name System) translates human-readable domain names (google.com) to IP addresses (142.250.185.78).

### DNS Resolution Process

```
User types: www.example.com
              │
              ▼
┌─────────────────────────────────────────────────────────────────┐
│ STEP 1: Check Local Caches                                      │
│                                                                 │
│   Browser Cache → OS Cache → Router Cache                       │
│   (If found, return immediately - CACHE HIT)                    │
└────────────────────────┬────────────────────────────────────────┘
                         │ (Cache Miss)
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│ STEP 2: Query Recursive Resolver                                │
│                                                                 │
│   ISP's DNS Server (or Google 8.8.8.8, Cloudflare 1.1.1.1)      │
│   → Checks its own cache                                        │
└────────────────────────┬────────────────────────────────────────┘
                         │ (Cache Miss)
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│ STEP 3: Query Root Server                                       │
│                                                                 │
│   Root server doesn't know example.com's IP                     │
│   → Returns: "Ask the .com TLD server"                          │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│ STEP 4: Query TLD (Top-Level Domain) Server                     │
│                                                                 │
│   .com TLD server doesn't know exact IP                         │
│   → Returns: "Ask example.com's authoritative nameserver"       │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│ STEP 5: Query Authoritative Nameserver                          │
│                                                                 │
│   example.com's nameserver knows the IP!                        │
│   → Returns: 93.184.216.34                                      │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│ STEP 6: Cache at All Levels                                     │
│                                                                 │
│   Recursive Resolver caches → OS caches → Browser caches        │
│   Future requests skip all these steps!                         │
└─────────────────────────────────────────────────────────────────┘
```

### Caching Layers in DNS

DNS caching happens at **multiple levels** to minimize latency:

| Layer | Location | TTL | Purpose |
|-------|----------|-----|---------|
| Browser Cache | Chrome, Firefox | 1-10 min | Fastest lookup |
| OS Cache | Windows, macOS, Linux | Minutes to hours | System-wide |
| Router Cache | Home/Office router | Hours | Network-wide |
| ISP Resolver | ISP's DNS server | Hours to days | Millions of users |
| Authoritative | Domain's nameserver | Hours to days | Canonical source |

**Why So Many Layers?**

Without caching, every DNS query would traverse the entire hierarchy (5+ network hops). With caching, most queries resolve in **1 hop or less**.

---

## In-Memory Databases

### Why RAM Over Disk?

Understanding the storage hierarchy is crucial:

```
┌─────────────────────────────────────────────────────────────────┐
│                    STORAGE HIERARCHY                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌────────────────┐                                             │
│  │   CPU CACHE    │  Speed: ~1 nanosecond                       │
│  │   (L1/L2/L3)   │  Size: KB to MB                             │
│  └───────┬────────┘  Cost: $$$$$                                │
│          │                                                      │
│  ┌───────▼────────┐                                             │
│  │      RAM       │  Speed: ~100 nanoseconds                    │
│  │ (Main Memory)  │  Size: 8-128 GB                             │
│  └───────┬────────┘  Cost: $$$                                  │
│          │                                                      │
│  ┌───────▼────────┐                                             │
│  │      SSD       │  Speed: ~100 microseconds                   │
│  │  (Fast Disk)   │  Size: 256 GB - 4 TB                        │
│  └───────┬────────┘  Cost: $$                                   │
│          │                                                      │
│  ┌───────▼────────┐                                             │
│  │      HDD       │  Speed: ~10 milliseconds                    │
│  │ (Traditional)  │  Size: 1-20 TB                              │
│  └────────────────┘  Cost: $                                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Speed Comparison

| Storage Type | Access Time | Comparison |
|--------------|-------------|------------|
| L1 Cache | 1 ns | 1 second |
| RAM | 100 ns | 1.5 minutes |
| SSD | 100 μs | 1.5 days |
| HDD | 10 ms | 4 months |

**Key Insight:** RAM is **100,000x faster** than HDD. This is why in-memory databases are so powerful.

### RAM Characteristics

| Feature | RAM | Disk (HDD/SSD) |
|---------|-----|----------------|
| Speed | Nanoseconds | Microseconds to Milliseconds |
| Capacity | 8-128 GB typical | 256 GB - 20 TB |
| Cost | Expensive ($5-10/GB) | Cheap ($0.01-0.10/GB) |
| Persistence | **Volatile** (lost on power off) | Persistent |
| Access Pattern | Random access (uniform speed) | Sequential faster than random |

### Redis and Memcached

These are the two most popular in-memory caching solutions:

| Feature | Redis | Memcached |
|---------|-------|-----------|
| Data Structures | Strings, Lists, Sets, Hashes, Sorted Sets | Strings only |
| Persistence | Yes (RDB, AOF) | No |
| Replication | Yes | No |
| Pub/Sub | Yes | No |
| Lua Scripting | Yes | No |
| Memory Efficiency | Less efficient | More efficient |
| Use Case | Feature-rich caching, sessions, queues | Simple, high-performance caching |

**For most backend applications, Redis is the recommended choice** due to its rich data structures and persistence options.

---

## Caching Strategies

### Cache-Aside (Lazy Loading)

**The most common caching pattern.** Application manages cache explicitly.

```
┌─────────────────────────────────────────────────────────────────┐
│                   CACHE-ASIDE PATTERN                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Client Request                                                │
│        │                                                        │
│        ▼                                                        │
│   ┌─────────┐      1. Check cache                               │
│   │  Cache  │◄─────────────────┐                                │
│   └────┬────┘                  │                                │
│        │                       │                                │
│   Cache Hit?                   │                                │
│   ┌────┴────┐                  │                                │
│   │         │                  │                                │
│  YES       NO                  │                                │
│   │         │                  │                                │
│   ▼         ▼                  │                                │
│ Return   2. Query Database     │                                │
│ cached   ┌─────────┐           │                                │
│ data     │   DB    │           │                                │
│          └────┬────┘           │                                │
│               │                │                                │
│               ▼                │                                │
│        3. Store in cache ──────┘                                │
│               │                                                 │
│               ▼                                                 │
│        4. Return data                                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**When to Use:**
- Read-heavy workloads
- Data that doesn't change frequently
- When stale data is acceptable for short periods

**Pros:**
- Only requested data is cached (no wasted memory)
- Cache failures don't break the system (fallback to DB)
- Simple to implement

**Cons:**
- First request always slow (cache miss)
- Potential for stale data

#### FastAPI Implementation

```python
import redis
import json
from fastapi import FastAPI, HTTPException
from typing import Optional
import asyncio

app = FastAPI()

# Redis connection
redis_client = redis.Redis(host='localhost', port=6379, db=0, decode_responses=True)

# Simulated database
fake_db = {
    "user:1": {"id": 1, "name": "Alice", "email": "alice@example.com"},
    "user:2": {"id": 2, "name": "Bob", "email": "bob@example.com"},
    "user:3": {"id": 3, "name": "Charlie", "email": "charlie@example.com"},
}

# Simulate slow database query
async def get_user_from_db(user_id: int) -> Optional[dict]:
    """Simulates a slow database query (100ms)"""
    await asyncio.sleep(0.1)  # Simulate DB latency
    return fake_db.get(f"user:{user_id}")


@app.get("/users/{user_id}")
async def get_user(user_id: int):
    """
    Cache-Aside Pattern:
    1. Check cache first
    2. If miss, query database
    3. Store result in cache
    4. Return data
    """
    cache_key = f"user:{user_id}"
    
    # Step 1: Check cache
    cached_data = redis_client.get(cache_key)
    
    if cached_data:
        # Cache HIT - return cached data
        print(f"Cache HIT for {cache_key}")
        return {"source": "cache", "data": json.loads(cached_data)}
    
    # Cache MISS - query database
    print(f"Cache MISS for {cache_key}")
    user = await get_user_from_db(user_id)
    
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    
    # Step 3: Store in cache with 5-minute TTL
    redis_client.setex(
        cache_key,
        300,  # TTL in seconds (5 minutes)
        json.dumps(user)
    )
    
    # Step 4: Return data
    return {"source": "database", "data": user}
```

---

### Write-Through

**Every write goes to both cache AND database simultaneously.**

```
┌─────────────────────────────────────────────────────────────────┐
│                   WRITE-THROUGH PATTERN                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Client Write Request                                          │
│        │                                                        │
│        ▼                                                        │
│   ┌─────────────────┐                                           │
│   │   Application   │                                           │
│   └────────┬────────┘                                           │
│            │                                                    │
│    ┌───────┴───────┐                                            │
│    │               │                                            │
│    ▼               ▼                                            │
│ ┌─────────┐   ┌─────────┐                                       │
│ │  Cache  │   │   DB    │                                       │
│ └─────────┘   └─────────┘                                       │
│                                                                 │
│ Both updated simultaneously (or in transaction)                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**When to Use:**
- Data consistency is critical
- You can tolerate slightly slower writes
- Read-heavy but writes must be immediately consistent

**Pros:**
- Cache is always fresh (no stale data)
- Reads always hit cache

**Cons:**
- Higher write latency (writing to two places)
- More complex write logic

#### FastAPI Implementation

```python
import redis
import json
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from typing import Optional

app = FastAPI()
redis_client = redis.Redis(host='localhost', port=6379, db=0, decode_responses=True)

# Simulated database (in production, use real DB)
fake_db = {}


class UserCreate(BaseModel):
    name: str
    email: str


class UserUpdate(BaseModel):
    name: Optional[str] = None
    email: Optional[str] = None


@app.post("/users")
async def create_user(user: UserCreate):
    """
    Write-Through Pattern:
    Write to BOTH database and cache in same operation
    """
    user_id = len(fake_db) + 1
    user_data = {"id": user_id, "name": user.name, "email": user.email}
    cache_key = f"user:{user_id}"
    
    # Write to database
    fake_db[cache_key] = user_data
    
    # Write to cache (simultaneously)
    redis_client.setex(
        cache_key,
        3600,  # 1 hour TTL
        json.dumps(user_data)
    )
    
    return {"message": "User created", "data": user_data}


@app.put("/users/{user_id}")
async def update_user(user_id: int, user: UserUpdate):
    """
    Write-Through: Update both DB and cache together
    """
    cache_key = f"user:{user_id}"
    
    # Get existing data
    existing = fake_db.get(cache_key)
    if not existing:
        raise HTTPException(status_code=404, detail="User not found")
    
    # Update fields
    if user.name:
        existing["name"] = user.name
    if user.email:
        existing["email"] = user.email
    
    # Write to database
    fake_db[cache_key] = existing
    
    # Write to cache (simultaneously)
    redis_client.setex(cache_key, 3600, json.dumps(existing))
    
    return {"message": "User updated", "data": existing}


@app.get("/users/{user_id}")
async def get_user(user_id: int):
    """
    With write-through, cache is always fresh.
    We can confidently read from cache.
    """
    cache_key = f"user:{user_id}"
    
    # Check cache first
    cached = redis_client.get(cache_key)
    if cached:
        return {"source": "cache", "data": json.loads(cached)}
    
    # Fallback to DB (shouldn't happen often with write-through)
    db_data = fake_db.get(cache_key)
    if not db_data:
        raise HTTPException(status_code=404, detail="User not found")
    
    # Re-populate cache
    redis_client.setex(cache_key, 3600, json.dumps(db_data))
    return {"source": "database", "data": db_data}
```

---

### Write-Behind (Write-Back)

**Writes go to cache immediately, then asynchronously to database.**

```
┌─────────────────────────────────────────────────────────────────┐
│                   WRITE-BEHIND PATTERN                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Client Write                                                  │
│        │                                                        │
│        ▼                                                        │
│   ┌─────────┐                                                   │
│   │  Cache  │ ◄── Immediate write (fast response)               │
│   └────┬────┘                                                   │
│        │                                                        │
│        │  (Async, batched)                                      │
│        ▼                                                        │
│   ┌─────────┐                                                   │
│   │   DB    │ ◄── Delayed write (background job)                │
│   └─────────┘                                                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**When to Use:**
- Write-heavy workloads
- Can tolerate eventual consistency
- Need fast write responses

**Pros:**
- Fastest write performance
- Reduces database load (batch writes)

**Cons:**
- Risk of data loss if cache fails before DB sync
- Complex to implement correctly
- Eventual consistency only

---

### Read-Through

**Cache sits between application and database. Application only talks to cache.**

```
┌─────────────────────────────────────────────────────────────────┐
│                   READ-THROUGH PATTERN                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌─────────────┐                                               │
│   │ Application │                                               │
│   └──────┬──────┘                                               │
│          │                                                      │
│          ▼                                                      │
│   ┌─────────────┐         ┌─────────────┐                       │
│   │    Cache    │ ◄─────► │   Database  │                       │
│   │  (manages   │         │             │                       │
│   │   loading)  │         │             │                       │
│   └─────────────┘         └─────────────┘                       │
│                                                                 │
│   Cache automatically fetches from DB on miss                   │
│   Application doesn't know about DB                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**When to Use:**
- When you want to abstract database access
- Using cache solutions that support this natively

---

### Strategy Comparison

| Strategy | Write Latency | Read Latency | Consistency | Complexity |
|----------|--------------|--------------|-------------|------------|
| Cache-Aside | N/A (manual) | Miss: High, Hit: Low | Eventual | Low |
| Write-Through | High | Always Low | Strong | Medium |
| Write-Behind | Low | Always Low | Eventual | High |
| Read-Through | N/A | Miss: High, Hit: Low | Eventual | Medium |

---

## Cache Eviction Policies

When cache memory is full, **something must be removed** to make room for new data.

### No Eviction

**Behavior:** When full, new writes fail with an error.

```python
# Redis configuration
# maxmemory-policy noeviction

# Result when full:
# redis.exceptions.ResponseError: OOM command not allowed
```

**When to Use:** Almost never. Only if you want strict control over what's cached.

---

### LRU (Least Recently Used)

**Evicts the item that hasn't been accessed for the longest time.**

```
Cache State (max 4 items):
┌────┬────┬────┬────┐
│ A  │ B  │ C  │ D  │  Current cache
└────┴────┴────┴────┘
  ↑
  Least recently used

Access sequence: D, C, B, D, C, B (A not accessed)

New item E arrives → Evict A (least recently used)

┌────┬────┬────┬────┐
│ E  │ B  │ C  │ D  │  Updated cache
└────┴────┴────┴────┘
```

**When to Use:**
- General-purpose caching
- When recent access is a good predictor of future access
- Most common choice

#### FastAPI Example with LRU-like Behavior

```python
import redis
import json
from fastapi import FastAPI
from collections import OrderedDict
from typing import Any

app = FastAPI()

# Simple in-memory LRU cache (for demonstration)
class LRUCache:
    def __init__(self, capacity: int):
        self.cache = OrderedDict()
        self.capacity = capacity
    
    def get(self, key: str) -> Any:
        if key not in self.cache:
            return None
        # Move to end (most recently used)
        self.cache.move_to_end(key)
        return self.cache[key]
    
    def put(self, key: str, value: Any) -> None:
        if key in self.cache:
            self.cache.move_to_end(key)
        self.cache[key] = value
        if len(self.cache) > self.capacity:
            # Remove least recently used (first item)
            self.cache.popitem(last=False)


# Create LRU cache with capacity of 100
cache = LRUCache(capacity=100)


@app.get("/products/{product_id}")
async def get_product(product_id: int):
    cache_key = f"product:{product_id}"
    
    # Check LRU cache
    cached = cache.get(cache_key)
    if cached:
        return {"source": "cache", "data": cached}
    
    # Simulate DB fetch
    product = {"id": product_id, "name": f"Product {product_id}", "price": product_id * 10}
    
    # Store in LRU cache
    cache.put(cache_key, product)
    
    return {"source": "database", "data": product}
```

---

### LFU (Least Frequently Used)

**Evicts the item accessed the fewest times.**

```
Cache State with access counts:
┌──────────┬──────────┬──────────┬──────────┐
│ A (5x)   │ B (10x)  │ C (6x)   │ D (23x)  │
└──────────┴──────────┴──────────┴──────────┘
     ↑
     Least frequently used

New item E arrives → Evict A (only 5 accesses)

┌──────────┬──────────┬──────────┬──────────┐
│ E (1x)   │ B (10x)  │ C (6x)   │ D (23x)  │
└──────────┴──────────┴──────────┴──────────┘
```

**When to Use:**
- When popular items should stay longer
- Content that has consistent, long-term popularity

**Cons:**
- New items get evicted quickly (low count)
- Old but no-longer-popular items may stay too long

---

### TTL (Time To Live)

**Items expire after a fixed time period.**

```python
import redis
from fastapi import FastAPI

app = FastAPI()
redis_client = redis.Redis(host='localhost', port=6379, db=0)


@app.get("/weather/{city}")
async def get_weather(city: str):
    cache_key = f"weather:{city}"
    
    # Check cache
    cached = redis_client.get(cache_key)
    if cached:
        return {"source": "cache", "data": cached.decode()}
    
    # Fetch from external API (simulated)
    weather_data = f"Sunny, 25°C in {city}"
    
    # Cache with 30-minute TTL
    redis_client.setex(
        cache_key,
        1800,  # 30 minutes in seconds
        weather_data
    )
    
    return {"source": "api", "data": weather_data}


@app.get("/trending")
async def get_trending():
    cache_key = "trending_topics"
    
    cached = redis_client.get(cache_key)
    if cached:
        return {"source": "cache", "data": cached.decode()}
    
    # Expensive computation (simulated)
    topics = ["#Python", "#FastAPI", "#Redis", "#Caching"]
    
    # Cache for 5 minutes
    redis_client.setex(cache_key, 300, str(topics))
    
    return {"source": "computed", "data": topics}
```

---

### Eviction Policy Comparison

| Policy | Best For | Pros | Cons |
|--------|----------|------|------|
| **No Eviction** | Strict control | Predictable | Writes fail when full |
| **LRU** | General purpose | Good for temporal locality | May evict frequently-used old items |
| **LFU** | Stable popularity | Keeps popular items | New items evicted quickly |
| **TTL** | Time-sensitive data | Simple, predictable | May evict before access |

**Redis Configuration:**

```bash
# Set max memory
maxmemory 100mb

# Set eviction policy
maxmemory-policy allkeys-lru  # LRU on all keys
# maxmemory-policy volatile-lru  # LRU only on keys with TTL
# maxmemory-policy allkeys-lfu  # LFU on all keys
# maxmemory-policy volatile-ttl  # Evict keys with shortest TTL
```

---

## Practical Use Cases

### Database Query Caching

**Problem:** Complex queries with multiple JOINs are slow and hit the database heavily.

**Solution:** Cache query results.

```python
import redis
import json
import hashlib
from fastapi import FastAPI
from typing import List
import asyncio

app = FastAPI()
redis_client = redis.Redis(host='localhost', port=6379, db=0, decode_responses=True)

# Simulated expensive database query
async def expensive_analytics_query() -> dict:
    """
    Simulates a complex query that:
    - Joins 5 tables
    - Aggregates millions of rows
    - Takes 2-3 seconds to execute
    """
    await asyncio.sleep(2)  # Simulate slow query
    return {
        "total_users": 150000,
        "active_users_today": 45000,
        "revenue_this_month": 1250000,
        "top_products": [
            {"id": 1, "name": "Widget Pro", "sales": 15000},
            {"id": 2, "name": "Gadget Plus", "sales": 12000},
            {"id": 3, "name": "Tool Master", "sales": 9500},
        ],
        "computed_at": "2025-01-31T10:00:00Z"
    }


@app.get("/dashboard/analytics")
async def get_dashboard_analytics():
    """
    Dashboard analytics endpoint.
    Without caching: 2-3 second response time
    With caching: <10ms response time
    """
    cache_key = "dashboard:analytics"
    
    # Check cache
    cached = redis_client.get(cache_key)
    if cached:
        return {
            "source": "cache",
            "data": json.loads(cached)
        }
    
    # Cache miss - run expensive query
    data = await expensive_analytics_query()
    
    # Cache for 1 hour (dashboard doesn't need real-time data)
    redis_client.setex(cache_key, 3600, json.dumps(data))
    
    return {
        "source": "database",
        "data": data
    }


@app.post("/dashboard/refresh")
async def refresh_dashboard():
    """
    Manually invalidate cache when needed
    """
    redis_client.delete("dashboard:analytics")
    return {"message": "Dashboard cache invalidated"}
```

**Real-World Examples:**
- **Amazon**: Caches product details, prices, inventory
- **Facebook**: Caches user profiles, friend lists
- **Twitter**: Caches trending topics, user timelines

---

### Session Storage

**Problem:** Storing sessions in database creates a DB call for **every request**.

**Solution:** Store sessions in Redis.

```python
import redis
import json
import uuid
import hashlib
from fastapi import FastAPI, HTTPException, Response, Cookie
from pydantic import BaseModel
from typing import Optional
from datetime import datetime

app = FastAPI()
redis_client = redis.Redis(host='localhost', port=6379, db=0, decode_responses=True)

# Session configuration
SESSION_TTL = 3600 * 24  # 24 hours


class LoginRequest(BaseModel):
    username: str
    password: str


class User(BaseModel):
    id: int
    username: str
    email: str
    role: str


# Fake user database
USERS_DB = {
    "alice": {"id": 1, "username": "alice", "email": "alice@example.com", 
              "password_hash": "hashed_password", "role": "admin"},
    "bob": {"id": 2, "username": "bob", "email": "bob@example.com",
            "password_hash": "hashed_password", "role": "user"},
}


def create_session(user_data: dict) -> str:
    """Create a new session in Redis"""
    session_id = str(uuid.uuid4())
    session_key = f"session:{session_id}"
    
    session_data = {
        "user_id": user_data["id"],
        "username": user_data["username"],
        "email": user_data["email"],
        "role": user_data["role"],
        "created_at": datetime.utcnow().isoformat(),
    }
    
    # Store session in Redis with TTL
    redis_client.setex(
        session_key,
        SESSION_TTL,
        json.dumps(session_data)
    )
    
    return session_id


def get_session(session_id: str) -> Optional[dict]:
    """Retrieve session from Redis"""
    session_key = f"session:{session_id}"
    session_data = redis_client.get(session_key)
    
    if session_data:
        # Refresh TTL on access
        redis_client.expire(session_key, SESSION_TTL)
        return json.loads(session_data)
    
    return None


def destroy_session(session_id: str) -> bool:
    """Delete session from Redis"""
    session_key = f"session:{session_id}"
    return redis_client.delete(session_key) > 0


@app.post("/auth/login")
async def login(login_req: LoginRequest, response: Response):
    """
    Login and create session in Redis
    """
    user = USERS_DB.get(login_req.username)
    
    if not user:
        raise HTTPException(status_code=401, detail="Invalid credentials")
    
    # In production, verify password hash here
    
    # Create session in Redis
    session_id = create_session(user)
    
    # Set session cookie
    response.set_cookie(
        key="session_id",
        value=session_id,
        httponly=True,
        max_age=SESSION_TTL,
        samesite="lax"
    )
    
    return {"message": "Login successful", "session_id": session_id}


@app.get("/auth/me")
async def get_current_user(session_id: Optional[str] = Cookie(None)):
    """
    Get current user from session.
    This is called on EVERY authenticated request.
    Redis makes this fast (<1ms).
    """
    if not session_id:
        raise HTTPException(status_code=401, detail="Not authenticated")
    
    session = get_session(session_id)
    
    if not session:
        raise HTTPException(status_code=401, detail="Session expired")
    
    return {"user": session}


@app.post("/auth/logout")
async def logout(response: Response, session_id: Optional[str] = Cookie(None)):
    """
    Logout and destroy session
    """
    if session_id:
        destroy_session(session_id)
    
    response.delete_cookie("session_id")
    return {"message": "Logged out"}
```

**Why Redis for Sessions?**
- Sessions are accessed on **every request**
- Database would be overwhelmed
- Redis: <1ms latency vs Database: 10-50ms

---

### API Response Caching

**Problem:** Your backend calls external APIs (weather, maps, payment) that have rate limits or per-call costs.

**Solution:** Cache API responses.

```python
import redis
import json
import httpx
from fastapi import FastAPI, HTTPException
from datetime import datetime

app = FastAPI()
redis_client = redis.Redis(host='localhost', port=6379, db=0, decode_responses=True)


# Simulated external API call
async def fetch_weather_from_api(city: str) -> dict:
    """
    In production, this would call a real weather API.
    Each call might cost $0.001 and have rate limits.
    """
    # Simulated API response
    return {
        "city": city,
        "temperature": 25,
        "conditions": "Sunny",
        "humidity": 60,
        "fetched_at": datetime.utcnow().isoformat()
    }


async def fetch_exchange_rates() -> dict:
    """
    Exchange rate APIs often have strict rate limits.
    Rates don't change by the second.
    """
    return {
        "base": "USD",
        "rates": {
            "EUR": 0.92,
            "GBP": 0.79,
            "INR": 83.12,
            "JPY": 148.50
        },
        "fetched_at": datetime.utcnow().isoformat()
    }


@app.get("/weather/{city}")
async def get_weather(city: str):
    """
    Cache weather data for 30 minutes.
    Weather doesn't change second by second.
    Reduces external API calls by 99%.
    """
    cache_key = f"weather:{city.lower()}"
    
    # Check cache
    cached = redis_client.get(cache_key)
    if cached:
        data = json.loads(cached)
        return {
            "source": "cache",
            "cached_at": data.get("fetched_at"),
            "data": data
        }
    
    # Fetch from external API
    weather = await fetch_weather_from_api(city)
    
    # Cache for 30 minutes
    redis_client.setex(cache_key, 1800, json.dumps(weather))
    
    return {
        "source": "api",
        "data": weather
    }


@app.get("/exchange-rates")
async def get_exchange_rates():
    """
    Cache exchange rates for 1 hour.
    Banks typically update rates once or twice per day.
    """
    cache_key = "exchange_rates"
    
    cached = redis_client.get(cache_key)
    if cached:
        return {"source": "cache", "data": json.loads(cached)}
    
    rates = await fetch_exchange_rates()
    redis_client.setex(cache_key, 3600, json.dumps(rates))
    
    return {"source": "api", "data": rates}
```

**Cost Savings Example:**

| Scenario | API Calls/Day | Cost at $0.001/call |
|----------|---------------|---------------------|
| No caching | 1,000,000 | $1,000/day |
| 30-min cache | 48 | $0.05/day |
| **Savings** | | **$999.95/day** |

---

### Rate Limiting

**Problem:** Prevent abuse, bot attacks, and server overload.

**Solution:** Track request counts per IP in Redis.

```python
import redis
import time
from fastapi import FastAPI, Request, HTTPException
from fastapi.responses import JSONResponse

app = FastAPI()
redis_client = redis.Redis(host='localhost', port=6379, db=0, decode_responses=True)


class RateLimiter:
    """
    Sliding window rate limiter using Redis.
    
    Example: 100 requests per minute per IP
    """
    
    def __init__(self, max_requests: int, window_seconds: int):
        self.max_requests = max_requests
        self.window_seconds = window_seconds
    
    def is_allowed(self, identifier: str) -> tuple[bool, dict]:
        """
        Check if request is allowed.
        Returns (allowed, info_dict)
        """
        key = f"ratelimit:{identifier}"
        current_time = int(time.time())
        window_start = current_time - self.window_seconds
        
        pipe = redis_client.pipeline()
        
        # Remove old entries outside the window
        pipe.zremrangebyscore(key, 0, window_start)
        
        # Count requests in current window
        pipe.zcard(key)
        
        # Add current request
        pipe.zadd(key, {str(current_time): current_time})
        
        # Set expiry on the key
        pipe.expire(key, self.window_seconds)
        
        results = pipe.execute()
        request_count = results[1]
        
        allowed = request_count < self.max_requests
        remaining = max(0, self.max_requests - request_count - 1)
        
        return allowed, {
            "limit": self.max_requests,
            "remaining": remaining,
            "reset_in": self.window_seconds,
            "current_count": request_count + 1
        }


# Create rate limiter: 100 requests per minute
rate_limiter = RateLimiter(max_requests=100, window_seconds=60)


@app.middleware("http")
async def rate_limit_middleware(request: Request, call_next):
    """
    Rate limiting middleware.
    Checks every request before processing.
    """
    # Get client IP (use X-Forwarded-For in production behind proxy)
    client_ip = request.headers.get("X-Forwarded-For", request.client.host)
    
    # Skip rate limiting for certain paths
    if request.url.path in ["/health", "/docs", "/openapi.json"]:
        return await call_next(request)
    
    allowed, info = rate_limiter.is_allowed(client_ip)
    
    if not allowed:
        return JSONResponse(
            status_code=429,
            content={
                "error": "Too Many Requests",
                "detail": f"Rate limit exceeded. Try again in {info['reset_in']} seconds.",
                "limit": info["limit"],
                "reset_in": info["reset_in"]
            },
            headers={
                "X-RateLimit-Limit": str(info["limit"]),
                "X-RateLimit-Remaining": "0",
                "X-RateLimit-Reset": str(info["reset_in"]),
                "Retry-After": str(info["reset_in"])
            }
        )
    
    # Process request
    response = await call_next(request)
    
    # Add rate limit headers
    response.headers["X-RateLimit-Limit"] = str(info["limit"])
    response.headers["X-RateLimit-Remaining"] = str(info["remaining"])
    
    return response


@app.get("/api/resource")
async def get_resource():
    """A protected endpoint"""
    return {"message": "Here's your data!", "timestamp": time.time()}


@app.get("/health")
async def health_check():
    """Health check (not rate limited)"""
    return {"status": "healthy"}
```

**Why Redis for Rate Limiting?**

| Storage | Access Time | Suitable? |
|---------|-------------|-----------|
| PostgreSQL | 10-50ms | ❌ Too slow for every request |
| Redis | <1ms | ✅ Perfect - minimal latency impact |
| In-memory (app) | <0.1ms | ⚠️ Doesn't work with multiple servers |

---

## Implementation with FastAPI and Redis

### Complete Application Example

Here's a complete FastAPI application demonstrating all caching patterns:

```python
"""
Complete FastAPI + Redis Caching Example
Run with: uvicorn main:app --reload
Requirements: pip install fastapi uvicorn redis
"""

import redis
import json
import uuid
import time
import hashlib
from datetime import datetime
from typing import Optional, List
from fastapi import FastAPI, HTTPException, Request, Depends
from fastapi.responses import JSONResponse
from pydantic import BaseModel
from contextlib import asynccontextmanager
import asyncio


# =============================================================================
# Configuration
# =============================================================================

REDIS_HOST = "localhost"
REDIS_PORT = 6379
CACHE_TTL_SHORT = 300      # 5 minutes
CACHE_TTL_MEDIUM = 3600    # 1 hour
CACHE_TTL_LONG = 86400     # 24 hours


# =============================================================================
# Redis Connection
# =============================================================================

redis_client: redis.Redis = None


@asynccontextmanager
async def lifespan(app: FastAPI):
    """Application lifespan - initialize and cleanup Redis"""
    global redis_client
    redis_client = redis.Redis(
        host=REDIS_HOST,
        port=REDIS_PORT,
        db=0,
        decode_responses=True
    )
    print("✅ Redis connected")
    yield
    redis_client.close()
    print("🔴 Redis disconnected")


app = FastAPI(
    title="Caching Demo API",
    description="Demonstrates various caching patterns with Redis",
    lifespan=lifespan
)


# =============================================================================
# Models
# =============================================================================

class Product(BaseModel):
    id: int
    name: str
    price: float
    category: str


class ProductCreate(BaseModel):
    name: str
    price: float
    category: str


# =============================================================================
# Simulated Database
# =============================================================================

PRODUCTS_DB = {
    1: Product(id=1, name="Laptop Pro", price=1299.99, category="Electronics"),
    2: Product(id=2, name="Wireless Mouse", price=49.99, category="Electronics"),
    3: Product(id=3, name="Coffee Maker", price=89.99, category="Kitchen"),
    4: Product(id=4, name="Running Shoes", price=129.99, category="Sports"),
}


async def simulate_db_query(delay: float = 0.1):
    """Simulate database latency"""
    await asyncio.sleep(delay)


# =============================================================================
# Caching Utilities
# =============================================================================

class CacheManager:
    """Centralized cache management"""
    
    @staticmethod
    def get(key: str) -> Optional[str]:
        """Get value from cache"""
        return redis_client.get(key)
    
    @staticmethod
    def set(key: str, value: str, ttl: int = CACHE_TTL_MEDIUM) -> None:
        """Set value in cache with TTL"""
        redis_client.setex(key, ttl, value)
    
    @staticmethod
    def delete(key: str) -> None:
        """Delete key from cache"""
        redis_client.delete(key)
    
    @staticmethod
    def delete_pattern(pattern: str) -> int:
        """Delete all keys matching pattern"""
        keys = redis_client.keys(pattern)
        if keys:
            return redis_client.delete(*keys)
        return 0
    
    @staticmethod
    def get_json(key: str) -> Optional[dict]:
        """Get JSON value from cache"""
        data = redis_client.get(key)
        return json.loads(data) if data else None
    
    @staticmethod
    def set_json(key: str, value: dict, ttl: int = CACHE_TTL_MEDIUM) -> None:
        """Set JSON value in cache"""
        redis_client.setex(key, ttl, json.dumps(value))


cache = CacheManager()


# =============================================================================
# Cache-Aside Pattern (Read)
# =============================================================================

@app.get("/products/{product_id}")
async def get_product(product_id: int):
    """
    Cache-Aside Pattern:
    1. Check cache
    2. On miss, query database
    3. Store in cache
    4. Return result
    """
    cache_key = f"product:{product_id}"
    
    # Step 1: Check cache
    cached = cache.get_json(cache_key)
    if cached:
        return {
            "source": "cache",
            "data": cached
        }
    
    # Step 2: Query database
    await simulate_db_query()
    product = PRODUCTS_DB.get(product_id)
    
    if not product:
        raise HTTPException(status_code=404, detail="Product not found")
    
    product_dict = product.model_dump()
    
    # Step 3: Store in cache
    cache.set_json(cache_key, product_dict, CACHE_TTL_MEDIUM)
    
    # Step 4: Return result
    return {
        "source": "database",
        "data": product_dict
    }


@app.get("/products")
async def list_products(category: Optional[str] = None):
    """
    List all products with optional category filter.
    Uses cache key based on query parameters.
    """
    cache_key = f"products:list:{category or 'all'}"
    
    cached = cache.get_json(cache_key)
    if cached:
        return {"source": "cache", "count": len(cached), "data": cached}
    
    await simulate_db_query(0.2)  # Simulate slower list query
    
    products = list(PRODUCTS_DB.values())
    if category:
        products = [p for p in products if p.category == category]
    
    products_data = [p.model_dump() for p in products]
    cache.set_json(cache_key, products_data, CACHE_TTL_SHORT)
    
    return {"source": "database", "count": len(products_data), "data": products_data}


# =============================================================================
# Write-Through Pattern (Write)
# =============================================================================

@app.post("/products")
async def create_product(product: ProductCreate):
    """
    Write-Through Pattern:
    Write to BOTH database and cache simultaneously
    """
    # Generate new ID
    new_id = max(PRODUCTS_DB.keys()) + 1
    
    new_product = Product(id=new_id, **product.model_dump())
    
    # Write to database
    await simulate_db_query()
    PRODUCTS_DB[new_id] = new_product
    
    # Write to cache (simultaneously)
    cache_key = f"product:{new_id}"
    cache.set_json(cache_key, new_product.model_dump(), CACHE_TTL_MEDIUM)
    
    # Invalidate list cache
    cache.delete_pattern("products:list:*")
    
    return {
        "message": "Product created",
        "data": new_product.model_dump()
    }


@app.put("/products/{product_id}")
async def update_product(product_id: int, product: ProductCreate):
    """
    Write-Through: Update both DB and cache
    """
    if product_id not in PRODUCTS_DB:
        raise HTTPException(status_code=404, detail="Product not found")
    
    updated_product = Product(id=product_id, **product.model_dump())
    
    # Update database
    await simulate_db_query()
    PRODUCTS_DB[product_id] = updated_product
    
    # Update cache
    cache_key = f"product:{product_id}"
    cache.set_json(cache_key, updated_product.model_dump(), CACHE_TTL_MEDIUM)
    
    # Invalidate list cache
    cache.delete_pattern("products:list:*")
    
    return {"message": "Product updated", "data": updated_product.model_dump()}


@app.delete("/products/{product_id}")
async def delete_product(product_id: int):
    """
    Delete from both DB and cache
    """
    if product_id not in PRODUCTS_DB:
        raise HTTPException(status_code=404, detail="Product not found")
    
    # Delete from database
    await simulate_db_query()
    del PRODUCTS_DB[product_id]
    
    # Delete from cache
    cache.delete(f"product:{product_id}")
    
    # Invalidate list cache
    cache.delete_pattern("products:list:*")
    
    return {"message": "Product deleted"}


# =============================================================================
# Expensive Query Caching
# =============================================================================

@app.get("/analytics/dashboard")
async def get_dashboard():
    """
    Cache expensive analytics queries
    """
    cache_key = "analytics:dashboard"
    
    cached = cache.get_json(cache_key)
    if cached:
        return {"source": "cache", "data": cached}
    
    # Simulate expensive aggregation query
    await simulate_db_query(2.0)  # 2 second query!
    
    analytics = {
        "total_products": len(PRODUCTS_DB),
        "total_value": sum(p.price for p in PRODUCTS_DB.values()),
        "by_category": {},
        "computed_at": datetime.utcnow().isoformat()
    }
    
    for product in PRODUCTS_DB.values():
        cat = product.category
        if cat not in analytics["by_category"]:
            analytics["by_category"][cat] = {"count": 0, "total_value": 0}
        analytics["by_category"][cat]["count"] += 1
        analytics["by_category"][cat]["total_value"] += product.price
    
    # Cache for 1 hour
    cache.set_json(cache_key, analytics, CACHE_TTL_MEDIUM)
    
    return {"source": "database", "data": analytics}


# =============================================================================
# Rate Limiting
# =============================================================================

class RateLimiter:
    def __init__(self, max_requests: int, window_seconds: int):
        self.max_requests = max_requests
        self.window_seconds = window_seconds
    
    def check(self, identifier: str) -> tuple[bool, int]:
        """Returns (allowed, remaining_requests)"""
        key = f"ratelimit:{identifier}"
        
        current = redis_client.incr(key)
        
        if current == 1:
            redis_client.expire(key, self.window_seconds)
        
        allowed = current <= self.max_requests
        remaining = max(0, self.max_requests - current)
        
        return allowed, remaining


rate_limiter = RateLimiter(max_requests=10, window_seconds=60)


@app.get("/limited-endpoint")
async def limited_endpoint(request: Request):
    """
    Rate-limited endpoint: 10 requests per minute per IP
    """
    client_ip = request.client.host
    
    allowed, remaining = rate_limiter.check(client_ip)
    
    if not allowed:
        raise HTTPException(
            status_code=429,
            detail="Rate limit exceeded. Try again later."
        )
    
    return {
        "message": "Request successful",
        "remaining_requests": remaining
    }


# =============================================================================
# Cache Management Endpoints
# =============================================================================

@app.get("/cache/stats")
async def cache_stats():
    """Get cache statistics"""
    info = redis_client.info()
    return {
        "used_memory": info["used_memory_human"],
        "connected_clients": info["connected_clients"],
        "total_keys": redis_client.dbsize(),
        "uptime_days": info["uptime_in_days"]
    }


@app.post("/cache/clear")
async def clear_cache():
    """Clear all cache (use with caution!)"""
    redis_client.flushdb()
    return {"message": "Cache cleared"}


@app.delete("/cache/{pattern}")
async def invalidate_cache(pattern: str):
    """Invalidate cache by pattern"""
    deleted = cache.delete_pattern(f"*{pattern}*")
    return {"message": f"Deleted {deleted} keys matching pattern"}


# =============================================================================
# Health Check
# =============================================================================

@app.get("/health")
async def health():
    """Health check endpoint"""
    try:
        redis_client.ping()
        redis_status = "connected"
    except:
        redis_status = "disconnected"
    
    return {
        "status": "healthy",
        "redis": redis_status,
        "timestamp": datetime.utcnow().isoformat()
    }
```

---

## Common Pitfalls & Debugging

### Pitfall 1: Cache Stampede

**Problem:** Cache expires, 1000 concurrent requests all hit the database simultaneously.

```
Cache expires at 10:00:00
┌──────────────────────────────────────────────────────────────┐
│ 10:00:00.001 - Request 1: Cache MISS → DB query             │
│ 10:00:00.002 - Request 2: Cache MISS → DB query             │
│ 10:00:00.003 - Request 3: Cache MISS → DB query             │
│ ...                                                          │
│ 10:00:00.100 - Request 1000: Cache MISS → DB query          │
│                                                              │
│ DATABASE OVERWHELMED! 🔥                                     │
└──────────────────────────────────────────────────────────────┘
```

**Solution: Locking/Mutex**

```python
import redis
import time
import json

def get_with_lock(cache_key: str, fetch_func, ttl: int = 3600):
    """
    Get from cache with stampede protection using locks
    """
    # Try cache first
    cached = redis_client.get(cache_key)
    if cached:
        return json.loads(cached)
    
    lock_key = f"lock:{cache_key}"
    
    # Try to acquire lock
    lock_acquired = redis_client.set(lock_key, "1", nx=True, ex=10)
    
    if lock_acquired:
        # We got the lock - fetch and cache
        try:
            data = fetch_func()
            redis_client.setex(cache_key, ttl, json.dumps(data))
            return data
        finally:
            redis_client.delete(lock_key)
    else:
        # Another request has the lock - wait and retry
        for _ in range(50):  # Wait up to 5 seconds
            time.sleep(0.1)
            cached = redis_client.get(cache_key)
            if cached:
                return json.loads(cached)
        
        # Timeout - fetch anyway
        return fetch_func()
```

---

### Pitfall 2: Stale Data

**Problem:** Cache contains outdated data after database update.

**Solution: Cache Invalidation**

```python
@app.put("/users/{user_id}")
async def update_user(user_id: int, user_data: dict):
    # Update database
    await db.update(user_id, user_data)
    
    # CRITICAL: Invalidate cache
    redis_client.delete(f"user:{user_id}")
    redis_client.delete_pattern(f"user_list:*")  # Invalidate list caches too
    
    return {"message": "Updated"}
```

---

### Pitfall 3: Caching Null/Empty Results

**Problem:** Database says "user not found", you cache `null`, future requests return stale `null`.

```python
# ❌ BAD: Caching null without short TTL
cached = redis_client.get(f"user:{user_id}")
if cached:
    return cached  # Returns "null" forever even if user is created later

# ✅ GOOD: Short TTL for negative cache
user = await db.get_user(user_id)
if user is None:
    # Cache negative result for short time only
    redis_client.setex(f"user:{user_id}", 60, "null")  # 1 minute
else:
    redis_client.setex(f"user:{user_id}", 3600, json.dumps(user))  # 1 hour
```

---

### Pitfall 4: Inconsistent Cache Keys

**Problem:** Different parts of code use different key formats.

```python
# ❌ BAD: Inconsistent keys
redis_client.set("user_123", data)      # In one file
redis_client.get("user:123")            # In another file - MISS!

# ✅ GOOD: Centralized key generation
class CacheKeys:
    @staticmethod
    def user(user_id: int) -> str:
        return f"user:{user_id}"
    
    @staticmethod
    def user_list(page: int = 1) -> str:
        return f"users:list:page:{page}"
    
    @staticmethod
    def product(product_id: int) -> str:
        return f"product:{product_id}"


# Usage
redis_client.set(CacheKeys.user(123), data)
redis_client.get(CacheKeys.user(123))  # Always consistent
```

---

### Pitfall 5: Not Setting TTL

**Problem:** Data cached forever, memory fills up, Redis crashes.

```python
# ❌ BAD: No TTL
redis_client.set("key", "value")  # Lives forever

# ✅ GOOD: Always set TTL
redis_client.setex("key", 3600, "value")  # Expires in 1 hour
```

---

### Pitfall 6: Large Objects in Cache

**Problem:** Caching 10MB objects, network latency becomes bottleneck.

```python
# ❌ BAD: Caching entire response
redis_client.set("search_results", json.dumps(huge_list))  # 10MB

# ✅ GOOD: Cache only what's needed
# Option 1: Pagination
redis_client.set("search:page:1", json.dumps(first_50_results))

# Option 2: Cache IDs only, fetch details separately
redis_client.set("search_ids", json.dumps([1, 2, 3, ...]))
```

---

### Debugging Tips

```python
# Check if key exists
exists = redis_client.exists("user:123")

# Check TTL remaining
ttl = redis_client.ttl("user:123")  # -1 = no expiry, -2 = doesn't exist

# Check key type
key_type = redis_client.type("user:123")  # string, list, hash, etc.

# Get all keys matching pattern (use sparingly!)
keys = redis_client.keys("user:*")

# Monitor Redis commands in real-time
# Run in terminal: redis-cli MONITOR

# Get memory usage of a key
memory = redis_client.memory_usage("user:123")
```

---

## Quick Revision Cheat Sheet

### Core Concepts

```
┌─────────────────────────────────────────────────────────────────┐
│                     CACHING CHEAT SHEET                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  WHAT: Store subset of data in faster storage                   │
│  WHY:  Reduce latency, reduce DB load, save costs               │
│  WHERE: CDN, DNS, RAM (Redis), Application memory               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Caching Strategies Quick Reference

| Strategy | When to Use | Write Speed | Read Speed | Consistency |
|----------|-------------|-------------|------------|-------------|
| **Cache-Aside** | Most cases, read-heavy | N/A | Hit: Fast, Miss: Slow | Eventual |
| **Write-Through** | Need strong consistency | Slower | Always fast | Strong |
| **Write-Behind** | Write-heavy, speed critical | Fastest | Always fast | Eventual |
| **Read-Through** | Want abstraction | N/A | Hit: Fast, Miss: Slow | Eventual |

### Eviction Policies Quick Reference

| Policy | Evicts | Best For |
|--------|--------|----------|
| **LRU** | Least recently accessed | General purpose |
| **LFU** | Least frequently accessed | Stable popularity patterns |
| **TTL** | Expired items | Time-sensitive data |

### Redis Commands Cheat Sheet

```bash
# Basic Operations
SET key value                    # Set string
GET key                          # Get string
SETEX key seconds value          # Set with expiration
DEL key                          # Delete key
EXISTS key                       # Check if exists

# TTL Operations
EXPIRE key seconds               # Set expiration
TTL key                          # Get remaining TTL
PERSIST key                      # Remove expiration

# Key Management
KEYS pattern                     # Find keys (use sparingly!)
SCAN cursor MATCH pattern        # Iterate keys safely
DBSIZE                           # Count all keys
FLUSHDB                          # Delete all keys

# JSON (via application)
SET user:1 '{"name":"Alice"}'    # Store JSON as string
GET user:1                       # Retrieve and parse in app
```

### Python/FastAPI Redis Patterns

```python
# Connection
redis_client = redis.Redis(host='localhost', port=6379, decode_responses=True)

# Basic cache-aside
def get_cached(key, fetch_func, ttl=3600):
    cached = redis_client.get(key)
    if cached:
        return json.loads(cached)
    data = fetch_func()
    redis_client.setex(key, ttl, json.dumps(data))
    return data

# Write-through
def save_with_cache(key, data, save_func):
    save_func(data)  # Save to DB
    redis_client.setex(key, 3600, json.dumps(data))  # Update cache

# Invalidation
def invalidate(pattern):
    keys = redis_client.keys(pattern)
    if keys:
        redis_client.delete(*keys)
```

### Common TTL Values

| Data Type | Recommended TTL |
|-----------|-----------------|
| Session tokens | 24 hours |
| User profiles | 1 hour |
| Product listings | 5-15 minutes |
| Search results | 5 minutes |
| Rate limit counters | 1 minute |
| Weather data | 30 minutes |
| Static content (CDN) | 1 week |

### Golden Rules

1. **Always set TTL** - Never cache forever
2. **Invalidate on write** - Update or delete cache when data changes
3. **Use consistent key naming** - `entity:id:attribute` pattern
4. **Don't cache large objects** - Keep cached items < 1MB
5. **Handle cache failures gracefully** - Fall back to database
6. **Monitor cache hit rate** - Aim for > 90%
7. **Use appropriate eviction policy** - LRU for most cases
8. **Protect against stampedes** - Use locking for hot keys
9. **Cache negative results carefully** - Short TTL for "not found"
10. **Test cache invalidation** - Stale data is worse than no cache

---

## Conclusion

Caching is a fundamental technique for building high-performance backend systems. Key takeaways:

1. **Caching reduces latency and load** by storing frequently-accessed data in faster storage
2. **Multiple levels exist**: Network (CDN, DNS), Hardware (CPU, RAM), Software (Redis)
3. **Choose the right strategy**: Cache-Aside for most cases, Write-Through for consistency
4. **Eviction policies matter**: LRU is the default choice for most applications
5. **Common use cases**: DB query caching, sessions, API responses, rate limiting
6. **Always handle edge cases**: Stampedes, stale data, cache failures

**Next Steps:**

1. Set up Redis locally and experiment
2. Implement cache-aside in your existing APIs
3. Add rate limiting to protect your endpoints
4. Monitor cache hit rates in production
5. Learn about distributed caching for horizontal scaling

Remember: **A cache miss should never break your application.** The database is always the source of truth.
