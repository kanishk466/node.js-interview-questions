# 🚀 Databases (MongoDB & PostgreSQL) — Interview Prep Guide (Advanced Level)

> **Level:** Advanced
> **Format:** What → Why → How → Example → Interview Answer
> **Prerequisite:** Basic + Intermediate (CRUD, JOINs, ACID, Normalization, Aggregation)

---

## 📑 Table of Contents

| # | Topic | Difficulty | Frequency |
|---|-------|-----------|-----------|
| 1 | [Hospital Schema Design](#1-database-schema-design-hospital-management) | ⭐⭐⭐ | 🔥🔥🔥 |
| 2 | [Query Optimization](#2-query-optimization-techniques) | ⭐⭐⭐ | 🔥🔥🔥🔥 |
| 3 | [Connection Pools](#3-database-connection-pools) | ⭐⭐⭐ | 🔥🔥🔥🔥 |
| 4 | [Database Migrations](#4-database-migrations) | ⭐⭐⭐ | 🔥🔥🔥 |
| 5 | [Sharding & Replication](#5-sharding--replication-in-mongodb) | ⭐⭐⭐⭐ | 🔥🔥🔥 |
| 6 | [CAP Theorem](#6-cap-theorem) | ⭐⭐⭐⭐ | 🔥🔥🔥🔥 |
| 7 | [Full-Text Search in PostgreSQL](#7-full-text-search-in-postgresql) | ⭐⭐⭐ | 🔥🔥🔥 |
| 8 | [Indexing Strategies](#8-indexing-strategies-for-different-query-patterns) | ⭐⭐⭐⭐ | 🔥🔥🔥🔥 |

---

## 🧠 Answer Framework

```
1️⃣  WHAT  →  "Yeh kya hai?"
2️⃣  WHY   →  "Iski zaroorat kyun hai?"
3️⃣  HOW   →  "Yeh kaam kaise karta hai?"
4️⃣  CODE  →  "Production-grade example..."
5️⃣  USE   →  "Real-world scenario..."
```

---
---

## 1. Database Schema Design — Hospital Management

> 🏆 **System Design + Database Design combo question!**

### 1️⃣ WHAT

Hospital Management System ka **complete schema design** — entities identify karna, relationships define karna, normalization apply karna, aur performance ke liye indexes lagana.

### 2️⃣ WHY

Schema design interviews mein check hota hai:
- **Entity identification** — kaunse tables chahiye?
- **Relationships** — 1:1, 1:N, M:N correctly map ho rahe hain?
- **Normalization** — redundancy kam hai?
- **Constraints** — data integrity hai?
- **Indexes** — queries fast hongi?
- **Scalability** — future growth handle hoga?

### 3️⃣ HOW — Entity Relationship Diagram

```
┌──────────────┐       ┌──────────────────┐       ┌──────────────┐
│  DEPARTMENTS │       │     DOCTORS      │       │   PATIENTS   │
├──────────────┤       ├──────────────────┤       ├──────────────┤
│ id (PK)      │◄──┐   │ id (PK)          │   ┌──►│ id (PK)      │
│ name         │   │   │ name             │   │   │ name         │
│ floor        │   └───│ department_id(FK)│   │   │ dob          │
│ head_doctor  │       │ specialization   │   │   │ gender       │
└──────────────┘       │ license_no (UQ)  │   │   │ blood_group  │
                       │ phone            │   │   │ phone (UQ)   │
                       │ is_active        │   │   │ email        │
                       └────────┬─────────┘   │   │ emergency_ct │
                                │             │   │ insurance_id │
                                ▼             │   └──────┬───────┘
                       ┌──────────────────┐   │          │
                       │  APPOINTMENTS    │   │          │
                       ├──────────────────┤   │          │
                       │ id (PK)          │   │          │
                       │ patient_id (FK)──│───┘          │
                       │ doctor_id (FK)───│───┘          │
                       │ date_time (UQ)   │              │
                       │ status           │              │
                       │ reason           │              │
                       └────────┬─────────┘              │
                                │                        │
                    ┌───────────┼───────────┐            │
                    ▼           ▼           ▼            ▼
            ┌──────────┐ ┌───────────┐ ┌──────────┐ ┌──────────┐
            │ MEDICAL  │ │PRESCRIP-  │ │ BILLING  │ │  LAB     │
            │ RECORDS  │ │TIONS      │ │          │ │  TESTS   │
            ├──────────┤ ├───────────┤ ├──────────┤ ├──────────┤
            │ id (PK)  │ │ id (PK)   │ │ id (PK)  │ │ id (PK)  │
            │ patient  │ │ record_id │ │ patient  │ │ patient  │
            │ doctor   │ │ medicine  │ │ appoint  │ │ doctor   │
            │ diagnosis│ │ dosage    │ │ amount   │ │ test_name│
            │ treatment│ │ frequency │ │ insurance│ │ result   │
            │ vitals   │ │ duration  │ │ status   │ │ status   │
            └──────────┘ └───────────┘ └──────────┘ └──────────┘
```

### 4️⃣ CODE — Complete PostgreSQL Schema

```sql
-- ═══════════════════════════════════════════
-- 1. DEPARTMENTS
-- ═══════════════════════════════════════════
CREATE TABLE departments (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL UNIQUE,
    floor INTEGER,
    phone VARCHAR(15),
    head_doctor_id INTEGER,  -- FK added later (circular ref)
    created_at TIMESTAMP DEFAULT NOW()
);

-- ═══════════════════════════════════════════
-- 2. DOCTORS
-- ═══════════════════════════════════════════
CREATE TABLE doctors (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    specialization VARCHAR(100) NOT NULL,
    department_id INTEGER NOT NULL REFERENCES departments(id),
    license_number VARCHAR(50) UNIQUE NOT NULL,
    phone VARCHAR(15) UNIQUE,
    email VARCHAR(255) UNIQUE,
    experience_years INTEGER DEFAULT 0 CHECK (experience_years >= 0),
    consultation_fee DECIMAL(10, 2) DEFAULT 500.00,
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Complete circular reference
ALTER TABLE departments
    ADD CONSTRAINT fk_head_doctor
    FOREIGN KEY (head_doctor_id) REFERENCES doctors(id)
    ON DELETE SET NULL;

-- ═══════════════════════════════════════════
-- 3. PATIENTS
-- ═══════════════════════════════════════════
CREATE TABLE patients (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    dob DATE NOT NULL,
    gender VARCHAR(10) NOT NULL CHECK (gender IN ('M', 'F', 'Other')),
    blood_group VARCHAR(5) CHECK (blood_group IN
        ('A+', 'A-', 'B+', 'B-', 'AB+', 'AB-', 'O+', 'O-')),
    phone VARCHAR(15) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE,
    address TEXT,
    emergency_contact VARCHAR(15) NOT NULL,
    emergency_name VARCHAR(100),
    insurance_provider VARCHAR(100),
    insurance_policy_no VARCHAR(50),
    allergies TEXT[],                     -- PostgreSQL Array!
    chronic_conditions TEXT[],
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- ═══════════════════════════════════════════
-- 4. APPOINTMENTS (M:N between Patient & Doctor)
-- ═══════════════════════════════════════════
CREATE TABLE appointments (
    id SERIAL PRIMARY KEY,
    patient_id INTEGER NOT NULL REFERENCES patients(id) ON DELETE CASCADE,
    doctor_id INTEGER NOT NULL REFERENCES doctors(id) ON DELETE RESTRICT,
    appointment_date TIMESTAMP NOT NULL,
    duration_minutes INTEGER DEFAULT 30,
    status VARCHAR(20) DEFAULT 'scheduled'
        CHECK (status IN ('scheduled', 'confirmed', 'in_progress',
                          'completed', 'cancelled', 'no_show')),
    reason TEXT,
    notes TEXT,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),

    -- Prevent double-booking same doctor at same time
    UNIQUE(doctor_id, appointment_date)
);

-- ═══════════════════════════════════════════
-- 5. MEDICAL RECORDS (1:N with Patient & Doctor)
-- ═══════════════════════════════════════════
CREATE TABLE medical_records (
    id SERIAL PRIMARY KEY,
    patient_id INTEGER NOT NULL REFERENCES patients(id),
    doctor_id INTEGER NOT NULL REFERENCES doctors(id),
    appointment_id INTEGER REFERENCES appointments(id),
    diagnosis TEXT NOT NULL,
    treatment TEXT,
    symptoms TEXT[],
    vitals JSONB DEFAULT '{}',
    -- Example: {"bp": "120/80", "temp": 98.6, "heart_rate": 72, "weight": 70}
    notes TEXT,
    is_confidential BOOLEAN DEFAULT false,
    created_at TIMESTAMP DEFAULT NOW()
);

-- ═══════════════════════════════════════════
-- 6. PRESCRIPTIONS (1:N with Medical Record)
-- ═══════════════════════════════════════════
CREATE TABLE prescriptions (
    id SERIAL PRIMARY KEY,
    record_id INTEGER NOT NULL REFERENCES medical_records(id) ON DELETE CASCADE,
    medicine_name VARCHAR(200) NOT NULL,
    generic_name VARCHAR(200),
    dosage VARCHAR(100) NOT NULL,          -- "500mg"
    frequency VARCHAR(50) NOT NULL,        -- "Twice daily"
    duration_days INTEGER,
    instructions TEXT,                     -- "After meals"
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT NOW()
);

-- ═══════════════════════════════════════════
-- 7. LAB TESTS
-- ═══════════════════════════════════════════
CREATE TABLE lab_tests (
    id SERIAL PRIMARY KEY,
    patient_id INTEGER NOT NULL REFERENCES patients(id),
    doctor_id INTEGER NOT NULL REFERENCES doctors(id),
    record_id INTEGER REFERENCES medical_records(id),
    test_name VARCHAR(200) NOT NULL,
    test_category VARCHAR(50),             -- "Blood", "Urine", "X-Ray", "MRI"
    result JSONB,
    -- Example: {"hemoglobin": 14.2, "wbc": 8500, "platelets": 250000}
    normal_range JSONB,
    status VARCHAR(20) DEFAULT 'pending'
        CHECK (status IN ('pending', 'in_progress', 'completed', 'cancelled')),
    sample_collected_at TIMESTAMP,
    result_delivered_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW()
);

-- ═══════════════════════════════════════════
-- 8. BILLING
-- ═══════════════════════════════════════════
CREATE TABLE billing (
    id SERIAL PRIMARY KEY,
    patient_id INTEGER NOT NULL REFERENCES patients(id),
    appointment_id INTEGER REFERENCES appointments(id),
    record_id INTEGER REFERENCES medical_records(id),
    description TEXT NOT NULL,
    amount DECIMAL(10, 2) NOT NULL CHECK (amount >= 0),
    insurance_covered DECIMAL(10, 2) DEFAULT 0,
    patient_responsibility DECIMAL(10, 2) GENERATED ALWAYS AS
        (amount - insurance_covered) STORED,  -- Computed column!
    status VARCHAR(20) DEFAULT 'pending'
        CHECK (status IN ('pending', 'partial', 'paid', 'overdue', 'written_off')),
    due_date DATE NOT NULL,
    paid_at TIMESTAMP,
    payment_method VARCHAR(20),
    created_at TIMESTAMP DEFAULT NOW()
);

-- ═══════════════════════════════════════════
-- 9. ROOMS & ADMISSIONS (For In-Patients)
-- ═══════════════════════════════════════════
CREATE TABLE rooms (
    id SERIAL PRIMARY KEY,
    room_number VARCHAR(10) UNIQUE NOT NULL,
    room_type VARCHAR(20) CHECK (room_type IN
        ('General', 'Semi-Private', 'Private', 'ICU', 'NICU')),
    floor INTEGER,
    daily_rate DECIMAL(10, 2) NOT NULL,
    is_occupied BOOLEAN DEFAULT false
);

CREATE TABLE admissions (
    id SERIAL PRIMARY KEY,
    patient_id INTEGER NOT NULL REFERENCES patients(id),
    doctor_id INTEGER NOT NULL REFERENCES doctors(id),
    room_id INTEGER REFERENCES rooms(id),
    admitted_at TIMESTAMP DEFAULT NOW(),
    discharged_at TIMESTAMP,
    reason TEXT NOT NULL,
    discharge_summary TEXT,
    status VARCHAR(20) DEFAULT 'admitted'
        CHECK (status IN ('admitted', 'discharged', 'transferred'))
);

-- ═══════════════════════════════════════════
-- 10. PERFORMANCE INDEXES
-- ═══════════════════════════════════════════

-- Appointment lookups (most frequent query!)
CREATE INDEX idx_appt_doctor_date ON appointments(doctor_id, appointment_date);
CREATE INDEX idx_appt_patient ON appointments(patient_id, appointment_date DESC);
CREATE INDEX idx_appt_status ON appointments(status) WHERE status IN ('scheduled', 'confirmed');

-- Patient search
CREATE INDEX idx_patient_name ON patients USING gin(to_tsvector('english', name));
CREATE INDEX idx_patient_phone ON patients(phone);
CREATE INDEX idx_patient_dob ON patients(dob);

-- Medical records
CREATE INDEX idx_records_patient ON medical_records(patient_id, created_at DESC);
CREATE INDEX idx_records_vitals ON medical_records USING gin(vitals);

-- Lab tests
CREATE INDEX idx_lab_patient ON lab_tests(patient_id, status);
CREATE INDEX idx_lab_result ON lab_tests USING gin(result);

-- Billing
CREATE INDEX idx_billing_patient ON billing(patient_id, status);
CREATE INDEX idx_billing_due ON billing(due_date) WHERE status IN ('pending', 'overdue');

-- ═══════════════════════════════════════════
-- 11. VIEWS (Common Queries)
-- ═══════════════════════════════════════════
CREATE VIEW patient_summary AS
SELECT
    p.id, p.name, p.phone, p.blood_group,
    COUNT(DISTINCT a.id) AS total_appointments,
    COUNT(DISTINCT mr.id) AS total_records,
    COALESCE(SUM(b.patient_responsibility), 0) AS outstanding_balance
FROM patients p
LEFT JOIN appointments a ON p.id = a.patient_id
LEFT JOIN medical_records mr ON p.id = mr.patient_id
LEFT JOIN billing b ON p.id = b.patient_id AND b.status != 'paid'
GROUP BY p.id;

-- ═══════════════════════════════════════════
-- 12. TRIGGERS (Auto-updates)
-- ═══════════════════════════════════════════
CREATE OR REPLACE FUNCTION update_timestamp()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_patients_update
    BEFORE UPDATE ON patients
    FOR EACH ROW EXECUTE FUNCTION update_timestamp();

CREATE TRIGGER trg_doctors_update
    BEFORE UPDATE ON doctors
    FOR EACH ROW EXECUTE FUNCTION update_timestamp();
```

### 5️⃣ INTERVIEW ANSWER

> *"For a hospital management system, I'd design around **8-10 core entities**: Patients, Doctors, Departments, Appointments, Medical Records, Prescriptions, Lab Tests, Billing, Rooms, and Admissions. Key relationships: Doctors belong to a Department (**M:1**), Appointments link Patients and Doctors (**M:N via junction**), Medical Records and Prescriptions are **1:N** from Appointments. I'd use **JSONB** for flexible data like vitals and lab results. Critical constraints include `UNIQUE(doctor_id, appointment_date)` to prevent double-booking, CHECK constraints on status enums, and a computed column for `patient_responsibility`. I'd add **partial indexes** on active appointments and overdue bills, **GIN indexes** on JSONB fields, and **triggers** for auto-updating timestamps. I'd also create **views** for common dashboard queries like patient summaries."*

📌 **One-liner:** Core entities → relationships → constraints → JSONB for flexible data → partial + GIN indexes → views + triggers

---
---

## 2. Query Optimization Techniques

### 1️⃣ WHAT

Query optimization = **slow queries ko fast banana** through indexes, query restructuring, execution plan analysis, caching, aur database configuration tuning.

### 2️⃣ WHY

| Metric | Unoptimized | Optimized | Impact |
|--------|------------|-----------|--------|
| Response Time | 5 seconds | 50ms | 100× faster |
| CPU Usage | 90% | 15% | Server stable |
| Memory | High (temp tables) | Low | No OOM crashes |
| Throughput | 100 req/s | 10,000 req/s | 100× more users |

### 3️⃣ HOW — Optimization Layers

```
┌─────────────────────────────────────────────────┐
│          QUERY OPTIMIZATION PYRAMID              │
│                                                  │
│  Level 5: Database Config                        │
│  ├── shared_buffers, work_mem, effective_cache   │
│  └── Connection pooling                          │
│                                                  │
│  Level 4: Architecture                           │
│  ├── Read replicas (read/write split)            │
│  ├── Caching layer (Redis)                       │
│  └── Denormalization for reads                   │
│                                                  │
│  Level 3: Query Rewriting                        │
│  ├── Avoid SELECT *                              │
│  ├── Replace subqueries with JOINs               │
│  ├── Use EXISTS instead of IN                    │
│  └── Pagination with cursor (not OFFSET)         │
│                                                  │
│  Level 2: Indexing                               │
│  ├── B-Tree, GIN, GiST, BRIN                     │
│  ├── Partial indexes                             │
│  ├── Covering indexes                            │
│  └── Composite index column order                │
│                                                  │
│  Level 1: EXPLAIN ANALYZE (Start Here!)          │
│  ├── Seq Scan → Index Scan                       │
│  ├── Nested Loop → Hash Join                     │
│  └── Sort → Index-based sort                     │
└─────────────────────────────────────────────────┘
```

### 4️⃣ CODE

```sql
-- ═══════════════════════════════════════════
-- 1. EXPLAIN ANALYZE (Always Start Here!)
-- ═══════════════════════════════════════════

-- ❌ Slow query
EXPLAIN ANALYZE
SELECT * FROM orders WHERE user_id = 123 AND status = 'completed';

-- Output (BAD):
-- Seq Scan on orders  (cost=0.00..45000.00 rows=50 width=200)
--   Filter: (user_id = 123 AND status = 'completed')
--   Rows Removed by Filter: 999950
-- Execution Time: 2500ms  💀

-- ✅ After adding index
CREATE INDEX idx_orders_user_status ON orders(user_id, status);

EXPLAIN ANALYZE
SELECT * FROM orders WHERE user_id = 123 AND status = 'completed';

-- Output (GOOD):
-- Index Scan using idx_orders_user_status on orders
--   (cost=0.42..8.44 rows=50 width=200)
--   Index Cond: (user_id = 123 AND status = 'completed')
-- Execution Time: 2ms  ✅

-- ═══════════════════════════════════════════
-- 2. AVOID SELECT * (Fetch Only What You Need)
-- ═══════════════════════════════════════════

-- ❌ BAD: Fetches all 50 columns including large TEXT/JSONB
SELECT * FROM users WHERE status = 'active';

-- ✅ GOOD: Only needed columns (can use covering index!)
SELECT id, name, email FROM users WHERE status = 'active';

-- ═══════════════════════════════════════════
-- 3. EXISTS vs IN (Subquery Optimization)
-- ═══════════════════════════════════════════

-- ❌ IN (evaluates entire subquery)
SELECT * FROM users
WHERE id IN (SELECT user_id FROM orders WHERE total > 10000);

-- ✅ EXISTS (stops at first match — faster!)
SELECT u.* FROM users u
WHERE EXISTS (
    SELECT 1 FROM orders o
    WHERE o.user_id = u.id AND o.total > 10000
);

-- ═══════════════════════════════════════════
-- 4. PAGINATION: OFFSET vs CURSOR
-- ═══════════════════════════════════════════

-- ❌ OFFSET (scans and discards rows — SLOW on large offsets!)
SELECT * FROM orders ORDER BY id LIMIT 20 OFFSET 100000;
-- Scans 100,020 rows, returns 20! 💀

-- ✅ Keyset/Cursor Pagination (instant!)
SELECT * FROM orders
WHERE id > 100000   -- Last seen ID from previous page
ORDER BY id
LIMIT 20;
-- Direct index lookup! ✅

-- ═══════════════════════════════════════════
-- 5. JOIN ORDER & Type
-- ═══════════════════════════════════════════

-- ❌ Subquery in SELECT (N+1 problem!)
SELECT
    u.name,
    (SELECT COUNT(*) FROM orders WHERE user_id = u.id) AS order_count
FROM users u;

-- ✅ JOIN with GROUP BY (single scan!)
SELECT u.name, COUNT(o.id) AS order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.name;

-- ═══════════════════════════════════════════
-- 6. PARTIAL INDEXES (Index Only What Matters)
-- ═══════════════════════════════════════════

-- ❌ Full index on status (indexes 95% 'completed' rows — waste!)
CREATE INDEX idx_orders_status ON orders(status);

-- ✅ Partial index (only active orders — 5% of data!)
CREATE INDEX idx_orders_active ON orders(status, created_at)
WHERE status IN ('pending', 'processing');
-- 20× smaller index, much faster!

-- ═══════════════════════════════════════════
-- 7. COVERING INDEX (Index-Only Scan!)
-- ═══════════════════════════════════════════

-- Query only needs: user_id, status, total
CREATE INDEX idx_orders_covering ON orders(user_id, status, total);

SELECT status, total FROM orders WHERE user_id = 123;
-- "Index Only Scan" — doesn't even touch the table! ✅✅

-- ═══════════════════════════════════════════
-- 8. CTE vs Subquery vs Temp Table
-- ═══════════════════════════════════════════

-- ✅ CTE (Common Table Expression — readable + optimizable in PG 12+)
WITH active_users AS (
    SELECT id, name FROM users WHERE is_active = true
),
user_orders AS (
    SELECT user_id, SUM(total) AS total_spent
    FROM orders
    GROUP BY user_id
)
SELECT au.name, uo.total_spent
FROM active_users au
JOIN user_orders uo ON au.id = uo.user_id
WHERE uo.total_spent > 10000;

-- ═══════════════════════════════════════════
-- 9. BATCH UPDATES (Avoid Lock Contention)
-- ═══════════════════════════════════════════

-- ❌ Update 1M rows at once (locks table for minutes!)
UPDATE users SET status = 'inactive'
WHERE last_login < NOW() - INTERVAL '1 year';

-- ✅ Batch update (1000 at a time)
DO $$
DECLARE
    batch_size INT := 1000;
    rows_updated INT;
BEGIN
    LOOP
        UPDATE users SET status = 'inactive'
        WHERE id IN (
            SELECT id FROM users
            WHERE last_login < NOW() - INTERVAL '1 year'
              AND status = 'active'
            LIMIT batch_size
        );
        GET DIAGNOSTICS rows_updated = ROW_COUNT;
        EXIT WHEN rows_updated = 0;
        COMMIT;  -- Release lock between batches
    END LOOP;
END $$;
```

```javascript
// ═══════════════════════════════════════════
// 10. MONGODB QUERY OPTIMIZATION
// ═══════════════════════════════════════════

// ❌ BAD: No index, full collection scan
db.orders.find({ "items.category": "electronics", total: { $gt: 5000 } });

// ✅ GOOD: Compound index matching query shape
db.orders.createIndex({ "items.category": 1, total: 1 });

// ❌ BAD: $where (JavaScript execution — SLOW!)
db.users.find({ $where: "this.age > 25 && this.name.startsWith('K')" });

// ✅ GOOD: Native operators + index
db.users.createIndex({ age: 1, name: 1 });
db.users.find({ age: { $gt: 25 }, name: /^K/ });

// Use explain() to verify
db.orders.find({ userId: 123 }).explain("executionStats");
// Check: "stage": "IXSCAN" ✅ vs "COLLSCAN" ❌
// Check: "totalDocsExamined" ≈ "nReturned" ✅
```

### Optimization Checklist

| # | Technique | Impact | Effort |
|---|-----------|--------|--------|
| 1 | EXPLAIN ANALYZE every slow query | ⭐⭐⭐⭐⭐ | Low |
| 2 | Add indexes on WHERE/JOIN/ORDER BY | ⭐⭐⭐⭐⭐ | Low |
| 3 | Avoid SELECT * | ⭐⭐⭐⭐ | Low |
| 4 | Keyset pagination (no OFFSET) | ⭐⭐⭐⭐ | Medium |
| 5 | EXISTS over IN for subqueries | ⭐⭐⭐ | Low |
| 6 | Partial indexes for filtered queries | ⭐⭐⭐⭐ | Low |
| 7 | Covering indexes (index-only scan) | ⭐⭐⭐⭐ | Medium |
| 8 | Batch large updates | ⭐⭐⭐⭐ | Medium |
| 9 | Connection pooling | ⭐⭐⭐⭐ | Medium |
| 10 | Cache hot queries (Redis) | ⭐⭐⭐⭐⭐ | Medium |

### 5️⃣ INTERVIEW ANSWER

> *"I approach query optimization systematically, starting with **EXPLAIN ANALYZE** to identify bottlenecks — looking for sequential scans, nested loops on large tables, and high row counts. The highest-impact fixes are **proper indexing** — B-Tree for equality/range, GIN for JSONB/arrays, partial indexes for filtered queries, and covering indexes for index-only scans. I rewrite queries to avoid `SELECT *`, replace `IN` subqueries with `EXISTS`, and use **keyset pagination** instead of `OFFSET` for large datasets. For bulk operations, I use **batch processing** to avoid lock contention. At the architecture level, I add **Redis caching** for hot queries and **read replicas** for read-heavy workloads. In MongoDB, I verify with `explain()` that queries use `IXSCAN` and that `totalDocsExamined` is close to `nReturned`."*

📌 **One-liner:** EXPLAIN first → Index properly → Rewrite queries → Cache hot data → Batch large ops

---
---

## 3. Database Connection Pools

### 1️⃣ WHAT

Connection pool ek **pre-created database connections ka cache** hai — har request ke liye naya connection banane ke bajaye, **existing connection reuse** kiya jata hai.

> **Analogy:** Taxi stand 🚕
> - Bina pool: Har customer ke liye nayi taxi manufacture karo → 200ms
> - Pool ke saath: Available taxi use karo → 0ms | Kaam hone pe wapas stand pe

### 2️⃣ WHY

| Metric | No Pool (New Connection Each Time) | With Pool |
|--------|-----------------------------------|-----------|
| Connection Time | ~100-300ms per request | ~0ms (reused) |
| Max Connections | OS/DB limit hit fast | Controlled (e.g., 20) |
| Memory | High (many TCP sockets) | Low (fixed pool) |
| DB Load | Connection storm → crash | Stable |
| Throughput | Low | High |

```
Without Pool:
Request 1 → TCP Handshake (50ms) → Auth (100ms) → Query (10ms) → Close (30ms) = 190ms
Request 2 → TCP Handshake (50ms) → Auth (100ms) → Query (10ms) → Close (30ms) = 190ms
Request 3 → TCP Handshake (50ms) → Auth (100ms) → Query (10ms) → Close (30ms) = 190ms
Total: 570ms + DB has 3 connections open/close 💀

With Pool (20 connections pre-created):
Request 1 → Grab Conn1 (0ms) → Query (10ms) → Return Conn1 (0ms) = 10ms
Request 2 → Grab Conn2 (0ms) → Query (10ms) → Return Conn2 (0ms) = 10ms
Request 3 → Grab Conn3 (0ms) → Query (10ms) → Return Conn3 (0ms) = 10ms
Total: 30ms + DB has stable 20 connections ✅
```

### 3️⃣ HOW — Pool Architecture

```
┌─────────────────────────────────────────────┐
│         CONNECTION POOL                      │
│                                              │
│  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐       │
│  │ Idle │ │ Idle │ │ BUSY │ │ BUSY │       │
│  │  #1  │ │  #2  │ │  #3  │ │  #4  │← Req  │
│  └──────┘ └──────┘ └──────┘ └──────┘       │
│  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐       │
│  │ BUSY │ │ Idle │ │ Idle │ │ Idle │       │
│  │  #5  │ │  #6  │ │  #7  │ │  #8  │       │
│  │← Req │ └──────┘ └──────┘ └──────┘       │
│  └──────┘                                    │
│                                              │
│  Config:                                     │
│  ├── min: 5    (always keep alive)           │
│  ├── max: 20   (never exceed)                │
│  ├── idleTimeout: 30s (close unused)         │
│  ├── acquireTimeout: 10s (wait for free)     │
│  └── maxLifetime: 1hr (prevent stale)        │
│                                              │
│  Flow:                                       │
│  Request → Acquire conn → Execute → Release  │
│  Pool full? → Wait (queue) → Timeout → Error │
└─────────────────────────────────────────────┘
```

### 4️⃣ CODE

```javascript
// ═══════════════════════════════════════════
// 1. PostgreSQL (pg library)
// ═══════════════════════════════════════════
const { Pool } = require("pg");

const pool = new Pool({
    host: process.env.DB_HOST,
    port: 5432,
    database: process.env.DB_NAME,
    user: process.env.DB_USER,
    password: process.env.DB_PASS,

    // Pool Configuration
    max: 20,                        // Max connections in pool
    min: 5,                         // Min idle connections
    idleTimeoutMillis: 30000,       // Close idle after 30s
    connectionTimeoutMillis: 10000, // Wait 10s for free connection
    maxLifetimeSeconds: 3600,       // Recycle after 1 hour
    maxUses: 7500,                  // Recycle after 7500 queries
});

// Monitor pool events
pool.on("connect", () => console.log("New connection created"));
pool.on("acquire", () => console.log("Connection acquired"));
pool.on("remove", () => console.log("Connection removed"));
pool.on("error", (err) => console.error("Pool error:", err));

// ── Usage Pattern 1: Simple Query (auto-release) ──
app.get("/users", async (req, res) => {
    const { rows } = await pool.query("SELECT id, name FROM users LIMIT 10");
    res.json(rows);  // Connection auto-released ✅
});

// ── Usage Pattern 2: Transaction (manual release!) ──
app.post("/transfer", async (req, res) => {
    const client = await pool.connect();  // Grab from pool

    try {
        await client.query("BEGIN");
        await client.query(
            "UPDATE accounts SET balance = balance - $1 WHERE id = $2",
            [5000, req.body.fromId]
        );
        await client.query(
            "UPDATE accounts SET balance = balance + $1 WHERE id = $2",
            [5000, req.body.toId]
        );
        await client.query("COMMIT");
        res.json({ success: true });
    } catch (err) {
        await client.query("ROLLBACK");
        res.status(500).json({ error: err.message });
    } finally {
        client.release();  // ⚠️ ALWAYS release back to pool!
    }
});

// ── Pool Monitoring (Production!) ──
setInterval(() => {
    console.log("Pool Stats:", {
        total: pool.totalCount,      // Total connections
        idle: pool.idleCount,        // Available
        waiting: pool.waitingCount,  // Requests waiting
    });
}, 30000);

// ═══════════════════════════════════════════
// 2. MongoDB (Mongoose)
// ═══════════════════════════════════════════
const mongoose = require("mongoose");

mongoose.connect(process.env.MONGODB_URI, {
    maxPoolSize: 20,           // Max connections (default: 100!)
    minPoolSize: 5,            // Min connections
    maxIdleTimeMS: 30000,      // Close idle after 30s
    waitQueueTimeoutMS: 10000, // Wait 10s for connection
    serverSelectionTimeoutMS: 5000,
    socketTimeoutMS: 45000,
    connectTimeoutMS: 10000,
});

// ═══════════════════════════════════════════
// 3. MySQL (mysql2)
// ═══════════════════════════════════════════
const mysql = require("mysql2/promise");

const mysqlPool = mysql.createPool({
    host: process.env.DB_HOST,
    user: process.env.DB_USER,
    password: process.env.DB_PASS,
    database: process.env.DB_NAME,
    waitForConnections: true,
    connectionLimit: 20,
    queueLimit: 0,             // Unlimited queue
    idleTimeout: 30000,
    enableKeepAlive: true,
    keepAliveInitialDelay: 10000,
});

// Usage
const [rows] = await mysqlPool.execute(
    "SELECT * FROM users WHERE id = ?", [userId]
);

// ═══════════════════════════════════════════
// 4. POOL SIZING FORMULA
// ═══════════════════════════════════════════
// Optimal Pool Size ≈ (CPU Cores × 2) + Effective Spindle Count
//
// Example: 4 cores, SSD
// Pool ≈ (4 × 2) + 1 = 9-10
//
// Guidelines:
//   Small app:    5-10
//   Medium app:   10-20
//   Large app:    20-50
//   ⚠️ More ≠ Better! Too many = DB overload + context switching

// ═══════════════════════════════════════════
// 5. GRACEFUL POOL SHUTDOWN
// ═══════════════════════════════════════════
process.on("SIGTERM", async () => {
    console.log("Closing pool...");
    await pool.end();          // Wait for active queries, then close
    await mongoose.disconnect();
    await mysqlPool.end();
    process.exit(0);
});
```

### Common Pool Mistakes

| Mistake | Consequence | Fix |
|---------|------------|-----|
| Forgetting `client.release()` | Pool exhaustion → deadlock | Always use `finally` block |
| Pool too large (100+) | DB overloaded, context switching | Use formula: (cores × 2) + 1 |
| Pool too small (2-3) | Requests queue up → timeouts | Monitor `waitingCount` |
| No idle timeout | Stale connections → errors | Set `idleTimeoutMillis` |
| No connection timeout | Infinite wait → hung requests | Set `connectionTimeoutMillis` |
| Not handling pool errors | Silent failures | Listen to `error` event |

### 5️⃣ INTERVIEW ANSWER

> *"A connection pool maintains a **pre-created set of reusable database connections**, eliminating the ~200ms overhead of TCP handshake and authentication per request. I configure pools with `max` (upper limit based on `(CPU cores × 2) + 1`), `min` (keep-alive), `idleTimeout` (cleanup unused), and `acquireTimeout` (prevent infinite waits). In PostgreSQL's `pg` library, simple queries auto-release connections, but for transactions I manually `connect()` and always `release()` in a `finally` block. A critical mistake is forgetting to release — it causes **pool exhaustion** where all connections are busy and new requests hang. I monitor pool stats (total, idle, waiting) in production and implement graceful pool shutdown on SIGTERM."*

📌 **One-liner:** Pool = reuse connections | max/min/idle config | Always release in finally | Monitor waiting count

---
---

## 4. Database Migrations

### 1️⃣ WHAT

Database migration = **version control for your database schema**. Jaise Git code track karta hai, migrations schema changes track karti hain — tables add/drop, columns add/rename, indexes create, data transform.

> **Analogy:** Building renovation 🏗️ — har change ka blueprint record karo, taaki rollback kar sako agar kuch galat ho.

### 2️⃣ WHY

| Problem Without Migrations | Solution With Migrations |
|---------------------------|-------------------------|
| "Kisne ye column add kiya?" | Git history mein sab record |
| Production DB ≠ Dev DB | Same migrations = same schema |
| Rollback impossible | `migrate down` = undo |
| Team conflicts | Sequential migration files |
| Manual SQL scripts | Automated, repeatable |

### 3️⃣ HOW — Migration Lifecycle

```
Developer writes migration file
         │
         ▼
┌─────────────────────────┐
│ 001_create_users.js     │  ← Version number + description
│ 002_add_email_column.js │
│ 003_create_orders.js    │
│ 004_add_indexes.js      │
└─────────┬───────────────┘
          │
          ▼
    Run: npm run migrate:up
          │
          ▼
┌─────────────────────────┐
│  Migration Table (DB)   │  ← Tracks which migrations ran
│  ┌─────┬──────────────┐ │
│  │ 001 │ 2024-01-01   │ │  ✅ Applied
│  │ 002 │ 2024-01-02   │ │  ✅ Applied
│  │ 003 │ 2024-01-03   │ │  ✅ Applied
│  │ 004 │ pending      │ │  ⏳ Not yet
│  └─────┴──────────────┘ │
└─────────────────────────┘
          │
          ▼
    Run: npm run migrate:down  ← Rollback last migration
```

### 4️⃣ CODE

```javascript
// ═══════════════════════════════════════════
// 1. KNEX.js (Most Popular for Node.js)
// ═══════════════════════════════════════════

// knexfile.js — Configuration
module.exports = {
    development: {
        client: "pg",
        connection: process.env.DATABASE_URL,
        migrations: { directory: "./migrations" },
        seeds: { directory: "./seeds" }
    },
    production: {
        client: "pg",
        connection: process.env.DATABASE_URL,
        pool: { min: 2, max: 10 },
        migrations: { directory: "./migrations" }
    }
};

// ── Migration: Create Users Table ──
// File: migrations/20240101000001_create_users.js
exports.up = function (knex) {
    return knex.schema.createTable("users", (table) => {
        table.increments("id").primary();
        table.string("name", 100).notNullable();
        table.string("email", 255).unique().notNullable();
        table.string("password_hash").notNullable();
        table.enum("role", ["user", "admin"]).defaultTo("user");
        table.boolean("is_active").defaultTo(true);
        table.timestamps(true, true);  // created_at, updated_at
    });
};

exports.down = function (knex) {
    return knex.schema.dropTableIfExists("users");
};

// ── Migration: Add Column ──
// File: migrations/20240102000002_add_phone_to_users.js
exports.up = function (knex) {
    return knex.schema.alterTable("users", (table) => {
        table.string("phone", 15).unique().nullable();
        table.index("phone");
    });
};

exports.down = function (knex) {
    return knex.schema.alterTable("users", (table) => {
        table.dropIndex("phone");
        table.dropColumn("phone");
    });
};

// ── Migration: Create Orders with FK ──
// File: migrations/20240103000003_create_orders.js
exports.up = function (knex) {
    return knex.schema.createTable("orders", (table) => {
        table.increments("id").primary();
        table.integer("user_id").unsigned().notNullable()
            .references("id").inTable("users")
            .onDelete("CASCADE").onUpdate("CASCADE");
        table.decimal("total", 10, 2).notNullable();
        table.enum("status", ["pending", "completed", "cancelled"])
            .defaultTo("pending");
        table.timestamps(true, true);
    });
};

exports.down = function (knex) {
    return knex.schema.dropTableIfExists("orders");
};

// ── Migration: Data Migration (Transform existing data!) ──
// File: migrations/20240104000004_split_name_field.js
exports.up = async function (knex) {
    // Step 1: Add new columns
    await knex.schema.alterTable("users", (table) => {
        table.string("first_name", 50);
        table.string("last_name", 50);
    });

    // Step 2: Migrate existing data
    const users = await knex("users").select("id", "name");
    for (const user of users) {
        const [first, ...rest] = user.name.split(" ");
        await knex("users").where("id", user.id).update({
            first_name: first,
            last_name: rest.join(" ") || null
        });
    }

    // Step 3: Drop old column (after verifying!)
    // await knex.schema.alterTable("users", (table) => {
    //     table.dropColumn("name");
    // });
};

exports.down = async function (knex) {
    // Reverse the data migration
    await knex.schema.alterTable("users", (table) => {
        table.dropColumn("first_name");
        table.dropColumn("last_name");
    });
};

// ═══════════════════════════════════════════
// 2. MIGRATION COMMANDS
// ═══════════════════════════════════════════
// npx knex migrate:make create_users     → Create new migration file
// npx knex migrate:latest                → Run all pending migrations (UP)
// npx knex migrate:rollback              → Undo last batch (DOWN)
// npx knex migrate:rollback --all        → Undo ALL migrations
// npx knex migrate:status                → Show migration status
// npx knex migrate:up                    → Run next pending migration
// npx knex migrate:down                  → Undo last migration
// npx knex seed:run                      → Run seed data

// ═══════════════════════════════════════════
// 3. MONGOOSE-MIGRATE (MongoDB)
// ═══════════════════════════════════════════
// MongoDB schema-less hai, but data migrations still zaroori hain!

// Example: Rename field in all documents
exports.up = async function (db) {
    await db.collection("users").updateMany(
        { userName: { $exists: true } },
        { $rename: { userName: "username" } }
    );
};

exports.down = async function (db) {
    await db.collection("users").updateMany(
        { username: { $exists: true } },
        { $rename: { username: "userName" } }
    );
};

// ═══════════════════════════════════════════
// 4. CI/CD INTEGRATION
// ═══════════════════════════════════════════
// .github/workflows/deploy.yml
// - name: Run migrations
//   run: npx knex migrate:latest
//   env:
//     DATABASE_URL: ${{ secrets.PROD_DB_URL }}
//
// ⚠️ Rules:
// 1. NEVER edit applied migrations (create new ones!)
// 2. Always test migrations on staging first
// 3. Backup DB before production migration
// 4. Keep migrations small and focused
// 5. Both UP and DOWN must work
```

### Migration Best Practices

| Practice | Why |
|----------|-----|
| **Never edit applied migrations** | Team sync breaks, rollback fails |
| **Small, focused migrations** | Easy to debug and rollback |
| **Always write DOWN** | Rollback must work |
| **Test on staging first** | Catch issues before production |
| **Backup before prod migration** | Safety net |
| **Data migrations in batches** | Avoid locking large tables |
| **Backward compatible** | Deploy code before removing columns |

### 5️⃣ INTERVIEW ANSWER

> *"Database migrations are **version control for schemas** — they track every change (create table, add column, create index) as sequential, reversible scripts. I use **Knex.js** for PostgreSQL migrations, where each migration has an `up` function (apply change) and a `down` function (rollback). Key rules I follow: **never edit applied migrations** (create new ones instead), always test on staging first, keep migrations small and focused, and ensure both up and down work correctly. For data migrations (like renaming fields across millions of rows), I process in **batches** to avoid table locks. In CI/CD, migrations run automatically before deployment. For MongoDB, I use similar migration tools for data transformations even though the schema is flexible."*

📌 **One-liner:** Migrations = Git for DB schema | up/down functions | Never edit applied | Batch data migrations

---
---

## 5. Sharding & Replication in MongoDB

### 1️⃣ WHAT

| Concept | Definition | Purpose |
|---------|-----------|---------|
| **Replication** | Data ki **copies** multiple servers pe | High Availability + Read Scaling |
| **Sharding** | Data ko **split** karke multiple servers pe | Write Scaling + Storage Scaling |

> **Simple:** Replication = same data, multiple copies 📋📋📋 | Sharding = data divided, multiple pieces 🧩🧩🧩

### 2️⃣ WHY

| Problem | Solution |
|---------|----------|
| Single server crash → downtime | Replication (failover) |
| Single server storage full | Sharding (distribute data) |
| Read traffic too high | Replication (read from secondaries) |
| Write traffic too high | Sharding (write to different shards) |
| Single server RAM limit | Sharding (each shard = smaller dataset) |

### 3️⃣ HOW — Architecture

```
═══════════════════════════════════════════
REPLICATION (Replica Set)
═══════════════════════════════════════════

         ┌──────────────┐
         │   PRIMARY    │  ← All writes go here
         │  (Node A)    │
         └──────┬───────┘
                │ Oplog (operations log)
        ┌───────┼───────┐
        ▼       ▼       ▼
  ┌──────────┐ ┌──────────┐ ┌──────────┐
  │SECONDARY │ │SECONDARY │ │ ARBITER  │
  │ (Node B) │ │ (Node C) │ │ (Node D) │
  │ Read ✅  │ │ Read ✅  │ │ Vote only│
  │ Write ❌ │ │ Write ❌ │ │ No data  │
  └──────────┘ └──────────┘ └──────────┘

  If Primary crashes → Secondary elected as new Primary ✅
  (Automatic failover in ~10-12 seconds)


═══════════════════════════════════════════
SHARDING (Sharded Cluster)
═══════════════════════════════════════════

         ┌──────────────┐
         │   MONGOS     │  ← Query Router (client connects here)
         │  (Router)    │
         └──────┬───────┘
                │ Routes based on shard key
    ┌───────────┼───────────┐
    ▼           ▼           ▼
┌────────┐ ┌────────┐ ┌────────┐
│ SHARD 1│ │ SHARD 2│ │ SHARD 3│
│ A-D    │ │ E-M    │ │ N-Z    │
│ (1TB)  │ │ (1TB)  │ │ (1TB)  │
│        │ │        │ │        │
│ Each shard is a Replica Set! │
│ [P][S][S] [P][S][S] [P][S][S]│
└────────┘ └────────┘ └────────┘

    ┌──────────────┐
    │ CONFIG SERVER│  ← Metadata: which data is on which shard
    │  (3 nodes)   │
    └──────────────┘
```

### 4️⃣ CODE

```javascript
// ═══════════════════════════════════════════
// 1. REPLICA SET SETUP
// ═══════════════════════════════════════════

// Start 3 mongod instances
// mongod --replSet myRS --port 27017 --dbpath /data/rs1
// mongod --replSet myRS --port 27018 --dbpath /data/rs2
// mongod --replSet myRS --port 27019 --dbpath /data/rs3

// Initialize replica set
rs.initiate({
    _id: "myRS",
    members: [
        { _id: 0, host: "localhost:27017", priority: 2 },  // Primary
        { _id: 1, host: "localhost:27018", priority: 1 },  // Secondary
        { _id: 2, host: "localhost:27019", priority: 1 }   // Secondary
    ]
});

// Check status
rs.status();
rs.isMaster();

// ── Read Preference (Application Level) ──
const { MongoClient, ReadPreference } = require("mongodb");

const client = new MongoClient("mongodb://localhost:27017,localhost:27018/?replicaSet=myRS", {
    readPreference: ReadPreference.SECONDARY_PREFERRED
    // Options: PRIMARY, SECONDARY, SECONDARY_PREFERRED, NEAREST
});

// Mongoose
mongoose.connect("mongodb://host1,host2,host3/mydb?replicaSet=myRS", {
    readPreference: "secondaryPreferred"
});

// ═══════════════════════════════════════════
// 2. SHARDING SETUP
// ═══════════════════════════════════════════

// Enable sharding on database
sh.enableSharding("myApp");

// Shard a collection (choose shard key carefully!)
sh.shardCollection("myApp.orders", { userId: 1 });

// ── Shard Key Strategies ──

// Ranged Sharding (good for range queries)
sh.shardCollection("myApp.logs", { timestamp: 1 });
// Shard 1: Jan-Mar | Shard 2: Apr-Jun | Shard 3: Jul-Sep

// Hashed Sharding (even distribution)
sh.shardCollection("myApp.users", { _id: "hashed" });
// Hash of _id determines shard → even distribution

// Compound Shard Key (best for most cases)
sh.shardCollection("myApp.orders", { userId: 1, createdAt: 1 });

// ═══════════════════════════════════════════
// 3. SHARD KEY SELECTION (Critical Decision!)
// ═══════════════════════════════════════════
```

### Shard Key Selection Guide

| Property | Good Shard Key | Bad Shard Key |
|----------|---------------|---------------|
| **Cardinality** | High (many unique values) | Low (e.g., `status` = 3 values) |
| **Frequency** | Evenly distributed | Hotspot (e.g., `country` = 90% "IN") |
| **Monotonicity** | Not always increasing | Always increasing (e.g., timestamp only) |
| **Query Pattern** | In most queries | Rarely queried |

### Replication vs Sharding

| Feature | Replication | Sharding |
|---------|------------|----------|
| **Purpose** | High Availability + Reads | Write + Storage Scaling |
| **Data** | Full copy on each node | Split across nodes |
| **Writes** | Primary only | All shards |
| **Reads** | All nodes | Specific shard(s) |
| **Failover** | Automatic | Per-shard replica set |
| **Complexity** | Low | High |
| **Use When** | < 1TB, need uptime | > 1TB, need throughput |

### 5️⃣ INTERVIEW ANSWER

> *"**Replication** in MongoDB uses **Replica Sets** — a primary node handles all writes, and secondaries replicate data via the oplog for read scaling and automatic failover. If the primary crashes, a secondary is elected within ~10 seconds. I configure read preferences like `secondaryPreferred` to distribute read load. **Sharding** horizontally partitions data across multiple servers using a **shard key**. The cluster consists of **mongos** (query router), **config servers** (metadata), and **shards** (each a replica set). Shard key selection is critical — it needs high cardinality, even distribution, and alignment with query patterns. I use **hashed keys** for even distribution and **compound keys** for targeted queries. In practice, I start with replication for HA and add sharding only when a single server can't handle the write throughput or storage."*

📌 **One-liner:** Replication = copies for HA + reads | Sharding = split for writes + storage | Shard key = most critical decision

---
---

## 6. CAP Theorem

> 🏆 **Most asked distributed systems theory question!**

### 1️⃣ WHAT

CAP Theorem states ki koi bhi **distributed database** ek saath **sirf 2 out of 3** guarantees de sakti hai:

| Property | Full Form | Meaning |
|----------|-----------|---------|
| **C** | **Consistency** | Har read ko **latest write** milta hai (all nodes same data) |
| **A** | **Availability** | Har request ko **response** milta hai (no downtime) |
| **P** | **Partition Tolerance** | System **network failures** ke baad bhi kaam karta hai |

> **Key Insight:** Network partitions **inevitable** hain (P is mandatory) → real choice is **C vs A**

### 2️⃣ WHY

Distributed systems mein **network failures** hoti hain — cables cut, switches fail, latency spikes. Jab 2 nodes communicate nahi kar paate (partition), tumhe choose karna padta hai:

```
Network Partition:
┌──────────┐    ✂️ NETWORK CUT ✂️    ┌──────────┐
│  Node A  │ ──── X X X X X ──── │  Node B  │
│ (Delhi)  │                       │ (Mumbai) │
└──────────┘                       └──────────┘

User writes "balance = 5000" to Node A

Now user reads from Node B:

Option 1: Choose CONSISTENCY (CP)
  → Node B: "Sorry, I can't confirm latest data" → ERROR ❌
  → Consistent but NOT available

Option 2: Choose AVAILABILITY (AP)
  → Node B: "balance = 10000" (old data!) → STALE ✅
  → Available but NOT consistent

You CANNOT have both during a partition! 💀
```

### 3️⃣ HOW — CAP Combinations

```
┌─────────────────────────────────────────────────┐
│              CAP THEOREM TRIANGLE                │
│                                                  │
│              Consistency (C)                     │
│                 / \                              │
│                /   \                             │
│               / CP  \                            │
│              /       \                           │
│             /         \                          │
│            /    CA     \                         │
│           / (impossible \                        │
│          /  in distributed)                      │
│         /                 \                      │
│        /       AP          \                     │
│       /                     \                    │
│  Availability (A) ──── Partition (P)             │
│                                                  │
│  CP: Consistent + Partition Tolerant             │
│  AP: Available + Partition Tolerant              │
│  CA: Consistent + Available (single node only!)  │
└─────────────────────────────────────────────────┘
```

| Type | Choice | Behavior During Partition | Examples |
|------|--------|--------------------------|----------|
| **CP** | Consistency + Partition | Rejects reads/writes to stale nodes | MongoDB (default), HBase, Redis Cluster, ZooKeeper |
| **AP** | Availability + Partition | Serves stale data, syncs later | Cassandra, DynamoDB, CouchDB, Riak |
| **CA** | Consistency + Availability | No partition tolerance (single node!) | Single-node PostgreSQL, single-node MySQL |

### 4️⃣ CODE — Real-World Examples

```javascript
// ═══════════════════════════════════════════
// CP SYSTEM: MongoDB (Default)
// ═══════════════════════════════════════════
// During partition:
// - Primary can't reach majority → steps down
// - No new writes accepted until quorum restored
// - Data is ALWAYS consistent ✅
// - But temporarily unavailable ❌

// MongoDB: Write Concern controls consistency level
db.orders.insertOne(
    { total: 5000 },
    { writeConcern: { w: "majority", wtimeout: 5000 } }
    // w: "majority" = wait for majority of replicas to confirm
    // CP behavior: if majority not available → write fails
);

// Read Concern
db.orders.find().readConcern("majority");
// Only reads data confirmed by majority → consistent

// ═══════════════════════════════════════════
// AP SYSTEM: Cassandra / DynamoDB
// ═══════════════════════════════════════════
// During partition:
// - ALL nodes accept reads and writes ✅
// - Data might be stale on some nodes ❌
// - "Eventual consistency" — syncs when partition heals

// Cassandra: Consistency Level (tunable!)
// ONE: Fastest, least consistent (AP)
// QUORUM: Balanced
// ALL: Most consistent, slowest (CP)

// DynamoDB: Read modes
// Eventually Consistent (default) → AP, faster, cheaper
// Strongly Consistent → CP-like, slower, 2× cost
```

### CAP in Practice — It's Not Binary!

```
Real World: CAP is a SPECTRUM, not a switch!

┌─────────────────────────────────────────────┐
│  Consistency                                │
│  Strong ←──────────────────→ Eventual       │
│  (CP)                        (AP)          │
│                                              │
│  MongoDB:   ████████░░ (mostly CP)          │
│  Cassandra: ░░░███████ (mostly AP)          │
│  PostgreSQL:██████████ (CP, single node)    │
│  Redis:     █████░░░░░ (CP, cluster)        │
│  DynamoDB:  ░░████████ (tunable AP→CP)      │
└─────────────────────────────────────────────┘

Modern databases let you TUNE consistency per query!
```

### 5️⃣ INTERVIEW ANSWER

> *"The **CAP theorem** states that a distributed system can guarantee at most **two of three** properties: **Consistency** (every read returns the latest write), **Availability** (every request gets a response), and **Partition Tolerance** (system works despite network failures). Since network partitions are inevitable in distributed systems, the real choice is between **CP** and **AP**. **CP systems** like MongoDB (default) and HBase prioritize consistency — during a partition, they reject requests to stale nodes. **AP systems** like Cassandra and DynamoDB prioritize availability — they serve potentially stale data and reconcile later via eventual consistency. In practice, CAP is a **spectrum**, not a binary choice — modern databases like DynamoDB let you tune consistency per query. For financial systems I'd choose CP; for social media feeds, AP is acceptable."*

📌 **One-liner:** CAP = pick 2 of 3 | P is mandatory → choose C (MongoDB) vs A (Cassandra) | Spectrum, not binary

---
---

## 7. Full-Text Search in PostgreSQL

### 1️⃣ WHAT

Full-Text Search (FTS) PostgreSQL ka **built-in text search engine** hai jo documents mein **words, phrases, aur relevance-based ranking** search karta hai — `LIKE '%word%'` se **100-1000× faster**.

> **Analogy:** `LIKE` = book ke har page pe word dhundho 💀 | FTS = book ka index use karo ✅

### 2️⃣ WHY

| Method | Speed (1M rows) | Features |
|--------|----------------|----------|
| `LIKE '%search%'` | ~5 seconds 💀 | No index, no stemming, no ranking |
| `ILIKE '%search%'` | ~5 seconds 💀 | Case-insensitive but still slow |
| **PostgreSQL FTS** | **~5ms** ✅ | Index, stemming, ranking, synonyms |
| Elasticsearch | ~2ms ✅ | Best but separate infra needed |

**FTS Features:**
- ✅ **Stemming:** "running" matches "run"
- ✅ **Stop words:** Ignores "the", "is", "and"
- ✅ **Ranking:** Most relevant results first
- ✅ **Weighting:** Title matches > body matches
- ✅ **Phrase search:** "exact phrase"
- ✅ **Boolean:** AND, OR, NOT

### 3️⃣ HOW — FTS Pipeline

```
Text → Normalization → Tokenization → Stemming → tsvector
                                              ↓
Query → Normalization → Tokenization → Stemming → tsquery
                                              ↓
                                    tsvector @@ tsquery = MATCH!
                                              ↓
                                    ts_rank() = RELEVANCE SCORE
```

### 4️⃣ CODE

```sql
-- ═══════════════════════════════════════════
-- 1. BASIC FTS CONCEPTS
-- ═══════════════════════════════════════════

-- tsvector: Document representation (indexed)
SELECT to_tsvector('english', 'The quick brown fox is running quickly');
-- 'brown':3 'fox':4 'quick':2 'quickli':7 'run':6
-- Note: "the" removed (stop word), "running" → "run" (stemmed)

-- tsquery: Search query
SELECT to_tsquery('english', 'running & fox');
-- 'run' & 'fox'

-- Match!
SELECT to_tsvector('english', 'The fox is running')
    @@ to_tsquery('english', 'running & fox');
-- true ✅

-- ═══════════════════════════════════════════
-- 2. SETUP: Articles Table with FTS
-- ═══════════════════════════════════════════
CREATE TABLE articles (
    id SERIAL PRIMARY KEY,
    title VARCHAR(500) NOT NULL,
    body TEXT NOT NULL,
    author VARCHAR(100),
    category VARCHAR(50),
    published_at TIMESTAMP DEFAULT NOW(),

    -- Dedicated FTS column (pre-computed for speed!)
    search_vector tsvector
);

-- ═══════════════════════════════════════════
-- 3. POPULATE SEARCH VECTOR
-- ═══════════════════════════════════════════

-- Option A: Manual update
UPDATE articles SET search_vector =
    setweight(to_tsvector('english', COALESCE(title, '')), 'A') ||
    setweight(to_tsvector('english', COALESCE(body, '')), 'B') ||
    setweight(to_tsvector('english', COALESCE(author, '')), 'C');
-- Weights: A (highest) > B > C > D (lowest)
-- Title matches rank higher than body matches!

-- Option B: Auto-update with trigger
CREATE OR REPLACE FUNCTION articles_search_trigger()
RETURNS TRIGGER AS $$
BEGIN
    NEW.search_vector :=
        setweight(to_tsvector('english', COALESCE(NEW.title, '')), 'A') ||
        setweight(to_tsvector('english', COALESCE(NEW.body, '')), 'B') ||
        setweight(to_tsvector('english', COALESCE(NEW.author, '')), 'C');
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_articles_search
    BEFORE INSERT OR UPDATE ON articles
    FOR EACH ROW EXECUTE FUNCTION articles_search_trigger();

-- ═══════════════════════════════════════════
-- 4. GIN INDEX (Critical for Performance!)
-- ═══════════════════════════════════════════
CREATE INDEX idx_articles_search ON articles USING GIN(search_vector);

-- GIN vs GiST for FTS:
-- GIN: Faster reads, slower writes, larger index (best for search-heavy)
-- GiST: Slower reads, faster writes, smaller index (best for write-heavy)

-- ═══════════════════════════════════════════
-- 5. SEARCH QUERIES
-- ═══════════════════════════════════════════

-- Basic search
SELECT title, author
FROM articles
WHERE search_vector @@ to_tsquery('english', 'javascript & tutorial');

-- Phrase search (exact phrase!)
SELECT title
FROM articles
WHERE search_vector @@ phraseto_tsquery('english', 'learn javascript');

-- OR search
SELECT title
FROM articles
WHERE search_vector @@ to_tsquery('english', 'javascript | typescript');

-- NOT search
SELECT title
FROM articles
WHERE search_vector @@ to_tsquery('english', 'javascript & !python');

-- Prefix search
SELECT title
FROM articles
WHERE search_vector @@ to_tsquery('english', 'java:*');
-- Matches: javascript, java, javelin

-- ═══════════════════════════════════════════
-- 6. RANKING (Most Important!)
-- ═══════════════════════════════════════════
SELECT
    title,
    author,
    ts_rank(search_vector, query) AS rank,           -- Basic rank
    ts_rank_cd(search_vector, query) AS rank_cd      -- Cover density rank
FROM articles,
    to_tsquery('english', 'javascript & tutorial') AS query
WHERE search_vector @@ query
ORDER BY rank DESC
LIMIT 10;

-- With weights (title matches rank higher!)
SELECT
    title,
    ts_rank(search_vector, query, 0) AS rank
FROM articles,
    to_tsquery('english', 'javascript') AS query
WHERE search_vector @@ query
ORDER BY rank DESC;
-- Weights already
