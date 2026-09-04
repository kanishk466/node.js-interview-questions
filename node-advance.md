# 🚀 Node.js & Express.js — Interview Prep Guide (Advanced Level)

> **Level:** Advanced
> **Format:** What → Why → How → Example → Interview Answer
> **Prerequisite:** Basic + Intermediate Level (Express, Middleware, Streams, Clustering)

---

## 📑 Table of Contents

| # | Topic | Difficulty | Frequency |
|---|-------|-----------|-----------|
| 1 | [Rate Limiting](#1-rate-limiting-in-express) | ⭐⭐⭐ | 🔥🔥🔥🔥 |
| 2 | [Performance Optimization](#2-nodejs-performance-optimization) | ⭐⭐⭐⭐ | 🔥🔥🔥🔥 |
| 3 | [Event Loop Blocking](#3-preventing-event-loop-blocking) | ⭐⭐⭐⭐ | 🔥🔥🔥🔥 |
| 4 | [Graceful Shutdown](#4-graceful-shutdown) | ⭐⭐⭐ | 🔥🔥🔥 |
| 5 | [Child Processes](#5-child-processes-in-nodejs) | ⭐⭐⭐ | 🔥🔥🔥 |
| 6 | [Connection Pooling](#6-connection-pooling) | ⭐⭐⭐ | 🔥🔥🔥🔥 |
| 7 | [Security Best Practices](#7-security-best-practices) | ⭐⭐⭐⭐ | 🔥🔥🔥🔥🔥 |
| 8 | [Memory Leaks](#8-memory-leaks-in-nodejs) | ⭐⭐⭐⭐ | 🔥🔥🔥🔥 |

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

## 1. Rate Limiting in Express

### 1️⃣ WHAT

Rate limiting ek **traffic control mechanism** hai jo **ek client ke requests ko ek time window mein limit** karta hai — taaki API abuse, brute-force attacks, aur server overload se bacha ja sake.

> **Analogy:** ATM machine — ek din mein 5 withdrawals allowed. 6th attempt pe "Limit exceeded" message.

```
Without Rate Limiting:                With Rate Limiting:
                                      
Client: req req req req req req       Client: req req req ✅ ✅ ✅
Server: 💀 OVERLOADED!               Server: req ❌ 429 Too Many Requests
                                      Server: ✅ Healthy
```

### 2️⃣ WHY

| Threat | Without Rate Limiting | With Rate Limiting |
|--------|----------------------|-------------------|
| **Brute Force** | 1M password attempts | 5 attempts → locked |
| **DDoS** | Server crash | Requests dropped |
| **API Abuse** | Unlimited scraping | 100 req/min cap |
| **Cost Spike** | Cloud bill 💸 | Controlled usage |
| **Fair Usage** | One user hogs resources | Equal distribution |

### 3️⃣ HOW — Rate Limiting Strategies

```
┌─────────────────────────────────────────────────────┐
│           RATE LIMITING ALGORITHMS                    │
│                                                      │
│  1. Fixed Window                                     │
│     [00:00──────01:00] = 100 requests               │
│     [01:00──────02:00] = 100 requests (reset)       │
│     ⚠️ Edge problem: 100 at 00:59 + 100 at 01:00   │
│                                                      │
│  2. Sliding Window (Better!)                         │
│     Last 60 seconds = 100 requests (rolling)         │
│     ✅ Smooth, no edge problem                       │
│                                                      │
│  3. Token Bucket                                     │
│     Bucket fills at fixed rate, each req = 1 token   │
│     ✅ Allows bursts                                 │
│                                                      │
│  4. Leaky Bucket                                     │
│     Requests enter bucket, processed at fixed rate   │
│     ✅ Smooth output                                 │
└─────────────────────────────────────────────────────┘
```

### 4️⃣ CODE

```bash
npm install express-rate-limit
npm install rate-limit-redis  # For distributed/production
```

```javascript
const express = require("express");
const rateLimit = require("express-rate-limit");
const RedisStore = require("rate-limit-redis");
const Redis = require("ioredis");
const app = express();

// ═══════════════════════════════════════════
// 1. BASIC RATE LIMITER (In-Memory — Dev Only)
// ═══════════════════════════════════════════
const globalLimiter = rateLimit({
    windowMs: 15 * 60 * 1000,   // 15 minutes
    max: 100,                    // 100 requests per window per IP
    message: {
        status: 429,
        error: "Too many requests, please try again later"
    },
    standardHeaders: true,       // X-RateLimit-* headers
    legacyHeaders: false,        // Disable X-RateLimit-* old headers
    keyGenerator: (req) => {
        return req.ip;           // Rate limit per IP
    }
});

app.use(globalLimiter);  // Apply to ALL routes

// ═══════════════════════════════════════════
// 2. STRICT LIMITER (Auth Routes — Anti Brute Force)
// ═══════════════════════════════════════════
const authLimiter = rateLimit({
    windowMs: 15 * 60 * 1000,
    max: 5,                      // Only 5 login attempts!
    message: {
        status: 429,
        error: "Too many login attempts. Try again in 15 minutes."
    },
    skipSuccessfulRequests: true // Don't count successful logins
});

app.post("/api/login", authLimiter, loginHandler);
app.post("/api/register", authLimiter, registerHandler);
app.post("/api/forgot-password", authLimiter, forgotPasswordHandler);

// ═══════════════════════════════════════════
// 3. API TIER LIMITER (Different Plans)
// ═══════════════════════════════════════════
const apiLimiter = rateLimit({
    windowMs: 60 * 1000,         // 1 minute
    max: async (req) => {
        // Dynamic limit based on user plan
        if (!req.user) return 10;           // Free: 10/min
        if (req.user.plan === "pro") return 100;   // Pro: 100/min
        if (req.user.plan === "enterprise") return 1000; // Enterprise: 1000/min
        return 10;
    },
    keyGenerator: (req) => {
        return req.user?.id || req.ip;  // Per user or per IP
    }
});

app.use("/api/", apiLimiter);

// ═══════════════════════════════════════════
// 4. PRODUCTION: Redis-Backed (Distributed!)
// ═══════════════════════════════════════════
const redisClient = new Redis(process.env.REDIS_URL);

const distributedLimiter = rateLimit({
    windowMs: 15 * 60 * 1000,
    max: 100,
    store: new RedisStore({
        sendCommand: (...args) => redisClient.call(...args),
        prefix: "rl:"  // Redis key prefix
    }),
    message: { error: "Rate limit exceeded" }
});

// Why Redis?
// ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
// │  Server 1    │  │  Server 2    │  │  Server 3    │
// │  (in-memory  │  │  (in-memory  │  │  (in-memory  │
// │   = separate │  │   = separate │  │   = separate │
// │   counts! ❌)│  │   counts! ❌)│  │   counts! ❌)│
// └──────┬───────┘  └──────┬───────┘  └──────┬───────┘
//        └─────────────┬────┴─────────────────┘
//                      ▼
//              ┌──────────────┐
//              │    Redis     │
//              │ Shared count │
//              │ across all! ✅│
//              └──────────────┘

app.use("/api/", distributedLimiter);

// ═══════════════════════════════════════════
// 5. CUSTOM RATE LIMITER (From Scratch)
// ═══════════════════════════════════════════
class RateLimiter {
    constructor({ windowMs, max }) {
        this.windowMs = windowMs;
        this.max = max;
        this.hits = new Map();  // IP → { count, resetTime }

        // Cleanup old entries every minute
        setInterval(() => this.cleanup(), 60 * 1000);
    }

    middleware() {
        return (req, res, next) => {
            const key = req.ip;
            const now = Date.now();
            const record = this.hits.get(key);

            if (!record || now > record.resetTime) {
                // New window
                this.hits.set(key, { count: 1, resetTime: now + this.windowMs });
                return next();
            }

            if (record.count >= this.max) {
                const retryAfter = Math.ceil((record.resetTime - now) / 1000);
                res.set("Retry-After", retryAfter);
                return res.status(429).json({
                    error: "Rate limit exceeded",
                    retryAfter
                });
            }

            record.count++;
            next();
        };
    }

    cleanup() {
        const now = Date.now();
        for (const [key, record] of this.hits) {
            if (now > record.resetTime) this.hits.delete(key);
        }
    }
}

const customLimiter = new RateLimiter({ windowMs: 60000, max: 50 });
app.use("/api/", customLimiter.middleware());

app.listen(3000);
```

### 5️⃣ INTERVIEW ANSWER

> *"I implement rate limiting in Express using the `express-rate-limit` package with different strategies for different routes. A **global limiter** (e.g., 100 requests per 15 minutes per IP) protects the entire API. **Stricter limiters** (e.g., 5 attempts per 15 minutes) protect authentication routes against brute-force attacks. In production with multiple server instances, I use a **Redis-backed store** so rate counts are shared across all nodes — in-memory stores don't work in distributed setups. I also implement **dynamic limits** based on user subscription tiers. Key headers like `Retry-After` and `X-RateLimit-Remaining` help clients handle limits gracefully. For critical systems, I'd use the **sliding window** or **token bucket** algorithm for smoother rate limiting."*

📌 **One-liner:** Rate limit = requests per time window per client | Redis for distributed | Strict on auth routes

---

---

## 2. Node.js Performance Optimization

### 1️⃣ WHAT

Performance optimization Node.js application ko **faster, more efficient, aur scalable** banane ka systematic process hai — covering CPU, memory, I/O, network, aur code-level improvements.

### 2️⃣ WHY

| Metric | Unoptimized | Optimized | Impact |
|--------|------------|-----------|--------|
| Response Time | 500ms | 50ms | 10× faster |
| Throughput | 1K req/s | 10K req/s | 10× more users |
| Memory | 1GB | 200MB | 5× cheaper infra |
| CPU Usage | 90% | 30% | No throttling |

### 3️⃣ HOW — Optimization Layers

```
┌─────────────────────────────────────────────────┐
│          PERFORMANCE OPTIMIZATION STACK          │
│                                                  │
│  Layer 1: CODE LEVEL                             │
│  ├── Async/Await (non-blocking)                  │
│  ├── Avoid sync methods in routes                │
│  ├── Efficient algorithms (O(n) vs O(n²))        │
│  └── Lazy loading modules                        │
│                                                  │
│  Layer 2: CACHING                                │
│  ├── In-memory (Map, node-cache)                 │
│  ├── Redis (distributed)                         │
│  ├── HTTP Cache (ETag, Cache-Control)            │
│  └── CDN (static assets)                         │
│                                                  │
│  Layer 3: DATABASE                               │
│  ├── Connection pooling                          │
│  ├── Indexes                                     │
│  ├── Query optimization                          │
│  └── Pagination                                  │
│                                                  │
│  Layer 4: ARCHITECTURE                           │
│  ├── Clustering (PM2)                            │
│  ├── Load Balancing (Nginx)                      │
│  ├── Microservices                               │
│  └── Message Queues (Bull, RabbitMQ)             │
│                                                  │
│  Layer 5: INFRASTRUCTURE                         │
│  ├── Gzip/Brotli compression                     │
│  ├── HTTP/2                                      │
│  ├── Keep-Alive connections                      │
│  └── Containerization (Docker)                   │
└─────────────────────────────────────────────────┘
```

### 4️⃣ CODE

```javascript
const express = require("express");
const compression = require("compression");
const helmet = require("helmet");
const Redis = require("ioredis");
const app = express();

// ═══════════════════════════════════════════
// 1. COMPRESSION (Reduce Response Size 60-80%)
// ═══════════════════════════════════════════
app.use(compression({
    level: 6,           // 1-9 (6 = best balance)
    threshold: 1024     // Only compress > 1KB
}));

// ═══════════════════════════════════════════
// 2. CACHING (Most Impactful!)
// ═══════════════════════════════════════════
const redis = new Redis(process.env.REDIS_URL);

// Cache middleware
function cache(duration) {
    return async (req, res, next) => {
        const key = `cache:${req.originalUrl}`;

        try {
            const cached = await redis.get(key);
            if (cached) {
                return res.json(JSON.parse(cached));  // ⚡ Instant!
            }
        } catch (err) {
            console.error("Cache error:", err);
        }

        // Override res.json to cache the response
        const originalJson = res.json.bind(res);
        res.json = (data) => {
            redis.setex(key, duration, JSON.stringify(data));
            return originalJson(data);
        };

        next();
    };
}

// Usage: Cache for 60 seconds
app.get("/api/products", cache(60), async (req, res) => {
    const products = await Product.find();  // DB call only if cache miss
    res.json(products);
});

// ═══════════════════════════════════════════
// 3. DATABASE OPTIMIZATION
// ═══════════════════════════════════════════
// ❌ BAD: N+1 Query Problem
app.get("/api/users-bad", async (req, res) => {
    const users = await User.find();
    for (const user of users) {
        user.orders = await Order.find({ userId: user._id });  // N queries!
    }
    res.json(users);
});

// ✅ GOOD: Single Query with Population
app.get("/api/users-good", async (req, res) => {
    const users = await User.find()
        .populate("orders")    // 1 query with JOIN
        .select("name email")  // Only needed fields
        .lean();               // Plain JS objects (faster!)
    res.json(users);
});

// ✅ Pagination (Don't send 1M records!)
app.get("/api/products", async (req, res) => {
    const page = parseInt(req.query.page) || 1;
    const limit = Math.min(parseInt(req.query.limit) || 20, 100);
    const skip = (page - 1) * limit;

    const [products, total] = await Promise.all([
        Product.find().skip(skip).limit(limit).lean(),
        Product.countDocuments()
    ]);

    res.json({
        data: products,
        pagination: { page, limit, total, pages: Math.ceil(total / limit) }
    });
});

// ═══════════════════════════════════════════
// 4. ASYNC PARALLEL EXECUTION
// ═══════════════════════════════════════════
// ❌ Sequential: 3s + 2s + 1s = 6s
app.get("/dashboard-slow", async (req, res) => {
    const users = await fetchUsers();
    const orders = await fetchOrders();
    const stats = await fetchStats();
    res.json({ users, orders, stats });
});

// ✅ Parallel: max(3s, 2s, 1s) = 3s
app.get("/dashboard-fast", async (req, res) => {
    const [users, orders, stats] = await Promise.all([
        fetchUsers(),
        fetchOrders(),
        fetchStats()
    ]);
    res.json({ users, orders, stats });
});

// ═══════════════════════════════════════════
// 5. STREAMING LARGE RESPONSES
// ═══════════════════════════════════════════
const fs = require("fs");

// ❌ Loads entire file in memory
app.get("/download-bad", async (req, res) => {
    const data = await fs.promises.readFile("huge-file.csv");
    res.send(data);  // 500MB in RAM! 💀
});

// ✅ Streams: ~64KB memory
app.get("/download-good", (req, res) => {
    const stream = fs.createReadStream("huge-file.csv");
    stream.pipe(res);
});

// ═══════════════════════════════════════════
// 6. HTTP KEEP-ALIVE & CONNECTION TUNING
// ═══════════════════════════════════════════
const http = require("http");
const server = http.createServer(app);

server.keepAliveTimeout = 65000;     // 65s (higher than LB timeout)
server.headersTimeout = 66000;       // Slightly higher
server.maxConnections = 10000;       // Max concurrent

// ═══════════════════════════════════════════
// 7. PROFILING (Find Bottlenecks!)
// ═══════════════════════════════════════════
// Run: node --prof server.js
// Then: node --prof-process isolate-*.log > profile.txt

// Or use clinic.js:
// npx clinic doctor -- node server.js
// npx clinic flame -- node server.js

// ═══════════════════════════════════════════
// 8. CLUSTERING (Use All CPU Cores)
// ═══════════════════════════════════════════
// pm2 start server.js -i max --no-daemon

app.listen(3000);
```

### Performance Checklist

| Area | Action | Impact |
|------|--------|--------|
| **Code** | Use async everywhere | ⭐⭐⭐⭐⭐ |
| **Cache** | Redis for hot data | ⭐⭐⭐⭐⭐ |
| **DB** | Indexes + lean() + populate | ⭐⭐⭐⭐⭐ |
| **DB** | Pagination | ⭐⭐⭐⭐ |
| **Network** | Gzip compression | ⭐⭐⭐⭐ |
| **Network** | CDN for static | ⭐⭐⭐⭐ |
| **Memory** | Streams for large files | ⭐⭐⭐⭐ |
| **CPU** | Clustering (PM2) | ⭐⭐⭐⭐ |
| **CPU** | Offload to workers | ⭐⭐⭐ |
| **Architecture** | Message queues | ⭐⭐⭐ |

### 5️⃣ INTERVIEW ANSWER

> *"I optimize Node.js applications across multiple layers. At the **code level**, I ensure all I/O is async, use `Promise.all()` for parallel operations, and avoid synchronous methods in request handlers. **Caching** gives the biggest ROI — I use Redis for frequently accessed data and HTTP cache headers for static responses. At the **database level**, I add proper indexes, use `.lean()` in Mongoose, avoid N+1 queries with population, and always implement pagination. For **large data**, I use streams instead of loading everything into memory. I enable **gzip compression**, use **clustering via PM2** to utilize all CPU cores, and set proper **keep-alive timeouts**. Finally, I profile with tools like **clinic.js** and Node's built-in `--prof` to identify actual bottlenecks rather than guessing."*

📌 **One-liner:** Cache + Async + DB indexes + Streams + Clustering + Compression = Performance

---

---

## 3. Preventing Event Loop Blocking

### 1️⃣ WHAT

Event loop blocking tab hota hai jab koi **synchronous, CPU-intensive operation** main thread ko itna busy rakhe ki **doosre requests, callbacks, aur timers process nahi ho paate**.

> **Rule of thumb:** Agar koi operation **~10ms se zyada** le raha hai main thread pe → it's blocking!

```
Healthy Event Loop:                    Blocked Event Loop:
                                       
Tick: [req][req][timer][req][I/O]      Tick: [██████████████████][req waiting...]
      ~2ms each ✅                           500ms blocking! 💀
      All requests served fast               Other requests TIMEOUT!
```

### 2️⃣ WHY

Node.js **single-threaded** hai. Ek blocking operation = **ALL users affected**:

| Blocking Operation | Duration | Impact |
|-------------------|----------|--------|
| `JSON.parse(hugeString)` | 100ms+ | All requests delayed |
| `crypto.pbkdf2Sync()` | 200ms+ | Server freeze |
| Large array `.sort()` | 50ms+ | Latency spike |
| `fs.readFileSync(bigFile)` | 500ms+ | Complete freeze |
| Regex on huge string | Variable | ReDoS attack! |
| Image processing | 1s+ | Server dead |

### 3️⃣ HOW — Detection + Prevention

```
┌─────────────────────────────────────────────────┐
│       EVENT LOOP BLOCKING STRATEGIES             │
│                                                  │
│  DETECTION:                                      │
│  ├── Monitor event loop lag (perf_hooks)         │
│  ├── clinic.js / 0x profiler                     │
│  └── APM tools (New Relic, Datadog)              │
│                                                  │
│  PREVENTION:                                     │
│  ├── 1. Use async versions of all I/O            │
│  ├── 2. Offload CPU work to Worker Threads       │
│  ├── 3. Break large tasks into chunks            │
│  ├── 4. Use streams for large data               │
│  ├── 5. Cache expensive computations             │
│  └── 6. Use child processes for heavy tasks      │
└─────────────────────────────────────────────────┘
```

### 4️⃣ CODE

```javascript
const express = require("express");
const { Worker } = require("worker_threads");
const { performance, PerformanceObserver } = require("perf_hooks");
const app = express();

// ═══════════════════════════════════════════
// 1. DETECT EVENT LOOP LAG (Monitoring!)
// ═══════════════════════════════════════════
function monitorEventLoopLag() {
    let lastCheck = Date.now();

    setInterval(() => {
        const now = Date.now();
        const lag = now - lastCheck - 1000;  // Expected: 1000ms

        if (lag > 100) {
            console.warn(`⚠️ Event loop lag: ${lag}ms`);
            // Alert team, log to APM, etc.
        }

        lastCheck = now;
    }, 1000);
}

monitorEventLoopLag();

// More precise: perf_hooks
const obs = new PerformanceObserver((list) => {
    const entry = list.getEntries()[0];
    if (entry.duration > 50) {
        console.warn(`⚠️ Slow event loop: ${entry.duration.toFixed(2)}ms`);
    }
});
obs.observe({ entryTypes: ["measure"] });

// ═══════════════════════════════════════════
// 2. ❌ BLOCKING vs ✅ NON-BLOCKING
// ═══════════════════════════════════════════
const fs = require("fs");
const crypto = require("crypto");

// ❌ BLOCKING: Sync crypto
app.get("/hash-bad", (req, res) => {
    const hash = crypto.pbkdf2Sync("password", "salt", 100000, 64, "sha512");
    res.send(hash.toString("hex"));  // Blocks thread for ~200ms!
});

// ✅ NON-BLOCKING: Async crypto
app.get("/hash-good", (req, res) => {
    crypto.pbkdf2("password", "salt", 100000, 64, "sha512", (err, hash) => {
        res.send(hash.toString("hex"));  // Thread free during computation!
    });
});

// ═══════════════════════════════════════════
// 3. WORKER THREADS (CPU-Intensive Tasks)
// ═══════════════════════════════════════════

// worker.js — Separate file
const { parentPort, workerData } = require("worker_threads");

function heavyComputation(data) {
    let result = 0;
    for (let i = 0; i < data.iterations; i++) {
        result += Math.sqrt(i) * Math.sin(i);
    }
    return result;
}

parentPort.postMessage(heavyComputation(workerData));

// server.js — Main thread
function runWorker(data) {
    return new Promise((resolve, reject) => {
        const worker = new Worker("./worker.js", { workerData: data });
        worker.on("message", resolve);
        worker.on("error", reject);
        worker.on("exit", (code) => {
            if (code !== 0) reject(new Error(`Worker exited with code ${code}`));
        });
    });
}

// ❌ Blocks main thread
app.get("/compute-bad", (req, res) => {
    let result = 0;
    for (let i = 0; i < 1e9; i++) result += Math.sqrt(i);  // 💀 2+ seconds!
    res.json({ result });
});

// ✅ Main thread stays free
app.get("/compute-good", async (req, res) => {
    const result = await runWorker({ iterations: 1e9 });
    res.json({ result });  // Worker did the heavy lifting
});

// ═══════════════════════════════════════════
// 4. CHUNKING (Break Large Tasks)
// ═══════════════════════════════════════════
// ❌ Process 1M items at once
function processAll(items) {
    return items.map(item => heavyTransform(item));  // Blocks!
}

// ✅ Process in chunks with setImmediate
async function processInChunks(items, chunkSize = 1000) {
    const results = [];

    for (let i = 0; i < items.length; i += chunkSize) {
        const chunk = items.slice(i, i + chunkSize);
        results.push(...chunk.map(item => heavyTransform(item)));

        // Yield to event loop between chunks!
        await new Promise(resolve => setImmediate(resolve));
    }

    return results;
}

// ═══════════════════════════════════════════
// 5. STREAMS FOR LARGE JSON
// ═══════════════════════════════════════════
const JSONStream = require("JSONStream");

// ❌ Parse 500MB JSON in memory
app.get("/data-bad", async (req, res) => {
    const raw = await fs.promises.readFile("huge.json", "utf-8");
    const data = JSON.parse(raw);  // 💀 Memory spike + CPU block!
    res.json(data);
});

// ✅ Stream-parse JSON
app.get("/data-good", (req, res) => {
    res.setHeader("Content-Type", "application/json");
    res.write("[");

    const stream = fs.createReadStream("huge.json")
        .pipe(JSONStream.parse("*"));

    let first = true;
    stream.on("data", (item) => {
        if (!first) res.write(",");
        res.write(JSON.stringify(item));
        first = false;
    });

    stream.on("end", () => res.end("]"));
});

// ═══════════════════════════════════════════
// 6. ReDoS PREVENTION (Regex Safety)
// ═══════════════════════════════════════════
// ❌ Vulnerable regex (exponential backtracking)
const badRegex = /^(a+)+$/;
// Input: "aaaaaaaaaaaaaaaaaaaaaaaaaaaa!" → HANGS FOREVER!

// ✅ Safe regex / use validator library
const validator = require("validator");
app.post("/validate", (req, res) => {
    if (!validator.isEmail(req.body.email)) {
        return res.status(400).json({ error: "Invalid email" });
    }
    res.json({ valid: true });
});

app.listen(3000);
```

### 5️⃣ INTERVIEW ANSWER

> *"Event loop blocking occurs when synchronous or CPU-intensive operations prevent Node.js from processing other requests. I prevent this through several strategies: First, I **monitor event loop lag** using `perf_hooks` or APM tools to detect issues early. I always use **async versions** of I/O and crypto operations. For CPU-heavy tasks like image processing or complex calculations, I offload to **Worker Threads** so the main thread stays responsive. For large data processing, I break work into **chunks with `setImmediate`** to yield to the event loop between batches. I use **streams** instead of loading large files into memory. I also guard against **ReDoS attacks** by using safe regex patterns or validation libraries. The key principle: no single operation on the main thread should take more than ~10ms."*

📌 **One-liner:** Monitor lag → async I/O → Worker Threads for CPU → chunk large tasks → streams for data

---

---

## 4. Graceful Shutdown

### 1️⃣ WHAT

Graceful shutdown ka matlab hai application ko **safely band karna** — bina active requests drop kiye, bina database connections corrupt kiye, bina data loss ke.

> **Analogy:** Restaurant closing — naye customers mat lo, jo andar hain unka order complete karo, kitchen saaf karo, phir lock karo.

```
❌ Hard Shutdown (SIGKILL / Ctrl+C):
   Request in progress → 💀 DROPPED
   DB write halfway → 💀 CORRUPTED
   WebSocket clients → 💀 DISCONNECTED
   Cache unsaved → 💀 LOST

✅ Graceful Shutdown:
   1. Stop accepting NEW connections
   2. Wait for ACTIVE requests to finish
   3. Close DB connections
   4. Flush logs & caches
   5. Exit cleanly
```

### 2️⃣ WHY

| Scenario | Without Graceful Shutdown | With Graceful Shutdown |
|----------|--------------------------|----------------------|
| Deployment | Active users get 502 | Requests complete |
| Container restart | Data corruption | Clean state |
| Scaling down | Dropped connections | Drained connections |
| PM2 restart | Lost in-flight data | Zero downtime |

### 3️⃣ HOW — Shutdown Sequence

```
Signal Received (SIGTERM / SIGINT)
         │
         ▼
┌─────────────────────┐
│ 1. Stop accepting   │  server.close()
│    new connections   │  (existing connections stay alive)
└─────────┬───────────┘
          ▼
┌─────────────────────┐
│ 2. Wait for active  │  Timeout: 30s
│    requests to end   │  (force close if exceeded)
└─────────┬───────────┘
          ▼
┌─────────────────────┐
│ 3. Close DB         │  mongoose.disconnect()
│    connections       │  redis.quit()
└─────────┬───────────┘
          ▼
┌─────────────────────┐
│ 4. Flush logs,      │  winston.flush()
│    caches, queues   │  bull.close()
└─────────┬───────────┘
          ▼
┌─────────────────────┐
│ 5. process.exit(0)  │  Clean exit ✅
└─────────────────────┘
```

### 4️⃣ CODE

```javascript
const express = require("express");
const http = require("http");
const mongoose = require("mongoose");
const Redis = require("ioredis");

const app = express();
const server = http.createServer(app);

const redis = new Redis(process.env.REDIS_URL);
const SHUTDOWN_TIMEOUT = 30000;  // 30 seconds

// ═══════════════════════════════════════════
// TRACK ACTIVE CONNECTIONS
// ═══════════════════════════════════════════
const activeConnections = new Set();

server.on("connection", (socket) => {
    activeConnections.add(socket);

    socket.on("close", () => {
        activeConnections.delete(socket);
    });
});

// ═══════════════════════════════════════════
// GRACEFUL SHUTDOWN FUNCTION
// ═══════════════════════════════════════════
async function gracefulShutdown(signal) {
    console.log(`\n🛑 ${signal} received. Starting graceful shutdown...`);

    // Step 1: Stop accepting new connections
    server.close(() => {
        console.log("✅ HTTP server closed (no new connections)");
    });

    // Step 2: Force close after timeout
    const forceCloseTimer = setTimeout(() => {
        console.error("⚠️ Forced shutdown after timeout!");
        process.exit(1);
    }, SHUTDOWN_TIMEOUT);

    // Step 3: Wait for active requests to finish
    if (activeConnections.size > 0) {
        console.log(`⏳ Waiting for ${activeConnections.size} active connections...`);

        // Close idle connections immediately
        for (const socket of activeConnections) {
            socket.end();  // Allow current request to finish
        }
    }

    try {
        // Step 4: Close database connections
        await mongoose.disconnect();
        console.log("✅ MongoDB disconnected");

        // Step 5: Close Redis
        await redis.quit();
        console.log("✅ Redis disconnected");

        // Step 6: Close message queues (if any)
        // await bullQueue.close();
        // console.log("✅ Queue closed");

        // Step 7: Flush logs
        // logger.flush();
        console.log("✅ Logs flushed");

        clearTimeout(forceCloseTimer);
        console.log("🎉 Graceful shutdown complete!");
        process.exit(0);

    } catch (err) {
        console.error("❌ Error during shutdown:", err);
        process.exit(1);
    }
}

// ═══════════════════════════════════════════
// LISTEN FOR SHUTDOWN SIGNALS
// ═══════════════════════════════════════════

// SIGTERM: Docker, Kubernetes, PM2, kill command
process.on("SIGTERM", () => gracefulShutdown("SIGTERM"));

// SIGINT: Ctrl+C in terminal
process.on("SIGINT", () => gracefulShutdown("SIGINT"));

// Unhandled errors (don't crash silently!)
process.on("uncaughtException", (err) => {
    console.error("💥 Uncaught Exception:", err);
    gracefulShutdown("UNCAUGHT_EXCEPTION");
});

process.on("unhandledRejection", (reason) => {
    console.error("💥 Unhandled Rejection:", reason);
    gracefulShutdown("UNHANDLED_REJECTION");
});

// ═══════════════════════════════════════════
// DOCKER / K8S HEALTH CHECK (Bonus)
// ═══════════════════════════════════════════
let isShuttingDown = false;

app.get("/health", (req, res) => {
    if (isShuttingDown) {
        return res.status(503).json({ status: "shutting_down" });
    }
    res.json({ status: "healthy", uptime: process.uptime() });
});

// Modified shutdown to set flag
const originalShutdown = gracefulShutdown;
gracefulShutdown = async (signal) => {
    isShuttingDown = true;  // Health check returns 503
    await originalShutdown(signal);
};

// ═══════════════════════════════════════════
// START SERVER
// ═══════════════════════════════════════════
async function start() {
    await mongoose.connect(process.env.MONGODB_URI);
    console.log("✅ MongoDB connected");

    server.listen(3000, () => {
        console.log("🚀 Server running on port 3000");
    });
}

start();
```

### 5️⃣ INTERVIEW ANSWER

> *"Graceful shutdown ensures the application closes **without dropping active requests or corrupting data**. When a `SIGTERM` or `SIGINT` signal is received, I first call `server.close()` to **stop accepting new connections** while letting existing ones complete. I track active connections using the server's `connection` event and set a **force-close timeout** (e.g., 30 seconds) as a safety net. Then I sequentially close **database connections** (MongoDB, Redis), flush **message queues and logs**, and finally call `process.exit(0)`. I also listen for `uncaughtException` and `unhandledRejection` to trigger the same shutdown flow. In Kubernetes environments, I add a `/health` endpoint that returns **503 during shutdown** so the load balancer stops routing traffic to the dying pod."*

📌 **One-liner:** SIGTERM → stop new connections → drain active → close DB/Redis → flush → exit(0)

---

---

## 5. Child Processes in Node.js

### 1️⃣ WHAT

Child processes Node.js ko **OS-level commands aur scripts execute** karne ki ability dete hain — ya toh **separate Node.js processes** ya **system commands** (like `ls`, `grep`, `ffmpeg`).

> **4 methods** hain child process create karne ke:

| Method | Use Case | Communication | Shell? |
|--------|----------|--------------|--------|
| `exec()` | Small shell commands | Buffer (callback) | ✅ Yes |
| `execFile()` | Executable files | Buffer (callback) | ❌ No (safer) |
| `spawn()` | Long-running processes | Streams | ❌ No (optional) |
| `fork()` | Node.js child processes | IPC channel | ❌ No |

### 2️⃣ WHY

| Scenario | Why Child Process? |
|----------|-------------------|
| Image/Video processing | `ffmpeg`, `sharp` CLI |
| PDF generation | `wkhtmltopdf` |
| Git operations | `git clone`, `git commit` |
| System monitoring | `top`, `df`, `free` |
| Heavy computation | Isolate from main process |
| Running Python/Java scripts | Cross-language integration |

### 3️⃣ HOW — Architecture

```
Parent Process (Node.js Server)
     │
     ├── fork() ──→ Child Node.js Process (IPC channel)
     │                  ├── process.on("message")
     │                  └── process.send()
     │
     ├── spawn() ─→ OS Process (streams: stdin/stdout/stderr)
     │                  ├── child.stdout.on("data")
     │                  └── child.stdin.write()
     │
     ├── exec() ──→ Shell Command (buffered output)
     │                  └── callback(err, stdout, stderr)
     │
     └── execFile()→ Executable (buffered, no shell)
                        └── callback(err, stdout, stderr)
```

### 4️⃣ CODE

```javascript
const { exec, execFile, spawn, fork } = require("child_process");
const path = require("path");
const express = require("express");
const app = express();

// ═══════════════════════════════════════════
// 1. exec() — Shell Commands (Small Output)
// ═══════════════════════════════════════════
app.get("/system-info", (req, res) => {
    // ⚠️ Output buffered in memory (max 1MB default)
    exec("uname -a && df -h && free -m", (err, stdout, stderr) => {
        if (err) {
            return res.status(500).json({ error: stderr });
        }
        res.type("text/plain").send(stdout);
    });
});

// ⚠️ SECURITY: Never pass user input to exec()!
// ❌ exec(`ls ${req.query.dir}`) → COMMAND INJECTION!
// ✅ Use execFile() or sanitize input

// ═══════════════════════════════════════════
// 2. execFile() — Safer (No Shell Parsing)
// ═══════════════════════════════════════════
app.get("/git-status", (req, res) => {
    // No shell = no injection risk ✅
    execFile("git", ["status", "--short"], (err, stdout) => {
        if (err) return res.status(500).json({ error: err.message });
        res.json({ status: stdout.trim().split("\n") });
    });
});

// ═══════════════════════════════════════════
// 3. spawn() — Long-Running / Large Output
// ═══════════════════════════════════════════
app.get("/logs", (req, res) => {
    res.setHeader("Content-Type", "text/plain");

    // Stream output — memory efficient!
    const tail = spawn("tail", ["-f", "/var/log/app.log"]);

    tail.stdout.on("data", (chunk) => {
        res.write(chunk);  // Stream to client in real-time
    });

    tail.stderr.on("data", (chunk) => {
        console.error("Tail error:", chunk.toString());
    });

    tail.on("close", (code) => {
        res.end(`\nProcess exited with code ${code}`);
    });

    // Kill child when client disconnects
    req.on("close", () => {
        tail.kill();
    });
});

// ═══════════════════════════════════════════
// 4. spawn() — Video Processing Example
// ═══════════════════════════════════════════
app.post("/convert-video", (req, res) => {
    const input = "input.mp4";
    const output = "output.webm";

    const ffmpeg = spawn("ffmpeg", [
        "-i", input,
        "-c:v", "libvpx-vp9",
        "-b:v", "1M",
        "-c:a", "libopus",
        output
    ]);

    let progress = "";

    ffmpeg.stderr.on("data", (data) => {
        progress += data.toString();
        // Parse progress from ffmpeg output
    });

    ffmpeg.on("close", (code) => {
        if (code === 0) {
            res.json({ success: true, output });
        } else {
            res.status(500).json({ error: "Conversion failed" });
        }
    });
});

// ═══════════════════════════════════════════
// 5. fork() — Node.js Child (IPC!)
// ═══════════════════════════════════════════

// heavy-worker.js (separate file)
process.on("message", (msg) => {
    if (msg.type === "COMPUTE") {
        let result = 0;
        for (let i = 0; i < msg.iterations; i++) {
            result += Math.sqrt(i) * Math.sin(i);
        }
        process.send({ type: "RESULT", result });
    }
});

// server.js
app.get("/compute", (req, res) => {
    const child = fork(path.join(__dirname, "heavy-worker.js"));

    child.send({ type: "COMPUTE", iterations: 1e8 });

    child.on("message", (msg) => {
        if (msg.type === "RESULT") {
            res.json({ result: msg.result });
            child.kill();  // Cleanup
        }
    });

    child.on("error", (err) => {
        res.status(500).json({ error: err.message });
    });
});

// ═══════════════════════════════════════════
// 6. PROMISE WRAPPER (Modern Usage)
// ═══════════════════════════════════════════
const { promisify } = require("util");
const execAsync = promisify(exec);

app.get("/disk-space", async (req, res) => {
    try {
        const { stdout } = await execAsync("df -h /");
        res.json({ disk: stdout.trim() });
    } catch (err) {
        res.status(500).json({ error: err.message });
    }
});

app.listen(3000);
```

### Method Selection Guide

```
Need to run a command?
    │
    ├── Small output, simple command?
    │     └── exec() / execAsync()
    │
    ├── User input involved? (Security!)
    │     └── execFile() (no shell injection)
    │
    ├── Large output / long-running?
    │     └── spawn() (streams)
    │
    └── Another Node.js script?
          └── fork() (IPC channel)
```

### 5️⃣ INTERVIEW ANSWER

> *"Node.js provides four child process methods: `exec()` for simple shell commands with buffered output, `execFile()` for running executables **without a shell** (safer against injection), `spawn()` for long-running processes with **streamed output**, and `fork()` for spawning **Node.js child processes** with a built-in IPC channel. I use `spawn()` for tasks like video processing with ffmpeg where output is large and continuous. I use `fork()` to offload CPU-intensive Node.js computations to a separate process while communicating via `process.send()` and `process.on('message')`. A critical security point: I **never pass user input directly to `exec()`** to prevent command injection — I use `execFile()` with an argument array instead. I also wrap these in Promises for cleaner async/await usage."*

📌 **One-liner:** exec = shell (small) | execFile = safe | spawn = streams (large) | fork = Node IPC

---

---

## 6. Connection Pooling

### 1️⃣ WHAT

Connection pooling ek **technique** hai jismein database connections ko **pre-create** karke ek "pool" mein rakha jata hai — har request ke liye naya connection banane aur destroy karne ke bajaye, **existing connection reuse** kiya jata hai.

> **Analogy:** Taxi stand — har customer ke liye nayi taxi manufacture mat karo, available taxi use karo, kaam hone pe wapas stand pe bhejo.

```
Without Pooling:                          With Pooling:
                                          
Request 1 → Connect (200ms)               Pool: [Conn1][Conn2][Conn3][Conn4]
           → Query (10ms)                         ↑
           → Disconnect (50ms)             Request 1 → Grab Conn1 (0ms!)
           → Total: 260ms 💀                          → Query (10ms)
                                                      → Return Conn1 (0ms)
Request 2 → Connect (200ms)                           → Total: 10ms ✅
           → Query (10ms)
           → Disconnect (50ms)             Request 2 → Grab Conn2 (0ms!)
           → Total: 260ms 💀                          → ...
```

### 2️⃣ WHY

| Metric | Without Pool | With Pool |
|--------|-------------|-----------|
| Connection Time | ~200ms per request | ~0ms (reused) |
| Max Connections | OS limit hit fast | Controlled (e.g., 20) |
| Memory | High (many sockets) | Low (fixed pool) |
| DB Load | Connection storm | Stable |
| Throughput | Low | High |

### 3️⃣ HOW — Pool Lifecycle

```
┌─────────────────────────────────────┐
│         CONNECTION POOL              │
│                                      │
│  ┌──────┐ ┌──────┐ ┌──────┐        │
│  │Idle  │ │Idle  │ │Busy  │ ← Req 1│
│  │Conn 1│ │Conn 2│ │Conn 3│        │
│  └──────┘ └──────┘ └──────┘        │
│  ┌──────┐ ┌──────┐                  │
│  │Busy  │ │Idle  │                  │
│  │Conn 4│ │Conn 5│                  │
│  │← Req2│ └──────┘                  │
│  └──────┘                            │
│                                      │
│  Config:                             │
│  ├── min: 5 (always keep alive)      │
│  ├── max: 20 (never exceed)          │
│  ├── idleTimeout: 30s                │
│  └── acquireTimeout: 10s             │
└─────────────────────────────────────┘
```

### 4️⃣ CODE

```javascript
// ═══════════════════════════════════════════
// 1. MONGODB (Mongoose) — Connection Pool
// ═══════════════════════════════════════════
const mongoose = require("mongoose");

mongoose.connect(process.env.MONGODB_URI, {
    // Pool Configuration
    maxPoolSize: 20,          // Max connections in pool (default: 100)
    minPoolSize: 5,           // Min connections to keep alive
    maxIdleTimeMS: 30000,     // Close idle connections after 30s
    waitQueueTimeoutMS: 10000, // Wait 10s for available connection
    serverSelectionTimeoutMS: 5000, // Timeout for server selection

    // Other important settings
    socketTimeoutMS: 45000,   // Close socket after 45s inactivity
    connectTimeoutMS: 10000,  // Connection timeout
    retryWrites: true,
    retryReads: true,
});

// Monitor pool events
mongoose.connection.on("connected", () => {
    console.log("✅ MongoDB connected");
    console.log(`Pool size: ${mongoose.connection.getClient().options.maxPoolSize}`);
});

mongoose.connection.on("error", (err) => {
    console.error("❌ MongoDB error:", err);
});

// ═══════════════════════════════════════════
// 2. PostgreSQL (pg) — Connection Pool
// ═══════════════════════════════════════════
const { Pool } = require("pg");

const pgPool = new Pool({
    host: process.env.DB_HOST,
    port: 5432,
    database: process.env.DB_NAME,
    user: process.env.DB_USER,
    password: process.env.DB_PASS,

    // Pool Configuration
    max: 20,              // Max connections
    min: 5,               // Min connections
    idleTimeoutMillis: 30000,    // Close idle after 30s
    connectionTimeoutMillis: 10000, // Timeout getting connection
    maxUses: 7500,        // Close after 7500 uses (prevents leaks)
});

// Monitor pool
pgPool.on("connect", () => console.log("New PG connection"));
pgPool.on("error", (err) => console.error("PG pool error:", err));
pgPool.on("remove", () => console.log("PG connection removed"));

// Usage in Express
app.get("/users", async (req, res) => {
    const client = await pgPool.connect();  // Grab from pool
    try {
        const result = await client.query("SELECT * FROM users LIMIT 10");
        res.json(result.rows);
    } finally {
        client.release();  // ⚠️ ALWAYS release back to pool!
    }
});

// Or simpler (auto-release):
app.get("/products", async (req, res) => {
    const { rows } = await pgPool.query("SELECT * FROM products");
    res.json(rows);  // Auto-released
});

// ═══════════════════════════════════════════
// 3. MySQL (mysql2) — Connection Pool
// ═══════════════════════════════════════════
const mysql = require("mysql2/promise");

const mysqlPool = mysql.createPool({
    host: process.env.DB_HOST,
    user: process.env.DB_USER,
    password: process.env.DB_PASS,
    database: process.env.DB_NAME,

    // Pool Configuration
    waitForConnections: true,
    connectionLimit: 20,      // Max connections
    queueLimit: 0,            // Unlimited queue (0 = unlimited)
    idleTimeout: 30000,
    enableKeepAlive: true,
    keepAliveInitialDelay: 10000,
});

app.get("/orders", async (req, res) => {
    const [rows] = await mysqlPool.execute(
        "SELECT * FROM orders WHERE user_id = ?",
        [req.user.id]  // Parameterized query (SQL injection safe!)
    );
    res.json(rows);
});

// ═══════════════════════════════════════════
// 4. Redis — Connection Pool (ioredis)
// ═══════════════════════════════════════════
const Redis = require("ioredis");

const redis = new Redis({
    host: process.env.REDIS_HOST,
    port: 6379,
    maxRetriesPerRequest: 3,
    retryStrategy(times) {
        const delay = Math.min(times * 200, 2000);
        return delay;  // Exponential backoff
    },
    lazyConnect: true,  // Don't connect until first command
});

// ═══════════════════════════════════════════
// 5. POOL MONITORING (Production!)
// ═══════════════════════════════════════════
setInterval(() => {
    console.log("PG Pool Stats:", {
        total: pgPool.totalCount,
        idle: pgPool.idleCount,
        waiting: pgPool.waitingCount,
    });
}, 30000);

// ═══════════════════════════════════════════
// 6. GRACEFUL POOL SHUTDOWN
// ═══════════════════════════════════════════
process.on("SIGTERM", async () => {
    await pgPool.end();          // Close all PG connections
    await mysqlPool.end();       // Close all MySQL connections
    await mongoose.disconnect(); // Close MongoDB
    await redis.quit();          // Close Redis
    process.exit(0);
});

app.listen(3000);
```

### Pool Sizing Formula

```
Optimal Pool Size ≈ (Core Count × 2) + Effective Spindle Count

Example:
  4 CPU cores, SSD (1 spindle)
  Pool size ≈ (4 × 2) + 1 = 9-10 connections

General guidelines:
  Small app:    5-10 connections
  Medium app:   10-20 connections
  Large app:    20-50 connections
  ⚠️ More ≠ Better! Too many = DB overload
```

### 5️⃣ INTERVIEW ANSWER

> *"Connection pooling maintains a **pre-created set of database connections** that are reused across requests instead of creating and destroying connections for each query. This eliminates the ~200ms connection overhead per request and prevents connection storms. I configure pools with `maxPoolSize` (upper limit), `minPoolSize` (keep-alive), and `idleTimeout` (cleanup unused). In Mongoose, I set `maxPoolSize: 20`; in PostgreSQL's `pg` library, I use `new Pool({ max: 20 })` and always call `client.release()` in a `finally` block. Pool sizing follows the formula `(cores × 2) + 1` — more connections aren't always better as they can overload the database. I monitor pool stats (total, idle, waiting) in production and implement graceful pool shutdown on SIGTERM."*

📌 **One-liner:** Pool = reuse connections, don't recreate | max/min/idle config | Always release!

---

---

## 7. Security Best Practices

> 🏆 **Most critical topic for production applications!**

### 1️⃣ WHAT

Security best practices woh **defensive measures** hain jo Node.js application ko common attacks se protect karti hain — injection, XSS, CSRF, DoS, data leaks, aur unauthorized access.

### 2️⃣ WHY

| Attack | Impact | Frequency |
|--------|--------|-----------|
| SQL/NoSQL Injection | Data theft/deletion | 🔥🔥🔥🔥🔥 |
| XSS | Session hijacking | 🔥🔥🔥🔥 |
| CSRF | Unauthorized actions | 🔥🔥🔥 |
| Brute Force | Account takeover | 🔥🔥🔥🔥 |
| DoS/DDoS | Server crash | 🔥🔥🔥 |
| Data Exposure | Privacy violation | 🔥🔥🔥🔥 |

### 3️⃣ HOW — Security Layers

```
┌─────────────────────────────────────────────────┐
│          SECURITY DEFENSE IN DEPTH               │
│                                                  │
│  Layer 1: HTTP HEADERS                           │
│  ├── Helmet (XSS, HSTS, CSP, etc.)              │
│  ├── CORS (origin control)                       │
│  └── Rate Limiting (DoS protection)              │
│                                                  │
│  Layer 2: INPUT VALIDATION                       │
│  ├── Joi / Zod (schema validation)               │
│  ├── Sanitize (XSS prevention)                   │
│  └── Parameterized queries (injection)           │
│                                                  │
│  Layer 3: AUTHENTICATION & AUTHORIZATION         │
│  ├── JWT (stateless auth)                        │
│  ├── bcrypt (password hashing)                   │
│  ├── RBAC (role-based access)                    │
│  └── 2FA (two-factor)                            │
│                                                  │
│  Layer 4: DATA PROTECTION                        │
│  ├── HTTPS everywhere                            │
│  ├── Encrypt sensitive fields                    │
│  ├── Don't log secrets                           │
│  └── Environment variables                       │
│                                                  │
│  Layer 5: DEPENDENCY SECURITY                    │
│  ├── npm audit                                   │
│  ├── Snyk / Dependabot                           │
│  └── Lock file versions                          │
└─────────────────────────────────────────────────┘
```

### 4️⃣ CODE

```bash
npm install helmet cors express-rate-limit express-mongo-sanitize
npm install xss-clean hpp cookie-parser jsonwebtoken bcryptjs
npm install joi helmet
```

```javascript
const express = require("express");
const helmet = require("helmet");
const cors = require("cors");
const rateLimit = require("express-rate-limit");
const mongoSanitize = require("express-mongo-sanitize");
const xss = require("xss-clean");
const hpp = require("hpp");
const cookieParser = require("cookie-parser");
const jwt = require("jsonwebtoken");
const bcrypt = require("bcryptjs");
const Joi = require("joi");

const app = express();

// ═══════════════════════════════════════════
// 1. HTTP SECURITY HEADERS (Helmet)
// ═══════════════════════════════════════════
app.use(helmet({
    contentSecurityPolicy: {
        directives: {
            defaultSrc: ["'self'"],
            scriptSrc: ["'self'", "'unsafe-inline'"],
            styleSrc: ["'self'", "'unsafe-inline'"],
            imgSrc: ["'self'", "data:", "https:"],
        }
    },
    hsts: { maxAge: 31536000, includeSubDomains: true },
    frameguard: { action: "deny" },        // Prevent clickjacking
    noSniff: true,                          // Prevent MIME sniffing
    xssFilter: true,                        // XSS filter
}));

// ═══════════════════════════════════════════
// 2. CORS (Strict Origin Control)
// ═══════════════════════════════════════════
app.use(cors({
    origin: process.env.NODE_ENV === "production"
        ? "https://myapp.com"
        : "http://localhost:3000",
    credentials: true,
    methods: ["GET", "POST", "PUT", "DELETE"],
}));

// ═══════════════════════════════════════════
// 3. RATE LIMITING (Brute Force + DoS)
// ═══════════════════════════════════════════
const limiter = rateLimit({
    windowMs: 15 * 60 * 1000,
    max: 100,
    message: { error: "Too many requests" }
});
app.use("/api/", limiter);

const authLimiter = rateLimit({
    windowMs: 15 * 60 * 1000,
    max: 5,
    message: { error: "Too many login attempts" }
});
app.use("/api/auth/", authLimiter);

// ═══════════════════════════════════════════
// 4. BODY PARSING (Size Limits!)
// ═══════════════════════════════════════════
app.use(express.json({ limit: "10kb" }));  // Prevent large payload DoS
app.use(express.urlencoded({ extended: true, limit: "10kb" }));
app.use(cookieParser());

// ═══════════════════════════════════════════
// 5. DATA SANITIZATION (NoSQL Injection + XSS)
// ═══════════════════════════════════════════
app.use(mongoSanitize());  // Removes $ and . from req.body/query/params
// Prevents: { "email": { "$gt": "" } } → NoSQL injection!

app.use(xss());  // Sanitizes HTML in req.body
// Prevents: <script>alert('xss')</script>

app.use(hpp());  // HTTP Parameter Pollution
// Prevents: ?sort=name&sort=age → uses only last value

// ═══════════════════════════════════════════
// 6. INPUT VALIDATION (Joi)
// ═══════════════════════════════════════════
const registerSchema = Joi.object({
    name: Joi.string().min(2).max(50).required(),
    email: Joi.string().email().required(),
    password: Joi.string()
        .min(8)
        .pattern(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/)
        .required()
        .messages({
            "string.pattern.base": "Must include uppercase, lowercase, and number"
        }),
});

function validate(schema) {
    return (req, res, next) => {
        const { error } = schema.validate(req.body, { abortEarly: false });
        if (error) {
            const messages = error.details.map(d => d.message);
            return res.status(400).json({ errors: messages });
        }
        next();
    };
}

app.post("/api/register", validate(registerSchema), async (req, res) => {
    // Safe to proceed — input validated!
    const { name, email, password } = req.body;

    // Hash password (12 rounds = ~250ms)
    const hashedPassword = await bcrypt.hash(password, 12);

    const user = await User.create({ name, email, password: hashedPassword });
    res.status(201).json({ id: user._id, name: user.name });
});

// ═══════════════════════════════════════════
// 7. JWT AUTHENTICATION (Secure Tokens)
// ═══════════════════════════════════════════
function generateToken(userId) {
    return jwt.sign(
        { id: userId },
        process.env.JWT_SECRET,       // Never hardcode!
        { expiresIn: "7d" }           // Token expiry
    );
}

function authenticate(req, res, next) {
    const token = req.headers.authorization?.split(" ")[1];

    if (!token) {
        return res.status(401).json({ error: "No token provided" });
    }

    try {
        const decoded = jwt.verify(token, process.env.JWT_SECRET);
        req.user = decoded;
        next();
    } catch (err) {
        if (err.name === "TokenExpiredError") {
            return res.status(401).json({ error: "Token expired" });
        }
        return res.status(403).json({ error: "Invalid token" });
    }
}

app.get("/api/profile", authenticate, async (req, res) => {
    const user = await User.findById(req.user.id).select("-password");
    res.json(user);
});

// ═══════════════════════════════════════════
// 8. SECURE COOKIES (If using cookies)
// ═══════════════════════════════════════════
app.post("/api/login", async (req, res) => {
    const { email, password } = req.body;
    const user = await User.findOne({ email });

    if (!user || !(await bcrypt.compare(password, user.password))) {
        return res.status(401).json({ error: "Invalid credentials" });
    }

    const token = generateToken(user._id);

    res.cookie("token", token, {
        httpOnly: true,     // ❌ JavaScript can't read (XSS safe)
        secure: true,       // ❌ HTTPS only
        sameSite: "strict", // ❌ CSRF protection
        maxAge: 7 * 24 * 60 * 60 * 1000,  // 7 days
    });

    res.json({ message: "Logged in" });
});

// ═══════════════════════════════════════════
// 9. ERROR HANDLING (Don't Leak Info!)
// ═══════════════════════════════════════════
app.use((err, req, res, next) => {
    console.error(err);  // Log internally

    // ❌ NEVER send stack traces or DB errors to client!
    res.status(err.statusCode || 500).json({
        status: "error",
        message: process.env.NODE_ENV === "production"
            ? "Internal server error"   // Generic in production
            : err.message               // Detailed in development
    });
});

// ═══════════════════════════════════════════
// 10. PROCESS SECURITY
// ═══════════════════════════════════════════
// Don't run as root!
if (process.getuid && process.getuid() === 0) {
    console.error("⚠️ Don't run Node.js as root!");
    process.exit(1);
}

app.listen(3000);
```

### Security Checklist

| # | Practice | Tool/Method |
|---|----------|-------------|
| 1 | Security Headers | `helmet` |
| 2 | CORS Control | `cors` (specific origins) |
| 3 | Rate Limiting | `express-rate-limit` |
| 4 | Input Validation | `joi` / `zod` |
| 5 | NoSQL Injection | `express-mongo-sanitize` |
| 6 | XSS Prevention | `xss-clean` + CSP |
| 7 | SQL Injection | Parameterized queries |
| 8 | Password Hashing | `bcrypt` (12+ rounds) |
| 9 | JWT Security | Short expiry + refresh tokens |
| 10 | HTTPS | Always in production |
| 11 | Env Variables | `.env` + `.gitignore` |
| 12 | Dependency Audit | `npm audit` / Snyk |
| 13 | Body Size Limit | `express.json({ limit })` |
| 14 | HPP Prevention | `hpp` |
| 15 | Error Messages | Generic in production |

### 5️⃣ INTERVIEW ANSWER

> *"I implement security in layers — **defense in depth**. At the HTTP level, I use **Helmet** for security headers (CSP, HSTS, X-Frame-Options), strict **CORS** configuration, and **rate limiting** to prevent brute-force and DoS attacks
