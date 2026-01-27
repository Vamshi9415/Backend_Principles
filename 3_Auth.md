# The Complete Authentication & Authorization Guide for Backend Engineers

> **Master Authentication & Authorization**: From Historical Context to Production Implementation

---

## Table of Contents

- [Part 1: Core Concepts](#part-1-core-concepts)
- [Part 2: Historical Evolution](#part-2-historical-evolution)
- [Part 3: Essential Components](#part-3-essential-components)
- [Part 4: Authentication Types](#part-4-authentication-types)
- [Part 5: Authorization](#part-5-authorization)
- [Part 6: Security Best Practices](#part-6-security-best-practices)
- [Part 7: Implementation Guides](#part-7-implementation-guides)
- [Part 8: Quick Reference](#part-8-quick-reference)

---

# Part 1: Core Concepts

## What Are Authentication and Authorization?

### Authentication: "Who Are You?"
**Definition**: The process of verifying a user's identity - confirming that users are who they claim to be.

**Real-World Analogy**: When you show your driver's license at airport security, you're authenticating yourself. The security officer verifies that you are the person shown on the ID.

**Technical Example**:
```
User enters: email@example.com + password123
Server checks: Does this match our records?
Result: Yes → User authenticated
        No → Access denied
```

### Authorization: "What Can You Do?"
**Definition**: The process of determining what permissions and capabilities an authenticated user has in a system.

**Real-World Analogy**: After passing airport security (authentication), your boarding pass determines which gate you can access, which seat you can occupy (authorization). Your ticket doesn't grant access to the pilot's cockpit or first class if you have an economy ticket.

**Technical Example**:
```
Authenticated user tries to delete file
System checks: Does this user have 'delete' permission?
Result: Admin role → Yes, proceed
        Regular user → No, forbidden
```

### The Workflow
```
1. Authentication (Verify Identity)
         ↓
2. Authorization (Check Permissions)
         ↓
3. Access Granted/Denied
```

**Why Both Matter**:
- **Authentication alone** tells you WHO someone is
- **Authorization** tells you WHAT they can do
- You need BOTH for proper security

**Example Scenario**:
```
Email app:
  Authentication: Prove you're john@example.com (password/biometric)
  Authorization: 
    ✓ Can read your own emails
    ✓ Can send emails as yourself
    ✗ Cannot read other users' emails
    ✗ Cannot access admin panel (unless admin)
```

---

# Part 2: Historical Evolution

## Why History Matters
Understanding how authentication evolved helps you:
- Appreciate modern solutions
- Understand trade-offs
- Avoid repeating past mistakes
- Choose appropriate methods

## Timeline of Authentication

### Pre-Industrial Era: Trust-Based Authentication

**How It Worked**:
- Village elders vouched for people
- Deals sealed with handshakes
- Identity = community recognition

**Real-World Analogy**: Small town where everyone knows each other. "That's John's son" is enough identification.

**The Problem**: Didn't scale beyond small communities.

```
┌──────────────────────────────┐
│  Village Elder says:         │
│  "I vouch for this person"   │
│  ↓                            │
│  Deal accepted               │
└──────────────────────────────┘

Works in small village
Fails in large city or between regions
```

### Medieval Period: Wax Seals (First Tokens)

**Innovation**: Physical tokens that prove identity

**How It Worked**:
```
1. Authority creates unique seal pattern
2. Seals pressed on documents
3. Recipients verify seal authenticity
4. Seal = authentication token
```

**Real-World Analogy**: Like a physical password. Just as you have a unique house key, nobles had unique seal patterns. A letter with the king's seal proved it was genuinely from the king.

**Security Principle**: "Something You Have"

**Vulnerabilities**:
- Could be forged (bypass attacks)
- Led to watermarks and encrypted codes

**Modern Parallel**: Physical security tokens, smart cards, YubiKeys

### Industrial Revolution: Passphrases

**Context**: Telegraph became critical infrastructure

**Innovation**: Pre-agreed passphrases (shared secrets)

```
Telegraph Operator A: "The eagle flies at midnight"
Telegraph Operator B: Validates phrase → Message trusted
```

**Security Principle Shift**: From "Something You Have" to **"Something You Know"**

**Characteristics**:
- Static passwords (same phrase repeatedly)
- Verbally or written communication
- Stored in memory

**The Risk**: If intercepted once, compromised forever

**Modern Parallel**: Passwords (but we rotate them now!)

### 1961: Digital Passwords

**Breakthrough**: MIT's CTSS (Compatible Time-Sharing Systems)

**Problem Solved**: Multiple users on one computer without data sharing

```
Before:
┌────────────────┐
│  Computer      │
│  All files     │
│  visible to    │
│  everyone      │
└────────────────┘

After (with passwords):
┌────────────────┐
│  Computer      │
│  User A → Password → Only A's files
│  User B → Password → Only B's files
│  User C → Password → Only C's files
└────────────────┘
```

**Critical Mistake**: Stored passwords in **plain text**

**The Incident**: Someone printed the password file

**The Lesson**: "Passwords should NEVER be stored in plain text"

**This principle is STILL critical in 2025!**

#### The Solution: Hashing

**What is Hashing?**
One-way cryptographic function that transforms text into irreversible fixed-length string.

```javascript
Input: "myPassword123"
  ↓ Hash Function (SHA-256)
Output: "ef92b778bce853e59f8c..."

Input: "password"  
  ↓ Hash Function (SHA-256)
Output: "5e884898da28047151d0..."

Input: "myPassword123" (same as first)
  ↓ Hash Function (SHA-256)  
Output: "ef92b778bce853e59f8c..." (same hash!)
```

**Key Properties**:
1. **Deterministic**: Same input → same output
2. **Fixed length**: Any input → same length output
3. **One-way**: Cannot reverse hash → original password
4. **Collision resistant**: Different inputs → different outputs

**How It Works**:
```javascript
// Registration
User submits: "myPassword123"
Server hashes: "ef92b778bce..."
Server stores: "ef92b778bce..." (only hash, not password)

// Login
User submits: "myPassword123"
Server hashes: "ef92b778bce..."
Server compares: stored_hash === new_hash
Match → Authenticated!
```

**Why Secure**: Even if database stolen, attackers only get hashes (can't reverse)

**Modern Enhancement - Salting**:
```javascript
Password: "myPassword123"
Salt: "x9k2m5p8" (random per user)
Combined: "myPassword123x9k2m5p8"
Hash: "9f7j4k2p..." (stored with salt)

// Same password, different users → different hashes!
```

### 1970s: Asymmetric Cryptography Revolution

**Pioneers**: Whitfield Diffie, Martin Hellman

**The Problem**: How to share encryption keys securely?

```
Symmetric Encryption Problem:
  Alice wants to send Bob encrypted message
  Challenge: How does Alice share the key with Bob?
  If key sent over network → Attacker intercepts → Game over
```

**The Solution**: **Two different keys**
- Public key: Shared openly, encrypts
- Private key: Secret, decrypts

```
┌────────────────────────────────┐
│  ASYMMETRIC CRYPTOGRAPHY       │
│                                │
│  Alice                  Bob    │
│    │                     │     │
│    │ Bob's Public Key    │     │
│    │◄────────────────────┤     │
│    │                     │     │
│    │ Encrypts with       │     │
│    │ Bob's Public Key    │     │
│    │                     │     │
│    │ Encrypted Message   │     │
│    ├─────────────────────►│     │
│    │                     │     │
│    │         Decrypts with     │
│    │         Bob's Private Key │
│    │         (never shared)    │
└────────────────────────────────┘
```

**Real-World Analogy**: Mailbox with a slot. Anyone (public key) can drop mail in, but only the person with the mailbox key (private key) can read it.

**Impact**: Backbone of modern authentication
- HTTPS/TLS
- SSH
- Digital signatures
- PKI (Public Key Infrastructure)

### 1990s: Multi-Factor Authentication (MFA)

**The Problem**: Passwords alone insufficient
- Brute force attacks
- Dictionary attacks
- Leaked password databases

**The Solution**: Combine multiple independent factors

```
┌─────────────────────────────────┐
│  THREE AUTHENTICATION FACTORS   │
│                                 │
│  1. Something You KNOW          │
│     → Passwords, PINs           │
│                                 │
│  2. Something You HAVE          │
│     → Phone, Token, Smart Card  │
│                                 │
│  3. Something You ARE           │
│     → Fingerprint, Face, Retina │
└─────────────────────────────────┘
```

**Real-World Analogy**: Bank vault access
1. PIN code (something you know)
2. Key card (something you have)
3. Fingerprint (something you are)

**Example - OTP (One-Time Password)**:
```
Login with password ✓
  ↓
System sends code to phone: "123456"
  ↓
User enters code
  ↓
Code matches + not expired → Access granted
```

**Why MFA Matters**: Even if password stolen, attacker still needs physical device/biometric

#### Biometric Authentication

**How It Works**:
```
1. Enrollment
   Capture fingerprint → Create mathematical template → Store securely

2. Authentication  
   Capture fingerprint → Create template → Compare with stored
   Match % > threshold (e.g., 95%) → Authenticate
```

**Challenges**:
- **False Positives**: Wrong person accepted (twin accessing sibling's phone)
- **False Negatives**: Right person rejected (injured finger)
- **Can't Change**: Stolen biometric data = permanent compromise

### 21st Century: Modern Authentication

**New Challenges**:
- Billions of users globally
- Mobile devices
- Microservices
- API-based architectures

**Modern Solutions**:

**1. OAuth 2.0**: Delegated authorization (Sign in with Google/Facebook)

**2. JWT (JSON Web Tokens)**: Stateless authentication for distributed systems

**3. Zero Trust**: "Never trust, always verify"

**4. Passwordless**:
- WebAuthn (hardware security keys)
- Magic links (email-based)
- Biometric authentication

**Example - WebAuthn**:
```
1. User clicks "Sign in with security key"
2. Inserts USB key or uses built-in biometric
3. Authenticates with PIN/biometric
4. Key signs challenge with private key
5. Server verifies with public key
6. Authenticated - no password needed!
```

### The Future

**1. Decentralized Identity (Blockchain)**
You own your identity, not companies

```
Traditional:
  Google owns your identity → grants access to websites

Decentralized:
  You own identity on blockchain → prove to any website
```

**2. Behavioral Biometrics**
Continuous authentication via behavior patterns
- Typing speed/rhythm
- Mouse movements
- Device interaction patterns

**3. Post-Quantum Cryptography**
Current encryption vulnerable to quantum computers

```
Current:
  Classical computer: Millions of years to break RSA
  Quantum computer: Hours to break RSA

Solution: Post-quantum algorithms (NIST standardizing now)
```

---

# Part 3: Essential Components

## The Three Pillars: Sessions, JWTs, and Cookies

These three components appear in virtually all authentication workflows.

### 1. Sessions: Server-Side Memory

#### The Problem HTTP Solves

**HTTP is Stateless**:
```
Request 1: GET /products
Server: Here are products
[Server forgets you exist]

Request 2: GET /cart
Server: Who are you? Don't remember you
```

**Real-World Analogy**: Coffee shop barista with amnesia. Every visit is like the first time.

**Why This Was a Problem**:
- Shopping carts need to remember items
- Users need to stay logged in
- Preferences need persistence

#### How Sessions Work

**Phase 1: Session Creation**
```
User logs in
  ↓
Server generates unique ID: "sess_a7k9m2x4p8"
  ↓
Server stores in Redis/DB:
  ┌─────────────────┐
  │ ID: sess_a7k9...│
  │ user_id: 12345  │
  │ role: "admin"   │
  │ cart: [...]     │
  │ expires: ...    │
  └─────────────────┘
  ↓
Sends ID to browser (as cookie)
```

**Phase 2: Using Session**
```
Every request:
  Browser sends session ID automatically
  ↓
  Server looks up in Redis
  ↓
  Retrieves user data
  ↓
  Processes request with full context
```

**Phase 3: Expiration**
```
After 15 minutes inactive:
  Server deletes from Redis
  User must login again
```

#### Storage Evolution

**1. File-Based (Early)**
```
/tmp/sessions/sess_abc123
Problem: Slow, doesn't scale
```

**2. Database-Backed**
```sql
CREATE TABLE sessions (
  session_id VARCHAR PRIMARY KEY,
  user_id INT,
  data JSON,
  expires_at TIMESTAMP
);
Problem: Still disk I/O, bottleneck
```

**3. In-Memory (Modern - Redis)**
```
Redis (RAM):
  Key: sess_abc123
  Value: {user_id: 123, role: "admin", ...}
  TTL: 900 seconds

Access time: <1ms (vs 10-100ms for DB)
```

**Why Redis**:
- Blazing fast (RAM-based)
- Built-in expiration (TTL)
- Atomic operations
- Can persist to disk

**Performance**:
```
File System: 10-100ms/read
Database:    5-50ms/read
Redis:       <1ms/read

For 1000 req/sec:
  Files: 10-100 sec CPU time
  DB:    5-50 sec CPU time
  Redis: <1 sec CPU time
```

### 2. JWTs: Self-Contained Tokens

#### The Problem Sessions Couldn't Solve

**1. Memory Overhead**
```
1M users × 5KB/session = 5GB RAM
10M users = 50GB RAM
100M users = 500GB RAM
```

**2. Distribution Nightmare**
```
User logs in → Server 1 (US-East) creates session
Next request → Load balancer → Server 2 (US-West)
Server 2: "Don't have your session!"

Solution: Replicate sessions → Latency, complexity
```

**Real-World Analogy**: Hotel key that only works at check-in location. To work everywhere, hotels must constantly sync - slow and error-prone.

#### JWT Structure

```
eyJhbGc...  .  eyJzdWI...  .  SflKxw...
   ↑              ↑              ↑
 HEADER        PAYLOAD       SIGNATURE
```

**Decoded**:

**Header** (Algorithm metadata):
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

**Payload** (User data):
```json
{
  "sub": "12345",           // User ID
  "email": "john@ex.com",
  "role": "admin",
  "iat": 1706356800,        // Issued at
  "exp": 1706360400         // Expires
}
```

**Signature** (Verification):
```
HMACSHA256(
  base64(header) + "." + base64(payload),
  "my-secret-key"
)
```

**Real-World Analogy**: Signed diploma. The diploma (payload) contains your info. The university's seal (signature) proves authenticity. Anyone can read it, but only the university could create that specific seal.

#### JWT Workflow

```javascript
// LOGIN
POST /login → Server validates credentials
  ↓
Server creates JWT:
  const payload = {
    sub: user.id,
    role: user.role,
    exp: Date.now() + 3600
  };
  const token = jwt.sign(payload, SECRET_KEY);
  ↓
Returns: {token: "eyJhbGc..."}

// AUTHENTICATED REQUEST
GET /api/profile
Headers: Authorization: Bearer eyJhbGc...
  ↓
Server extracts token
  ↓
jwt.verify(token, SECRET_KEY)
  ↓
If valid: decoded = {sub: "123", role: "admin"}
  ↓
Process request (NO database lookup!)
```
+-----------+        1) Login         +-------------+
|           | ---------------------> |             |
|  Browser  |   (username,password) |   Server    |
|           |                        | (Auth API)  |
+-----------+                        +-------------+
                                         |
                                         | 2) Verify credentials
                                         v
                                   +-------------+
                                   |   Database  |
                                   +-------------+
                                         |
                                         | OK
                                         v
+-----------+        3) JWT Token        +-------------+
|           | <---------------------  |             |
|  Browser  |     (Signed Token)      |   Server    |
|           |                        |             |
+-----------+                        +-------------+


=======================================================


+-----------+   4) API Request + JWT   +-------------+
|           | ---------------------->  |             |
|  Browser  |  Authorization: Bearer  |   Server    |
|           |       <JWT>             |             |
+-----------+                        +-------------+
                                         |
                                         | 5) Verify JWT Signature
                                         | 6) Check Expiry
                                         v
                                   +-------------+
                                   |   Request   |
                                   |   Accepted  |
                                   +-------------+
                                         |
                                         v
                                   +-------------+
                                   |  Response   |
                                   +-------------+
                                         |
                                         v
                                    Browser

#### Advantages

**1. Stateless = Scalable**
```
Request → Any server → Verifies JWT → Responds
No Redis needed
No session sync
Add servers instantly
```

**2. Microservices Friendly**
```
Gateway verifies JWT once
Passes to all services
Each service can verify independently
```

**3. Cross-Domain**
```
example.com
api.example.com
mobile-api.example.com
All accept same JWT!
```

#### Disadvantages

**1. Cannot Revoke**
```
Problem: User account compromised

Session: Delete from Redis → Instant lockout ✓
JWT: Token valid until expiration ✗
    Must wait hours!
```

**Solutions**:
```javascript
// Option 1: Short expiration + Refresh tokens
access_token: expires 15min
refresh_token: expires 7 days, stored server-side, can revoke

// Option 2: Blacklist
REVOKED_JWTS = Set();
if (REVOKED_JWTS.has(token.jti)) reject();
// But this defeats "stateless"!

// Option 3: Change secret (nuclear option)
Change SECRET_KEY → All users logged out globally
```

**2. Token Size**
```
Session ID: ~20 bytes
JWT:        ~200-500 bytes

Sent with EVERY request
1000 req/sec × 500 bytes = 500KB/sec = 43GB/day overhead
```

**3. Cannot Update**
```
User promoted admin → manager

Session: Update Redis → immediate effect
JWT: Token says "manager", must wait for expiration
```

### 3. Cookies: Automatic Token Transport

#### The Problem Cookies Solve

Without cookies:
```javascript
// Manual token management
fetch('/api/profile', {
  headers: {
    'X-Session-ID': 'sess_abc'  // Must remember every time!
  }
});
```

**Problems**: Easy to forget, repetitive, vulnerable

#### How Cookies Work

**Server Sets Cookie**:
```http
Set-Cookie: session_id=sess_abc; HttpOnly; Secure; SameSite=Strict
```

**Browser Automatically Sends**:
```http
GET /api/profile
Cookie: session_id=sess_abc
```

**Real-World Analogy**: Loyalty card. Shop gives you card (server sets cookie), you keep it (browser stores), you automatically show it every visit (browser sends automatically).

#### Cookie Attributes (Critical for Security)

**1. HttpOnly**
```
Set-Cookie: token=abc; HttpOnly

JavaScript CANNOT access cookie
Prevents XSS attacks:
  Malicious script tries: document.cookie
  Returns: "" (empty - cookie hidden)
```

**2. Secure**
```
Set-Cookie: token=abc; Secure

Cookie ONLY sent over HTTPS
Prevents man-in-the-middle attacks:
  HTTP: Cookie NOT sent
  HTTPS: Cookie sent
```

**3. SameSite**
```
Strict: Only same-site requests
Lax: Safe cross-site (GET links)
None: All requests (needs Secure)

Prevents CSRF attacks:
  Attacker site tries to use your cookie
  Browser: "Nope, different site"
```

**4. Max-Age**
```
Max-Age=3600     // 1 hour
Max-Age=86400    // 24 hours
Max-Age=604800   // 7 days
```

**Security Checklist**:
```
✓ Always HttpOnly for auth cookies
✓ Always Secure in production
✓ Use SameSite=Strict or Lax
✓ Set appropriate Max-Age
✓ Use HTTPS
```

---

# Part 4: Authentication Types

## 1. Stateful Authentication

### Complete Workflow

```
CLIENT                          SERVER
  │
  │  POST /login
  │  {email, password}
  ├──────────────────────────────►
  │                               │ Validate credentials
  │                               │ Generate session_id
  │                               │ Store in Redis:
  │                               │   sess_x7k2: {
  │                               │     user_id: 123,
  │                               │     role: "admin"
  │                               │   }
  │  Set-Cookie: session_id
  │◄──────────────────────────────┤
  │
  │  GET /api/profile
  │  Cookie: session_id=sess_x7k2
  ├──────────────────────────────►
  │                               │ Redis.get(sess_x7k2)
  │                               │ → user_id: 123
  │                               │ → role: "admin"
  │  {user: {...}}
  │◄──────────────────────────────┤
```

### Pros

✓ **Centralized Control**: View/revoke all sessions
✓ **Immediate Revocation**: Delete from Redis = instant lockout
✓ **Dynamic Updates**: Change role in Redis = immediate effect
✓ **Rich Session Data**: Store complex objects
✓ **Better Security**: Session ID is meaningless random string

### Cons

✗ **Scalability**: Need shared Redis for multiple servers
✗ **Memory Cost**: 1M users × 5KB = 5GB RAM
✗ **Latency**: Extra Redis lookup every request
✗ **Single Point of Failure**: Redis down = all users logged out

### Implementation Example

```javascript
const express = require('express');
const Redis = require('ioredis');
const crypto = require('crypto');

const app = express();
const redis = new Redis();

// LOGIN
app.post('/api/login', async (req, res) => {
  const user = await validateUser(req.body);
  
  // Generate session ID
  const sessionId = crypto.randomBytes(32).toString('hex');
  
  // Store in Redis
  await redis.setex(
    `sess:${sessionId}`,
    3600,  // 1 hour TTL
    JSON.stringify({
      user_id: user.id,
      role: user.role,
      created_at: Date.now()
    })
  );
  
  // Set cookie
  res.cookie('session_id', sessionId, {
    httpOnly: true,
    secure: true,
    sameSite: 'strict',
    maxAge: 3600000
  });
  
  res.json({message: "Logged in"});
});

// AUTH MIDDLEWARE
const requireAuth = async (req, res, next) => {
  const sessionId = req.cookies.session_id;
  
  if (!sessionId) {
    return res.status(401).json({error: "Not authenticated"});
  }
  
  const data = await redis.get(`sess:${sessionId}`);
  
  if (!data) {
    return res.status(401).json({error: "Session expired"});
  }
  
  req.user = JSON.parse(data);
  
  // Extend session
  await redis.expire(`sess:${sessionId}`, 3600);
  
  next();
};

// PROTECTED ROUTE
app.get('/api/profile', requireAuth, (req, res) => {
  res.json({
    user_id: req.user.user_id,
    role: req.user.role
  });
});

// LOGOUT
app.post('/api/logout', requireAuth, async (req, res) => {
  await redis.del(`sess:${req.cookies.session_id}`);
  res.clearCookie('session_id');
  res.json({message: "Logged out"});
});
```

## 2. Stateless Authentication (JWT)

### Complete Workflow

```
CLIENT                          SERVER
  │
  │  POST /login
  │  {email, password}
  ├──────────────────────────────►
  │                               │ Validate credentials
  │                               │ Create JWT:
  │                               │   payload: {
  │                               │     sub: "123",
  │                               │     role: "admin",
  │                               │     exp: 1706360400
  │                               │   }
  │                               │ Sign with SECRET_KEY
  │  {token: "eyJhbGc..."}
  │◄──────────────────────────────┤
  │
  │  GET /api/profile
  │  Authorization: Bearer eyJhbGc...
  ├──────────────────────────────►
  │                               │ jwt.verify(token, SECRET)
  │                               │ → {sub: "123", role: "admin"}
  │                               │ (NO database lookup!)
  │  {user: {...}}
  │◄──────────────────────────────┤
```

### Pros

✓ **Stateless**: No server storage needed
✓ **Scalable**: Works across distributed servers
✓ **Fast**: No database lookup (CPU-only verification)
✓ **Microservices**: Each service verifies independently
✓ **Cross-domain**: Works everywhere

### Cons

✗ **Cannot Revoke**: Token valid until expiration
✗ **Token Size**: 200-500 bytes vs 20 bytes for session ID
✗ **Cannot Update**: User info locked until expiration
✗ **Security Concerns**: Must be stored carefully

### Implementation Example

```javascript
const express = require('express');
const jwt = require('jsonwebtoken');

const app = express();
const SECRET_KEY = process.env.JWT_SECRET;

// LOGIN
app.post('/api/login', async (req, res) => {
  const user = await validateUser(req.body);
  
  // Create JWT
  const token = jwt.sign(
    {
      sub: user.id.toString(),
      email: user.email,
      role: user.role,
      exp: Math.floor(Date.now() / 1000) + 3600  // 1 hour
    },
    SECRET_KEY,
    {algorithm: 'HS256'}
  );
  
  res.json({
    token: token,
    expires_in: 3600
  });
});

// AUTH MIDDLEWARE
const verifyToken = (req, res, next) => {
  const authHeader = req.headers.authorization;
  
  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res.status(401).json({error: "No token"});
  }
  
  const token = authHeader.split(' ')[1];
  
  try {
    const decoded = jwt.verify(token, SECRET_KEY);
    req.user = decoded;
    next();
  } catch (err) {
    if (err.name === 'TokenExpiredError') {
      return res.status(401).json({error: "Token expired"});
    }
    return res.status(401).json({error: "Invalid token"});
  }
};

// PROTECTED ROUTE
app.get('/api/profile', verifyToken, (req, res) => {
  res.json({
    user_id: req.user.sub,
    email: req.user.email,
    role: req.user.role
  });
});

// REFRESH TOKEN PATTERN
app.post('/api/refresh', async (req, res) => {
  const {refresh_token} = req.body;
  
  // Verify refresh token (stored server-side)
  const refreshData = await db.refreshTokens.findOne({
    token: refresh_token,
    expires_at: {$gt: new Date()}
  });
  
  if (!refreshData) {
    return res.status(401).json({error: "Invalid refresh token"});
  }
  
  // Issue new access token
  const accessToken = jwt.sign(
    {sub: refreshData.user_id, role: refreshData.role},
    SECRET_KEY,
    {expiresIn: '15m'}
  );
  
  res.json({access_token: accessToken});
});
```

## 3. API Key Authentication

### The Use Case

**Problem**: You want programmatic access to your server without UI/login flow

**Solution**: API keys - long-lived tokens for machine-to-machine communication

### Real-World Example: ChatGPT API

```
UI Access (Human):
  User → Chat Interface → Types message → Gets response
  
API Access (Machine):
  Your Server → HTTP Request + API Key → ChatGPT Server → Response
  
Why? Your app wants ChatGPT features without the UI
```

### How API Keys Work

```
1. User generates key in dashboard
   → Server creates: "sk_live_abc123def456..."
   
2. User stores key securely
   → Environment variable or secret manager
   
3. Use key in requests
   → Header: Authorization: Bearer sk_live_abc...
   
4. Server validates
   → Look up key in database
   → Check permissions, quotas, expiration
   → Process request
```

### Characteristics

- **Long-lived**: Days, months, or years
- **Revocable**: Can delete from database
- **Permission-scoped**: Read-only, read-write, etc.
- **Rate-limited**: Enforce quotas per key

### Implementation Example

```javascript
// GENERATE API KEY
app.post('/api/keys', requireAuth, async (req, res) => {
  const apiKey = `sk_${crypto.randomBytes(32).toString('hex')}`;
  
  const hashedKey = crypto
    .createHash('sha256')
    .update(apiKey)
    .digest('hex');
  
  await db.apiKeys.insert({
    key_hash: hashedKey,
    user_id: req.user.id,
    permissions: ['read', 'write'],
    created_at: new Date(),
    last_used: null
  });
  
  // Show once, then only show hash
  res.json({
    api_key: apiKey,
    message: "Save this key - you won't see it again!"
  });
});

// VALIDATE API KEY
const validateApiKey = async (req, res, next) => {
  const apiKey = req.headers.authorization?.replace('Bearer ', '');
  
  if (!apiKey) {
    return res.status(401).json({error: "API key required"});
  }
  
  const hashedKey = crypto
    .createHash('sha256')
    .update(apiKey)
    .digest('hex');
  
  const keyData = await db.apiKeys.findOne({key_hash: hashedKey});
  
  if (!keyData) {
    return res.status(401).json({error: "Invalid API key"});
  }
  
  // Update last used
  await db.apiKeys.update(
    {_id: keyData._id},
    {$set: {last_used: new Date()}}
  );
  
  req.apiKey = keyData;
  next();
};

// PROTECTED API ENDPOINT
app.get('/api/data', validateApiKey, async (req, res) => {
  if (!req.apiKey.permissions.includes('read')) {
    return res.status(403).json({error: "Permission denied"});
  }
  
  const data = await fetchData(req.apiKey.user_id);
  res.json(data);
});
```

### Best Practices

```
✓ Hash keys before storing (like passwords)
✓ Show key only once at creation
✓ Support key rotation
✓ Implement rate limiting
✓ Log all API key usage
✓ Allow users to revoke keys
✓ Set expiration dates
✓ Use different keys for dev/prod
```

## 4. OAuth 2.0 & OpenID Connect

### The Delegation Problem

**Scenario**: Travel app needs to scan your Gmail for flight tickets

**Bad Solution** (Before OAuth):
```
Travel App: "Give us your Gmail password"
User: Provides password
Travel App: Full access to email, contacts, everything!

Problems:
  ✗ Share password = give complete access
  ✗ Can't revoke without changing password everywhere
  ✗ No permission scoping
  ✗ Huge security risk
```

**Good Solution** (OAuth 2.0):
```
Travel App: "We need read-only access to your email"
User: Redirected to Google → Approves specific permission
Travel App: Gets limited token (read-only, emails only)

Benefits:
  ✓ No password sharing
  ✓ Scoped permissions (read-only)
  ✓ Revokable anytime
  ✓ Time-limited
```

### OAuth 2.0 Components

```
┌──────────────────────────────────────────┐
│  OAUTH 2.0 ACTORS                        │
├──────────────────────────────────────────┤
│                                          │
│  1. RESOURCE OWNER (User)                │
│     → You, who owns the data             │
│                                          │
│  2. CLIENT (App requesting access)       │
│     → Travel app                         │
│                                          │
│  3. RESOURCE SERVER (API with data)      │
│     → Gmail servers                      │
│                                          │
│  4. AUTHORIZATION SERVER (Issues tokens) │
│     → Google's auth servers              │
│                                          │
└──────────────────────────────────────────┘
```

### OAuth 2.0 Flow (Authorization Code)

```
1. CLIENT redirects USER to AUTHORIZATION SERVER
   → "TravelApp wants to read your emails"
   
2. USER authenticates and grants permission
   → Logs in to Google, clicks "Allow"
   
3. AUTHORIZATION SERVER sends code to CLIENT
   → Redirect: travelapp.com/callback?code=xyz123
   
4. CLIENT exchanges code for access token
   → POST to Google with code
   → Receives: {access_token: "abc...", refresh_token: "def..."}
   
5. CLIENT uses access_token to access RESOURCE SERVER
   → GET https://gmail.googleapis.com/emails
   → Header: Authorization: Bearer abc...
   
6. RESOURCE SERVER validates token and returns data
   → Google verifies token
   → Returns emails
```

### OAuth 2.0 Grant Types

**1. Authorization Code** (Most secure, for web apps)
```
User-facing apps with server backend
Supports refresh tokens
Most common flow
```

**2. Implicit** (Deprecated, was for SPAs)
```
Tokens in URL (insecure)
No longer recommended
Use Authorization Code + PKCE instead
```

**3. Client Credentials** (Machine-to-machine)
```
Server-to-server
No user involvement
App authenticates as itself
```

**4. Device Code** (Limited input devices)
```
Smart TVs, IoT devices
Shows code on screen
User enters code on phone/computer
```

### OpenID Connect: Adding Identity

**OAuth 2.0**: Authorization only ("what can you access?")
**OpenID Connect**: Authorization + Authentication ("who are you?")

**What OpenID Connect Adds**:
```
OAuth 2.0 returns:
  {access_token: "abc123"}
  
OpenID Connect returns:
  {
    access_token: "abc123",
    id_token: "eyJhbGc..."  // JWT with user identity!
  }
```

**ID Token** (JWT):
```json
{
  "sub": "google|123456",
  "email": "john@example.com",
  "name": "John Doe",
  "picture": "https://...",
  "iss": "https://accounts.google.com",
  "aud": "your-app-id",
  "iat": 1706356800,
  "exp": 1706360400
}
```

### "Sign in with Google" Flow

```
1. User clicks "Sign in with Google"
   
2. Redirected to Google
   → https://accounts.google.com/oauth?
     client_id=your-app&
     redirect_uri=yourapp.com/callback&
     scope=openid email profile
     
3. User logs in to Google, approves
   
4. Google redirects back with code
   → yourapp.com/callback?code=xyz123
   
5. Your server exchanges code for tokens
   → POST to Google
   → Receives: {access_token, id_token}
   
6. Decode id_token to get user info
   → {email, name, picture}
   
7. Create user in your database (if new)
   → Or find existing user
   
8. Create your own session/JWT
   → User now logged into your app!
```

### Implementation Example

```javascript
const express = require('express');
const axios = require('axios');

const app = express();

const GOOGLE_CONFIG = {
  client_id: process.env.GOOGLE_CLIENT_ID,
  client_secret: process.env.GOOGLE_CLIENT_SECRET,
  redirect_uri: 'http://localhost:3000/auth/google/callback',
  auth_url: 'https://accounts.google.com/o/oauth2/v2/auth',
  token_url: 'https://oauth2.googleapis.com/token'
};

// INITIATE LOGIN
app.get('/auth/google', (req, res) => {
  const authUrl = `${GOOGLE_CONFIG.auth_url}?` +
    `client_id=${GOOGLE_CONFIG.client_id}&` +
    `redirect_uri=${GOOGLE_CONFIG.redirect_uri}&` +
    `response_type=code&` +
    `scope=openid email profile`;
  
  res.redirect(authUrl);
});

// CALLBACK
app.get('/auth/google/callback', async (req, res) => {
  const {code} = req.query;
  
  try {
    // Exchange code for tokens
    const tokenResponse = await axios.post(GOOGLE_CONFIG.token_url, {
      code,
      client_id: GOOGLE_CONFIG.client_id,
      client_secret: GOOGLE_CONFIG.client_secret,
      redirect_uri: GOOGLE_CONFIG.redirect_uri,
      grant_type: 'authorization_code'
    });
    
    const {access_token, id_token} = tokenResponse.data;
    
    // Decode ID token (or verify with Google)
    const userInfo = jwt.decode(id_token);
    
    // Find or create user
    let user = await db.users.findOne({google_id: userInfo.sub});
    
    if (!user) {
      user = await db.users.insert({
        google_id: userInfo.sub,
        email: userInfo.email,
        name: userInfo.name,
        picture: userInfo.picture,
        created_at: new Date()
      });
    }
    
    // Create your own session/JWT
    const sessionId = await createSession(user);
    
    res.cookie('session_id', sessionId, {
      httpOnly: true,
      secure: true,
      sameSite: 'strict'
    });
    
    res.redirect('/dashboard');
    
  } catch (err) {
    console.error('OAuth error:', err);
    res.redirect('/login?error=oauth_failed');
  }
});
```

---

# Part 5: Authorization

## Role-Based Access Control (RBAC)

### Why Authorization Matters

**Scenario**: Note-taking app

```
Requirements:
  1. Regular users: Create, read, update, delete own notes
  2. Moderators: Can read all notes, delete inappropriate ones
  3. Admins: Full access including user management
  
Problem: How to enforce different permissions?
```

**Bad Solution**: Check user_id everywhere
```javascript
if (user.id === 1 || user.id === 5 || user.id === 9) {
  // Allow admin actions
}
// Unmaintainable!
```

**Good Solution**: Role-Based Access Control

### How RBAC Works

```
┌────────────────────────────────────┐
│  RBAC COMPONENTS                   │
├────────────────────────────────────┤
│                                    │
│  USERS                             │
│    └─ Assigned to ROLES            │
│                                    │
│  ROLES                             │
│    └─ Have PERMISSIONS             │
│                                    │
│  PERMISSIONS                       │
│    └─ Allow ACTIONS on RESOURCES   │
│                                    │
└────────────────────────────────────┘
```

**Example Structure**:
```
User: john@example.com
  ↓ assigned
Role: Editor
  ↓ has permissions
Permissions:
  - notes:read
  - notes:write
  - notes:delete (own only)
  
User: admin@example.com
  ↓ assigned
Role: Admin
  ↓ has permissions
Permissions:
  - notes:read (all)
  - notes:write (all)
  - notes:delete (all)
  - users:manage
  - settings:configure
```

### Implementation Example

```javascript
// DEFINE ROLES AND PERMISSIONS
const ROLES = {
  user: {
    permissions: [
      'notes:read:own',
      'notes:write:own',
      'notes:delete:own'
    ]
  },
  moderator: {
    permissions: [
      'notes:read:all',
      'notes:delete:all'
    ]
  },
  admin: {
    permissions: [
      'notes:read:all',
      'notes:write:all',
      'notes:delete:all',
      'users:manage',
      'settings:configure'
    ]
  }
};

// AUTHORIZATION MIDDLEWARE
const requirePermission = (permission) => {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({error: "Not authenticated"});
    }
    
    const userRole = req.user.role;
    const permissions = ROLES[userRole]?.permissions || [];
    
    if (!permissions.includes(permission)) {
      return res.status(403).json({error: "Permission denied"});
    }
    
    next();
  };
};

// RESOURCE OWNERSHIP CHECK
const requireOwnership = async (req, res, next) => {
  const noteId = req.params.id;
  const note = await db.notes.findById(noteId);
  
  if (!note) {
    return res.status(404).json({error: "Note not found"});
  }
  
  // Admins and moderators can access all
  if (['admin', 'moderator'].includes(req.user.role)) {
    req.note = note;
    return next();
  }
  
  // Regular users only own notes
  if (note.user_id !== req.user.user_id) {
    return res.status(403).json({error: "Not your note"});
  }
  
  req.note = note;
  next();
};

// ROUTES
app.get('/api/notes', 
  requireAuth,
  requirePermission('notes:read:own'),
  async (req, res) => {
    let notes;
    
    if (ROLES[req.user.role].permissions.includes('notes:read:all')) {
      // Admin/moderator: all notes
      notes = await db.notes.find({});
    } else {
      // Regular user: own notes only
      notes = await db.notes.find({user_id: req.user.user_id});
    }
    
    res.json(notes);
  }
);

app.delete('/api/notes/:id',
  requireAuth,
  requireOwnership,
  requirePermission('notes:delete:own'),
  async (req, res) => {
    await db.notes.delete(req.note._id);
    res.json({message: "Note deleted"});
  }
);

app.post('/api/users/:id/role',
  requireAuth,
  requirePermission('users:manage'),
  async (req, res) => {
    const {role} = req.body;
    
    if (!ROLES[role]) {
      return res.status(400).json({error: "Invalid role"});
    }
    
    await db.users.update(
      {_id: req.params.id},
      {$set: {role}}
    );
    
    res.json({message: "Role updated"});
  }
);
```

### Database Schema

```sql
-- Users table
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  role VARCHAR(50) DEFAULT 'user',
  created_at TIMESTAMP DEFAULT NOW()
);

-- Roles table (optional, for dynamic roles)
CREATE TABLE roles (
  id SERIAL PRIMARY KEY,
  name VARCHAR(50) UNIQUE NOT NULL,
  description TEXT
);

-- Permissions table
CREATE TABLE permissions (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) UNIQUE NOT NULL,
  description TEXT
);

-- Role-Permission mapping
CREATE TABLE role_permissions (
  role_id INT REFERENCES roles(id),
  permission_id INT REFERENCES permissions(id),
  PRIMARY KEY (role_id, permission_id)
);

-- Notes table
CREATE TABLE notes (
  id SERIAL PRIMARY KEY,
  user_id INT REFERENCES users(id),
  title VARCHAR(255),
  content TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);
```

---

# Part 6: Security Best Practices

## 1. Error Messages: Don't Leak Information

### The Problem

**Bad (Information Leak)**:
```javascript
// Login attempt
if (!user) {
  return res.json({error: "User not found"});
}
if (!passwordMatch) {
  return res.json({error: "Incorrect password"});
}
```

**Why Bad**: Attacker learns:
- "User not found" → Email doesn't exist, try another
- "Incorrect password" → Email exists, attack this account!

**Good (Generic Message)**:
```javascript
if (!user || !passwordMatch) {
  return res.json({error: "Invalid credentials"});
}
```

**Principle**: Never confirm which part failed

### Examples of Information Leaks

**Registration**:
```javascript
// ❌ Bad
if (await db.users.findOne({email})) {
  return res.json({error: "Email already registered"});
}

// ✓ Good  
if (await db.users.findOne({email})) {
  // Still create account (idempotent)
  // Or generic: "Check your email for confirmation"
  return res.json({message: "Account created, check email"});
}
```

**Password Reset**:
```javascript
// ❌ Bad
if (!user) {
  return res.json({error: "Email not found"});
}

// ✓ Good
// Always say email sent (even if not)
return res.json({message: "If email exists, reset link sent"});
```

## 2. Timing Attacks

### The Vulnerability

```javascript
// Typical authentication flow
1. Find user by username     // ~5ms
2. Check if account locked    // ~2ms
3. Compare password hashes    // ~100ms (bcrypt)

If username invalid: Response in ~7ms
If password invalid: Response in ~107ms

Attacker measures timing:
  Fast response (7ms) → username invalid
  Slow response (107ms) → username valid, password wrong
```

### The Attack

```python
# Attacker's script
times = []
for username in potential_usernames:
    start = time()
    response = login(username, "wrong_password")
    elapsed = time() - start
    times.append((username, elapsed))

# Usernames with longer response times likely valid
valid_usernames = [u for u, t in times if t > 100ms]
```

### The Solution

**Option 1: Constant-Time Operations**
```javascript
const crypto = require('crypto');

// Use constant-time comparison
function constantTimeCompare(a, b) {
  if (a.length !== b.length) {
    return false;
  }
  
  // Node.js built-in constant-time
  return crypto.timingSafeEqual(
    Buffer.from(a),
    Buffer.from(b)
  );
}
```

**Option 2: Simulate Delay**
```javascript
app.post('/login', async (req, res) => {
  const start = Date.now();
  
  const user = await db.users.findOne({email: req.body.email});
  
  let valid = false;
  if (user) {
    valid = await bcrypt.compare(req.body.password, user.password_hash);
  } else {
    // Fake hash comparison even if user doesn't exist
    await bcrypt.compare(req.body.password, '$2b$10$fakehash...');
  }
  
  // Ensure minimum response time (e.g., 200ms)
  const elapsed = Date.now() - start;
  if (elapsed < 200) {
    await new Promise(resolve => setTimeout(resolve, 200 - elapsed));
  }
  
  if (!valid) {
    return res.status(401).json({error: "Invalid credentials"});
  }
  
  // Continue with login...
});
```

## 3. Password Storage

### Never Store Plain Text

```javascript
// ❌ NEVER DO THIS
await db.users.insert({
  email: "user@example.com",
  password: "MyPassword123"  // DANGER!
});
```

**Why**: Database breach = all passwords exposed

### Always Hash + Salt

```javascript
const bcrypt = require('bcrypt');

// Registration
app.post('/register', async (req, res) => {
  const {email, password} = req.body;
  
  // Generate salt and hash
  const saltRounds = 12;  // Higher = more secure, slower
  const password_hash = await bcrypt.hash(password, saltRounds);
  
  await db.users.insert({
    email,
    password_hash  // Store hash, not password!
  });
  
  res.json({message: "Registered"});
});

// Login
app.post('/login', async (req, res) => {
  const {email, password} = req.body;
  
  const user = await db.users.findOne({email});
  
  if (!user) {
    return res.status(401).json({error: "Invalid credentials"});
  }
  
  // Compare provided password with stored hash
  const valid = await bcrypt.compare(password, user.password_hash);
  
  if (!valid) {
    return res.status(401).json({error: "Invalid credentials"});
  }
  
  // Continue with login...
});
```

**How bcrypt Works**:
```
Password: "MyPassword123"
Salt: "$2b$12$randomsalthere"
Hash: bcrypt(password, salt) = "$2b$12$randomsalthere$hashedpasswordhere"

Stored in DB: "$2b$12$randomsalthere$hashedpasswordhere"

Login:
  User enters: "MyPassword123"
  Extract salt from stored hash: "$2b$12$randomsalthere"
  Compute: bcrypt("MyPassword123", "$2b$12$randomsalthere")
  Compare: computed hash === stored hash
  Match → Valid password
```

### Password Requirements

```javascript
function validatePassword(password) {
  const errors = [];
  
  if (password.length < 12) {
    errors.push("Minimum 12 characters");
  }
  
  if (!/[a-z]/.test(password)) {
    errors.push("Must contain lowercase letter");
  }
  
  if (!/[A-Z]/.test(password)) {
    errors.push("Must contain uppercase letter");
  }
  
  if (!/[0-9]/.test(password)) {
    errors.push("Must contain number");
  }
  
  if (!/[!@#$%^&*]/.test(password)) {
    errors.push("Must contain special character");
  }
  
  // Check against common passwords
  const commonPasswords = ['password', '123456', 'qwerty', ...];
  if (commonPasswords.includes(password.toLowerCase())) {
    errors.push("Password too common");
  }
  
  return errors;
}
```

## 4. Rate Limiting

### Prevent Brute Force

```javascript
const rateLimit = require('express-rate-limit');

// Login rate limiting
const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 minutes
  max: 5,  // 5 attempts per window
  message: "Too many login attempts, try again later",
  standardHeaders: true,
  legacyHeaders: false,
});

app.post('/login', loginLimiter, async (req, res) => {
  // Login logic...
});

// IP-based rate limiting
const apiLimiter = rateLimit({
  windowMs: 60 * 1000,  // 1 minute
  max: 100,  // 100 requests per minute
});

app.use('/api', apiLimiter);
```

### Account Lockout

```javascript
app.post('/login', async (req, res) => {
  const user = await db.users.findOne({email: req.body.email});
  
  if (!user) {
    return res.status(401).json({error: "Invalid credentials"});
  }
  
  // Check if account locked
  if (user.locked_until && user.locked_until > new Date()) {
    const minutesLeft = Math.ceil(
      (user.locked_until - new Date()) / 60000
    );
    return res.status(429).json({
      error: `Account locked. Try again in ${minutesLeft} minutes`
    });
  }
  
  const valid = await bcrypt.compare(req.body.password, user.password_hash);
  
  if (!valid) {
    // Increment failed attempts
    const failedAttempts = (user.failed_login_attempts || 0) + 1;
    
    if (failedAttempts >= 5) {
      // Lock account for 30 minutes
      await db.users.update(
        {_id: user._id},
        {
          $set: {
            failed_login_attempts: failedAttempts,
            locked_until: new Date(Date.now() + 30 * 60 * 1000)
          }
        }
      );
      
      return res.status(429).json({
        error: "Too many failed attempts. Account locked for 30 minutes"
      });
    }
    
    await db.users.update(
      {_id: user._id},
      {$set: {failed_login_attempts: failedAttempts}}
    );
    
    return res.status(401).json({error: "Invalid credentials"});
  }
  
  // Successful login - reset failed attempts
  await db.users.update(
    {_id: user._id},
    {$set: {failed_login_attempts: 0, locked_until: null}}
  );
  
  // Continue with login...
});
```

---

# Part 7: Implementation Guides

## Complete Production Example (Node.js + Express)

```javascript
// ========================================
// server.js - Complete Production Setup
// ========================================

const express = require('express');
const cookieParser = require('cookie-parser');
const Redis = require('ioredis');
const bcrypt = require('bcrypt');
const crypto = require('crypto');
const rateLimit = require('express-rate-limit');
const helmet = require('helmet');

// ========================================
// Configuration
// ========================================

const app = express();
const redis = new Redis({
  host: process.env.REDIS_HOST || 'localhost',
  port: process.env.REDIS_PORT || 6379,
  password: process.env.REDIS_PASSWORD,
  retryStrategy(times) {
    return Math.min(times * 50, 2000);
  }
});

const SESSION_TTL = 3600; // 1 hour
const SESSION_PREFIX = 'sess:';
const SALT_ROUNDS = 12;

// ========================================
// Middleware
// ========================================

app.use(helmet()); // Security headers
app.use(express.json());
app.use(cookieParser());

// Rate limiting
const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,
  message: "Too many login attempts"
});

// Authentication middleware
const requireAuth = async (req, res, next) => {
  try {
    const sessionId = req.cookies.session_id;
    
    if (!sessionId) {
      return res.status(401).json({error: "Not authenticated"});
    }
    
    const sessionData = await redis.get(SESSION_PREFIX + sessionId);
    
    if (!sessionData) {
      res.clearCookie('session_id');
      return res.status(401).json({error: "Session expired"});
    }
    
    req.session = JSON.parse(sessionData);
    req.sessionId = sessionId;
    
    // Extend session
    await redis.expire(SESSION_PREFIX + sessionId, SESSION_TTL);
    
    next();
  } catch (err) {
    console.error('Auth error:', err);
    res.status(500).json({error: "Authentication error"});
  }
};

// Authorization middleware
const requireRole = (...roles) => {
  return (req, res, next) => {
    if (!req.session || !roles.includes(req.session.role)) {
      return res.status(403).json({error: "Insufficient permissions"});
    }
    next();
  };
};

// ========================================
// Helper Functions
// ========================================

async function createSession(user, req) {
  const sessionId = crypto.randomBytes(32).toString('hex');
  
  const sessionData = {
    user_id: user.id,
    email: user.email,
    role: user.role,
    created_at: new Date().toISOString(),
    ip_address: req.ip,
    user_agent: req.headers['user-agent']
  };
  
  await redis.setex(
    SESSION_PREFIX + sessionId,
    SESSION_TTL,
    JSON.stringify(sessionData)
  );
  
  return sessionId;
}

function validatePassword(password) {
  const errors = [];
  
  if (password.length < 12) {
    errors.push("Minimum 12 characters");
  }
  if (!/[a-z]/.test(password)) {
    errors.push("Must contain lowercase");
  }
  if (!/[A-Z]/.test(password)) {
    errors.push("Must contain uppercase");
  }
  if (!/[0-9]/.test(password)) {
    errors.push("Must contain number");
  }
  if (!/[!@#$%^&*]/.test(password)) {
    errors.push("Must contain special character");
  }
  
  return errors;
}

// ========================================
// Routes
// ========================================

// REGISTER
app.post('/api/register', async (req, res) => {
  try {
    const {email, password, name} = req.body;
    
    // Validate input
    if (!email || !password || !name) {
      return res.status(400).json({error: "All fields required"});
    }
    
    // Validate password
    const passwordErrors = validatePassword(password);
    if (passwordErrors.length > 0) {
      return res.status(400).json({errors: passwordErrors});
    }
    
    // Check if user exists
    const existing = await db.users.findOne({email});
    if (existing) {
      // Don't reveal user exists - say "check email"
      return res.json({message: "Check your email to verify account"});
    }
    
    // Hash password
    const password_hash = await bcrypt.hash(password, SALT_ROUNDS);
    
    // Create user
    const user = await db.users.insert({
      email,
      password_hash,
      name,
      role: 'user',
      created_at: new Date(),
      verified: false
    });
    
    // Send verification email (not implemented here)
    // await sendVerificationEmail(user);
    
    res.json({message: "Check your email to verify account"});
    
  } catch (err) {
    console.error('Registration error:', err);
    res.status(500).json({error: "Registration failed"});
  }
});

// LOGIN
app.post('/api/login', loginLimiter, async (req, res) => {
  try {
    const start = Date.now();
    const {email, password} = req.body;
    
    if (!email || !password) {
      return res.status(400).json({error: "Email and password required"});
    }
    
    const user = await db.users.findOne({email});
    
    let valid = false;
    if (user) {
      // Check if locked
      if (user.locked_until && user.locked_until > new Date()) {
        const minutes = Math.ceil((user.locked_until - new Date()) / 60000);
        return res.status(429).json({
          error: `Account locked. Try again in ${minutes} minutes`
        });
      }
      
      valid = await bcrypt.compare(password, user.password_hash);
    } else {
      // Fake comparison to prevent timing attacks
      await bcrypt.compare(password, '$2b$12$fakehashtopreventtimingattack');
    }
    
    // Ensure minimum response time (200ms)
    const elapsed = Date.now() - start;
    if (elapsed < 200) {
      await new Promise(resolve => setTimeout(resolve, 200 - elapsed));
    }
    
    if (!valid) {
      // Increment failed attempts if user exists
      if (user) {
        const failedAttempts = (user.failed_login_attempts || 0) + 1;
        
        if (failedAttempts >= 5) {
          await db.users.update(
            {_id: user._id},
            {
              $set: {
                failed_login_attempts: failedAttempts,
                locked_until: new Date(Date.now() + 30 * 60 * 1000)
              }
            }
          );
          
          return res.status(429).json({
            error: "Too many failed attempts. Account locked for 30 minutes"
          });
        }
        
        await db.users.update(
          {_id: user._id},
          {$inc: {failed_login_attempts: 1}}
        );
      }
      
      return res.status(401).json({error: "Invalid credentials"});
    }
    
    // Reset failed attempts
    await db.users.update(
      {_id: user._id},
      {$set: {failed_login_attempts: 0, locked_until: null}}
    );
    
    // Create session
    const sessionId = await createSession(user, req);
    
    // Set cookie
    res.cookie('session_id', sessionId, {
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
      sameSite: 'strict',
      maxAge: SESSION_TTL * 1000
    });
    
    res.json({
      message: "Logged in successfully",
      user: {
        id: user.id,
        email: user.email,
        name: user.name,
        role: user.role
      }
    });
    
  } catch (err) {
    console.error('Login error:', err);
    res.status(500).json({error: "Login failed"});
  }
});

// GET CURRENT USER
app.get('/api/me', requireAuth, (req, res) => {
  res.json({
    user_id: req.session.user_id,
    email: req.session.email,
    role: req.session.role
  });
});

// LOGOUT
app.post('/api/logout', requireAuth, async (req, res) => {
  await redis.del(SESSION_PREFIX + req.sessionId);
  
  res.clearCookie('session_id', {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'strict'
  });
  
  res.json({message: "Logged out successfully"});
});

// LOGOUT ALL DEVICES
app.post('/api/logout-all', requireAuth, async (req, res) => {
  const keys = await redis.keys(SESSION_PREFIX + '*');
  
  for (const key of keys) {
    const data = await redis.get(key);
    if (data) {
      const session = JSON.parse(data);
      if (session.user_id === req.session.user_id) {
        await redis.del(key);
      }
    }
  }
  
  res.clearCookie('session_id');
  res.json({message: "Logged out from all devices"});
});

// ADMIN: VIEW ALL SESSIONS
app.get('/api/admin/sessions',
  requireAuth,
  requireRole('admin'),
  async (req, res) => {
    const keys = await redis.keys(SESSION_PREFIX + '*');
    const sessions = [];
    
    for (const key of keys) {
      const data = await redis.get(key);
      const ttl = await redis.ttl(key);
      
      if (data) {
        sessions.push({
          session_id: key.replace(SESSION_PREFIX, ''),
          ...JSON.parse(data),
          expires_in: ttl
        });
      }
    }
    
    res.json(sessions);
  }
);

// ADMIN: REVOKE SESSION
app.delete('/api/admin/sessions/:sessionId',
  requireAuth,
  requireRole('admin'),
  async (req, res) => {
    await redis.del(SESSION_PREFIX + req.params.sessionId);
    res.json({message: "Session revoked"});
  }
);

// ========================================
// Server Startup
// ========================================

const PORT = process.env.PORT || 3000;

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});

// Graceful shutdown
process.on('SIGTERM', async () => {
  console.log('SIGTERM received, closing connections...');
  await redis.quit();
  process.exit(0);
});
```

---

# Part 8: Quick Reference

## Decision Matrix: Which Authentication Method?

```
┌─────────────────────────────────────────────────────────────┐
│  AUTHENTICATION METHOD SELECTOR                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Traditional Web App (Server-rendered pages)               │
│    → Stateful (Sessions + Cookies)                         │
│                                                             │
│  SPA (React/Vue/Angular) + API                             │
│    → Stateless (JWT in HttpOnly cookies)                   │
│                                                             │
│  Mobile App                                                 │
│    → Stateless (JWT in secure storage)                     │
│                                                             │
│  Microservices                                              │
│    → Stateless (JWT across services)                       │
│                                                             │
│  Third-party API Access                                     │
│    → API Keys                                               │
│                                                             │
│  "Sign in with Google/Facebook"                            │
│    → OAuth 2.0 + OpenID Connect                            │
│                                                             │
│  Server-to-Server                                           │
│    → Client Credentials (OAuth 2.0) or API Keys            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Authentication Cheat Sheet

### Session-Based (Stateful)

**When to Use**: Traditional web apps, need immediate revocation

**Pros**: Secure, revocable, updates in real-time
**Cons**: Doesn't scale horizontally easily, needs Redis

```javascript
// Setup
const Redis = require('ioredis');
const redis = new Redis();

// Login
const sessionId = crypto.randomBytes(32).toString('hex');
await redis.setex(`sess:${sessionId}`, 3600, JSON.stringify(userData));
res.cookie('session_id', sessionId, {httpOnly: true, secure: true});

// Verify
const data = await redis.get(`sess:${sessionId}`);
if (!data) return unauthorized();

// Logout
await redis.del(`sess:${sessionId}`);
res.clearCookie('session_id');
```

### JWT-Based (Stateless)

**When to Use**: SPAs, mobile apps, microservices

**Pros**: Scalable, no server storage, cross-domain
**Cons**: Can't revoke, larger payload

```javascript
const jwt = require('jsonwebtoken');

// Login
const token = jwt.sign(
  {sub: user.id, role: user.role},
  process.env.JWT_SECRET,
  {expiresIn: '1h'}
);
res.json({token});

// Verify
try {
  const decoded = jwt.verify(token, process.env.JWT_SECRET);
  req.user = decoded;
} catch (err) {
  return unauthorized();
}

// "Logout" (client-side only)
// Delete token from client storage
localStorage.removeItem('token');
```

### API Keys

**When to Use**: Machine-to-machine, programmatic access

**Pros**: Simple, long-lived, revocable
**Cons**: Must protect like passwords

```javascript
// Generate
const apiKey = `sk_${crypto.randomBytes(32).toString('hex')}`;
const hash = crypto.createHash('sha256').update(apiKey).digest('hex');
await db.apiKeys.insert({key_hash: hash, user_id: user.id});

// Verify
const providedKey = req.headers.authorization.replace('Bearer ', '');
const hash = crypto.createHash('sha256').update(providedKey).digest('hex');
const keyData = await db.apiKeys.findOne({key_hash: hash});
if (!keyData) return unauthorized();
```

## Security Checklist

```
Authentication:
  ✓ Hash passwords with bcrypt (12+ rounds)
  ✓ Generic error messages ("Invalid credentials")
  ✓ Rate limit login attempts
  ✓ Account lockout after failed attempts
  ✓ Prevent timing attacks (constant-time comparison)
  ✓ Require strong passwords (12+ chars, mixed case, numbers, symbols)
  ✓ Implement MFA for sensitive accounts
  ✓ Use HTTPS only
  
Cookies:
  ✓ HttpOnly (prevent XSS)
  ✓ Secure (HTTPS only)
  ✓ SameSite=Strict or Lax (prevent CSRF)
  ✓ Appropriate Max-Age
  
Sessions:
  ✓ Generate cryptographically secure session IDs
  ✓ Set reasonable TTL (15min - 1hr)
  ✓ Extend on activity
  ✓ Invalidate on logout
  ✓ Store in Redis (fast, built-in expiration)
  
JWTs:
  ✓ Strong secret key (256+ bits)
  ✓ Short expiration (15min - 1hr)
  ✓ Refresh token pattern for longer sessions
  ✓ Store in HttpOnly cookies (not localStorage)
  ✓ Verify signature on every request
  ✓ Check expiration
  
Authorization:
  ✓ Implement RBAC
  ✓ Principle of least privilege
  ✓ Check permissions on every protected route
  ✓ Verify resource ownership
  ✓ Audit permission changes
  
General:
  ✓ Never store passwords in plain text
  ✓ Never log sensitive data (passwords, tokens)
  ✓ Rotate secrets regularly
  ✓ Monitor for suspicious activity
  ✓ Keep dependencies updated
  ✓ Security headers (helmet.js)
  ✓ CORS configuration
  ✓ Input validation and sanitization
```

## Common Vulnerabilities

### XSS (Cross-Site Scripting)

**Attack**: Inject malicious script to steal cookies/tokens

**Prevention**:
```javascript
// ✓ HttpOnly cookies (JavaScript can't access)
res.cookie('session', id, {httpOnly: true});

// ✓ Content Security Policy
app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    scriptSrc: ["'self'"]
  }
}));

// ✓ Sanitize user input
const sanitize = require('sanitize-html');
const clean = sanitize(userInput);
```

### CSRF (Cross-Site Request Forgery)

**Attack**: Trick user into making unwanted request

**Prevention**:
```javascript
// ✓ SameSite cookies
res.cookie('session', id, {sameSite: 'strict'});

// ✓ CSRF tokens
const csrf = require('csurf');
app.use(csrf({cookie: true}));

// ✓ Check Origin/Referer headers
if (req.headers.origin !== 'https://yourapp.com') {
  return forbidden();
}
```

### SQL Injection

**Attack**: Inject SQL through user input

**Prevention**:
```javascript
// ✓ Use parameterized queries
const query = 'SELECT * FROM users WHERE email = ?';
db.query(query, [email]);

// ✓ Use ORM (Sequelize, TypeORM)
const user = await User.findOne({where: {email}});

// ❌ NEVER concatenate
const query = `SELECT * FROM users WHERE email = '${email}'`; // DANGER!
```

### Brute Force

**Attack**: Try many passwords

**Prevention**:
```javascript
// ✓ Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5
});
app.post('/login', limiter, ...);

// ✓ Account lockout
if (failedAttempts >= 5) {
  user.locked_until = new Date(Date.now() + 30 * 60 * 1000);
}

// ✓ CAPTCHA after failures
if (failedAttempts >= 3) {
  requireCaptcha = true;
}
```

## Troubleshooting Guide

### "Session expired" immediately

**Causes**:
- Cookie not being set/sent
- Domain mismatch
- SameSite too strict
- HTTPS/Secure mismatch

**Debug**:
```javascript
// Check cookie in response
console.log('Response headers:', res.headers['set-cookie']);

// Check cookie in request
console.log('Request cookies:', req.cookies);

// Verify domain
res.cookie('session', id, {
  domain: '.example.com',  // Works for all subdomains
  sameSite: 'lax'          // Try lax instead of strict
});
```

### JWT "Invalid signature"

**Causes**:
- Wrong secret key
- Token tampered
- Token format invalid

**Debug**:
```javascript
// Verify secret
console.log('Using secret:', process.env.JWT_SECRET);

// Decode without verifying (see payload)
const decoded = jwt.decode(token);
console.log('Token payload:', decoded);

// Check algorithm
const decoded = jwt.verify(token, secret, {algorithms: ['HS256']});
```

### CORS errors

**Causes**:
- Missing CORS headers
- Credentials not allowed
- Wrong origin

**Fix**:
```javascript
const cors = require('cors');

app.use(cors({
  origin: 'https://yourfrontend.com',
  credentials: true  // Allow cookies
}));

// Or manually
app.use((req, res, next) => {
  res.header('Access-Control-Allow-Origin', 'https://yourfrontend.com');
  res.header('Access-Control-Allow-Credentials', 'true');
  res.header('Access-Control-Allow-Methods', 'GET,POST,PUT,DELETE');
  res.header('Access-Control-Allow-Headers', 'Content-Type,Authorization');
  next();
});
```

### Redis connection failed

**Causes**:
- Redis not running
- Wrong host/port
- Network issue

**Fix**:
```javascript
const redis = new Redis({
  host: 'localhost',
  port: 6379,
  retryStrategy(times) {
    console.log(`Redis retry attempt ${times}`);
    return Math.min(times * 50, 2000);
  }
});

redis.on('error', (err) => {
  console.error('Redis error:', err);
});

redis.on('connect', () => {
  console.log('Redis connected');
});

// Test connection
redis.ping((err, result) => {
  console.log('Redis ping:', result); // Should be "PONG"
});
```

---

## Final Recommendations

### For New Projects

1. **Use an Auth Provider**: Auth0, Clerk, Supabase, Firebase Auth
   - Handles complexity for you
   - Security best practices built-in
   - Save months of development

2. **If Building Yourself**:
   - Start with sessions (simpler, more secure)
   - Use bcrypt for passwords (12+ rounds)
   - Redis for session storage
   - Implement MFA early

3. **Always**:
   - HTTPS only
   - HttpOnly cookies
   - Rate limiting
   - Generic error messages
   - Strong password requirements

### For Learning

1. **Implement session-based first**: Understand fundamentals
2. **Then try JWT**: Understand stateless
3. **Implement OAuth**: Learn delegation
4. **Read RFCs**: OAuth 2.0 (RFC 6749), JWT (RFC 7519)
5. **Study breaches**: Learn from real-world failures

### Resources

**Standards**:
- OAuth 2.0: https://oauth.net/2/
- OpenID Connect: https://openid.net/connect/
- JWT: https://jwt.io/

**Libraries**:
- Node.js: passport.js, jose, bcrypt
- Python: authlib, PyJWT, passlib
- Go: golang-jwt, oauth2

**Auth Providers**:
- Auth0 (enterprise)
- Clerk (modern, great DX)
- Supabase (open source)
- Firebase Auth (Google)

---

## Conclusion

Authentication and authorization are **critical** to application security. This guide covered:

- ✓ Historical evolution and why we have current methods
- ✓ Sessions vs JWTs vs API keys vs OAuth
- ✓ When to use each approach
- ✓ Security best practices
- ✓ Common vulnerabilities and mitigations
- ✓ Production-ready code examples

**Remember**:
- Security is **not optional**
- Use proven solutions (don't roll your own crypto)
- Consider auth providers for production
- Keep learning - security evolves

**The most important principle**: *Defense in depth* - multiple layers of security, so if one fails, others protect you.

---

**This guide is comprehensive but not exhaustive. Security is an ongoing journey. Stay updated, stay vigilant, stay secure.**
