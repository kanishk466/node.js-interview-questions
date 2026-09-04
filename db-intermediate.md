# 🚀 Databases (MongoDB & PostgreSQL) — Interview Prep Guide (Intermediate Level)

> **Level:** Intermediate
> **Format:** What → Why → How → Example → Interview Answer
> **Prerequisite:** Basic Level (SQL vs NoSQL, CRUD, Indexes, Keys)

---

## 📑 Table of Contents

| # | Topic | Difficulty | Frequency |
|---|-------|-----------|-----------|
| 1 | [Database Relationships](#1-database-relationships) | ⭐⭐ | 🔥🔥🔥🔥 |
| 2 | [Database Normalization](#2-database-normalization) | ⭐⭐ | 🔥🔥🔥 |
| 3 | [ACID Properties](#3-acid-properties) | ⭐⭐ | 🔥🔥🔥🔥🔥 |
| 4 | [Transactions](#4-transactions) | ⭐⭐ | 🔥🔥🔥🔥 |
| 5 | [Joins in PostgreSQL](#5-joins-in-postgresql) | ⭐⭐ | 🔥🔥🔥🔥🔥 |
| 6 | [Aggregation in MongoDB](#6-aggregation-in-mongodb) | ⭐⭐ | 🔥🔥🔥🔥 |
| 7 | [Embedding vs Referencing](#7-embedding-vs-referencing-in-mongodb) | ⭐⭐ | 🔥🔥🔥🔥🔥 |
| 8 | [Mongoose Schemas & Models](#8-mongoose-schemas--models) | ⭐⭐ | 🔥🔥🔥🔥 |

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

## 1. Database Relationships

### 1️⃣ WHAT

Database relationships define karti hain ki **ek table ke records doosre table ke records se kaise connected** hain. 3 main types hain:

| Relationship | Symbol | Description | Real-World Example |
|-------------|--------|-------------|-------------------|
| **One-to-One (1:1)** | 1 ↔ 1 | Ek record ka ek hi match | User ↔ Profile |
| **One-to-Many (1:N)** | 1 → N | Ek record ke multiple matches | User → Orders |
| **Many-to-Many (M:N)** | M ↔ N | Dono taraf multiple matches | Students ↔ Courses |

### 2️⃣ WHY

Real-world data **interconnected** hota hai. Bina relationships ke:
- ❌ Data duplicate hoga (har order mein user ka poora naam, email, address)
- ❌ Update anomaly (user ka naam change → 1000 orders update karne padenge)
- ❌ Data integrity nahi hogi

### 3️⃣ HOW — Visual + Implementation

```
═══════════════════════════════════════════
1:1 — One-to-One (User ↔ Profile)
═══════════════════════════════════════════

  users                    profiles
┌────┬─────────┐         ┌────┬───────┬─────────┐
│ id │ name    │         │ id │ bio   │ user_id │ ← UNIQUE FK!
├────┼─────────┤         ├────┼───────┼─────────┤
│ 1  │ Kanishk │◄────────│ 1  │ Dev   │ 1       │
│ 2  │ Rahul   │◄────────│ 2  │ Designer│ 2     │
└────┴─────────┘         └────┴───────┴─────────┘

Key: user_id UNIQUE hai → ek user = ek profile


═══════════════════════════════════════════
1:N — One-to-Many (User → Orders)
═══════════════════════════════════════════

  users                    orders
┌────┬─────────┐         ┌────┬───────┬─────────┐
│ id │ name    │         │ id │ total │ user_id │ ← FK (not unique!)
├────┼─────────┤         ├────┼───────┼─────────┤
│ 1  │ Kanishk │◄───┬────│ 1  │ 5000  │ 1       │
│    │         │◄───┤────│ 2  │ 3000  │ 1       │ ← Same user, multiple orders
│ 2  │ Rahul   │◄───┘────│ 3  │ 7000  │ 2       │
└────┴─────────┘         └────┴───────┴─────────┘

Key: user_id repeat ho sakta hai → ek user = many orders


═══════════════════════════════════════════
M:N — Many-to-Many (Students ↔ Courses)
═══════════════════════════════════════════

  students                 student_courses (Junction!)       courses
┌────┬─────────┐         ┌────────────┬───────────┐        ┌────┬───────┐
│ id │ name    │         │ student_id │ course_id │        │ id │ title │
├────┼─────────┤         ├────────────┼───────────┤        ├────┼───────┤
│ 1  │ Kanishk │◄────────│ 1          │ 1         │───────►│ 1  │ Math  │
│ 2  │ Rahul   │◄────────│ 1          │ 2         │───────►│ 2  │ CS    │
└────┴─────────┘         │ 2          │ 1         │───────►│    │       │
                         │ 2          │ 3         │───────►│ 3  │ Eng   │
                         └────────────┴───────────┘        └────┴───────┘

Key: Junction table dono PK ko FK ke roop mein rakhta hai
     Kanishk → Math + CS
     Rahul → Math + Eng
     Math → Kanishk + Rahul
```

### 4️⃣ CODE

```sql
-- ═══ 1:1 — User ↔ Profile ═══
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL
);

CREATE TABLE profiles (
    id SERIAL PRIMARY KEY,
    bio TEXT,
    avatar_url VARCHAR(500),
    user_id INTEGER UNIQUE NOT NULL REFERENCES users(id)
    --         ^^^^^^ UNIQUE = 1:1 guarantee!
);

-- ═══ 1:N — User → Orders ═══
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    total DECIMAL(10, 2) NOT NULL,
    status VARCHAR(20) DEFAULT 'pending',
    user_id INTEGER NOT NULL REFERENCES users(id)
    -- FK without UNIQUE = 1:N (same user_id can repeat)
);

-- Query: User ke saare orders
SELECT u.name, o.total, o.status
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE u.id = 1;

-- ═══ M:N — Students ↔ Courses (Junction Table!) ═══
CREATE TABLE students (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

CREATE TABLE courses (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    credits INTEGER DEFAULT 3
);

-- Junction / Pivot / Bridge Table
CREATE TABLE student_courses (
    student_id INTEGER REFERENCES students(id) ON DELETE CASCADE,
    course_id INTEGER REFERENCES courses(id) ON DELETE CASCADE,
    enrolled_at TIMESTAMP DEFAULT NOW(),
    grade VARCHAR(5),
    PRIMARY KEY (student_id, course_id)  -- Composite PK = no duplicate enrollment
);

-- Query: Student ke saare courses
SELECT s.name, c.title, sc.grade
FROM students s
JOIN student_courses sc ON s.id = sc.student_id
JOIN courses c ON sc.course_id = c.id
WHERE s.name = 'Kanishk';

-- Query: Course ke saare students
SELECT c.title, s.name
FROM courses c
JOIN student_courses sc ON c.id = sc.course_id
JOIN students s ON sc.student_id = s.id
WHERE c.title = 'Math';
```

```javascript
// ═══ MongoDB: Relationships ═══

// 1:1 — Embedded (most common for 1:1)
db.users.insertOne({
    name: "Kanishk",
    profile: { bio: "Full-stack dev", avatar: "kan.jpg" }  // Embedded!
});

// 1:N — Referenced
db.users.insertOne({
    _id: ObjectId("user1"),
    name: "Kanishk",
    orders: [ObjectId("order1"), ObjectId("order2")]  // Array of refs
});

// M:N — Referenced both sides
db.students.insertOne({
    name: "Kanishk",
    courses: [ObjectId("course1"), ObjectId("course2")]
});
db.courses.insertOne({
    title: "Math",
    students: [ObjectId("user1"), ObjectId("user2")]
});
```

### 5️⃣ INTERVIEW ANSWER

> *"There are three types of database relationships. **One-to-One** (e.g., User-Profile) is enforced with a `UNIQUE` foreign key — each record maps to exactly one record in the other table. **One-to-Many** (e.g., User-Orders) is the most common — the child table has a foreign key referencing the parent, and multiple children can share the same parent ID. **Many-to-Many** (e.g., Students-Courses) requires a **junction table** with foreign keys to both parent tables and a composite primary key to prevent duplicates. In MongoDB, 1:1 and 1:N are often handled via **embedding**, while M:N uses **arrays of ObjectIds** with `populate()`."*

📌 **One-liner:** 1:1 = UNIQUE FK | 1:N = FK in child (repeatable) | M:N = junction table with composite PK

---

---

## 2. Database Normalization

### 1️⃣ WHAT

Normalization ek **systematic process** hai jismein data ko **multiple tables mein organize** kiya jata hai taaki:
- **Redundancy kam** ho (same data baar-baar store na ho)
- **Data integrity** maintain rahe (update/delete anomalies na aayein)

> **Simple:** "Ek fact, ek jagah" — har piece of information sirf ek jagah store ho.

### 2️⃣ WHY — Problems Without Normalization

```
Unnormalized Table (orders):
┌────┬─────────┬──────────────┬──────────┬───────┬──────────────┐
│ id │ customer│ customer_email│ product  │ price │ product_cat  │
├────┼─────────┼──────────────┼──────────┼───────┼──────────────┤
│ 1  │ Kanishk │ k@test.com   │ Laptop   │ 50000 │ Electronics  │
│ 2  │ Kanishk │ k@test.com   │ Mouse    │ 500   │ Electronics  │ ← Email repeat!
│ 3  │ Rahul   │ r@test.com   │ Laptop   │ 50000 │ Electronics  │ ← Product repeat!
└────┴─────────┴──────────────┴──────────┴───────┴──────────────┘

Problems:
❌ Update Anomaly: Kanishk ka email change → 2 rows update karne padenge
❌ Delete Anomaly: Order 3 delete → "Laptop" product info bhi gayab!
❌ Insert Anomaly: Naya product add bina order ke? Nahi kar sakte!
❌ Storage Waste: "k@test.com" 100 orders = 100 baar stored
```

### 3️⃣ HOW — Normal Forms (Step by Step)

```
═══════════════════════════════════════════
UNF → 1NF: Atomic Values (No multi-valued cells)
═══════════════════════════════════════════

❌ UNF (Unnormalized):
┌────┬─────────┬──────────────────────┐
│ id │ student │ courses              │
├────┼─────────┼──────────────────────┤
│ 1  │ Kanishk │ Math, Science, Eng   │ ← Multiple values in one cell!
└────┴─────────┴──────────────────────┘

✅ 1NF (First Normal Form):
┌────┬─────────┬─────────┐
│ id │ student │ course  │  ← One value per cell
├────┼─────────┼─────────┤
│ 1  │ Kanishk │ Math    │
│ 2  │ Kanishk │ Science │
│ 3  │ Kanishk │ Eng     │
└────┴─────────┴─────────┘
Rule: Har cell mein EK value. No arrays, no comma-separated lists.


═══════════════════════════════════════════
1NF → 2NF: No Partial Dependencies
═══════════════════════════════════════════

❌ 1NF (Partial dependency on composite key):
┌────────────┬───────────┬──────────┬───────────────┐
│ student_id │ course_id │ student  │ course_credits│
├────────────┼───────────┼──────────┼───────────────┤
│ 1          │ 1         │ Kanishk  │ 4             │
│ 1          │ 2         │ Kanishk  │ 3             │
└────────────┴───────────┴──────────┴───────────────┘
  PK = (student_id, course_id)
  Problem: "student" sirf student_id pe depend karta hai (partial!)

✅ 2NF (Second Normal Form):
Students:  ┌────┬─────────┐
           │ id │ name    │  ← student info alag
           └────┴─────────┘
Courses:   ┌────┬─────────┐
           │ id │ credits │  ← course info alag
           └────┴─────────┘
Enrollments: ┌────────────┬───────────┐
             │ student_id │ course_id │  ← sirf relationship
             └────────────┴───────────┘
Rule: Har non-key column POORE primary key pe depend kare.


═══════════════════════════════════════════
2NF → 3NF: No Transitive Dependencies
═══════════════════════════════════════════

❌ 2NF (Transitive dependency):
┌────┬─────────┬──────────┬──────────────┐
│ id │ student │ dept_id  │ dept_name    │
├────┼─────────┼──────────┼──────────────┤
│ 1  │ Kanishk │ 1        │ CS           │ ← dept_name depends on dept_id,
│ 2  │ Rahul   │ 1        │ CS           │   NOT directly on id!
└────┴─────────┴──────────┴──────────────┘
  id → dept_id → dept_name (transitive chain!)

✅ 3NF (Third Normal Form):
Students:    ┌────┬─────────┬─────────┐
             │ id │ name    │ dept_id │  ← FK to departments
             └────┴─────────┴─────────┘
Departments: ┌────┬──────────┐
             │ id │ name     │  ← dept info alag
             └────┴──────────┘
Rule: Non-key column doosre non-key column pe depend nahi karna chahiye.
```

**Yaad Rakhne Ka Trick:**

| NF | Rule | Catchphrase |
|----|------|-------------|
| **1NF** | Atomic values | "Ek cell, ek value" |
| **2NF** | No partial dependency | "Poore key pe depend karo" |
| **3NF** | No transitive dependency | "Key pe depend karo, non-key pe nahi" |

### 4️⃣ CODE

```sql
-- ═══ BEFORE: Unnormalized (Bad!) ═══
CREATE TABLE orders_bad (
    order_id INT,
    customer_name VARCHAR(100),    -- Repeated for every order!
    customer_email VARCHAR(255),   -- Repeated!
    customer_phone VARCHAR(15),    -- Repeated!
    product_name VARCHAR(100),     -- Repeated!
    product_price DECIMAL,         -- Repeated!
    quantity INT
);

-- ═══ AFTER: 3NF Normalized (Good!) ═══
CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    phone VARCHAR(15)
);

CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    price DECIMAL(10, 2) NOT NULL
);

CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_id INT REFERENCES customers(id),
    order_date TIMESTAMP DEFAULT NOW()
);

CREATE TABLE order_items (
    id SERIAL PRIMARY KEY,
    order_id INT REFERENCES orders(id),
    product_id INT REFERENCES products(id),
    quantity INT NOT NULL,
    unit_price DECIMAL(10, 2) NOT NULL  -- Price at time of order
);

-- Query (JOINs bring data back together)
SELECT c.name, c.email, p.name AS product, oi.quantity, oi.unit_price
FROM customers c
JOIN orders o ON c.id = o.customer_id
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id
WHERE c.name = 'Kanishk';
```

### When to Denormalize?

| Scenario | Normalize (3NF) | Denormalize |
|----------|----------------|-------------|
| OLTP (Transactions) | ✅ | ❌ |
| OLAP (Analytics/Reporting) | ❌ | ✅ |
| Read-heavy dashboards | ❌ | ✅ |
| Data integrity critical | ✅ | ❌ |
| MongoDB (document model) | Partial | ✅ (Embedding) |

### 5️⃣ INTERVIEW ANSWER

> *"Normalization is the process of organizing data to **reduce redundancy** and prevent **update, insert, and delete anomalies**. The three main normal forms are: **1NF** ensures atomic values — no multi-valued cells. **2NF** eliminates partial dependencies — every non-key column must depend on the **entire** primary key, not just part of it. **3NF** eliminates transitive dependencies — non-key columns shouldn't depend on other non-key columns. In practice, I normalize to **3NF** for OLTP systems. For read-heavy analytics or MongoDB, I sometimes **denormalize** intentionally by embedding related data for query performance."*

📌 **One-liner:** 1NF = atomic | 2NF = no partial dep | 3NF = no transitive dep | Goal = one fact, one place

---

---

## 3. ACID Properties

> 🏆 **Most asked database theory question — guaranteed in almost every interview!**

### 1️⃣ WHAT

ACID **4 properties** ka acronym hai jo **database transactions ki reliability** guarantee karti hain:

| Letter | Property | One-Liner | Analogy |
|--------|----------|-----------|---------|
| **A** | **Atomicity** | "All or Nothing" | Ya poora kaam ho, ya bilkul nahi |
| **C** | **Consistency** | "Valid → Valid" | Rules kabhi break nahi hote |
| **I** | **Isolation** | "No interference" | Ek kaam doosre ko disturb nahi karta |
| **D** | **Durability** | "Permanent save" | Save ho gaya toh power cut mein bhi safe |

### 2️⃣ WHY — The Banking Example

```
Scenario: Kanishk ₹5000 transfer karta hai Rahul ko

Step 1: Kanishk.balance -= 5000   (₹10000 → ₹5000)
Step 2: Rahul.balance   += 5000   (₹2000 → ₹7000)

═══════════════════════════════════════════
❌ WITHOUT ACID (Nightmare!):
═══════════════════════════════════════════
Step 1 done (Kanishk: ₹5000) → 💥 CRASH!
Step 2 never happened (Rahul: ₹2000)
Result: ₹5000 GAYAB! Kanishk ka paisa gaya, Rahul ko mila nahi! 💀

═══════════════════════════════════════════
✅ WITH ACID (Safe!):
═══════════════════════════════════════════
Atomicity: Step 1 + Step 2 = ONE unit. Crash → ROLLBACK → Kanishk: ₹10000 ✅
Consistency: Total money before = ₹12000, after = ₹12000 ✅ (rules maintained)
Isolation: Beech mein koi aur transaction balance nahi padh sakta ✅
Durability: COMMIT ke baad power cut → data safe (WAL se recover) ✅
```

### 3️⃣ HOW — Each Property Deep Dive

```
═══════════════════════════════════════════
A — ATOMICITY ("All or Nothing")
═══════════════════════════════════════════
Transaction ke saare operations ek UNIT hain:
  → Sab succeed → COMMIT ✅
  → Ek bhi fail → ROLLBACK ❌ (poora undo)

Implementation: Undo Log (rollback segment)
  → Har operation ka "reverse" record rakha jata hai
  → Failure pe reverse operations execute hote hain


═══════════════════════════════════════════
C — CONSISTENCY ("Valid State → Valid State")
═══════════════════════════════════════════
Transaction database ko ek VALID state se doosre VALID state mein le jaata hai.
Constraints kabhi violate nahi hote:
  → CHECK constraints (balance >= 0)
  → UNIQUE constraints (email unique)
  → FOREIGN KEY constraints (user must exist)
  → NOT NULL constraints

Example:
  balance CHECK (balance >= 0)
  Transaction: balance = -500 → ❌ REJECTED (constraint violation)


═══════════════════════════════════════════
I — ISOLATION ("No Interference")
═══════════════════════════════════════════
Concurrent transactions ek doosre ko affect nahi karti.

Problem without isolation:
  T1: Read balance = 10000
  T2: Update balance = 5000 (not committed yet)
  T1: Read balance = 5000 ← DIRTY READ! 💀 (T2 rollback ho sakta hai!)

Isolation Levels (Low → High):
  1. Read Uncommitted  → Dirty reads possible ⚠️
  2. Read Committed    → No dirty reads ✅ (PostgreSQL default)
  3. Repeatable Read   → No dirty + no non-repeatable reads ✅ (MySQL default)
  4. Serializable      → Full isolation (slowest) ✅✅


═══════════════════════════════════════════
D — DURABILITY ("Permanent Save")
═══════════════════════════════════════════
Ek baar COMMIT ho gaya → data PERMANENTLY save hai.
  → Power failure, crash, restart → data safe!

Implementation: WAL (Write-Ahead Logging)
  → COMMIT se pehle changes disk pe log hote hain
  → Crash ke baad WAL se recover hota hai
  → "Commit = Disk write" guarantee
```

### 4️⃣ CODE

```sql
-- ═══ PostgreSQL: ACID in Action ═══

-- Atomicity + Consistency
BEGIN;
    UPDATE accounts SET balance = balance - 5000 WHERE id = 1;
    -- Check: balance >= 0? (Consistency constraint)
    UPDATE accounts SET balance = balance + 5000 WHERE id = 2;
COMMIT;  -- Atomic: both succeed or both rollback

-- Isolation Level Setting
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
    SELECT balance FROM accounts WHERE id = 1;  -- Consistent snapshot
    UPDATE accounts SET balance = balance - 1000 WHERE id = 1;
COMMIT;

-- Durability: After COMMIT, data is on disk (WAL)
-- Even if server crashes NOW, data is safe ✅
```

```javascript
// ═══ MongoDB: ACID Transactions (v4.0+, requires Replica Set) ═══
const session = await mongoose.startSession();
session.startTransaction();

try {
    // Atomicity: Both succeed or both rollback
    await Account.updateOne(
        { _id: kanishkId },
        { $inc: { balance: -5000 } },
        { session }
    );
    await Account.updateOne(
        { _id: rahulId },
        { $inc: { balance: 5000 } },
        { session }
    );

    await session.commitTransaction();  // Durability: now permanent
    console.log("Transfer successful!");
} catch (error) {
    await session.abortTransaction();  // Atomicity: rollback!
    console.error("Transfer failed:", error);
} finally {
    session.endSession();
}
```

### Isolation Levels Quick Reference

| Level | Dirty Read | Non-Repeatable Read | Phantom Read | Speed |
|-------|-----------|--------------------|-------------|-------|
| Read Uncommitted | ✅ Possible | ✅ Possible | ✅ Possible | ⚡ Fastest |
| Read Committed | ❌ Prevented | ✅ Possible | ✅ Possible | ⚡ Fast |
| Repeatable Read | ❌ Prevented | ❌ Prevented | ✅ Possible | 🐢 Medium |
| Serializable | ❌ Prevented | ❌ Prevented | ❌ Prevented | 🐢🐢 Slowest |

### 5️⃣ INTERVIEW ANSWER

> *"ACID stands for **Atomicity, Consistency, Isolation, and Durability** — the four guarantees that make database transactions reliable. **Atomicity** ensures all-or-nothing execution — if any operation fails, the entire transaction rolls back. **Consistency** ensures data moves from one valid state to another, respecting all constraints like CHECK, UNIQUE, and FOREIGN KEY. **Isolation** ensures concurrent transactions don't interfere — controlled via isolation levels from Read Committed to Serializable. **Durability** ensures committed data survives system crashes through Write-Ahead Logging. PostgreSQL provides full ACID compliance natively. MongoDB supports multi-document ACID transactions since version 4.0 with replica sets."*

📌 **One-liner:** A=All or Nothing | C=Valid→Valid | I=No interference | D=Survives crash (WAL)

---

---

## 4. Transactions

### 1️⃣ WHAT

Transaction **ek ya multiple database operations ka group** hai jo **ek single logical unit** ki tarah execute hota hai — ya toh **poora succeed** (COMMIT) hoga ya **poora undo** (ROLLBACK) hoga.

> **Simple:** Transaction = ACID ka practical implementation

### 2️⃣ WHY

Jab **multiple related operations** ko atomically perform karna ho:

| Scenario | Operations | Why Transaction? |
|----------|-----------|-----------------|
| **Fund Transfer** | Debit A + Credit B | Ek fail → dono rollback |
| **Order Placement** | Create order + Reduce stock + Charge payment | 3 steps, all or nothing |
| **User Registration** | Create user + Create profile + Send email token | Consistent state |
| **Inventory Update** | Reduce stock + Create shipment + Update ledger | No partial updates |

### 3️⃣ HOW — Transaction Lifecycle

```
                  ┌───────────┐
                  │   BEGIN   │  ← Transaction starts
                  └─────┬─────┘
                        │
                  ┌─────▼─────┐
                  │ Operation 1│  ← e.g., Debit account A
                  └─────┬─────┘
                        │
                  ┌─────▼─────┐
                  │ Operation 2│  ← e.g., Credit account B
                  └─────┬─────┘
                        │
                   Success?
                  /        \
               YES           NO
                │             │
          ┌─────▼─────┐ ┌────▼──────┐
          │  COMMIT   │ │ ROLLBACK  │
          │ (Save ✅) │ │ (Undo ❌) │
          └───────────┘ └───────────┘

Savepoints (Partial Rollback):
BEGIN → Op1 → SAVEPOINT sp1 → Op2 → SAVEPOINT sp2 → Op3 (FAIL!)
                                                    → ROLLBACK TO sp2
                                                    → Op3 retry → COMMIT
```

### 4️⃣ CODE

```sql
-- ═══ PostgreSQL: Basic Transaction ═══
BEGIN;  -- ya START TRANSACTION;

    INSERT INTO orders (user_id, total, status)
    VALUES (1, 5000, 'confirmed');

    UPDATE products SET stock = stock - 1
    WHERE id = 101 AND stock > 0;  -- Stock check!

    UPDATE users SET total_spent = total_spent + 5000
    WHERE id = 1;

COMMIT;  -- Sab successful → save!
-- Ya ROLLBACK; agar koi error aaye

-- ═══ PostgreSQL: Savepoints (Partial Rollback) ═══
BEGIN;
    INSERT INTO orders (user_id, total) VALUES (1, 5000);
    SAVEPOINT order_created;

    UPDATE inventory SET stock = stock - 1 WHERE product_id = 101;
    -- Oops! Stock 0 hai → error!

    ROLLBACK TO order_created;  -- Sirf inventory undo, order rakho
    -- Try alternative product...
    UPDATE inventory SET stock = stock - 1 WHERE product_id = 102;

COMMIT;

-- ═══ PostgreSQL: Transaction with Error Handling (PL/pgSQL) ═══
DO $$
BEGIN
    UPDATE accounts SET balance = balance - 5000 WHERE id = 1;

    -- Check if balance went negative
    IF (SELECT balance FROM accounts WHERE id = 1) < 0 THEN
        RAISE EXCEPTION 'Insufficient funds!';
    END IF;

    UPDATE accounts SET balance = balance + 5000 WHERE id = 2;

    EXCEPTION
        WHEN OTHERS THEN
            RAISE NOTICE 'Transaction failed: %', SQLERRM;
            -- Auto rollback on exception
END $$;
```

```javascript
// ═══ MongoDB (Mongoose): Transaction ═══
async function placeOrder(userId, productId, amount) {
    const session = await mongoose.startSession();
    session.startTransaction();

    try {
        // Step 1: Create order
        const order = await Order.create(
            [{ userId, productId, amount, status: "confirmed" }],
            { session }
        );

        // Step 2: Reduce stock
        const product = await Product.findOneAndUpdate(
            { _id: productId, stock: { $gt: 0 } },
            { $inc: { stock: -1 } },
            { session, new: true }
        );

        if (!product) {
            throw new Error("Out of stock!");  // → ROLLBACK
        }

        // Step 3: Update user's total spent
        await User.findByIdAndUpdate(
            userId,
            { $inc: { totalSpent: amount } },
            { session }
        );

        // All good → COMMIT
        await session.commitTransaction();
        return { success: true, orderId: order[0]._id };

    } catch (error) {
        // Any error → ROLLBACK
        await session.abortTransaction();
        throw error;

    } finally {
        // ALWAYS end session
        session.endSession();
    }
}

// Usage
try {
    const result = await placeOrder(userId, productId, 5000);
    console.log("Order placed:", result.orderId);
} catch (err) {
    console.error("Order failed:", err.message);
}

// ═══ PostgreSQL (Node.js with pg): Transaction ═══
const { Pool } = require("pg");
const pool = new Pool();

async function transferMoney(fromId, toId, amount) {
    const client = await pool.connect();

    try {
        await client.query("BEGIN");

        await client.query(
            "UPDATE accounts SET balance = balance - $1 WHERE id = $2",
            [amount, fromId]
        );

        await client.query(
            "UPDATE accounts SET balance = balance + $1 WHERE id = $2",
            [amount, toId]
        );

        await client.query("COMMIT");
        return { success: true };

    } catch (err) {
        await client.query("ROLLBACK");
        throw err;

    } finally {
        client.release();  // Return to pool!
    }
}
```

### 5️⃣ INTERVIEW ANSWER

> *"A transaction groups multiple database operations into a **single atomic unit** — either all operations succeed with `COMMIT` or all are undone with `ROLLBACK`. This is critical for operations like fund transfers or order processing where partial completion would corrupt data. In PostgreSQL, I use `BEGIN/COMMIT/ROLLBACK` and can use **savepoints** for partial rollbacks within a transaction. In MongoDB, multi-document transactions require **replica sets** and use `session.startTransaction()` with `commitTransaction()` and `abortTransaction()`. I always wrap transactions in try/catch/finally to ensure sessions are properly closed and connections returned to the pool."*

📌 **One-liner:** Transaction = BEGIN → operations → COMMIT/ROLLBACK | All or nothing | try/catch/finally

---

---

## 5. Joins in PostgreSQL

> 🏆 **Most asked SQL question — coding round mein bhi aata hai!**

### 1️⃣ WHAT

JOINs **multiple tables ko combine** karte hain based on a related column — ek single result set produce karte hain.

### 2️⃣ WHY

Normalized databases mein data **multiple tables** mein hota hai. JOINs usse **wapas ek saath** laate hain for querying.

### 3️⃣ HOW — 6 Types of JOINs

```
Table A (Users)          Table B (Orders)
┌────┬─────────┐        ┌────┬───────┬─────────┐
│ id │ name    │        │ id │ total │ user_id │
├────┼─────────┤        ├────┼───────┼─────────┤
│ 1  │ Kanishk │        │ 1  │ 5000  │ 1       │
│ 2  │ Rahul   │        │ 2  │ 3000  │ 1       │
│ 3  │ Priya   │        │ 3  │ 7000  │ 4       │ ← user 4 doesn't exist!
└────┴─────────┘        └────┴───────┴─────────┘

═══════════════════════════════════════════
INNER JOIN (Matching Only — Most Common)
═══════════════════════════════════════════
Result: Sirf woh rows jo DONO tables mein match karti hain
┌─────────┬───────┐
│ Kanishk │ 5000  │  ← user 1 matches
│ Kanishk │ 3000  │  ← user 1 matches
└─────────┴───────┘
Priya ❌ (no orders) | Order 3 ❌ (no user 4)

═══════════════════════════════════════════
LEFT JOIN (All Left + Matching Right)
═══════════════════════════════════════════
Result: SAARE left table rows + matching right (NULL if no match)
┌─────────┬───────┐
│ Kanishk │ 5000  │
│ Kanishk │ 3000  │
│ Rahul   │ NULL  │  ← No orders, but still included!
│ Priya   │ NULL  │  ← No orders, but still included!
└─────────┴───────┘

═══════════════════════════════════════════
RIGHT JOIN (All Right + Matching Left)
═══════════════════════════════════════════
Result: SAARE right table rows + matching left
┌─────────┬───────┐
│ Kanishk │ 5000  │
│ Kanishk │ 3000  │
│ NULL    │ 7000  │  ← No user 4, but order included!
└─────────┴───────┘

═══════════════════════════════════════════
FULL OUTER JOIN (All From Both)
═══════════════════════════════════════════
┌─────────┬───────┐
│ Kanishk │ 5000  │
│ Kanishk │ 3000  │
│ Rahul   │ NULL  │
│ Priya   │ NULL  │
│ NULL    │ 7000  │
└─────────┴───────┘

═══════════════════════════════════════════
CROSS JOIN (Cartesian Product)
═══════════════════════════════════════════
Har row × Har row = 3 × 3 = 9 rows!
⚠️ Rarely used — performance killer on large tables

═══════════════════════════════════════════
SELF JOIN (Table joins itself)
═══════════════════════════════════════════
employees: id, name, manager_id
Result: Employee name + Manager name (same table!)
```

### 4️⃣ CODE

```sql
-- Setup
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(255)
);

CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INT REFERENCES users(id),
    total DECIMAL(10, 2),
    status VARCHAR(20)
);

CREATE TABLE order_items (
    id SERIAL PRIMARY KEY,
    order_id INT REFERENCES orders(id),
    product_name VARCHAR(100),
    quantity INT,
    price DECIMAL(10, 2)
);

-- ═══ INNER JOIN ═══
-- Users with their orders (sirf jinke orders hain)
SELECT u.name, u.email, o.total, o.status
FROM users u
INNER JOIN orders o ON u.id = o.user_id;

-- ═══ LEFT JOIN ═══
-- ALL users + their orders (including users with no orders)
SELECT u.name, COALESCE(o.total, 0) AS total
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;
-- COALESCE: NULL → 0

-- Find users with NO orders (LEFT JOIN trick!)
SELECT u.name
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.id IS NULL;  -- ← No matching order!

-- ═══ RIGHT JOIN ═══
-- ALL orders + user info (including orphan orders)
SELECT u.name, o.total
FROM users u
RIGHT JOIN orders o ON u.id = o.user_id;

-- ═══ FULL OUTER JOIN ═══
SELECT u.name, o.total
FROM users u
FULL OUTER JOIN orders o ON u.id = o.user_id;

-- ═══ MULTIPLE JOINs (3+ tables!) ═══
SELECT
    u.name AS customer,
    o.id AS order_id,
    oi.product_name,
    oi.quantity,
    oi.price,
    (oi.quantity * oi.price) AS line_total
FROM users u
JOIN orders o ON u.id = o.user_id
JOIN order_items oi ON o.id = oi.order_id
WHERE o.status = 'completed'
ORDER BY o.id, oi.id;

-- ═══ SELF JOIN ═══
-- Employee + Manager (same table!)
SELECT
    e.name AS employee,
    m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
-- Result:
-- Kanishk | Rahul    (Rahul is Kanishk's manager)
-- Rahul   | Priya    (Priya is Rahul's manager)
-- Priya   | NULL     (Priya has no manager — CEO!)

-- ═══ JOIN with Aggregation ═══
-- Total spending per user
SELECT
    u.name,
    COUNT(o.id) AS order_count,
    SUM(o.total) AS total_spent,
    AVG(o.total) AS avg_order
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.name
ORDER BY total_spent DESC NULLS LAST;

-- ═══ CROSS JOIN (Use carefully!) ═══
-- All possible user-product combinations
SELECT u.name, p.name AS product
FROM users u
CROSS JOIN products p;
-- ⚠️ 1000 users × 500 products = 500,000 rows!
```

### JOIN Performance Tips

| Tip | Why |
|-----|-----|
| **Index JOIN columns** | `ON u.id = o.user_id` → index on `user_id`! |
| **Filter early** | `WHERE` before `JOIN` reduces rows |
| **SELECT specific columns** | `SELECT u.name` not `SELECT *` |
| **Use EXPLAIN ANALYZE** | Check if Nested Loop / Hash / Merge Join |
| **Avoid CROSS JOIN** | Cartesian product = O(n×m) rows |

### 5️⃣ INTERVIEW ANSWER

> *"PostgreSQL supports several JOIN types. **INNER JOIN** returns only matching rows from both tables — it's the most common. **LEFT JOIN** returns all rows from the left table with matching right rows, filling NULLs where there's no match — useful for finding records without related data. **RIGHT JOIN** is the reverse, and **FULL OUTER JOIN** returns all rows from both tables. **SELF JOIN** joins a table to itself — useful for hierarchical data like employee-manager relationships. **CROSS JOIN** produces a Cartesian product and should be used cautiously. For performance, I always ensure JOIN columns are **indexed** and use `EXPLAIN ANALYZE` to verify the query plan uses Hash or Merge joins instead of slow Nested Loops on large datasets."*

📌 **One-liner:** INNER = match only | LEFT = all left + match | RIGHT = all right | FULL = all both | SELF = hierarchy

---

---

## 6. Aggregation in MongoDB

### 1️⃣ WHAT

Aggregation MongoDB ka **data processing pipeline** hai — documents ko **filter, transform, group, sort, aur calculate** karta hai step by step. SQL ke `GROUP BY`, `SUM`, `AVG`, `JOIN` ka equivalent.

> **Analogy:** Factory assembly line 🏭
> Raw materials → Cut → Weld → Paint → Package → Final product
> Documents → $match → $group → $sort → $project → Result

### 2️⃣ WHY

Raw data se **meaningful insights** nikalne ke liye:
- Dashboard analytics (monthly revenue, top users)
- Reports (sales by category, avg order value)
- Data transformation (reshaping documents)
- Joins across collections (`$lookup`)

### 3️⃣ HOW — Pipeline Concept

```
Input: 10,000 order documents
         │
         ▼
┌─────────────────┐
│ Stage 1: $match │  → Filter: status = "completed" (10,000 → 7,000)
└────────┬────────┘
         ▼
┌─────────────────┐
│ Stage 2: $group │  → Group by userId, SUM amount (7,000 → 500 users)
└────────┬────────┘
         ▼
┌─────────────────┐
│ Stage 3: $sort  │  → Sort by totalSpent descending
└────────┬────────┘
         ▼
┌─────────────────┐
│ Stage 4: $limit │  → Top 10 spenders
└────────┬────────┘
         ▼
Output: 10 documents (top spenders with totals)
```

**Key Rule:** Pipeline stages execute **sequentially** — output of one = input of next.

### 4️⃣ CODE

```javascript
// Sample Data:
// orders: [
//   { userId: 1, product: "Laptop", amount: 50000, status: "completed", date: ISODate },
//   { userId: 1, product: "Mouse", amount: 500, status: "completed", date: ISODate },
//   { userId: 2, product: "Phone", amount: 25000, status: "cancelled", date: ISODate },
//   ...
// ]

// ═══ 1. BASIC: Total Revenue ═══
db.orders.aggregate([
    { $match: { status: "completed" } },
    { $group: {
        _id: null,                              // null = group ALL
        totalRevenue: { $sum: "$amount" },
        avgOrder: { $avg: "$amount" },
        maxOrder: { $max: "$amount" },
        minOrder: { $min: "$amount" },
        orderCount: { $sum: 1 }
    }}
]);
// { totalRevenue: 75500, avgOrder: 37750, orderCount: 2 }

// ═══ 2. GROUP BY FIELD: Revenue per User ═══
db.orders.aggregate([
    { $match: { status: "completed" } },
    { $group: {
        _id: "$userId",                         // Group by userId
        totalSpent: { $sum: "$amount" },
        orderCount: { $sum: 1 },
        products: { $push: "$product" }          // Array of products
    }},
    { $sort: { totalSpent: -1 } },              // Highest first
    { $limit: 5 }                                // Top 5
]);
// { _id: 1, totalSpent: 50500, orderCount: 2, products: ["Laptop", "Mouse"] }

// ═══ 3. DATE GROUPING: Monthly Revenue ═══
db.orders.aggregate([
    { $match: { status: "completed" } },
    { $group: {
        _id: {
            year: { $year: "$date" },
            month: { $month: "$date" }
        },
        revenue: { $sum: "$amount" },
        orders: { $sum: 1 }
    }},
    { $sort: { "_id.year": 1, "_id.month": 1 } }
]);
// { _id: { year: 2024, month: 1 }, revenue: 75500, orders: 2 }

// ═══ 4. $lookup (MongoDB JOIN!) ═══
db.orders.aggregate([
    { $match: { status: "completed" } },
    { $lookup: {
        from: "users",                           // Join with users collection
        localField: "userId",                    // Field in orders
        foreignField: "_id",                     // Field in users
        as: "userDetails"                        // Output array
    }},
    { $unwind: "$userDetails" },                 // Flatten array to object
    { $project: {
        "userDetails.name": 1,
        "userDetails.email": 1,
        product: 1,
        amount: 1,
        _id: 0
    }}
]);
// { userDetails: { name: "Kanishk", email: "k@test.com" }, product: "Laptop", amount: 50000 }

// ═══ 5. CONDITIONAL AGGREGATION ═══
db.orders.aggregate([
    { $group: {
        _id: "$status",
        count: { $sum: 1 },
        totalAmount: { $sum: "$amount" },
        avgAmount: { $avg: "$amount" }
    }}
]);
// { _id: "completed", count: 2, totalAmount: 50500 }
// { _id: "cancelled", count: 1, totalAmount: 25000 }

// ═══ 6. COMPLEX: Revenue by Category with Ranking ═══
db.orders.aggregate([
    { $match: { status: "completed", date: { $gte: ISODate("2024-01-01") } } },
    { $lookup: {
        from: "products",
        localField: "productId",
        foreignField: "_id",
        as: "product"
    }},
    { $unwind: "$product" },
    { $group: {
        _id: "$product.category",
        revenue: { $sum: "$amount" },
        orderCount: { $sum: 1 },
        avgOrderValue: { $avg: "$amount" }
    }},
    { $sort: { revenue: -1 } },
    { $limit: 10 },
    { $project: {
        category: "$_id",
        revenue: { $round: ["$revenue", 2] },
        orderCount: 1,
        avgOrderValue: { $round: ["$avgOrderValue", 2] },
        _id: 0
    }}
]);
```

### Aggregation Operators Quick Reference

| Stage | Purpose | SQL Equivalent |
|-------|---------|---------------|
| `$match` | Filter documents | `WHERE` |
| `$group` | Group & aggregate | `GROUP BY` |
| `$sort` | Sort results | `ORDER BY` |
| `$limit` | Limit results | `LIMIT` |
| `$skip` | Skip results | `OFFSET` |
| `$project` | Select/reshape fields | `SELECT` |
| `$lookup` | Join collections | `LEFT JOIN` |
| `$unwind` | Flatten array | — |
| `$addFields` | Add computed fields | — |
| `$facet` | Multiple pipelines | — |

| Accumulator | Purpose |
|------------|---------|
| `$sum` | Total |
| `$avg` | Average |
| `$min` / `$max` | Min / Max |
| `$push` | Array of values |
| `$addToSet` | Unique array |
| `$first` / `$last` | First / Last value |
| `$count` | Count documents |

### Performance Tips

| Tip | Why |
|-----|-----|
| `$match` **pehle** rakho | Index use hota hai, data kam hota hai |
| `$sort` **pehle** rakho (after $match) | Index-based sorting |
| `$project` **early** | Less data flowing through pipeline |
| `allowDiskUse: true` | 100MB pipeline limit bypass |

### 5️⃣ INTERVIEW ANSWER

> *"MongoDB's aggregation framework processes data through a **pipeline of stages** where each stage transforms the data sequentially. Key stages include `$match` for filtering, `$group` for aggregation with operators like `$sum`, `$avg`, `$push`, `$sort` for ordering, `$lookup` for joining collections (equivalent to LEFT JOIN), and `$project` for reshaping output. The pipeline concept is similar to Unix pipes. For performance, I always place `$match` and `$sort` stages **early** in the pipeline to leverage indexes and reduce the data flowing through subsequent stages. I use aggregation extensively for dashboards, reports, and analytics."*

📌 **One-liner:** Aggregation = pipeline of stages | $match → $group → $sort → $lookup | $match early for performance

---

---

## 7. Embedding vs Referencing in MongoDB

> 🏆 **Most asked MongoDB design question — schema design interviews mein guaranteed!**

### 1️⃣ WHAT

| Strategy | Approach | Data Location | SQL Equivalent |
|----------|----------|--------------|---------------|
| **Embedding** | Related data **inside** the document (nested) | Same document | Denormalized |
| **Referencing** | Related data in **separate collection**, store ObjectId | Different collections | Normalized (FK) |

### 2️⃣ WHY

MongoDB mein **JOINs expensive** hain (`$lookup` slow hai). Data model design decide karta hai:
- **Read speed** vs **Write speed**
- **Data consistency** vs **Query simplicity**
- **Document size** vs **Number of queries**

### 3️⃣ HOW — Visual Comparison

```
═══════════════════════════════════════════
EMBEDDING (Data Inside Document)
═══════════════════════════════════════════

users collection:
{
    "_id": ObjectId("u1"),
    "name": "Kanishk",
    "email": "k@test.com",
    "addresses": [                        ← Embedded!
        { "city": "Delhi", "pin": "110001", "type": "home" },
        { "city": "Mumbai", "pin": "400001", "type": "office" }
    ],
    "profile": {                          ← Embedded!
        "bio": "Full-stack dev",
        "avatar": "kan.jpg"
    }
}

✅ One query = ALL data (super fast reads!)
✅ No JOIN needed
✅ Atomic updates (single document)
❌ Document size grows (16MB limit!)
❌ Data duplication if shared across documents
❌ Updating embedded data in many docs = slow


═══════════════════════════════════════════
REFERENCING (Separate Collections)
═══════════════════════════════════════════

users collection:
{
    "_id": ObjectId("u1"),
    "name": "Kanishk",
    "orderIds": [ObjectId("o1"), ObjectId("o2"), ...]  ← References!
}

orders collection:
{ "_id": ObjectId("o1"), "total": 5000, "userId": ObjectId("u1") }
{ "_id": ObjectId("o2"), "total": 3000, "userId": ObjectId("u1") }

✅ No 16MB limit (unbounded data)
✅ Independent updates (change order without touching user)
✅ No data duplication
❌ Multiple queries needed ($lookup or populate)
❌ Slower reads
❌ No atomic cross-document updates (without transactions)
```

### 4️⃣ CODE

```javascript
// ═══ EMBEDDING Example ═══
// Best for: 1:1, 1:Few, read-heavy, rarely changing data

const userSchema = new mongoose.Schema({
    name: String,
    email: String,

    // Embed addresses (user ke saath hi aayenge — always needed together)
    addresses: [{
        city: String,
        state: String,
        pin: String,
        type: { type: String, enum: ["home", "office", "other"] }
    }],

    // Embed profile (1:1 — always accessed with user)
    profile: {
        bio: String,
        avatar: String,
        socialLinks: {
            github: String,
            linkedin: String
        }
    },

    // Embed tags (small, bounded array)
    tags: [String]  // ["developer", "nodejs", "mongodb"]
});

// ONE query gets everything!
const user = await User.findById(id);
console.log(user.addresses);  // Already loaded ✅
console.log(user.profile.bio); // Already loaded ✅


// ═══ REFERENCING Example ═══
// Best for: 1:Many (large), M:N, independently changing data

const authorSchema = new mongoose.Schema({
    name: String,
    email: String
});

const bookSchema = new mongoose.Schema({
    title: String,
    isbn: String,
    author: {
        type: mongoose.Schema.Types.ObjectId,
        ref: "Author"    // ← Reference!
    },
    reviews: [{
        type: mongoose.Schema.Types.ObjectId,
        ref: "Review"    // ← Reference! (could be 1000s)
    }]
});

const reviewSchema = new mongoose.Schema({
    rating: Number,
    comment: String,
    book: { type: mongoose.Schema.Types.ObjectId, ref: "Book" },
    user: { type: mongoose.Schema.Types.ObjectId, ref: "User" }
});

// TWO queries (or populate)
const book = await Book.findById(id).populate("author", "name");
console.log(book.author.name);  // "Robert Martin"

// Deep populate
const bookWithReviews = await Book.findById(id)
    .populate("author")
    .populate({
        path: "reviews",
        populate: { path: "user", select: "name" }
    });


// ═══ HYBRID Approach (Best of Both!) ═══
// Embed frequently accessed, reference the rest

const userSchema = new mongoose.Schema({
    name: String,
    email: String,

    // Embed: Small, frequently accessed, rarely changes
    profile: { bio: String, avatar: String },
    preferences: { theme: String, language: String },

    // Reference: Large, grows unbounded, changes independently
    orders: [{ type: ObjectId, ref: "Order" }],      // Could be 1000s!
    posts: [{ type: ObjectId, ref: "Post" }],         // Could be 100s!
    followers: [{ type: ObjectId, ref: "User" }]      // Could be 10000s!
});
```

### Decision Matrix

| Factor | Embed ✅ | Reference ✅ |
|--------|---------|-------------|
| **Data Size** | Small (< 16MB total) | Large / unbounded |
| **Read Pattern** | Always needed together | Sometimes needed |
| **Update Frequency** | Rarely changes | Frequently changes |
| **Relationship** | 1:1, 1:Few (< 100) | 1:Many (1000s), M:N |
| **Consistency** | Eventual OK | Strong needed |
| **Query Count** | Want single query | OK with multiple queries |
| **Data Duplication** | OK to duplicate | Must avoid |
| **Example** | User + Address, User + Profile | User + Orders, Post + Comments |

### 5️⃣ INTERVIEW ANSWER

> *"**Embedding** stores related data within the same document — ideal for 1:1 or 1:few relationships where data is always accessed together and rarely changes independently. It provides **fast reads** with a single query and atomic updates. **Referencing** stores related data in separate collections with ObjectId links — ideal for 1:many (large) or many:many relationships, or when data updates independently. It requires `populate()` or `$lookup` for joins. The **16MB document size limit** is a critical constraint for embedding. My rule of thumb: **embed for read performance** (profiles, addresses, small arrays), **reference for scalability** (orders, comments, followers). In practice, I often use a **hybrid approach** — embedding frequently accessed data and referencing large, unbounded collections."*

📌 **One-liner:** Embed = fast read, small data, together | Reference = large data, independent, scalable | Hybrid = best of both

---

---

## 8. Mongoose Schemas & Models

### 1️⃣ WHAT

| Concept | Definition | Analogy |
|---------|-----------|---------|
| **Schema** | Document ka **blueprint** — fields, types, validation, defaults | Building ka naksha 🏗️ |
| **Model** | Schema se bana **constructor** — database se interact karta hai | Actual building 🏢 |

> **MongoDB schema-less hai** — but Mongoose **application-level structure** add karta hai for validation, consistency, aur developer experience.

### 2️⃣ WHY

Bina Mongoose ke:
- ❌ Koi validation nahi — koi bhi data insert ho sakta hai
- ❌ No type safety — age mein string aa sakti hai
- ❌ No default values — har baar manually set karo
- ❌ No relationships — references handle karna mushkil
- ❌ No hooks — password hashing manually karna padega

Mongoose ke saath:
- ✅ Automatic validation
- ✅ Type casting
- ✅ Defaults & timestamps
- ✅ `populate()` for relationships
- ✅ Pre/post hooks (middleware)
- ✅ Virtuals & methods

### 3️⃣ HOW — Schema → Model → Database

```
Schema (Blueprint)           Model (Interface)           MongoDB
─────────────────            ─────────────────           ───────
{                            User.create({...})     →    Insert document
  name: String,              User.find({...})       →    Query documents
  email: String,             User.updateOne(...)    →    Update document
  age: Number                User.deleteOne(...)    →    Delete document
}                            User.findById(id)      →    Find by _id
```

### 4️⃣ CODE

```javascript
const mongoose = require("mongoose");
const bcrypt = require("bcryptjs");

// ═══════════════════════════════════════════
// 1. SCHEMA DEFINITION (Blueprint)
// ═══════════════════════════════════════════
const userSchema = new mongoose.Schema({
    // ── Basic Types with Validation ──
    name: {
        type: String,
        required: [true, "Name is required"],     // Custom error message
        trim: true,                                // Remove whitespace
        minlength: [2, "Name too short"],
        maxlength: [50, "Name too long"]
    },
    email: {
        type: String,
        required: true,
        unique: true,                              // Creates unique index!
        lowercase: true,                           // Auto-convert to lowercase
        match: [/^\S+@\S+\.\S+$/, "Invalid email"] // Regex validation
    },
    age: {
        type: Number,
        min: [18, "Must be 18+"],
        max: [120, "Invalid age"]
    },
    role: {
        type: String,
        enum: {
            values: ["user", "admin", "moderator"],
            message: "Invalid role"
        },
        default: "user"                            // Default value
    },
    password: {
        type: String,
        required: true,
        select: false                              // Never include in queries by default!
    },
    isActive: {
        type: Boolean,
        default: true
    },

    // ── Nested Object ──
    address: {
        city: String,
        state: String,
        pin: { type: String, match: /^\d{6}$/ }
    },

    // ── Arrays ──
    hobbies: [String],                             // Simple array
    scores: [{                                     // Array of objects
        subject: String,
        marks: { type: Number, min: 0, max: 100 }
    }],

    // ── References (Relationships!) ──
    orders: [{
        type: mongoose.Schema.Types.ObjectId,
        ref: "Order"                               // Reference to Order model
    }],
    bestFriend: {
        type: mongoose.Schema.Types.ObjectId,
        ref: "User"                                // Self-reference!
    }

}, {
    // ── Schema Options ──
    timestamps: true,                              // Auto createdAt + updatedAt
    toJSON: { virtuals: true },                    // Include virtuals in JSON
    toObject: { virtuals: true }
});

// ═══════════════════════════════════════════
// 2. VIRTUALS (Computed Fields — Not Stored!)
// ═══════════════════════════════════════════
userSchema.virtual("fullName").get(function () {
    return `${this.firstName} ${this.lastName}`;
});

userSchema.virtual("orderCount").get(function () {
    return this.orders ? this.orders.length : 0;
});

// ═══════════════════════════════════════════
// 3. INSTANCE METHODS (Per-Document)
// ═══════════════════════════════════════════
userSchema.methods.comparePassword = async function (candidatePassword) {
    return bcrypt.compare(candidatePassword, this.password);
};

userSchema.methods.toJSON = function () {
    const obj = this.toObject();
    delete obj.password;     // Never expose password!
    delete obj.__v;
    return obj;
};

// ═══════════════════════════════════════════
// 4. STATIC METHODS (Collection-Level)
// ═══════════════════════════════════════════
userSchema.statics.findActiveUsers = function () {
    return this.find({ isActive: true });
};

userSchema.statics.findByEmail = function (email) {
    return this.findOne({ email: email.toLowerCase() });
};

// ═══════════════════════════════════════════
// 5. MIDDLEWARE / HOOKS (Pre & Post)
// ═══════════════════════════════════════════

// Pre-save: Hash password before saving
userSchema.pre("save", async function (next) {
    // Only hash if password is new or modified
    if (!this.isModified("password")) return next();

    this.password = await bcrypt.hash(this.password, 12);
    next();
});

// Pre-find: Exclude inactive users by default
userSchema.pre(/^find/, function (next) {
    this.find({ isActive: { $ne: false } });
    next();
});

// Post-save: Log after save
userSchema.post("save", function (doc) {
    console.log(`✅ User saved: ${doc.name} (${doc._id})`);
});

// Pre-delete: Cleanup related data
userSchema.pre("findOneAndDelete", async function (next) {
    const userId = this.getFilter()._id;
    await mongoose.model("Order").deleteMany({ user: userId });
    next();
});

// ═══════════════════════════════════════════
// 6. MODEL CREATION
// ═══════════════════════════════════════════
const User = mongoose.model("User", userSchema);
// "User" → Collection name = "users" (auto pluralized + lowercase)

// ═══════════════════════════════════════════
// 7. USAGE IN APPLICATION
// ═══════════════════════════════════════════
async function demo() {
    // ── CREATE ──
    const user = await User.create({
        name: "Kanishk",
        email: "K@Test.com",      // Auto lowercased → "k@test.com"
        age: 25,
        password: "secret123",    // Auto hashed by pre-save hook!
        hobbies: ["coding", "gaming"]
    });

    // ── READ ──
    const users = await User.find({ age: { $gte: 18 } })
        .populate("orders")        // JOIN with orders!
        .populate("bestFriend", "name email")
        .select("name email age")  // Projection
        .sort({ createdAt: -1 })
        .limit(10)
        .lean();                   // Plain JS objects (faster!)

    const singleUser = await User.findById("64f8a...");
    const byEmail = await User.findByEmail("k@test.com");  // Static method!

    // ── UPDATE ──
    const updated = await User.findByIdAndUpdate(
        user._id,
        { $set: { age: 26 }, $push: { hobbies: "reading" } },
        { new: true, runValidators: true }  // Return updated + validate!
    );

    // ── DELETE ──
    await User.findByIdAndDelete(user._id);  // Triggers pre-delete hook!

    // ── INSTANCE METHOD ──
    const isMatch = await user.comparePassword("secret123");  // true

    // ── STATIC METHOD ──
    const active = await User.findActiveUsers();
}
```

### Schema Features Summary

| Feature | Purpose | Example |
|---------|---------|---------|
| **Types** | Data type enforcement | `String`, `Number`, `Boolean`, `ObjectId`, `Date` |
| **Validation** | Data rules | `required`, `min`, `max`, `enum`, `match` |
| **Defaults** | Auto values | `default: "user"`, `default: Date.now` |
| **Virtuals** | Computed fields | `fullName`, `orderCount` |
| **Methods** | Instance functions | `comparePassword()` |
| **Statics** | Collection functions | `findActiveUsers()` |
| **Pre Hooks** | Before operations | Hash password before save |
| **Post Hooks** | After operations | Log after save |
| **References** | Relationships | `ref: "Order"` + `populate()` |
| **Timestamps** | Auto dates | `createdAt`, `updatedAt` |
| **Indexes** | Performance | `unique: true`, `index: true` |

### 5️⃣ INTERVIEW ANSWER

> *"A **Mongoose Schema** defines the structure of MongoDB documents — field types, validation rules, defaults, indexes, and constraints. A **Model** is the compiled version of a schema that provides the interface for CRUD operations against the database. Schemas support powerful features: **virtuals** for computed fields not stored in DB, **instance methods** for per-document logic like password comparison, **static methods** for collection-level queries, and **middleware hooks** (pre/post)
