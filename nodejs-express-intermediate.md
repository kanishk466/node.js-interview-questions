# 🚀 Node.js & Express.js — Interview Prep Guide (Intermediate Level)

> **Level:** Intermediate
> **Format:** What → Why → How → Example → Interview Answer
> **Prerequisite:** Basic Level (Node.js, Express, Middleware, Routes)

---

## 📑 Table of Contents

| # | Topic | Difficulty | Frequency |
|---|-------|-----------|-----------|
| 1 | [Event-Driven Architecture](#1-event-driven-architecture-of-nodejs) | ⭐⭐⭐ | 🔥🔥🔥🔥 |
| 2 | [Streams in Node.js](#2-streams-in-nodejs) | ⭐⭐⭐ | 🔥🔥🔥 |
| 3 | [File Uploads in Express](#3-file-uploads-in-express) | ⭐⭐ | 🔥🔥🔥🔥 |
| 4 | [Sync vs Async in Node.js](#4-sync-vs-async-in-nodejs) | ⭐⭐ | 🔥🔥🔥🔥🔥 |
| 5 | [Clustering in Node.js](#5-clustering-in-nodejs) | ⭐⭐⭐ | 🔥🔥🔥 |
| 6 | [Error Handling Middleware](#6-error-handling-middleware-in-express) | ⭐⭐ | 🔥🔥🔥🔥🔥 |
| 7 | [body-parser Middleware](#7-body-parser-middleware) | ⭐⭐ | 🔥🔥🔥 |
| 8 | [CORS in Express](#8-cors-in-express) | ⭐⭐ | 🔥🔥🔥🔥🔥 |

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

## 1. Event-Driven Architecture of Node.js

### 1️⃣ WHAT

Node.js **Event-Driven Architecture** pe based hai — matlab server **events ka wait** karta hai aur jab koi event fire hota hai, uske liye registered **callback (listener)** execute karta hai.

> **Simple:** "Jab ye ho → tab ye karo" — yahi Node.js ka core philosophy hai.

```
Traditional Server (Thread-based):         Node.js (Event-Driven):
┌──────────────────────┐                   ┌──────────────────────┐
│ Request 1 → Thread 1 │ (blocked)         │ Request 1 ─┐         │
│ Request 2 → Thread 2 │ (blocked)         │ Request 2 ─┤ Event   │
│ Request 3 → Thread 3 │ (blocked)         │ Request 3 ─┤ Loop    │
│ ...                  │                   │ Request 999┤ (single │
│ 1000 requests =      │                   │            │  thread)│
│ 1000 threads! 💀     │                   │ Non-blocking! ✅     │
└──────────────────────┘                   └──────────────────────┘
```

### 2️⃣ WHY

| Problem (Thread Model) | Solution (Event Model) |
|------------------------|----------------------|
| Har request = naya thread | Ek thread, hazaron connections |
| Thread creation = expensive | No thread overhead |
| Context switching = slow | No context switching |
| Memory per thread ~1-2MB | Minimal memory footprint |
| 1000 users = 1000 threads | 1000 users = 1 thread + event queue |

### 3️⃣ HOW — Core Components

```
┌─────────────────────────────────────────────────────┐
│              EVENT-DRIVEN ARCHITECTURE               │
│                                                      │
│  ┌─────────────┐    ┌──────────────┐                │
│  │ Event       │    │ Event        │                │
│  │ Emitters    │───→│ Listeners    │                │
│  │ (fire event)│    │ (callbacks)  │                │
│  └──────┬──────┘    └──────┬───────┘                │
│         │                  │                         │
│         ▼                  ▼                         │
│  ┌──────────────────────────────────┐               │
│  │         EVENT LOOP               │               │
│  │                                  │               │
│  │  ┌────────┐  ┌────────────────┐  │               │
│  │  │  Tick  │  │  Phases:       │  │               │
│  │  │ Queue  │  │  1. Timers     │  │               │
│  │  │        │  │  2. I/O Poll   │  │               │
│  │  │        │  │  3. Check      │  │               │
│  │  │        │  │  4. Close      │  │               │
│  │  └────────┘  └────────────────┘  │               │
│  └──────────────────────────────────┘               │
│         │                                            │
│         ▼                                            │
│  ┌──────────────┐                                    │
│  │  libuv       │                                    │
│  │  Thread Pool │ (for heavy I/O: file, DNS, crypto) │
│  │  (4 threads  │                                    │
│  │   default)   │                                    │
│  └──────────────┘                                    │
└─────────────────────────────────────────────────────┘
```

**Key Concepts:**

| Concept | Role | Example |
|---------|------|---------|
| **EventEmitter** | Events fire karta hai | `emitter.emit('data')` |
| **Event Listener** | Events sunta hai | `emitter.on('data', cb)` |
| **Event Loop** | Events ko process karta hai | Continuous cycle |
| **Callback Queue** | Pending callbacks store | FIFO |
| **libuv** | OS-level async I/O | File read, network |

### 4️⃣ CODE

```javascript
// ═══════════════════════════════════════════
// 1. EventEmitter — Node.js ka Core
// ═══════════════════════════════════════════
const EventEmitter = require("events");

class OrderService extends EventEmitter {
    placeOrder(order) {
        console.log(`Order placed: ${order.id}`);

        // Event fire karo!
        this.emit("orderPlaced", order);
    }
}

const orderService = new OrderService();

// Listeners register karo (decoupled!)
orderService.on("orderPlaced", (order) => {
    console.log(`📧 Email sent for order ${order.id}`);
});

orderService.on("orderPlaced", (order) => {
    console.log(`📦 Inventory updated for order ${order.id}`);
});

orderService.on("orderPlaced", (order) => {
    console.log(`📊 Analytics logged for order ${order.id}`);
});

// Trigger!
orderService.placeOrder({ id: 101, item: "Laptop" });
// Output:
// Order placed: 101
// 📧 Email sent for order 101
// 📦 Inventory updated for order 101
// 📊 Analytics logged for order 101

// ═══════════════════════════════════════════
// 2. HTTP Server = Event-Driven!
// ═══════════════════════════════════════════
const http = require("http");

const server = http.createServer();

// "request" event pe listener
server.on("request", (req, res) => {
    res.end("Hello!");
});

// "connection" event pe listener
server.on("connection", (socket) => {
    console.log("New client connected!");
});

server.listen(3000);

// ═══════════════════════════════════════════
// 3. Real-World: Chat Server Pattern
// ═══════════════════════════════════════════
const chatEmitter = new EventEmitter();

chatEmitter.on("message", ({ user, text }) => {
    console.log(`[${user}]: ${text}`);
    // Broadcast to all connected clients
});

chatEmitter.on("userJoined", (user) => {
    console.log(`🎉 ${user} joined the chat!`);
});

// Simulate events
chatEmitter.emit("userJoined", "Kanishk");
chatEmitter.emit("message", { user: "Kanishk", text: "Hello everyone!" });

// ═══════════════════════════════════════════
// 4. once() — Listen Only Once
// ═══════════════════════════════════════════
const emitter = new EventEmitter();

emitter.once("init", () => {
    console.log("Initialized! (only first time)");
});

emitter.emit("init");  // "Initialized!"
emitter.emit("init");  // (nothing — already fired once)
```

### 5️⃣ INTERVIEW ANSWER

> *"Node.js follows an **event-driven, non-blocking I/O architecture**. Instead of creating a new thread for each request (like traditional servers), Node.js uses a **single thread with an Event Loop**. When an I/O operation is initiated, Node.js offloads it to the OS or libuv thread pool and registers a **callback**. When the operation completes, an **event is emitted** and the callback is placed in the event queue. The Event Loop picks it up when the call stack is empty. This architecture allows Node.js to handle **thousands of concurrent connections** with minimal memory overhead. The `EventEmitter` class is the foundation — even the HTTP server is essentially an EventEmitter that listens for 'request' events."*

📌 **One-liner:** Event-Driven = "jab ye ho → tab ye karo" | Single thread + Event Loop + Callbacks

---

---

## 2. Streams in Node.js

### 1️⃣ WHAT

Streams **data ko chunks (chhote-chhote hisson)** mein read ya write karne ka tarika hai — **poora data ek saath memory mein load kiye bina**.

> **Analogy:** Bucket vs Pipe
> - **Bucket (Buffer):** Poora paani bharo → phir leke jao (memory heavy)
> - **Pipe (Stream):** Paani continuously flow karta hai (memory efficient)

```
Without Streams:                    With Streams:
┌──────────────┐                    ┌──────────────┐
│ 2GB File     │                    │ 2GB File     │
│    ↓         │                    │    ↓         │
│ Load ALL in  │                    │ Chunk 1 (64KB)│ → Process
│ Memory (2GB) │ 💀 CRASH!          │ Chunk 2 (64KB)│ → Process
│    ↓         │                    │ Chunk 3 (64KB)│ → Process
│ Process      │                    │ ...           │ → Process
└──────────────┘                    └──────────────┘
                                    Memory: ~64KB only! ✅
```

### 2️⃣ WHY

| Scenario | Without Streams | With Streams |
|----------|----------------|--------------|
| 2GB video upload | 2GB RAM chahiye | ~64KB RAM |
| Large CSV processing | Crash / hang | Smooth |
| Real-time data | Not possible | Possible |
| File transfer | Slow start | Instant start |

### 3️⃣ HOW — 4 Types of Streams

| Type | Direction | Use Case | Example |
|------|-----------|----------|---------|
| **Readable** | Source → App | Data read karna | `fs.createReadStream()` |
| **Writable** | App → Destination | Data write karna | `fs.createWriteStream()` |
| **Duplex** | Both ways | Read + Write | `net.Socket` (TCP) |
| **Transform** | Both + Modify | Data transform | `zlib.createGzip()` |

```
Stream Flow:

Readable ──→ [pipe()] ──→ Writable
  (source)                  (destination)

Readable ──→ [pipe()] ──→ Transform ──→ [pipe()] ──→ Writable
  (source)                 (modify)                   (destination)
```

### 4️⃣ CODE

```javascript
const fs = require("fs");
const zlib = require("zlib");
const { Transform } = require("stream");

// ═══════════════════════════════════════════
// 1. READABLE STREAM
// ═══════════════════════════════════════════
const readStream = fs.createReadStream("large-file.txt", {
    encoding: "utf-8",
    highWaterMark: 64 * 1024  // 64KB chunks
});

readStream.on("data", (chunk) => {
    console.log(`Received chunk: ${chunk.length} bytes`);
});

readStream.on("end", () => {
    console.log("File reading complete!");
});

readStream.on("error", (err) => {
    console.error("Read error:", err.message);
});

// ═══════════════════════════════════════════
// 2. WRITABLE STREAM
// ═══════════════════════════════════════════
const writeStream = fs.createWriteStream("output.txt");

writeStream.write("Hello ");
writeStream.write("World!");
writeStream.end();  // Signal: no more data

writeStream.on("finish", () => {
    console.log("Write complete!");
});

// ═══════════════════════════════════════════
// 3. PIPE — Most Common Pattern!
// ═══════════════════════════════════════════
// File copy (memory efficient!)
fs.createReadStream("source.mp4")
  .pipe(fs.createWriteStream("destination.mp4"));

// ═══════════════════════════════════════════
// 4. TRANSFORM STREAM (Modify Data)
// ═══════════════════════════════════════════
// File compress karo (Gzip)
fs.createReadStream("input.txt")
  .pipe(zlib.createGzip())              // Transform: compress
  .pipe(fs.createWriteStream("input.txt.gz"));

// ═══════════════════════════════════════════
// 5. CUSTOM TRANSFORM STREAM
// ═══════════════════════════════════════════
const upperCaseTransform = new Transform({
    transform(chunk, encoding, callback) {
        const upper = chunk.toString().toUpperCase();
        callback(null, upper);  // Pass transformed data
    }
});

fs.createReadStream("input.txt")
  .pipe(upperCaseTransform)
  .pipe(fs.createWriteStream("output-upper.txt"));

// ═══════════════════════════════════════════
// 6. DUPLEX STREAM (Read + Write)
// ═══════════════════════════════════════════
const { Duplex } = require("stream");

const duplexStream = new Duplex({
    read(size) {
        this.push("Data from source");
        this.push(null);  // End
    },
    write(chunk, encoding, callback) {
        console.log("Writing:", chunk.toString());
        callback();
    }
});

// ═══════════════════════════════════════════
// 7. EXPRESS: Stream File Download
// ═══════════════════════════════════════════
const express = require("express");
const app = express();

app.get("/download", (req, res) => {
    const filePath = "large-video.mp4";

    res.setHeader("Content-Type", "video/mp4");
    res.setHeader("Content-Disposition", 'attachment; filename="video.mp4"');

    // Stream file — poora RAM mein load nahi hoga!
    const stream = fs.createReadStream(filePath);
    stream.pipe(res);

    stream.on("error", (err) => {
        res.status(500).send("Download failed");
    });
});
```

### 5️⃣ INTERVIEW ANSWER

> *"Streams in Node.js are objects that allow reading or writing data **continuously in chunks** rather than loading the entire data into memory at once. There are four types: **Readable** (data source), **Writable** (data destination), **Duplex** (both read and write, like TCP sockets), and **Transform** (duplex that modifies data, like gzip compression). The most powerful feature is `.pipe()`, which connects streams together — for example, piping a read stream through a gzip transform into a write stream for file compression. Streams are essential for handling **large files, video streaming, and real-time data** with minimal memory usage."*

📌 **One-liner:** Streams = data in chunks, not all at once | Readable → Writable → Duplex → Transform | `.pipe()`

---

---

## 3. File Uploads in Express

### 1️⃣ WHAT

File upload ka matlab hai client (browser) se server pe files bhejna — images, PDFs, videos, documents. Express mein ye **`multipart/form-data`** encoding se hota hai aur **`multer`** middleware sabse popular solution hai.

### 2️⃣ WHY

- Express by default **multipart data parse nahi** karta
- `express.json()` sirf JSON handle karta hai
- File upload ke liye **special parsing** chahiye (binary data, boundaries)
- Security: file type, size, count **validate** karna zaroori hai

### 3️⃣ HOW — Upload Flow

```
Client (Browser)                        Server (Express)
───────────────                         ────────────────
                                        
<form enctype="multipart/form-data">    
  <input type="file" />                 
  <button>Upload</button>               
</form>                                 
     │                                  
     │ POST /upload                     
     │ Content-Type: multipart/form-data
     │ [binary file data]               
     │                                  
     └─────────────────────────────────→  Multer Middleware
                                              │
                                         ┌────┴────┐
                                         │ Validate │
                                         │ Type/Size│
                                         └────┬────┘
                                              │
                                         ┌────┴────┐
                                         │  Save    │
                                         │  to Disk │
                                         │  /Memory │
                                         └────┬────┘
                                              │
                                         Route Handler
                                         (req.file)
```

### 4️⃣ CODE

```bash
npm install multer
```

```javascript
const express = require("express");
const multer = require("multer");
const path = require("path");
const app = express();

// ═══════════════════════════════════════════
// 1. DISK STORAGE (Most Common)
// ═══════════════════════════════════════════
const storage = multer.diskStorage({
    destination: (req, file, cb) => {
        cb(null, "uploads/");  // Folder must exist!
    },
    filename: (req, file, cb) => {
        // Unique filename: timestamp + original name
        const uniqueName = `${Date.now()}-${file.originalname}`;
        cb(null, uniqueName);
    }
});

// ═══════════════════════════════════════════
// 2. FILE FILTER (Security!)
// ═══════════════════════════════════════════
const fileFilter = (req, file, cb) => {
    const allowedTypes = /jpeg|jpg|png|gif|pdf/;
    const extName = allowedTypes.test(
        path.extname(file.originalname).toLowerCase()
    );
    const mimeType = allowedTypes.test(file.mimetype);

    if (extName && mimeType) {
        cb(null, true);   // Accept
    } else {
        cb(new Error("Only images and PDFs allowed!"), false);  // Reject
    }
};

// ═══════════════════════════════════════════
// 3. CONFIGURE MULTER
// ═══════════════════════════════════════════
const upload = multer({
    storage: storage,
    fileFilter: fileFilter,
    limits: {
        fileSize: 5 * 1024 * 1024,  // 5MB max
        files: 3                     // Max 3 files per request
    }
});

// ═══════════════════════════════════════════
// 4. SINGLE FILE UPLOAD
// ═══════════════════════════════════════════
app.post("/upload/single", upload.single("avatar"), (req, res) => {
    if (!req.file) {
        return res.status(400).json({ error: "No file uploaded" });
    }

    res.json({
        message: "File uploaded successfully!",
        file: {
            name: req.file.filename,
            originalName: req.file.originalname,
            size: req.file.size,
            path: req.file.path
        }
    });
});

// ═══════════════════════════════════════════
// 5. MULTIPLE FILES UPLOAD
// ═══════════════════════════════════════════
app.post("/upload/multiple", upload.array("photos", 5), (req, res) => {
    res.json({
        message: `${req.files.length} files uploaded!`,
        files: req.files.map(f => ({
            name: f.filename,
            size: f.size
        }))
    });
});

// ═══════════════════════════════════════════
// 6. MIXED FIELDS (Different field names)
// ═══════════════════════════════════════════
app.post("/upload/mixed", upload.fields([
    { name: "avatar", maxCount: 1 },
    { name: "documents", maxCount: 3 }
]), (req, res) => {
    res.json({
        avatar: req.files["avatar"]?.[0]?.filename,
        documents: req.files["documents"]?.map(f => f.filename)
    });
});

// ═══════════════════════════════════════════
// 7. MEMORY STORAGE (For cloud upload)
// ═══════════════════════════════════════════
const memoryUpload = multer({ storage: multer.memoryStorage() });

app.post("/upload/cloud", memoryUpload.single("file"), (req, res) => {
    // req.file.buffer contains the file data
    // Upload to AWS S3, Cloudinary, etc.
    // s3.upload({ Body: req.file.buffer, ... })

    res.json({
        message: "Ready for cloud upload",
        size: req.file.buffer.length
    });
});

// ═══════════════════════════════════════════
// 8. ERROR HANDLING
// ═══════════════════════════════════════════
app.use((err, req, res, next) => {
    if (err instanceof multer.MulterError) {
        if (err.code === "LIMIT_FILE_SIZE") {
            return res.status(400).json({ error: "File too large! Max 5MB" });
        }
        return res.status(400).json({ error: err.message });
    }
    if (err) {
        return res.status(400).json({ error: err.message });
    }
    next();
});

app.listen(3000, () => console.log("Upload server running"));
```

```html
<!-- Client-Side Form -->
<form action="/upload/single" method="POST" enctype="multipart/form-data">
    <input type="file" name="avatar" accept="image/*" />
    <button type="submit">Upload</button>
</form>
```

### 5️⃣ INTERVIEW ANSWER

> *"Express doesn't handle `multipart/form-data` by default, so I use the **multer** middleware for file uploads. I configure it with **disk storage** for saving files locally or **memory storage** for cloud uploads. Security is critical — I always implement a **file filter** to validate MIME types and extensions, and set **file size limits** to prevent abuse. For single files I use `upload.single()`, for multiple files `upload.array()`, and for mixed fields `upload.fields()`. I also handle multer-specific errors like `LIMIT_FILE_SIZE` in the error middleware. In production, I typically stream uploads directly to **AWS S3 or Cloudinary** using memory storage."*

📌 **One-liner:** Multer = Express file upload middleware | disk/memory storage | filter + limits = security

---

---

## 4. Sync vs Async in Node.js

### 1️⃣ WHAT

| Mode | Behavior | Blocking? |
|------|----------|-----------|
| **Synchronous** | Ek kaam **khatam hone ke baad** hi agla shuru hota hai | ✅ Yes — poora thread block |
| **Asynchronous** | Kaam **background** mein start karo, agla kaam continue karo | ❌ No — thread free rehta hai |

> **Node.js single-threaded hai** — isliye sync code **poore server ko freeze** kar deta hai!

### 2️⃣ WHY — Critical Difference

```
Synchronous (BLOCKING):
──────────────────────────────────────────────
Task 1: Read File (3 sec)  ████████████
Task 2: API Call (2 sec)                ████████
Task 3: DB Query (1 sec)                        ████
Total: 6 seconds | Thread BLOCKED the entire time! 💀

Asynchronous (NON-BLOCKING):
──────────────────────────────────────────────
Task 1: Read File (3 sec)  ████████████
Task 2: API Call (2 sec)   ████████
Task 3: DB Query (1 sec)   ████
Total: ~3 seconds | Thread FREE during I/O! ✅
```

### 3️⃣ HOW — Node.js Async Patterns

```
Evolution of Async in Node.js:

1. Callbacks (Old)
   fs.readFile("f.txt", (err, data) => { ... })

2. Promises (ES6)
   fs.promises.readFile("f.txt").then(data => { ... })

3. Async/Await (ES2017 — Modern!)
   const data = await fs.promises.readFile("f.txt")
```

### 4️⃣ CODE

```javascript
const fs = require("fs");

// ═══════════════════════════════════════════
// SYNCHRONOUS (BLOCKING) — AVOID IN PRODUCTION!
// ═══════════════════════════════════════════
console.log("Start");

// Ye 3 second lega — poora server RUK jayega!
const data = fs.readFileSync("large-file.txt", "utf-8");
console.log("File read:", data.length);

console.log("End");
// Order: Start → File read → End (sequential, blocking)

// ═══════════════════════════════════════════
// ASYNCHRONOUS — CALLBACK (Old Way)
// ═══════════════════════════════════════════
console.log("Start");

fs.readFile("large-file.txt", "utf-8", (err, data) => {
    if (err) return console.error(err);
    console.log("File read:", data.length);
});

console.log("End");
// Order: Start → End → File read (non-blocking!)

// ═══════════════════════════════════════════
// ASYNCHRONOUS — PROMISES
// ═══════════════════════════════════════════
const fsPromises = fs.promises;

fsPromises.readFile("large-file.txt", "utf-8")
    .then(data => console.log("File read:", data.length))
    .catch(err => console.error(err));

// ═══════════════════════════════════════════
// ASYNCHRONOUS — ASYNC/AWAIT (Best!)
// ═══════════════════════════════════════════
async function readFileAsync() {
    try {
        const data = await fsPromises.readFile("large-file.txt", "utf-8");
        console.log("File read:", data.length);
    } catch (err) {
        console.error("Error:", err.message);
    }
}

readFileAsync();

// ═══════════════════════════════════════════
// REAL-WORLD: Express Route Comparison
// ═══════════════════════════════════════════
const express = require("express");
const app = express();

// ❌ SYNC — Server freeze hoga!
app.get("/sync", (req, res) => {
    const data = fs.readFileSync("huge-data.json", "utf-8");  // BLOCKS!
    res.json(JSON.parse(data));
});

// ✅ ASYNC — Server responsive rahega!
app.get("/async", async (req, res) => {
    try {
        const data = await fsPromises.readFile("huge-data.json", "utf-8");
        res.json(JSON.parse(data));
    } catch (err) {
        res.status(500).json({ error: "Failed to read data" });
    }
});

// ═══════════════════════════════════════════
// PARALLEL EXECUTION (Async Superpower!)
// ═══════════════════════════════════════════
async function getDashboardData() {
    // ❌ Sequential: 3 + 2 + 1 = 6 seconds
    // const users = await fetchUsers();
    // const orders = await fetchOrders();
    // const stats = await fetchStats();

    // ✅ Parallel: max(3, 2, 1) = 3 seconds!
    const [users, orders, stats] = await Promise.all([
        fetchUsers(),
        fetchOrders(),
        fetchStats()
    ]);

    return { users, orders, stats };
}
```

### When to Use What?

| Scenario | Use | Why |
|----------|-----|-----|
| Server routes / APIs | **Async** ✅ | Don't block other requests |
| Startup config loading | **Sync** OK | Server start hone se pehle |
| CLI scripts | **Sync** OK | Single user, sequential |
| File operations in routes | **Async** ✅ | Large files block thread |
| CPU-heavy tasks | **Worker Threads** | Neither sync nor async helps |

### 5️⃣ INTERVIEW ANSWER

> *"In Node.js, **synchronous** code executes line by line and **blocks the single thread** until completion, while **asynchronous** code offloads I/O operations to the OS/libuv and continues executing the next lines. Since Node.js is single-threaded, synchronous operations like `fs.readFileSync()` can **freeze the entire server** for all users. I always use **async/await** with `fs.promises` for file operations, database queries, and API calls in route handlers. For multiple independent async operations, I use `Promise.all()` for parallel execution. Synchronous methods are acceptable only during **application startup** before the server begins accepting requests."*

📌 **One-liner:** Sync = blocking (server freeze!) | Async = non-blocking (server responsive) | Always async in routes

---

---

## 5. Clustering in Node.js

### 1️⃣ WHAT

Clustering Node.js ki **single-thread limitation** ko overcome karne ka tarika hai — ye **multiple worker processes** banata hai, har ek **alag CPU core** pe chalta hai, but **same port share** karta hai.

```
Without Clustering:                    With Clustering (4 cores):
┌──────────────────┐                   ┌──────────────────┐
│   CPU Core 1     │                   │   CPU Core 1     │
│   ┌────────────┐ │                   │   ┌────────────┐ │
│   │ Node.js    │ │                   │   │ Worker 1   │ │
│   │ (1 thread) │ │                   │   └────────────┘ │
│   └────────────┘ │                   ├──────────────────┤
│   CPU Core 2     │                   │   CPU Core 2     │
│   (IDLE 😴)      │                   │   ┌────────────┐ │
│   CPU Core 3     │                   │   │ Worker 2   │ │
│   (IDLE 😴)      │                   │   └────────────┘ │
│   CPU Core 4     │                   ├──────────────────┤
│   (IDLE 😴)      │                   │   CPU Core 3 → Worker 3 │
└──────────────────┘                   │   CPU Core 4 → Worker 4 │
                                       │   All share port 3000!  │
Utilization: ~25% 💀                   └──────────────────┘
                                       Utilization: ~100% ✅
```

### 2️⃣ WHY

| Problem | Solution |
|---------|----------|
| Node.js = 1 thread = 1 CPU core | Cluster = N workers = N cores |
| 4-core server pe 75% CPU waste | All cores utilized |
| Single crash = server down | Worker crash = master restarts it |
| Limited throughput | N× throughput |

### 3️⃣ HOW — Architecture

```
┌─────────────────────────────────────┐
│         MASTER PROCESS              │
│  (Manages workers, shares port)     │
│                                     │
│  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐  │
│  │ W1  │ │ W2  │ │ W3  │ │ W4  │  │
│  │PID: │ │PID: │ │PID: │ │PID: │  │
│  │1001 │ │1002 │ │1003 │ │1004 │  │
│  └──┬──┘ └──┬──┘ └──┬──┘ └──┬──┘  │
│     └───────┴───────┴───────┘      │
│              │                      │
│         Same Port: 3000             │
│    (OS load-balances requests)      │
└─────────────────────────────────────┘
```

### 4️⃣ CODE

```javascript
// ═══════════════════════════════════════════
// cluster.js — Basic Clustering
// ═══════════════════════════════════════════
const cluster = require("cluster");
const os = require("os");
const express = require("express");

const numCPUs = os.cpus().length;  // e.g., 4 or 8

if (cluster.isMaster) {
    // ── MASTER PROCESS ──
    console.log(`Master ${process.pid} is running`);
    console.log(`Forking ${numCPUs} workers...`);

    // Fork workers
    for (let i = 0; i < numCPUs; i++) {
        cluster.fork();
    }

    // Worker crash → restart!
    cluster.on("exit", (worker, code, signal) => {
        console.log(`Worker ${worker.process.pid} died (${signal || code})`);
        console.log("Starting a new worker...");
        cluster.fork();  // Auto-restart!
    });

    // Worker online
    cluster.on("online", (worker) => {
        console.log(`Worker ${worker.process.pid} is online ✅`);
    });

} else {
    // ── WORKER PROCESS ──
    const app = express();

    app.get("/", (req, res) => {
        // Simulate CPU work
        let sum = 0;
        for (let i = 0; i < 1e7; i++) sum += i;

        res.json({
            message: "Hello from cluster!",
            workerPid: process.pid,
            result: sum
        });
    });

    app.listen(3000, () => {
        console.log(`Worker ${process.pid} listening on port 3000`);
    });
}

// ═══════════════════════════════════════════
// PRODUCTION: PM2 (Better than manual cluster!)
// ═══════════════════════════════════════════
// Install: npm install -g pm2
// Run: pm2 start server.js -i max
//
// -i max = auto-detect CPUs and fork
// Built-in: restart, logging, monitoring
//
// Other PM2 commands:
// pm2 list          → Show all processes
// pm2 monit         → Real-time monitoring
// pm2 logs          → View logs
// pm2 restart all   → Restart all
// pm2 delete all    → Stop all

// ═══════════════════════════════════════════
// ALTERNATIVE: Worker Threads (CPU-heavy tasks)
// ═══════════════════════════════════════════
// Clustering = multiple PROCESSES (separate memory)
// Worker Threads = multiple THREADS (shared memory)

const { Worker } = require("worker_threads");

function runHeavyTask(data) {
    return new Promise((resolve, reject) => {
        const worker = new Worker("./heavy-task.js", {
            workerData: data
        });

        worker.on("message", resolve);
        worker.on("error", reject);
    });
}

// Usage in Express route:
app.get("/process", async (req, res) => {
    const result = await runHeavyTask(req.body.data);
    res.json({ result });
});
```

### Clustering vs Worker Threads

| Feature | Cluster | Worker Threads |
|---------|---------|---------------|
| **Type** | Multiple processes | Multiple threads |
| **Memory** | Separate (isolated) | Shared (ArrayBuffer) |
| **Use Case** | Scale HTTP server | CPU-heavy computation |
| **Crash Impact** | One worker dies, others OK | Thread crash can affect process |
| **Communication** | IPC (message passing) | SharedArrayBuffer / messages |
| **Production** | PM2 | For specific tasks |

### 5️⃣ INTERVIEW ANSWER

> *"Node.js runs on a **single thread**, which means it can only utilize **one CPU core**. Clustering solves this by using the built-in `cluster` module to fork **multiple worker processes** — ideally one per CPU core — all sharing the **same port**. The master process manages workers and automatically **restarts** any that crash. The OS load-balances incoming requests across workers. In production, I prefer using **PM2** with `pm2 start server.js -i max`, which handles clustering, process monitoring, and auto-restart out of the box. For CPU-intensive tasks within a single process, I use **Worker Threads** instead, which share memory and are lighter than full processes."*

📌 **One-liner:** Cluster = N workers × N CPU cores × same port | PM2 in production | Worker Threads for CPU tasks

---

---

## 6. Error Handling Middleware in Express

### 1️⃣ WHAT

Error handling middleware ek **special Express middleware** hai jo **4 arguments** leta hai — `(err, req, res, next)` — aur application mein kahin bhi throw hone wale errors ko **catch aur handle** karta hai.

> **Express ka rule:** 4 arguments = error handler | 3 arguments = normal middleware

### 2️⃣ WHY

Bina centralized error handling ke:
- ❌ Har route mein `try/catch` likhna padega (repetitive)
- ❌ Unhandled errors → **stack trace leak** to client (security risk!)
- ❌ Inconsistent error responses
- ❌ App crash ho sakta hai

Centralized error handler se:
- ✅ Ek jagah sab errors handle
- ✅ Consistent error format
- ✅ Security (no stack traces in production)
- ✅ Logging (errors track karo)

### 3️⃣ HOW — Error Flow

```
Route Handler
     │
     ├── Sync Error: throw new Error("Oops")
     │        ↓
     │   Express auto-catches → Error Middleware ✅
     │
     ├── Async Error (Express 5): throw in async
     │        ↓
     │   Auto-catches → Error Middleware ✅
     │
     ├── Async Error (Express 4): throw in async
     │        ↓
     │   NOT auto-caught! → next(err) zaroori ⚠️
     │
     └── Manual: next(new Error("Oops"))
              ↓
         Error Middleware ✅

Error Middleware (LAST in chain!)
     │
     ├── Log the error
     ├── Determine status code
     ├── Format response
     └── Send to client
```

### 4️⃣ CODE

```javascript
const express = require("express");
const app = express();
app.use(express.json());

// ═══════════════════════════════════════════
// 1. CUSTOM ERROR CLASS (Best Practice!)
// ═══════════════════════════════════════════
class AppError extends Error {
    constructor(message, statusCode) {
        super(message);
        this.statusCode = statusCode;
        this.isOperational = true;  // Known error vs unknown bug
        Error.captureStackTrace(this, this.constructor);
    }
}

// ═══════════════════════════════════════════
// 2. ROUTES WITH ERRORS
// ═══════════════════════════════════════════

// Sync error — Express auto-catches
app.get("/sync-error", (req, res) => {
    throw new Error("Something broke!");  // Auto → error handler
});

// Async error — Express 4 needs next(err)
app.get("/async-error", async (req, res, next) => {
    try {
        const user = await findUser(req.params.id);  // Might fail
        res.json(user);
    } catch (err) {
        next(err);  // ← Pass to error handler!
    }
});

// Manual error with custom status
app.get("/not-found", (req, res, next) => {
    next(new AppError("Resource not found", 404));
});

// Validation error
app.post("/users", (req, res, next) => {
    const { name, email } = req.body;

    if (!name || !email) {
        return next(new AppError("Name and email are required", 400));
    }

    res.status(201).json({ name, email });
});

// ═══════════════════════════════════════════
// 3. ASYNC WRAPPER (Avoid try/catch everywhere!)
// ═══════════════════════════════════════════
const asyncHandler = (fn) => (req, res, next) => {
    Promise.resolve(fn(req, res, next)).catch(next);
};

// Now no try/catch needed!
app.get("/users/:id", asyncHandler(async (req, res) => {
    const user = await findUser(req.params.id);
    if (!user) throw new AppError("User not found", 404);
    res.json(user);
}));

// ═══════════════════════════════════════════
// 4. 404 HANDLER (Unknown Routes)
// ═══════════════════════════════════════════
app.use((req, res, next) => {
    next(new AppError(`Route ${req.originalUrl} not found`, 404));
});

// ═══════════════════════════════════════════
// 5. GLOBAL ERROR HANDLER (MUST BE LAST!)
// ═══════════════════════════════════════════
app.use((err, req, res, next) => {
    // 1. Log the error
    console.error(`[ERROR] ${err.message}`);
    if (process.env.NODE_ENV === "development") {
        console.error(err.stack);
    }

    // 2. Determine status code
    const statusCode = err.statusCode || 500;

    // 3. Handle specific error types
    if (err.name === "ValidationError") {
        // Mongoose validation error
        return res.status(400).json({
            status: "fail",
            errors: Object.values(err.errors).map(e => e.message)
        });
    }

    if (err.code === 11000) {
        // MongoDB duplicate key
        return res.status(409).json({
            status: "fail",
            message: "Duplicate field value"
        });
    }

    // 4. Send response
    res.status(statusCode).json({
        status: err.isOperational ? "fail" : "error",
        message: err.isOperational
            ? err.message
            : "Internal server error",  // Don't leak details!
        ...(process.env.NODE_ENV === "development" && {
            stack: err.stack
        })
    });
});

// ═══════════════════════════════════════════
// 6. UNHANDLED REJECTIONS (Process-level)
// ═══════════════════════════════════════════
process.on("unhandledRejection", (err) => {
    console.error("UNHANDLED REJECTION! Shutting down...");
    console.error(err);
    process.exit(1);
});

process.on("uncaughtException", (err) => {
    console.error("UNCAUGHT EXCEPTION! Shutting down...");
    console.error(err);
    process.exit(1);
});

app.listen(3000);
```

### 5️⃣ INTERVIEW ANSWER

> *"Express error-handling middleware is a special middleware with **four arguments** — `(err, req, res, next)` — that must be placed **after all routes**. When `next(error)` is called or an error is thrown synchronously, Express skips normal middleware and jumps to this handler. For async routes in Express 4, I use an **async wrapper** utility that catches promise rejections and forwards them to `next()`. I create a custom **AppError class** with status codes for operational errors. In the global handler, I differentiate between **operational errors** (validation, not found) and **programming errors** (bugs), sending detailed messages only for the former. In production, I never expose stack traces. I also listen for `unhandledRejection` and `uncaughtException` at the process level."*

📌 **One-liner:** 4 args `(err, req, res, next)` = error handler | Last in chain | `next(err)` to trigger

---

---

## 7. body-parser Middleware

### 1️⃣ WHAT

`body-parser` ek middleware hai jo **incoming request bodies** ko parse karta hai — JSON, URL-encoded form data, raw text, etc. — aur `req.body` pe available karata hai.

> **Important Update:** Express 4.16+ mein `body-parser` **built-in** ho gaya hai — alag se install karne ki zaroorat nahi!

```
Request Without body-parser:          Request With body-parser:
req.body = undefined ❌               req.body = { name: "Kanishk" } ✅
```

### 2️⃣ WHY

HTTP requests mein body **raw string/buffer** ke roop mein aata hai. Usse JavaScript object mein convert karna padta hai:

```
Raw HTTP Request Body:
'{"name":"Kanishk","age":25}'   ← String!

After Parsing:
{ name: "Kanishk", age: 25 }   ← JavaScript Object! ✅
```

### 3️⃣ HOW — Evolution

```
Express < 4.16:
  npm install body-parser
  const bodyParser = require("body-parser")
  app.use(bodyParser.json())

Express >= 4.16 (Current):
  app.use(express.json())          ← Same thing, built-in!
  app.use(express.urlencoded())    ← Same thing, built-in!
```

### 4️⃣ CODE

```javascript
const express = require("express");
const app = express();

// ═══════════════════════════════════════════
// MODERN WAY (Express 4.16+) — No npm install!
// ═══════════════════════════════════════════

// 1. JSON bodies
app.use(express.json());
// Parses: {"name": "Kanishk"}
// Content-Type: application/json

// 2. URL-encoded form data
app.use(express.urlencoded({ extended: true }));
// Parses: name=Kanishk&age=25
// Content-Type: application/x-www-form-urlencoded
// extended: true → nested objects support (uses qs library)
// extended: false → simple values only (uses querystring)

// 3. With size limit (Security!)
app.use(express.json({ limit: "10kb" }));
// Rejects bodies larger than 10KB → prevents DoS

// ═══════════════════════════════════════════
// LEGACY WAY (Still works, but unnecessary)
// ═══════════════════════════════════════════
// const bodyParser = require("body-parser");
// app.use(bodyParser.json());
// app.use(bodyParser.urlencoded({ extended: true }));
// app.use(bodyParser.raw());     // Buffer
// app.use(bodyParser.text());    // Plain text

// ═══════════════════════════════════════════
// USAGE IN ROUTES
// ═══════════════════════════════════════════

// JSON Request
app.post("/api/users", (req, res) => {
    const { name, email } = req.body;  // ✅ Parsed!
    console.log(typeof req.body);      // "object"
    res.json({ received: { name, email } });
});

// Form Data Request
app.post("/login", (req, res) => {
    const { username, password } = req.body;  // ✅ Parsed!
    res.json({ user: username });
});

// ═══════════════════════════════════════════
// RAW & TEXT (Special Cases)
// ═══════════════════════════════════════════

// Webhook from Stripe (needs raw body for signature verification)
app.post("/webhook/stripe",
    express.raw({ type: "application/json" }),
    (req, res) => {
        const signature = req.headers["stripe-signature"];
        const rawBody = req.body;  // Buffer!
        // stripe.webhooks.constructEvent(rawBody, signature, secret);
        res.json({ received: true });
    }
);

// Plain text
app.post("/log",
    express.text(),
    (req, res) => {
        console.log("Log:", req.body);  // String!
        res.send("Logged");
    }
);

app.listen(3000);
```

### Content-Type Mapping

| Content-Type | Parser | `req.body` Type |
|-------------|--------|----------------|
| `application/json` | `express.json()` | Object |
| `application/x-www-form-urlencoded` | `express.urlencoded()` | Object |
| `text/plain` | `express.text()` | String |
| `application/octet-stream` | `express.raw()` | Buffer |
| `multipart/form-data` | `multer` (not body-parser!) | `req.file` / `req.files` |

### 5️⃣ INTERVIEW ANSWER

> *"The `body-parser` middleware parses incoming request bodies and makes them available on `req.body`. Since **Express 4.16**, it's been **built into Express** — so `express.json()` and `express.urlencoded()` are the modern equivalents, and there's no need to install the separate `body-parser` package. I use `express.json()` for API requests and `express.urlencoded({ extended: true })` for HTML form submissions. I always set a **size limit** like `express.json({ limit: '10kb' })` to prevent denial-of-service attacks. One important note: `body-parser` does **not** handle `multipart/form-data` — for file uploads, I use **multer** instead."*

📌 **One-liner:** body-parser = raw body → `req.body` object | Built-in since Express 4.16 | Not for file uploads

---

---

## 8. CORS in Express

### 1️⃣ WHAT

**CORS (Cross-Origin Resource Sharing)** ek browser security mechanism hai jo **ek origin (domain) se doosre origin pe requests** ko control karta hai.

> **Simple:** Browser frontend (localhost:3000) ko backend (localhost:5000) pe request bhejne se **rok deta hai** — unless backend explicitly allow kare.

```
Same Origin (ALLOWED ✅):
Frontend: https://myapp.com
Backend:  https://myapp.com/api
→ Same protocol + domain + port = OK!

Cross Origin (BLOCKED ❌ by default):
Frontend: https://myapp.com
Backend:  https://api.myapp.com     ← Different subdomain!
Frontend: http://localhost:3000
Backend:  http://localhost:5000     ← Different port!
Frontend: https://myapp.com
Backend:  http://myapp.com          ← Different protocol!
```

### 2️⃣ WHY

**Security!** Bina CORS ke:
- ❌ Koi bhi malicious website aapke API ko access kar sakti hai
- ❌ CSRF attacks possible
- ❌ Unauthorized data access

CORS se:
- ✅ Server decide karta hai kaun access kar sakta hai
- ✅ Specific origins allow kar sakte ho
- ✅ Methods aur headers control kar sakte ho

### 3️⃣ HOW — CORS Flow

```
Browser                          Server
───────                          ──────

1. Preflight Request (OPTIONS):
   "Kya main https://myapp.com se
    POST /api/users kar sakta hoon?"
   ─────────────────────────────→

2. Server Response:
   "Haan, Access-Control-Allow-Origin:
    https://myapp.com"
   ←─────────────────────────────

3. Actual Request (POST):
   Browser ab actual request bhejta hai
   ─────────────────────────────→

4. Response:
   Data milta hai ✅
   ←─────────────────────────────
```

**CORS Headers:**

| Header | Purpose |
|--------|---------|
| `Access-Control-Allow-Origin` | Kaunsa origin allowed (`*` = all) |
| `Access-Control-Allow-Methods` | Kaunse methods (GET, POST, etc.) |
| `Access-Control-Allow-Headers` | Kaunse headers allowed |
| `Access-Control-Allow-Credentials` | Cookies bhej sakte ho? |
| `Access-Control-Max-Age` | Preflight cache duration |

### 4️⃣ CODE

```bash
npm install cors
```

```javascript
const express = require("express");
const cors = require("cors");
const app = express();

// ═══════════════════════════════════════════
// 1. ALLOW ALL ORIGINS (Development Only!)
// ═══════════════════════════════════════════
app.use(cors());
// Headers: Access-Control-Allow-Origin: *
// ⚠️ Production mein kabhi mat karo!

// ═══════════════════════════════════════════
// 2. SPECIFIC ORIGIN (Production!)
// ═══════════════════════════════════════════
app.use(cors({
    origin: "https://myapp.com",
    methods: ["GET", "POST", "PUT", "DELETE"],
    allowedHeaders: ["Content-Type", "Authorization"],
    credentials: true,    // Cookies allow
    maxAge: 86400         // Preflight cache: 24 hours
}));

// ═══════════════════════════════════════════
// 3. MULTIPLE ORIGINS
// ═══════════════════════════════════════════
const allowedOrigins = [
    "https://myapp.com",
    "https://admin.myapp.com",
    "http://localhost:3000"   // Development
];

app.use(cors({
    origin: (origin, callback) => {
        // origin undefined = same-origin requests (Postman, curl)
        if (!origin || allowedOrigins.includes(origin)) {
            callback(null, true);
        } else {
            callback(new Error("Not allowed by CORS"));
        }
    },
    credentials: true
}));

// ═══════════════════════════════════════════
// 4. DYNAMIC ORIGIN (Environment-based)
// ═══════════════════════════════════════════
const corsOptions = {
    origin: process.env.NODE_ENV === "production"
        ? "https://myapp.com"
        : "http://localhost:3000"
};

app.use(cors(corsOptions));

// ═══════════════════════════════════════════
// 5. ROUTE-SPECIFIC CORS
// ═══════════════════════════════════════════
// Public API — anyone can access
app.get("/api/public", cors(), (req, res) => {
    res.json({ data: "public" });
});

// Admin API — only admin panel
app.get("/api/admin",
    cors({ origin: "https://admin.myapp.com" }),
    (req, res) => {
        res.json({ data: "admin only" });
    }
);

// ═══════════════════════════════════════════
// 6. MANUAL CORS (Without Package)
// ═══════════════════════════════════════════
app.use((req, res, next) => {
    res.setHeader("Access-Control-Allow-Origin", "https://myapp.com");
    res.setHeader("Access-Control-Allow-Methods", "GET,POST,PUT,DELETE");
    res.setHeader("Access-Control-Allow-Headers", "Content-Type,Authorization");
    res.setHeader("Access-Control-Allow-Credentials", "true");

    // Handle preflight
    if (req.method === "OPTIONS") {
        return res.sendStatus(204);
    }

    next();
});

// ═══════════════════════════════════════════
// 7. ERROR HANDLING FOR CORS
// ═══════════════════════════════════════════
app.use((err, req, res, next) => {
    if (err.message === "Not allowed by CORS") {
        return res.status(403).json({ error: "CORS: Origin not allowed" });
    }
    next(err);
});

app.listen(5000, () => console.log("CORS-enabled server on 5000"));
```

### Common CORS Errors & Fixes

| Error | Cause | Fix |
|-------|-------|-----|
| `No 'Access-Control-Allow-Origin'` | CORS not configured | Add `cors()` middleware |
| `Origin not allowed` | Wrong origin in config | Add frontend URL to allowed list |
| `Preflight failed` | OPTIONS not handled | `cors()` handles it automatically |
| `Credentials not supported` | `credentials: true` but origin is `*` | Use specific origin, not `*` |
| `Header not allowed` | Custom header blocked | Add to `allowedHeaders` |

### 5️⃣ INTERVIEW ANSWER

> *"**CORS** is a browser security mechanism that restricts cross-origin HTTP requests. When a frontend at one origin tries to access an API at a different origin, the browser blocks it unless the server explicitly allows it via CORS headers. In Express, I use the **`cors`** package and configure it with **specific allowed origins** in production — never `*` with credentials. I set allowed methods, headers, and enable credentials when cookies are involved. For multi-origin setups, I use a **dynamic origin function** that checks against a whitelist. The package automatically handles **preflight OPTIONS requests**. CORS is a **server-side configuration** — the browser enforces it, but the server decides the policy."*

📌 **One-liner:** CORS = browser security | Server allows origins | `cors({ origin: "..." })` | Never `*` in production

---

---

## ⚡ Lightning Revision Sheet (5 Min Before Interview)

| # | Topic | One-Line Answer |
|---|-------|----------------|
| 1 | **Event-Driven** | Single thread + Event Loop + EventEmitter = non-blocking I/O |
| 2 | **Streams** | Data in chunks (Readable, Writable, Duplex, Transform) via `.pipe()` |
| 3 | **File Uploads** | Multer middleware | disk/memory storage | filter + limits |
| 4 | **Sync vs Async** | Sync = blocks thread ❌ | Async = non-blocking ✅ | Always async in routes |
| 5 | **Clustering** | N workers × N CPU cores × same port | PM2 in production |
| 6 | **Error Middleware** | 4 args `(err, req, res, next)` | Last in chain | `next(err)` |
| 7 | **body-parser** | Raw body → `req.body` | Built-in since Express 4.16 | Not for files |
| 8 | **CORS** | Browser security | `cors({ origin })` | Specific origins in prod |

---

## 🔗 Node.js Intermediate Concept Map

```
Node.js Core
├── Event-Driven Architecture
│   ├── EventEmitter (on/emit)
│   ├── Event Loop (phases)
│   └── libuv (thread pool)
│
├── Streams
│   ├── Readable → Writable (pipe)
│   ├── Transform (modify)
│   └── Duplex (both)
│
├── Sync vs Async
│   ├── Callbacks → Promises → async/await
│   └── Promise.all() (parallel)
│
├── Clustering
│   ├── cluster module (fork workers)
│   ├── PM2 (production)
│   └── Worker Threads (CPU tasks)
│
└── Express.js
    ├── File Uploads (multer)
    ├── Error Handling (4-arg middleware)
    ├── body-parser (express.json())
    └── CORS (cors package)
```

---
