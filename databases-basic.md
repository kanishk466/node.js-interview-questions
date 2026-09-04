# 🚀 Databases (MongoDB & PostgreSQL) — Interview Prep Guide (Basic Level)

> **Level:** Basic
> **Format:** What → Why → How → Example → Interview Answer
> **Language:** Hinglish (understanding) + English (interview delivery)

---

## 📑 Table of Contents

| # | Topic | Difficulty | Frequency |
|---|-------|-----------|-----------|
| 1 | [SQL vs NoSQL](#1-sql-vs-nosql-databases) | ⭐ | 🔥🔥🔥🔥🔥 |
| 2 | [MongoDB vs PostgreSQL](#2-when-to-choose-mongodb-over-postgresql) | ⭐ | 🔥🔥🔥🔥🔥 |
| 3 | [Collection in MongoDB](#3-what-is-a-collection-in-mongodb) | ⭐ | 🔥🔥🔥 |
| 4 | [Document in MongoDB](#4-what-is-a-document-in-mongodb) | ⭐ | 🔥🔥🔥 |
| 5 | [CRUD in MongoDB](#5-crud-operations-in-mongodb) | ⭐ | 🔥🔥🔥🔥 |
| 6 | [Indexes](#6-indexes-in-databases) | ⭐ | 🔥🔥🔥🔥 |
| 7 | [Primary Key](#7-what-is-a-primary-key) | ⭐ | 🔥🔥🔥🔥 |
| 8 | [Foreign Key](#8-what-is-a-foreign-key) | ⭐ | 🔥🔥🔥🔥 |

---

## 🧠 Answer Framework

```
1️⃣  WHAT  →  "Yeh kya hai?"
2️⃣  WHY   →  "Iski zaroorat kyun hai?"
3️⃣  HOW   →  "Yeh kaam kaise karta hai?"
4️⃣  CODE  →  "Ek example dikhata hoon..."
5️⃣  USE   →  "Practical mein main isse tab use karta hoon jab..."
```

---

---

## 1. SQL vs NoSQL Databases

> 🏆 **Most asked database question — almost guaranteed in every interview!**

### 1️⃣ WHAT

Databases ki duniya mein **2 major categories** hain:

| Feature | SQL (Relational) | NoSQL (Non-Relational) |
|---------|-----------------|----------------------|
| **Full Form** | Structured Query Language | Not Only SQL |
| **Data Model** | Tables (Rows & Columns) | Documents, Key-Value, Graph, Column |
| **Schema** | **Fixed** (pehle define karo) | **Flexible** (kabhi bhi change karo) |
| **Scaling** | **Vertical** (bigger server 💰) | **Horizontal** (more servers 🖥️🖥️🖥️) |
| **Query Language** | SQL (`SELECT * FROM...`) | Varies (JSON-like queries) |
| **Relationships** | JOINs (strong) | Embedding / Referencing |
| **Transactions** | ACID (strong guarantee) | BASE (eventual consistency) |
| **Examples** | PostgreSQL, MySQL, Oracle, SQL Server | MongoDB, Redis, Cassandra, DynamoDB |

### 2️⃣ WHY

Dono alag problems solve karte hain — koi ek "better" nahi hai, **context matter** karta hai:

```
SQL = Excel Sheet jaisa (Structured)
┌────┬─────────┬──────────────┬─────┐
│ ID │ Name    │ Email        │ Age │  ← Fixed columns
├────┼─────────┼──────────────┼─────┤
│ 1  │ Kanishk │ k@test.com   │ 25  │  ← Structured rows
│ 2  │ Rahul   │ r@test.com   │ 28  │
└────┴─────────┴──────────────┴─────┘

NoSQL = JSON Document jaisa (Flexible)
{
  "_id": 1,
  "name": "Kanishk",
  "email": "k@test.com",
  "age": 25,
  "hobbies": ["coding", "gaming"],     ← Array! (SQL mein alag table)
  "address": { "city": "Delhi" }       ← Nested! (SQL mein JOIN)
}
```

### 3️⃣ HOW — Decision Framework

```
SQL choose karo jab:
  ✅ Data structured aur predictable hai
  ✅ Complex queries aur JOINs chahiye
  ✅ ACID transactions critical hain (banking, payments)
  ✅ Data integrity sabse zaroori hai
  ✅ Reporting aur analytics heavy hai

NoSQL choose karo jab:
  ✅ Data structure frequently change hota hai
  ✅ Rapid prototyping / MVP banana hai
  ✅ Horizontal scaling chahiye (big data, millions of users)
  ✅ Hierarchical / nested data hai (JSON-like)
  ✅ High write throughput chahiye (IoT, logging)
  ✅ Different records ke different fields hain (product catalog)
```

### 4️⃣ CODE

```sql
-- ═══ SQL (PostgreSQL) ═══

-- Fixed schema define karna zaroori hai
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    age INTEGER CHECK (age >= 18),
    created_at TIMESTAMP DEFAULT NOW()
);

-- Data insert
INSERT INTO users (name, email, age)
VALUES ('Kanishk', 'k@test.com', 25);

-- Complex JOIN query
SELECT u.name, o.total, p.name AS product
FROM users u
JOIN orders o ON u.id = o.user_id
JOIN products p ON o.product_id = p.id
WHERE o.total > 1000
ORDER BY o.total DESC;
```

```javascript
// ═══ NoSQL (MongoDB) ═══

// No schema needed — directly insert!
db.users.insertOne({
    name: "Kanishk",
    email: "k@test.com",
    age: 25,
    hobbies: ["coding", "gaming"],        // Array — no extra table!
    address: { city: "Delhi", pin: 110001 }, // Nested — no JOIN!
    createdAt: new Date()
});

// Query (JSON-like)
db.users.find({
    age: { $gte: 18 },
    "address.city": "Delhi"
}).sort({ age: -1 });
```

### 5️⃣ INTERVIEW ANSWER

> *"**SQL databases** are relational, use fixed schemas with tables and rows, support complex JOINs, and provide strong **ACID** guarantees — ideal for structured data like financial systems. **NoSQL databases** offer flexible schemas, horizontal scalability, and various data models like document, key-value, and graph — ideal for rapidly evolving data, real-time apps, and big data. The choice depends on data structure, consistency requirements, and scaling needs. In modern applications, many teams use **both** — PostgreSQL for transactional data and MongoDB or Redis for flexible, high-throughput workloads."*

📌 **One-liner:** SQL = structured + ACID + JOINs | NoSQL = flexible + scalable + JSON-like

---

---

## 2. When to Choose MongoDB Over PostgreSQL

### 1️⃣ WHAT

Ye **decision-making** question hai — interviewer check karta hai ki tumhe **real-world trade-offs** samajh aate hain ya nahi.

### 2️⃣ WHY

Wrong database choice = performance issues, scaling problems, development slowdown. Interview mein "it depends" bolna kaafi nahi — **specific scenarios** batane padte hain.

### 3️⃣ HOW — Scenario-Based Comparison

| Scenario | MongoDB ✅ | PostgreSQL ✅ | Why? |
|----------|-----------|--------------|------|
| **E-commerce catalog** | ✅ | | Products ke different attributes (laptop has RAM, t-shirt has size) |
| **Banking / Payments** | | ✅ | ACID critical — paisa lose nahi ho sakta |
| **CMS / Blog** | ✅ | | Content types vary — article, video, gallery |
| **Complex reporting** | | ✅ | JOINs + aggregations + window functions |
| **IoT / Sensor data** | ✅ | | High write throughput, flexible schema |
| **User auth & roles** | | ✅ | Referential integrity, complex relationships |
| **Rapid MVP** | ✅ | | Schema-less = fast iteration |
| **Geospatial** | ✅ | ✅ | Both good — MongoDB GeoJSON, PG PostGIS |
| **Social media feed** | ✅ | | Nested comments, likes, varied content |
| **Multi-tenant SaaS** | | ✅ | Row-level security, complex JOINs |

### 4️⃣ CODE — Same Problem, Different Approach

```javascript
// ═══ MongoDB: Product Catalog (Flexible Schema) ═══
// Har product ke DIFFERENT fields — no problem!
db.products.insertMany([
    {
        name: "Dell Laptop",
        category: "electronics",
        specs: { ram: "16GB", gpu: "RTX 4060", storage: "512GB SSD" }
    },
    {
        name: "Nike T-Shirt",
        category: "clothing",
        specs: { size: "XL", color: "Black", fabric: "Cotton" }
    },
    {
        name: "Clean Code Book",
        category: "books",
        specs: { author: "Robert Martin", pages: 464, isbn: "978-0132350884" }
    }
]);
// ✅ Each product has completely different specs — natural fit!
```

```sql
-- ═══ PostgreSQL: Same thing is MESSY ═══
-- Option 1: Many NULL columns 😩
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(200),
    category VARCHAR(50),
    ram VARCHAR(20),        -- NULL for t-shirts & books
    gpu VARCHAR(50),        -- NULL for t-shirts & books
    size VARCHAR(5),        -- NULL for laptops & books
    color VARCHAR(20),      -- NULL for laptops & books
    author VARCHAR(100),    -- NULL for laptops & t-shirts
    pages INTEGER           -- NULL for laptops & t-shirts
);
-- 💀 50%+ columns NULL — wasteful!

-- Option 2: JSONB column (PG ka MongoDB-like feature!)
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(200),
    category VARCHAR(50),
    specs JSONB  -- Flexible like MongoDB!
);
-- ✅ Better! But MongoDB is still more natural for this.
```

```sql
-- ═══ PostgreSQL: Banking (ACID Critical) ═══
BEGIN;
    UPDATE accounts SET balance = balance - 5000 WHERE id = 1;
    UPDATE accounts SET balance = balance + 5000 WHERE id = 2;
    -- Agar beech mein crash → ROLLBACK → paisa safe! ✅
COMMIT;
-- MongoDB can do this too (v4.0+) but PG is native & battle-tested.
```

### 5️⃣ INTERVIEW ANSWER

> *"I choose **MongoDB** when the data schema is **flexible or evolving** — like product catalogs where each product has different attributes, CMS content, or IoT sensor data. It's also better for **rapid prototyping** since there's no schema migration overhead, and for **high write throughput** scenarios. I choose **PostgreSQL** when **data integrity and ACID compliance** are critical — like financial systems, when I need **complex JOINs and reporting**, or when relationships between entities are well-defined and stable. In many real-world projects, I actually use **both**: PostgreSQL for core transactional data and MongoDB for flexible, high-volume data stores."*

📌 **One-liner:** MongoDB = flexible schema + fast iteration + high writes | PostgreSQL = ACID + JOINs + data integrity

---

---

## 3. What is a Collection in MongoDB?

### 1️⃣ WHAT

Collection MongoDB mein **documents ka group** hota hai — bilkul SQL ke **table** jaisa, but with key differences.

| SQL | MongoDB |
|-----|---------|
| Database | Database |
| **Table** | **Collection** |
| Row | Document |
| Column | Field |

### 2️⃣ WHY

Data ko **logically organize** karne ke liye — users ka data alag, orders ka alag, products ka alag. Bina collections ke sab data ek jagah hota — chaos!

### 3️⃣ HOW — Structure

```
MongoDB Server
└── Database: "myApp"
    ├── Collection: "users"        ← User documents
    ├── Collection: "orders"       ← Order documents
    ├── Collection: "products"     ← Product documents
    ├── Collection: "sessions"     ← Session documents
    └── Collection: "logs"         ← Log documents
```

**Key Differences from SQL Table:**

| Feature | SQL Table | MongoDB Collection |
|---------|-----------|-------------------|
| Schema | Fixed (predefined) | Flexible (dynamic) |
| Creation | `CREATE TABLE` zaroori | Auto-created on first insert |
| Columns | Same for all rows | Different fields per document |
| Size Limit | No limit | No limit (but document ≤ 16MB) |

### 4️⃣ CODE

```javascript
// ═══ Auto-Create (Most Common) ═══
// Pehla document insert karo → collection automatically ban jayegi!
db.users.insertOne({ name: "Kanishk", age: 25 });
// "users" collection ab exist karti hai ✅

// ═══ Explicit Create (With Options) ═══
db.createCollection("logs", {
    capped: true,          // Fixed size — purane logs auto-delete
    size: 10485760,        // 10MB max
    max: 5000              // Max 5000 documents
});
// Use case: Application logs — sirf latest 5000 rakho

// ═══ Useful Commands ═══
show collections;          // List all collections
db.users.stats();          // Collection statistics
db.users.countDocuments(); // Count documents
db.users.drop();           // Delete entire collection ⚠️

// ═══ Mongoose (Application Level) ═══
const mongoose = require("mongoose");

// Mongoose model create karo → collection auto-banti hai
const User = mongoose.model("User", userSchema);
// "User" → collection name = "users" (auto pluralized + lowercase)
```

### 5️⃣ INTERVIEW ANSWER

> *"A **collection** in MongoDB is a grouping of documents, equivalent to a **table** in SQL. The key differences are: collections don't enforce a **fixed schema** — documents within the same collection can have different fields. Collections are created **automatically** when the first document is inserted, though you can explicitly create them with options like **capped collections** for fixed-size, FIFO storage — useful for logs. In Mongoose, the collection name is auto-generated by pluralizing the model name."*

📌 **One-liner:** Collection = MongoDB ka Table | Schema-less | Auto-created on first insert

---

---

## 4. What is a Document in MongoDB?

### 1️⃣ WHAT

Document MongoDB ka **basic data unit** hai — SQL ki **row** ka equivalent. Ye **BSON** (Binary JSON) format mein store hota hai.

> **Simple:** JSON object jo database mein save hota hai = Document

### 2️⃣ WHY

Documents JSON-like hote hain → developers ke liye **natural feel**, nested data **easy**, schema **flexible**. Frontend (React) se backend (Node) se database (Mongo) — **same JSON format** everywhere!

### 3️⃣ HOW — Structure Comparison

```
SQL Row (Flat, Fixed):
┌────┬─────────┬──────────────┬─────┬──────────┐
│ id │ name    │ email        │ age │ city     │
├────┼─────────┼──────────────┼─────┼──────────┤
│ 1  │ Kanishk │ k@test.com   │ 25  │ Delhi    │
└────┴─────────┴──────────────┴─────┴──────────┘
❌ Array nahi daal sakte
❌ Nested object nahi daal sakte

MongoDB Document (Rich, Flexible):
{
    "_id": ObjectId("64f8a1b2c3d4e5f6"),  ← Auto-generated PK
    "name": "Kanishk",
    "email": "k@test.com",
    "age": 25,
    "skills": ["JavaScript", "Node.js", "MongoDB"],  ← Array! ✅
    "address": {                                       ← Nested! ✅
        "city": "Delhi",
        "pin": 110001,
        "geo": { "lat": 28.6, "lng": 77.2 }           ← Deep nesting! ✅
    },
    "isActive": true,
    "createdAt": ISODate("2024-01-15T10:30:00Z")
}
```

**BSON Data Types:**

| Type | Example | Notes |
|------|---------|-------|
| **String** | `"Kanishk"` | UTF-8 |
| **Number** | `25`, `3.14` | Int32, Int64, Double |
| **Boolean** | `true`, `false` | |
| **Date** | `ISODate("2024-01-01")` | UTC timestamp |
| **ObjectId** | `ObjectId("64f...")` | 12-byte unique ID |
| **Array** | `[1, "two", { three: 3 }]` | Mixed types allowed! |
| **Object** | `{ city: "Delhi" }` | Nested document |
| **Null** | `null` | |
| **Binary** | `BinData(...)` | Files, images |
| **Regex** | `/pattern/i` | |

### 4️⃣ CODE

```javascript
// ═══ Create Document ═══
db.users.insertOne({
    name: "Kanishk",
    email: "k@test.com",
    age: 25,
    isActive: true,
    skills: ["JavaScript", "Node.js"],
    address: {
        city: "Delhi",
        pin: 110001
    },
    createdAt: new Date()
});

// ═══ Read Document ═══
const user = db.users.findOne({ email: "k@test.com" });
console.log(user.name);           // "Kanishk"
console.log(user.skills[0]);      // "JavaScript"
console.log(user.address.city);   // "Delhi"

// ═══ Update Document (Partial!) ═══
db.users.updateOne(
    { name: "Kanishk" },
    {
        $set: { age: 26 },                    // Update field
        $push: { skills: "MongoDB" },         // Add to array
        $set: { "address.city": "Mumbai" }    // Update nested field
    }
);

// ═══ Mongoose Document ═══
const user = new User({
    name: "Kanishk",
    email: "k@test.com",
    age: 25
});
await user.save();  // Document saved with auto _id + timestamps
```

### 5️⃣ INTERVIEW ANSWER

> *"A **document** in MongoDB is the fundamental unit of data, equivalent to a **row** in SQL. It's stored in **BSON** (Binary JSON) format, which supports rich data types like arrays, nested objects, dates, ObjectIds, and more. Unlike SQL rows, documents in the same collection can have **different structures** — one user document might have a `profile` object while another doesn't. Each document automatically gets a unique `_id` field (ObjectId) if not provided. The 16MB per-document size limit is an important constraint to keep in mind."*

📌 **One-liner:** Document = MongoDB ki Row | BSON format | Arrays + Nested objects | 16MB limit

---

---

## 5. CRUD Operations in MongoDB

> 🏆 **Most practical question — coding round mein bhi aata hai!**

### 1️⃣ WHAT

CRUD = **C**reate, **R**ead, **U**pdate, **D**elete — database ke **4 fundamental operations**.

### 2️⃣ WHY

Har application ka **core data interaction** inhi 4 operations pe based hota hai — user signup (Create), profile view (Read), profile edit (Update), account delete (Delete).

### 3️⃣ HOW — CRUD Mapping

| CRUD | MongoDB Method | SQL Equivalent | HTTP Method |
|------|---------------|---------------|-------------|
| **C**reate | `insertOne()`, `insertMany()` | `INSERT INTO` | POST |
| **R**ead | `find()`, `findOne()` | `SELECT` | GET |
| **U**pdate | `updateOne()`, `updateMany()` | `UPDATE` | PUT/PATCH |
| **D**elete | `deleteOne()`, `deleteMany()` | `DELETE` | DELETE |

### 4️⃣ CODE

```javascript
// ═══════════════════════════════════════════
// 🟢 CREATE
// ═══════════════════════════════════════════

// Single document
db.users.insertOne({
    name: "Kanishk",
    email: "k@test.com",
    age: 25,
    role: "developer"
});
// Returns: { acknowledged: true, insertedId: ObjectId("...") }

// Multiple documents
db.users.insertMany([
    { name: "Rahul", email: "r@test.com", age: 28, role: "designer" },
    { name: "Priya", email: "p@test.com", age: 24, role: "developer" },
    { name: "Amit", email: "a@test.com", age: 30, role: "manager" }
]);
// Returns: { acknowledged: true, insertedIds: { 0: ObjectId, 1: ... } }

// ═══════════════════════════════════════════
// 🔵 READ (Most versatile!)
// ═══════════════════════════════════════════

// All documents
db.users.find();

// Filter (WHERE clause)
db.users.find({ role: "developer" });
db.users.find({ age: { $gte: 25, $lte: 30 } });  // 25 <= age <= 30
db.users.find({ $or: [{ role: "developer" }, { age: { $lt: 25 } }] });

// Single document
db.users.findOne({ email: "k@test.com" });

// Projection (SELECT specific fields)
db.users.find(
    {},                          // No filter (all docs)
    { name: 1, email: 1, _id: 0 } // Only name & email, exclude _id
);

// Sort + Limit + Skip (Pagination!)
db.users.find()
    .sort({ age: -1 })     // Descending by age
    .skip(0)               // Offset (page 1 = 0, page 2 = 10)
    .limit(10);            // 10 per page

// Count
db.users.countDocuments({ role: "developer" });

// ═══════════════════════════════════════════
// 🟡 UPDATE
// ═══════════════════════════════════════════

// Update ONE document
db.users.updateOne(
    { email: "k@test.com" },       // Filter: kaunsa document?
    { $set: { age: 26 } }          // Update: kya change karna hai?
);

// Update MANY documents
db.users.updateMany(
    { role: "developer" },
    { $set: { department: "Engineering" } }
);

// Useful Update Operators:
db.users.updateOne(
    { name: "Kanishk" },
    {
        $set: { "address.city": "Mumbai" },  // Set nested field
        $inc: { age: 1 },                     // Increment (age++)
        $push: { skills: "MongoDB" },         // Add to array
        $pull: { skills: "Java" },            // Remove from array
        $addToSet: { tags: "backend" },       // Add unique to array
        $unset: { tempField: "" }             // Remove field entirely
    }
);

// Upsert (Update if exists, Insert if not!)
db.users.updateOne(
    { email: "new@test.com" },
    { $set: { name: "New User", age: 22, role: "intern" } },
    { upsert: true }  // ← Magic flag!
);

// Replace entire document (⚠️ Careful!)
db.users.replaceOne(
    { name: "Kanishk" },
    { name: "Kanishk", email: "new@test.com", age: 26 }
    // Purane fields jo nahi diye → GONE!
);

// ═══════════════════════════════════════════
// 🔴 DELETE
// ═══════════════════════════════════════════

// Delete ONE
db.users.deleteOne({ email: "a@test.com" });

// Delete MANY
db.users.deleteMany({ role: "intern" });

// Delete ALL (⚠️ DANGER!)
db.users.deleteMany({});  // Poora collection khaali!

// Drop collection (structure + data — both gone!)
db.users.drop();
```

### Mongoose CRUD (Application Level)

```javascript
const User = require("./models/User");

// Create
const user = await User.create({ name: "Kanishk", email: "k@test.com" });

// Read
const users = await User.find({ role: "developer" }).sort("-age").limit(10);
const user = await User.findById("64f8a...");
const user = await User.findOne({ email: "k@test.com" });

// Update
await User.findByIdAndUpdate(id, { age: 26 }, { new: true });
await User.updateMany({ role: "dev" }, { $set: { dept: "Eng" } });

// Delete
await User.findByIdAndDelete(id);
await User.deleteMany({ isActive: false });
```

### 5️⃣ INTERVIEW ANSWER

> *"CRUD operations in MongoDB map to: **Create** using `insertOne()` and `insertMany()`, **Read** using `find()` and `findOne()` with filters, projections, sorting, and pagination, **Update** using `updateOne()` and `updateMany()` with operators like `$set`, `$inc`, `$push`, and `$pull`, and **Delete** using `deleteOne()` and `deleteMany()`. MongoDB also supports **upsert** — update if the document exists, insert if it doesn't. I always use specific filters to avoid accidental bulk updates or deletes, and in production I use Mongoose methods like `findByIdAndUpdate` for cleaner code."*

📌 **One-liner:** C=insert | R=find | U=updateOne ($set/$inc/$push) | D=deleteOne | +upsert

---

---

## 6. Indexes in Databases

> 🏆 **Performance question — senior-level interviews mein guaranteed!**

### 1️⃣ WHAT

Index ek **special data structure** (usually B-Tree) hai jo database ko **specific fields pe fast lookup** karne deta hai — **poora table/collection scan kiye bina**.

> **Analogy:** Book ka **index page** 📖
> - Bina index: Poora book page-by-page padho → "Chapter 5 kahan hai?" 💀 Slow
> - Index ke saath: Index page pe dekho → "Chapter 5 → Page 142" ✅ Instant!

### 2️⃣ WHY

| Scenario | Without Index (Full Scan) | With Index (B-Tree Lookup) |
|----------|--------------------------|---------------------------|
| Find user by email (1M users) | ~2-5 seconds 💀 | ~2-5 milliseconds ✅ |
| Sort by date | Full scan + sort | Pre-sorted in index |
| Unique check (email) | Check all 1M rows | Instant hash/B-tree lookup |
| Range query (age 25-30) | Scan all | Jump to range directly |

**Trade-off (Zaroori hai!):**

| | Benefit | Cost |
|---|---------|------|
| **Read** | ⚡ 100-1000× faster | — |
| **Write** | — | 🐢 Slower (index update on every insert/update/delete) |
| **Storage** | — | 💾 Extra disk space (10-30% of data size) |

> **Rule:** Har field pe index mat banao! Sirf **frequently queried** fields pe.

### 3️⃣ HOW — How Index Works Internally

```
Without Index (COLLSCAN — Collection Scan):
Query: email = "k@test.com"

[Doc1] → [Doc2] → [Doc3] → ... → [Doc500000] → FOUND! 💀
Time: O(n) — 500,000 documents check kiye

With Index (IXSCAN — Index Scan):
Query: email = "k@test.com"

B-Tree Index on email:
         "m@test.com"
        /            \
  "a@test.com"    "z@test.com"
     /    \
"k@test.com"  "l@test.com"
     ↓
  Doc Pointer → Doc500000  ✅ FOUND!
Time: O(log n) — ~19 comparisons only!
```

### 4️⃣ CODE

```javascript
// ═══ MongoDB Indexes ═══

// Single field index
db.users.createIndex({ email: 1 });          // 1 = Ascending
db.users.createIndex({ createdAt: -1 });     // -1 = Descending

// Unique index (duplicate prevention!)
db.users.createIndex({ email: 1 }, { unique: true });
// Ab 2 users same email nahi rakh sakte

// Compound index (multi-field queries)
db.orders.createIndex({ userId: 1, createdAt: -1 });
// Fast for: db.orders.find({ userId: 123 }).sort({ createdAt: -1 })

// Text index (full-text search!)
db.articles.createIndex({ title: "text", content: "text" });
db.articles.find({ $text: { $search: "javascript tutorial" } });

// Check if index is being used
db.users.find({ email: "k@test.com" }).explain("executionStats");
// Look for:
//   "stage": "IXSCAN"  ✅ Index used!
//   "stage": "COLLSCAN" ❌ Full scan — index needed!

// List all indexes
db.users.getIndexes();

// Drop index (if not needed)
db.users.dropIndex("email_1");
```

```sql
-- ═══ PostgreSQL Indexes ═══

-- Single column
CREATE INDEX idx_users_email ON users(email);

-- Unique index
CREATE UNIQUE INDEX idx_users_username ON users(username);

-- Compound index
CREATE INDEX idx_orders_user_date ON orders(user_id, created_at DESC);

-- Partial index (only active users!)
CREATE INDEX idx_active_users ON users(email) WHERE is_active = true;

-- GIN index (JSONB, arrays, full-text)
CREATE INDEX idx_products_tags ON products USING GIN(tags);

-- Check query plan
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'k@test.com';
-- Look for:
--   "Index Scan"  ✅
--   "Seq Scan"    ❌ Full table scan
```

### Index Types Quick Reference

| Index Type | Use Case | MongoDB | PostgreSQL |
|-----------|----------|---------|------------|
| **Single** | One field queries | `{ email: 1 }` | `ON users(email)` |
| **Compound** | Multi-field queries | `{ userId: 1, date: -1 }` | `ON orders(user_id, date)` |
| **Unique** | No duplicates | `{ unique: true }` | `UNIQUE INDEX` |
| **Text** | Full-text search | `{ title: "text" }` | `USING GIN(to_tsvector(...))` |
| **Partial** | Subset of data | `{ partialFilterExpression }` | `WHERE condition` |
| **TTL** | Auto-expire data | `{ expireAfterSeconds: 3600 }` | — |

### 5️⃣ INTERVIEW ANSWER

> *"Indexes are special data structures — typically **B-Trees** — that allow the database to find data **without scanning every row**. They dramatically speed up read queries, turning a full collection scan (O(n)) into a logarithmic lookup (O(log n)). The trade-off is **slower writes** since the index must be updated on every insert, update, and delete, plus **increased storage**. I create indexes on fields frequently used in `WHERE`, `SORT`, and `JOIN` clauses. I always verify with `EXPLAIN` that queries use indexes (`IXSCAN` vs `COLLSCAN`). I use **compound indexes** for multi-field queries and **unique indexes** for constraints like email. A key mistake to avoid is over-indexing — too many indexes slow down writes significantly."*

📌 **One-liner:** Index = book ka index page | Read ⚡ fast | Write 🐢 slow | Verify with EXPLAIN

---

---

## 7. What is a Primary Key?

### 1️⃣ WHAT

Primary Key (PK) ek **column ya field** hai jo table/collection mein **har record ko uniquely identify** karta hai.

**4 Golden Rules:**

| Rule | Meaning | Example |
|------|---------|---------|
| ✅ **Unique** | Koi duplicate nahi | 2 users ka same ID nahi ho sakta |
| ✅ **Not NULL** | Kabhi empty nahi | ID hamesha honi chahiye |
| ✅ **Immutable** | Change nahi karna | ID badalna = bad practice |
| ✅ **Single per table** | Sirf ek PK | Ek table mein ek hi PK |

### 2️⃣ WHY

- **Identity:** Har record ko uniquely pehchanna
- **Relationships:** Foreign keys PK ko reference karte hain
- **Performance:** PK **automatically indexed** hota hai → instant lookup
- **Integrity:** Duplicate records prevent karta hai

### 3️⃣ HOW — Visual

```
users Table:
┌─────────────┬─────────┬──────────────┬─────┐
│  id (PK) 🔑 │ name    │ email        │ age │
├─────────────┼─────────┼──────────────┼─────┤
│  1          │ Kanishk │ k@test.com   │ 25  │  ← PK = 1 (unique!)
│  2          │ Rahul   │ r@test.com   │ 28  │  ← PK = 2
│  3          │ Priya   │ p@test.com   │ 24  │  ← PK = 3
│  1          │ Amit    │ a@test.com   │ 30  │  ← ❌ DUPLICATE! Not allowed!
└─────────────┴─────────┴──────────────┴─────┘
```

### 4️⃣ CODE

```sql
-- ═══ PostgreSQL ═══

-- Auto-incrementing integer (Most Common)
CREATE TABLE users (
    id SERIAL PRIMARY KEY,    -- 1, 2, 3, 4... auto!
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL
);

INSERT INTO users (name, email) VALUES ('Kanishk', 'k@test.com');
-- id = 1 (auto-generated!)

-- UUID (Better for distributed systems!)
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    -- "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
    name VARCHAR(100) NOT NULL
);
-- Why UUID?
-- ✅ Globally unique (multiple servers pe collision nahi)
-- ✅ Security (guess nahi kar sakte — unlike 1, 2, 3)
-- ❌ Larger storage (16 bytes vs 4 bytes)
-- ❌ Slower indexing

-- Composite Primary Key (Multiple columns)
CREATE TABLE order_items (
    order_id INTEGER,
    product_id INTEGER,
    quantity INTEGER,
    PRIMARY KEY (order_id, product_id)  -- Dono milke unique!
);
```

```javascript
// ═══ MongoDB ═══

// _id = Automatic Primary Key
db.users.insertOne({ name: "Kanishk" });
// Result:
// {
//   "_id": ObjectId("64f8a1b2c3d4e5f6a7b8c9d0"),  ← PK!
//   "name": "Kanishk"
// }

// _id properties:
// ✅ Unique across entire collection
// ✅ Auto-generated if not provided
// ✅ Automatically indexed
// ✅ 12-byte ObjectId (timestamp + machine + process + counter)

// Custom _id (if needed)
db.users.insertOne({
    _id: "user_kanishk_001",  // Custom PK (string)
    name: "Kanishk"
});

// Mongoose
const user = await User.create({ name: "Kanishk" });
console.log(user._id);  // ObjectId("64f8a...")
```

### PK Type Comparison

| Type | Size | Speed | Distributed? | Guessable? | Best For |
|------|------|-------|-------------|-----------|----------|
| **SERIAL (INT)** | 4 bytes | ⚡ Fast | ❌ No | ✅ Yes | Single-server apps |
| **BIGSERIAL** | 8 bytes | ⚡ Fast | ❌ No | ✅ Yes | Large tables |
| **UUID** | 16 bytes | 🐢 Slower | ✅ Yes | ❌ No | Distributed systems |
| **ObjectId** | 12 bytes | ⚡ Fast | ✅ Yes | ❌ No | MongoDB |

### 5️⃣ INTERVIEW ANSWER

> *"A **primary key** is a column or field that **uniquely identifies** each record in a table. It must be **unique, not null, and ideally immutable**. In PostgreSQL, I typically use `SERIAL` for auto-incrementing integers in single-server setups, or `UUID` for distributed systems where global uniqueness is needed. In MongoDB, the `_id` field serves as the primary key and is automatically generated as a 12-byte `ObjectId`. Primary keys are **automatically indexed**, making lookups by PK extremely fast. I avoid using business data like email as PK since it might change — a surrogate key like UUID is safer."*

📌 **One-liner:** Primary Key = unique ID per row | Not NULL | Auto-indexed | SERIAL or UUID

---

---

## 8. What is a Foreign Key?

### 1️⃣ WHAT

Foreign Key (FK) ek field hai jo **doosre table ke Primary Key ko reference** karta hai — dono tables ke beech **relationship** establish karta hai.

> **Simple:** FK = "Ye record us table ke us record se connected hai"

### 2️⃣ WHY

| Benefit | Explanation |
|---------|-------------|
| **Referential Integrity** | Orphan records nahi honge — order bina user ke nahi ban sakta |
| **Relationships** | Tables ko logically connect karna (1:N, M:N) |
| **Data Consistency** | Invalid references prevent karna (user 999 exist nahi karta → order nahi banega) |
| **Cascading** | Parent delete → children auto-delete (optional) |

### 3️⃣ HOW — Visual

```
users (Parent Table)                    orders (Child Table)
┌──────┬─────────┐                      ┌──────┬───────┬───────────────┐
│ id 🔑│ name    │                      │ id 🔑│ total │ user_id (FK) 🔗│
├──────┼─────────┤                      ├──────┼───────┼───────────────┤
│  1   │ Kanishk │◄─────────────────────│  1   │ 5000  │  1            │
│  2   │ Rahul   │◄─────────────────────│  2   │ 3000  │  2            │
│  3   │ Priya   │                      │  3   │ 7000  │  1            │
└──────┴─────────┘                      │  4   │ 2000  │  999          │ ← ❌ INVALID!
  PK: id                                └──────┴───────┴───────────────┘   User 999 nahi hai!
                                          PK: id     FK: user_id → users.id
```

**FK Rules in Action:**
- ✅ `user_id = 1` → Valid (user 1 exists)
- ❌ `user_id = 999` → Rejected! (user 999 doesn't exist)
- ❌ Delete user 1 → Blocked! (orders reference user 1) — unless CASCADE

### 4️⃣ CODE

```sql
-- ═══ PostgreSQL: Foreign Key ═══

-- Parent table
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL
);

-- Child table with FK
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    total DECIMAL(10, 2) NOT NULL,
    status VARCHAR(20) DEFAULT 'pending',
    user_id INTEGER NOT NULL,

    -- Foreign Key Constraint
    CONSTRAINT fk_orders_user
        FOREIGN KEY (user_id)
        REFERENCES users(id)
        ON DELETE CASCADE      -- User delete → orders auto-delete
        ON UPDATE CASCADE      -- User id change → orders auto-update
);

-- Test referential integrity
INSERT INTO users (name, email) VALUES ('Kanishk', 'k@test.com');  -- id = 1

INSERT INTO orders (total, user_id) VALUES (5000, 1);   -- ✅ Valid!
INSERT INTO orders (total, user_id) VALUES (3000, 999); -- ❌ ERROR!
-- "insert or update on table orders violates foreign key constraint"

-- ON DELETE Options:
-- CASCADE:     Child records bhi delete ho jayenge
-- SET NULL:    FK field NULL ho jayega (user_id = NULL)
-- SET DEFAULT: FK field default value le lega
-- RESTRICT:    Delete block hoga (default behavior)
-- NO ACTION:   Same as RESTRICT (checked at end of transaction)

-- ═══ Multiple FKs (M:N Junction Table) ═══
CREATE TABLE student_courses (
    student_id INTEGER REFERENCES students(id) ON DELETE CASCADE,
    course_id INTEGER REFERENCES courses(id) ON DELETE CASCADE,
    enrolled_at TIMESTAMP DEFAULT NOW(),
    PRIMARY KEY (student_id, course_id)  -- Composite PK
);
```

```javascript
// ═══ MongoDB: No Native FK (Application-Level References) ═══

// MongoDB mein FK constraint nahi hota — reference pattern use karte hain
db.orders.insertOne({
    total: 5000,
    userId: ObjectId("64f8a1b2c3d4e5f6"),  // Reference to users._id
    // ⚠️ MongoDB validate nahi karega ki ye user exist karta hai!
    // Application level pe check karna padega
});

// Mongoose handles this with "ref" + "populate"
const orderSchema = new mongoose.Schema({
    total: { type: Number, required: true },
    user: {
        type: mongoose.Schema.Types.ObjectId,
        ref: "User",     // ← Reference to User model
        required: true
    }
});

const Order = mongoose.model("Order", orderSchema);

// Create order (Mongoose doesn't validate FK by default!)
const order = await Order.create({
    total: 5000,
    user: user._id
});

// Read with "JOIN" (populate)
const orders = await Order.find()
    .populate("user", "name email");  // ← User data fetch karo!
// Result: { total: 5000, user: { name: "Kanishk", email: "k@test.com" } }
```

### SQL vs MongoDB FK Comparison

| Feature | PostgreSQL (SQL) | MongoDB (NoSQL) |
|---------|-----------------|-----------------|
| **Native FK** | ✅ Yes (database level) | ❌ No (application level) |
| **Validation** | Automatic | Manual / Mongoose plugin |
| **CASCADE** | Built-in | Manual (middleware/hooks) |
| **Performance** | Optimized JOINs | `$lookup` (slower) |
| **Integrity** | Guaranteed | Developer responsibility |

### 5️⃣ INTERVIEW ANSWER

> *"A **foreign key** is a field in one table that references the **primary key** of another table, establishing a relationship between them. It enforces **referential integrity** — you cannot insert a record with a foreign key value that doesn't exist in the parent table. In PostgreSQL, I define FKs with the `REFERENCES` keyword and configure `ON DELETE` behavior — `CASCADE` to auto-delete children, `SET NULL` to nullify the reference, or `RESTRICT` to block deletion. MongoDB doesn't have native foreign keys, but I use the **reference pattern** with ObjectId fields and Mongoose's `populate()` method to achieve similar relationships at the application level."*

📌 **One-liner:** Foreign Key = doosre table ka PK reference | Referential integrity | CASCADE/RESTRICT

---

---

## ⚡ Lightning Revision Sheet (5 Min Before Interview)

| # | Topic | One-Line Answer |
|---|-------|----------------|
| 1 | **SQL vs NoSQL** | SQL = structured + ACID + JOINs | NoSQL = flexible + scalable + JSON |
| 2 | **MongoDB vs PG** | MongoDB = flexible schema + fast iteration | PG = ACID + JOINs + integrity |
| 3 | **Collection** | MongoDB ka Table | Schema-less | Auto-created on first insert |
| 4 | **Document** | MongoDB ki Row | BSON format | Arrays + Nested | 16MB limit |
| 5 | **CRUD** | C=insert | R=find | U=updateOne ($set/$inc) | D=deleteOne |
| 6 | **Indexes** | B-Tree lookup | Read ⚡ fast | Write 🐢 slow | Verify with EXPLAIN |
| 7 | **Primary Key** | Unique ID per row | Not NULL | Auto-indexed | SERIAL/UUID/ObjectId |
| 8 | **Foreign Key** | Doosre table ka PK ref | Referential integrity | CASCADE/RESTRICT |

---

## 🔗 Database Concept Map

```
Database
├── SQL (PostgreSQL)
│   ├── Tables (Rows + Columns)
│   ├── Fixed Schema
│   ├── Primary Key (SERIAL / UUID)
│   ├── Foreign Key (REFERENCES + CASCADE)
│   ├── JOINs (INNER, LEFT, RIGHT, FULL)
│   └── ACID Transactions
│
├── NoSQL (MongoDB)
│   ├── Collections (Documents)
│   ├── Flexible Schema (BSON)
│   ├── _id (ObjectId = PK)
│   ├── References (populate) / Embedding
│   ├── Aggregation Pipeline
│   └── BASE (Eventual Consistency)
│
└── Shared Concepts
    ├── Indexes (B-Tree → fast lookup)
    ├── CRUD (Create, Read, Update, Delete)
    ├── Relationships (1:1, 1:N, M:N)
    └── Query Optimization (EXPLAIN)
```
