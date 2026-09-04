# 🚀 Node.js & Express.js — Interview Prep Guide (Basic Level)

> **Level:** Basic
> **Format:** What → Why → How → Example → Interview Answer
> **Language:** Hinglish (understanding) + English (interview delivery)

---

## 📑 Table of Contents

| # | Topic | Difficulty | Frequency |
|---|-------|-----------|-----------|
| 1 | [What is Node.js?](#1-what-is-nodejs-and-why-is-it-used) | ⭐ | 🔥🔥🔥🔥🔥 |
| 2 | [npm & package.json](#2-what-is-npm-and-what-is-packagejson) | ⭐ | 🔥🔥🔥🔥 |
| 3 | [Middleware in Express](#3-explain-the-purpose-of-middleware-in-expressjs) | ⭐⭐ | 🔥🔥🔥🔥🔥 |
| 4 | [Basic Express Server](#4-how-do-you-create-a-basic-express-server) | ⭐ | 🔥🔥🔥🔥🔥 |
| 5 | [`app.get()` vs `app.post()`](#5-what-is-the-difference-between-appget-and-apppost) | ⭐ | 🔥🔥🔥🔥 |
| 6 | [Environment Variables](#6-how-do-you-handle-environment-variables-in-nodejs) | ⭐ | 🔥🔥🔥 |
| 7 | [`next()` in Middleware](#7-what-is-the-purpose-of-the-next-function-in-middleware) | ⭐⭐ | 🔥🔥🔥🔥 |
| 8 | [Serving Static Files](#8-how-do-you-serve-static-files-in-express) | ⭐ | 🔥🔥🔥 |

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

## 1. What is Node.js and Why is it Used?

### 1️⃣ WHAT

Node.js ek **open-source, cross-platform JavaScript runtime environment** hai jo **V8 engine** (Chrome ka engine) pe chalta hai aur JavaScript ko **browser ke bahar** — server pe — run karne deta hai.

> **Simple:** JavaScript ab sirf browser tak limited nahi — Node.js se server, CLI tools, desktop apps sab bana sakte ho.

```
Before Node.js:                    After Node.js:
┌──────────────┐                   ┌──────────────┐
│   Browser    │                   │   Browser    │
│   (JS runs)  │                   │   (JS runs)  │
└──────────────┘                   └──────────────┘
┌──────────────┐                   ┌──────────────┐
│   Server     │                   │   Server     │
│ (Java/Python │                   │  (JS runs!)  │
│  /PHP only)  │                   │   ✅ Node.js │
└──────────────┘                   └──────────────┘
```

### 2️⃣ WHY — Node.js kyun choose karein?

| Feature | Benefit | Real Impact |
|---------|---------|-------------|
| **Non-blocking I/O** | Async operations block nahi karte | 1000s concurrent connections handle |
| **Single Language** | Frontend + Backend = JavaScript | Team efficiency ↑, context switch ↓ |
| **V8 Engine** | JIT compilation → blazing fast | High performance |
| **npm Ecosystem** | 2M+ packages available | Don't reinvent the wheel |
| **Event-Driven** | Events pe react karta hai | Real-time apps (chat, gaming) |
| **Lightweight** | Microservices ke liye perfect | Scalable architecture |

### 3️⃣ HOW — Architecture

```
┌─────────────────────────────────────┐
│         YOUR APPLICATION            │
│     (JavaScript Code / Express)     │
├─────────────────────────────────────┤
│         NODE.JS RUNTIME             │
│  ┌─────────┐  ┌─────────────────┐  │
│  │  V8     │  │  libuv          │  │
│  │  Engine │  │  (Event Loop +  │  │
│  │  (JS →  │  │   Thread Pool)  │  │
│  │  Machine│  │                 │  │
│  │  Code)  │  │  ┌───────────┐  │  │
│  └─────────┘  │  │ File I/O  │  │  │
│               │  │ Network   │  │  │
│               │  │ DNS       │  │  │
│               │  └───────────┘  │  │
│               └─────────────────┘  │
├─────────────────────────────────────┤
│         OPERATING SYSTEM            │
└─────────────────────────────────────┘
```

**Key Components:**
- **V8 Engine:** JavaScript code ko machine code mein compile karta hai
- **libuv:** Event loop, async I/O, thread pool handle karta hai
- **C/C++ Bindings:** OS-level operations (file system, network) access karta hai

### 4️⃣ CODE

```javascript
// ── Node.js Hello World (No browser needed!) ──
// Run: node app.js

const http = require("http");

const server = http.createServer((req, res) => {
    res.writeHead(200, { "Content-Type": "text/plain" });
    res.end("Hello from Node.js Server! 🚀");
});

server.listen(3000, () => {
    console.log("Server running at http://localhost:3000");
});

// ── Node.js can also do: ──
const fs = require("fs");
const path = require("path");
const os = require("os");

// Read files
const data = fs.readFileSync("./data.txt", "utf-8");

// OS info
console.log("Platform:", os.platform());   // win32 / linux / darwin
console.log("CPUs:", os.cpus().length);    // 8

// Path handling
console.log(path.join(__dirname, "uploads", "file.txt"));
```

### 5️⃣ INTERVIEW ANSWER

> *"Node.js is an **open-source, cross-platform JavaScript runtime** built on Chrome's **V8 engine** that allows JavaScript to run outside the browser — primarily on the server side. It uses an **event-driven, non-blocking I/O model**, making it highly efficient for handling thousands of concurrent connections. Key advantages include using a **single language** across the full stack, access to the massive **npm ecosystem**, and excellent performance for **I/O-heavy and real-time applications** like chat apps, APIs, and streaming services. It's not ideal for CPU-intensive tasks, where multi-threaded languages might be better."*

📌 **One-liner:** Node.js = JavaScript on the server | V8 + Event Loop + Non-blocking I/O

---

---

## 2. What is npm and What is package.json?

### 1️⃣ WHAT

| Term | Full Form | Definition |
|------|-----------|------------|
| **npm** | **N**ode **P**ackage **M**anager | JavaScript ka **package manager** — libraries install, share, aur manage karta hai |
| **package.json** | — | Project ka **identity card** — metadata, dependencies, scripts sab isme hota hai |

> **Analogy:** npm = Play Store / App Store for JavaScript | package.json = App ka manifest file

### 2️⃣ WHY

- **Code reuse** — lakhon ready-made packages (express, mongoose, bcrypt, etc.)
- **Dependency management** — kaunsa version kaunsa package use karta hai
- **Scripts** — `npm start`, `npm test`, `npm run dev` jaise shortcuts
- **Team collaboration** — `npm install` se poora setup ek command mein

### 3️⃣ HOW — npm Workflow

```
Step 1: npm init -y
        ↓
   package.json created 📄

Step 2: npm install express
        ↓
   ├── express downloaded to node_modules/ 📦
   ├── package.json updated (dependencies)
   └── package-lock.json created (exact versions)

Step 3: require("express") in code
        ↓
   Node.js looks in node_modules/ → finds express → loads it ✅
```

### 4️⃣ CODE

```json
// ── package.json (Example) ──
{
  "name": "my-api",
  "version": "1.0.0",
  "description": "My first Express API",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js",
    "test": "jest"
  },
  "dependencies": {
    "express": "^4.18.2",
    "mongoose": "^7.5.0",
    "dotenv": "^16.3.1"
  },
  "devDependencies": {
    "nodemon": "^3.0.1",
    "jest": "^29.7.0"
  },
  "keywords": ["api", "express"],
  "author": "Kanishk",
  "license": "MIT"
}
```

```bash
# ── Important npm Commands ──

npm init -y              # package.json create (default values)
npm install express      # dependency install (production)
npm install -D nodemon   # devDependency install
npm install -g pm2       # global install
npm uninstall express    # remove package
npm update               # update all packages
npm start                # runs "start" script
npm run dev              # runs "dev" script
npm list                 # show installed packages
npm outdated             # check outdated packages
```

```
📁 Project Structure:
my-api/
├── node_modules/       ← Installed packages (DON'T commit to Git!)
├── package.json        ← Project metadata + dependencies
├── package-lock.json   ← Exact version lock (DO commit!)
├── .gitignore          ← node_modules/ ko ignore karo
└── server.js           ← Your code
```

**dependencies vs devDependencies:**

| Type | When | Examples |
|------|------|----------|
| `dependencies` | Production mein chahiye | express, mongoose, bcrypt |
| `devDependencies` | Sirf development mein | nodemon, jest, eslint |

**Version Symbols:**

| Symbol | Meaning | Example |
|--------|---------|---------|
| `^4.18.2` | Minor updates OK (4.x.x) | `^4.18.2` → `4.19.0` ✅, `5.0.0` ❌ |
| `~4.18.2` | Patch updates OK (4.18.x) | `~4.18.2` → `4.18.3` ✅, `4.19.0` ❌ |
| `4.18.2` | Exact version only | `4.18.2` → only `4.18.2` |

### 5️⃣ INTERVIEW ANSWER

> *"**npm** is the default **package manager** for Node.js, used to install, manage, and share JavaScript packages. **package.json** is the project's manifest file that contains metadata like project name, version, scripts, and most importantly, the list of **dependencies** and **devDependencies**. When someone clones the project, a simple `npm install` reads package.json and installs all required packages. The **package-lock.json** ensures consistent dependency versions across all environments. I use `dependencies` for production packages and `devDependencies` for development-only tools like nodemon and jest."*

📌 **One-liner:** npm = JS package store | package.json = project ID card with dependencies & scripts

---

---

## 3. Explain the Purpose of Middleware in Express.js

> 🏆 **Most asked Express question!**

### 1️⃣ WHAT

Middleware ek **function** hai jo **request aur response ke beech** mein kaam karta hai. Ye request ko **process, modify, validate, ya reject** kar sakta hai before it reaches the final route handler.

> **Analogy:** Airport security check — aap (request) plane (response) tak pahunchne se pehle security (middleware) se guzarte ho.

```
Request Flow:

Client Request
     ↓
┌─────────────┐
│ Middleware 1 │ → Logging (req ka record rakho)
└──────┬──────┘
       ↓
┌─────────────┐
│ Middleware 2 │ → Auth Check (token valid hai?)
└──────┬──────┘
       ↓
┌─────────────┐
│ Middleware 3 │ → Body Parser (JSON parse karo)
└──────┬──────┘
       ↓
┌─────────────┐
│ Route Handler│ → Final logic (DB query, response)
└──────┬──────┘
       ↓
  Response to Client
```

### 2️⃣ WHY

Bina middleware ke:
- Har route mein **duplicate code** likhna padega (auth, logging, parsing)
- Code **messy** aur **unmaintainable** ho jayega
- **Separation of concerns** nahi hoga

Middleware se:
- ✅ **Reusable** logic ek jagah
- ✅ **Clean** route handlers
- ✅ **Chain** bana sakte ho (multiple middlewares)
- ✅ **Third-party** middleware use kar sakte ho (cors, helmet, morgan)

### 3️⃣ HOW — Types of Middleware

| Type | Scope | Example |
|------|-------|---------|
| **Application-level** | Saari requests pe | `app.use(logger)` |
| **Router-level** | Specific router pe | `router.use(auth)` |
| **Route-specific** | Ek route pe | `app.get("/admin", isAdmin, handler)` |
| **Built-in** | Express ke saath aata hai | `express.json()`, `express.static()` |
| **Third-party** | npm se install | `cors()`, `helmet()`, `morgan()` |
| **Error-handling** | Errors catch karta hai | `(err, req, res, next)` — **4 arguments!** |

### 4️⃣ CODE

```javascript
const express = require("express");
const app = express();

// ═══════════════════════════════════════════
// 1. BUILT-IN MIDDLEWARE
// ═══════════════════════════════════════════
app.use(express.json());           // JSON body parse karo
app.use(express.urlencoded());     // Form data parse karo

// ═══════════════════════════════════════════
// 2. CUSTOM APPLICATION-LEVEL MIDDLEWARE
// ═══════════════════════════════════════════
// Har request pe chalega
app.use((req, res, next) => {
    console.log(`[${new Date().toISOString()}] ${req.method} ${req.url}`);
    next();  // ← Bahut zaroori! Warna request yahi atak jayegi
});

// ═══════════════════════════════════════════
// 3. THIRD-PARTY MIDDLEWARE
// ═══════════════════════════════════════════
const cors = require("cors");
const helmet = require("helmet");

app.use(cors());       // Cross-origin requests allow
app.use(helmet());     // Security headers add

// ═══════════════════════════════════════════
// 4. ROUTE-SPECIFIC MIDDLEWARE
// ═══════════════════════════════════════════
const authenticate = (req, res, next) => {
    const token = req.headers.authorization;

    if (!token) {
        return res.status(401).json({ error: "No token provided" });
    }

    // Token verify logic...
    req.user = { id: 1, name: "Kanishk" };  // Attach user to request
    next();
};

// Sirf /profile pe auth lagega
app.get("/profile", authenticate, (req, res) => {
    res.json({ user: req.user });
});

// Public route — no auth needed
app.get("/public", (req, res) => {
    res.json({ message: "Anyone can access" });
});

// ═══════════════════════════════════════════
// 5. ERROR-HANDLING MIDDLEWARE (4 args!)
// ═══════════════════════════════════════════
app.use((err, req, res, next) => {
    console.error("Error:", err.message);
    res.status(500).json({ error: "Internal Server Error" });
});

app.listen(3000, () => console.log("Server running"));
```

### 5️⃣ INTERVIEW ANSWER

> *"Middleware in Express is a **function that has access to the request, response, and the next function** in the request-response cycle. It can execute code, modify the request/response objects, end the cycle, or call `next()` to pass control to the next middleware. There are five types: **application-level, router-level, route-specific, built-in** (like `express.json()`), and **error-handling** middleware (which takes four arguments). Middleware enables **separation of concerns** — I use it for logging, authentication, body parsing, CORS, and error handling, keeping route handlers clean and focused on business logic."*

📌 **One-liner:** Middleware = gatekeeper between request and response | `req, res, next` | Chainable

---

---

## 4. How Do You Create a Basic Express Server?

### 1️⃣ WHAT

Express.js Node.js ka **minimalist web framework** hai jo HTTP server banana, routes define karna, aur middleware use karna **bahut easy** banata hai.

> **Analogy:** Node.js = Raw bricks | Express = Ready-made house structure

### 2️⃣ WHY

Raw Node.js `http` module se server banana **verbose** hai:

```javascript
// ❌ Raw Node.js (15+ lines for simple routing)
const http = require("http");
const server = http.createServer((req, res) => {
    if (req.method === "GET" && req.url === "/") {
        res.writeHead(200, { "Content-Type": "text/html" });
        res.end("<h1>Home</h1>");
    } else if (req.method === "GET" && req.url === "/about") {
        // ... more if-else chains 😩
    }
});
```

Express se same kaam **3-4 lines** mein:

```javascript
// ✅ Express (clean!)
app.get("/", (req, res) => res.send("<h1>Home</h1>"));
app.get("/about", (req, res) => res.send("<h1>About</h1>"));
```

### 3️⃣ HOW — Step by Step

```
Step 1: mkdir my-server && cd my-server
Step 2: npm init -y
Step 3: npm install express
Step 4: Create server.js
Step 5: node server.js
Step 6: Open http://localhost:3000 ✅
```

### 4️⃣ CODE

```javascript
// ═══════════════════════════════════════════
// server.js — Complete Basic Express Server
// ═══════════════════════════════════════════

// Step 1: Import Express
const express = require("express");

// Step 2: Create App Instance
const app = express();

// Step 3: Port Configuration
const PORT = process.env.PORT || 3000;

// Step 4: Built-in Middleware
app.use(express.json());  // Parse JSON request bodies

// Step 5: Define Routes
// GET - Home
app.get("/", (req, res) => {
    res.json({
        message: "Welcome to my Express API! 🚀",
        endpoints: ["/users", "/users/:id"]
    });
});

// GET - All Users
app.get("/users", (req, res) => {
    const users = [
        { id: 1, name: "Kanishk" },
        { id: 2, name: "Rahul" }
    ];
    res.json(users);
});

// GET - Single User (Route Parameter)
app.get("/users/:id", (req, res) => {
    const userId = req.params.id;
    res.json({ id: userId, name: "Kanishk" });
});

// POST - Create User
app.post("/users", (req, res) => {
    const { name, email } = req.body;

    if (!name || !email) {
        return res.status(400).json({ error: "Name and email required" });
    }

    const newUser = { id: 3, name, email };
    res.status(201).json(newUser);
});

// Step 6: 404 Handler (Unknown Routes)
app.use((req, res) => {
    res.status(404).json({ error: "Route not found" });
});

// Step 7: Error Handler
app.use((err, req, res, next) => {
    console.error(err.stack);
    res.status(500).json({ error: "Something went wrong!" });
});

// Step 8: Start Server
app.listen(PORT, () => {
    console.log(`✅ Server running at http://localhost:${PORT}`);
});
```

```bash
# ── Run the server ──
node server.js

# ── Test with curl ──
curl http://localhost:3000/
curl http://localhost:3000/users
curl -X POST http://localhost:3000/users \
     -H "Content-Type: application/json" \
     -d '{"name":"Kanishk","email":"k@test.com"}'
```

### 5️⃣ INTERVIEW ANSWER

> *"To create a basic Express server, I first install Express via npm, then import it and create an app instance using `express()`. I configure middleware like `express.json()` for body parsing, define routes using methods like `app.get()` and `app.post()`, add a 404 handler for unknown routes, and an error-handling middleware at the end. Finally, I call `app.listen()` with a port number to start the server. I typically use `process.env.PORT` for the port to make it deployment-ready."*

📌 **One-liner:** `express()` → middleware → routes → `app.listen(PORT)`

---

---

## 5. What is the Difference Between `app.get()` and `app.post()`?

### 1️⃣ WHAT

Dono **HTTP methods** ke corresponding Express route handlers hain:

| Method | Purpose | Data Location | Idempotent? |
|--------|---------|---------------|-------------|
| `app.get()` | **Read / Fetch** data | URL query params (`?key=value`) | ✅ Yes (same result every time) |
| `app.post()` | **Create / Send** data | Request body (`req.body`) | ❌ No (creates new resource) |

### 2️⃣ WHY — HTTP Method Semantics

HTTP protocol ke **CRUD operations** map hote hain:

```
CRUD       HTTP Method    Express          SQL
────       ───────────    ───────          ───
Create  →  POST        →  app.post()    →  INSERT
Read    →  GET         →  app.get()     →  SELECT
Update  →  PUT/PATCH   →  app.put()     →  UPDATE
Delete  →  DELETE      →  app.delete()  →  DELETE
```

### 3️⃣ HOW — Key Differences

| Feature | `app.get()` | `app.post()` |
|---------|-------------|--------------|
| **Use Case** | Data retrieve karna | Data create/submit karna |
| **Data Source** | `req.query`, `req.params` | `req.body` |
| **Data Visibility** | URL mein dikhta hai ⚠️ | Body mein hidden ✅ |
| **Caching** | Browser cache karta hai | Cache nahi hota |
| **Bookmark** | Bookmark kar sakte ho | Nahi kar sakte |
| **Data Size** | Limited (~2048 chars URL) | Unlimited (body) |
| **Security** | Sensitive data mat bhejo | Safer for sensitive data |
| **Browser** | Address bar se directly call | Form submit / fetch / axios |

### 4️⃣ CODE

```javascript
const express = require("express");
const app = express();
app.use(express.json());

// ═══════════════════════════════════════════
// GET — Read Data
// ═══════════════════════════════════════════

// Route Parameters
app.get("/users/:id", (req, res) => {
    const userId = req.params.id;  // URL se aaya
    res.json({ id: userId, name: "Kanishk" });
});
// Call: GET /users/123

// Query Parameters
app.get("/search", (req, res) => {
    const { q, page, limit } = req.query;  // ?q=js&page=1&limit=10
    res.json({ search: q, page, limit });
});
// Call: GET /search?q=javascript&page=1&limit=10

// ═══════════════════════════════════════════
// POST — Create Data
// ═══════════════════════════════════════════
app.post("/users", (req, res) => {
    const { name, email, password } = req.body;  // Body se aaya

    // Validation
    if (!name || !email || !password) {
        return res.status(400).json({ error: "All fields required" });
    }

    // Simulate DB insert
    const newUser = { id: Date.now(), name, email };

    res.status(201).json({
        message: "User created successfully",
        user: newUser
    });
});
// Call: POST /users  Body: {"name":"Kanishk","email":"k@test.com","password":"123"}

// ═══════════════════════════════════════════
// COMPARISON IN ACTION
// ═══════════════════════════════════════════

// ❌ WRONG: Sensitive data in GET
app.get("/login", (req, res) => {
    const { username, password } = req.query;
    // URL: /login?username=admin&password=secret123  😱 VISIBLE!
});

// ✅ RIGHT: Sensitive data in POST
app.post("/login", (req, res) => {
    const { username, password } = req.body;
    // Body mein hidden ✅
});
```

### 5️⃣ INTERVIEW ANSWER

> *"`app.get()` and `app.post()` correspond to the HTTP **GET** and **POST** methods. **GET** is used to **retrieve data** — parameters are sent via URL (query strings or route params), it's cacheable, bookmarkable, and idempotent. **POST** is used to **create or submit data** — data is sent in the **request body**, it's not cached, and it's safer for sensitive information like passwords. In RESTful APIs, GET maps to **Read** operations and POST maps to **Create** operations. I always use POST for sensitive data since GET parameters are visible in the URL and browser history."*

📌 **One-liner:** GET = read (URL data, cacheable) | POST = create (body data, secure)

---

---

## 6. How Do You Handle Environment Variables in Node.js?

### 1️⃣ WHAT

Environment variables **OS-level key-value pairs** hain jo application ke **configuration** store karte hain — jaise database URL, API keys, port numbers, secrets.

> **Rule:** Sensitive data **kabhi code mein hardcode mat karo!**

```
❌ BAD:  const dbUrl = "mongodb://user:pass123@cluster.mongodb.net";
✅ GOOD: const dbUrl = process.env.MONGODB_URI;
```

### 2️⃣ WHY

| Reason | Explanation |
|--------|-------------|
| **Security** | API keys, passwords code mein nahi dikhte |
| **Environment Separation** | Dev, Staging, Production ke alag configs |
| **12-Factor App** | Industry best practice |
| **Git Safety** | `.env` file `.gitignore` mein → secrets safe |

### 3️⃣ HOW — Setup Flow

```
Step 1: npm install dotenv
        ↓
Step 2: Create .env file (root directory)
        ↓
        PORT=3000
        DB_URL=mongodb://localhost:27017/mydb
        JWT_SECRET=supersecretkey123
        ↓
Step 3: Add .env to .gitignore!
        ↓
Step 4: Load in code: require("dotenv").config()
        ↓
Step 5: Access: process.env.PORT
```

### 4️⃣ CODE

```bash
# ── .env file (NEVER commit to Git!) ──
PORT=3000
NODE_ENV=development
MONGODB_URI=mongodb://localhost:27017/myapp
JWT_SECRET=my_super_secret_jwt_key_123
API_KEY=sk-abc123xyz
EMAIL_USER=admin@myapp.com
EMAIL_PASS=apppassword123
```

```bash
# ── .gitignore ──
node_modules/
.env          # ← MOST IMPORTANT LINE!
.env.local
```

```javascript
// ═══════════════════════════════════════════
// server.js
// ═══════════════════════════════════════════

// Step 1: Load .env file (TOP of file!)
require("dotenv").config();

const express = require("express");
const app = express();

// Step 2: Access environment variables
const PORT = process.env.PORT || 3000;
const NODE_ENV = process.env.NODE_ENV || "development";
const DB_URL = process.env.MONGODB_URI;
const JWT_SECRET = process.env.JWT_SECRET;

// Step 3: Use them
console.log(`Running in ${NODE_ENV} mode`);

// Database connection (example)
// mongoose.connect(DB_URL);

// JWT signing (example)
// const token = jwt.sign({ userId: 1 }, JWT_SECRET);

app.get("/", (req, res) => {
    res.json({
        environment: NODE_ENV,
        port: PORT
        // ⚠️ NEVER send secrets in response!
    });
});

app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
});

// ═══════════════════════════════════════════
// CONFIG PATTERN (Production Best Practice)
// ═══════════════════════════════════════════
// config.js
module.exports = {
    port: process.env.PORT || 3000,
    nodeEnv: process.env.NODE_ENV || "development",
    db: {
        uri: process.env.MONGODB_URI,
    },
    jwt: {
        secret: process.env.JWT_SECRET,
        expiresIn: process.env.JWT_EXPIRES || "7d",
    },
    isProduction: process.env.NODE_ENV === "production",
};

// Usage in other files:
// const config = require("./config");
// mongoose.connect(config.db.uri);
```

```bash
# ── Different Environments ──

# Development
NODE_ENV=development node server.js

# Production (Linux/Mac)
NODE_ENV=production PORT=8080 node server.js

# Using .env files per environment
# .env.development
# .env.staging
# .env.production
```

### 5️⃣ INTERVIEW ANSWER

> *"I handle environment variables using the **`dotenv`** package. I create a `.env` file in the project root with key-value pairs like `PORT`, `DB_URL`, and `JWT_SECRET`, and I **always add `.env` to `.gitignore`** to prevent committing secrets. At the top of my entry file, I call `require('dotenv').config()` to load these variables, and then access them via `process.env.VARIABLE_NAME`. For production, environment variables are typically set directly on the hosting platform like AWS, Heroku, or Docker. I also create a centralized **config module** that exports all environment variables with sensible defaults."*

📌 **One-liner:** `.env` file + `dotenv` package + `process.env.KEY` + `.gitignore` = safe config

---

---

## 7. What is the Purpose of the `next()` Function in Middleware?

### 1️⃣ WHAT

`next()` ek **callback function** hai jo Express middleware ko batata hai:

> **"Mera kaam ho gaya, ab agle middleware/route handler ko control bhejo."**

Bina `next()` ke → request **yahin atak jayegi** → client ko kabhi response nahi milega → **timeout!**

```
Middleware Chain:

Request → [MW1] →next()→ [MW2] →next()→ [MW3] →next()→ [Route Handler] → Response
              ↑               ↑               ↑
         "Mera kaam       "Mera kaam      "Mera kaam
          done, aage       done, aage       done, aage
          bhejo"           bhejo"           bhejo"
```

### 2️⃣ WHY

Express mein middleware **sequentially** execute hote hain. `next()` chain ko **move forward** karta hai. Iske bina:

- ❌ Request hang ho jayegi
- ❌ Response kabhi nahi jayega
- ❌ Client timeout error dega

### 3️⃣ HOW — 3 Ways to Use `next()`

| Usage | What Happens | When to Use |
|-------|-------------|-------------|
| `next()` | Agle middleware pe bhejo | Normal flow continue |
| `next("route")` | Agle **route** pe skip karo (middleware skip) | Conditional routing |
| `next(error)` | **Error handler** pe bhejo | Kuch galat hua |

### 4️⃣ CODE

```javascript
const express = require("express");
const app = express();

// ═══════════════════════════════════════════
// 1. next() — Normal Flow
// ═══════════════════════════════════════════
const logger = (req, res, next) => {
    console.log(`${req.method} ${req.url}`);
    next();  // ← Agle middleware/handler pe bhejo
};

const authCheck = (req, res, next) => {
    const token = req.headers.authorization;
    if (!token) {
        return res.status(401).json({ error: "Unauthorized" });
        // ↑ Yahan next() nahi call kiya → chain RUK gayi → response bheja
    }
    req.user = { id: 1 };
    next();  // ← Token valid → aage bhejo
};

app.get("/dashboard", logger, authCheck, (req, res) => {
    res.json({ message: `Welcome user ${req.user.id}` });
});

// ═══════════════════════════════════════════
// 2. next(error) — Error Handling
// ═══════════════════════════════════════════
const validateUser = (req, res, next) => {
    const { email } = req.body;

    if (!email || !email.includes("@")) {
        // Error create karo aur next() mein pass karo
        const error = new Error("Invalid email format");
        error.statusCode = 400;
        return next(error);  // ← Seedha error handler pe jayega!
    }

    next();
};

app.post("/register", validateUser, (req, res) => {
    res.json({ message: "Registered!" });
});

// Error Handler (4 arguments — Express pehchanta hai!)
app.use((err, req, res, next) => {
    const status = err.statusCode || 500;
    res.status(status).json({
        error: err.message || "Internal Server Error"
    });
});

// ═══════════════════════════════════════════
// 3. What Happens WITHOUT next()?
// ═══════════════════════════════════════════
app.get("/hang", (req, res, next) => {
    console.log("This runs...");
    // next() nahi call kiya
    // res bhi nahi bheja
    // ⚠️ Client wait karta rahega... TIMEOUT!
});

// ═══════════════════════════════════════════
// 4. next("route") — Skip to Next Route
// ═══════════════════════════════════════════
app.get("/skip",
    (req, res, next) => {
        if (req.query.skip === "true") {
            return next("route");  // Agle route pe jump!
        }
        res.send("First handler");
    },
    (req, res) => {
        res.send("Second handler (skipped if next('route'))");
    }
);

app.get("/skip", (req, res) => {
    res.send("Fallback route handler");
});
```

### Common Mistakes

| Mistake | Result | Fix |
|---------|--------|-----|
| `next()` bhool gaye | Request hang | Har path pe `next()` ya `res.send()` |
| `next()` ke baad code | Double response error | `return next()` use karo |
| Error handler mein 3 args | Normal middleware treat hoga | **4 args** `(err, req, res, next)` |
| Async mein `next()` nahi | Unhandled rejection | `try/catch` + `next(error)` |

```javascript
// ❌ WRONG: Double response
app.use((req, res, next) => {
    next();
    res.send("Oops");  // Error! Headers already sent
});

// ✅ RIGHT: return next()
app.use((req, res, next) => {
    return next();     // Function exit → no further execution
});
```

### 5️⃣ INTERVIEW ANSWER

> *"The `next()` function is a **callback** provided by Express that passes control to the **next middleware or route handler** in the chain. Without calling `next()`, the request gets stuck and the client eventually times out. There are three ways to use it: `next()` for normal flow, `next(error)` to jump to the error-handling middleware, and `next('route')` to skip to the next matching route. A critical best practice is to always use `return next()` to prevent code execution after the call, which could cause 'headers already sent' errors. In async middleware, I wrap logic in try/catch and pass errors to `next(error)`."*

📌 **One-liner:** `next()` = "mera kaam done, aage bhejo" | `next(err)` = error handler | `return next()` = safe

---

---

## 8. How Do You Serve Static Files in Express?

### 1️⃣ WHAT

Static files woh files hain jo **server pe bina kisi processing ke directly client ko bheji** jaati hain — jaise HTML, CSS, JavaScript, images, fonts, PDFs.

> Express ka **built-in middleware** `express.static()` ye kaam karta hai.

```
Static Files Examples:
📁 public/
├── 📄 index.html
├── 🎨 style.css
├── ⚡ app.js
├── 🖼️ logo.png
├── 📱 favicon.ico
└── 📂 fonts/
    └── roboto.woff2
```

### 2️⃣ WHY

- Frontend assets (HTML/CSS/JS) serve karne ke liye
- Uploaded images/files serve karne ke liye
- Documentation / PDF serve karne ke liye
- Bina kisi route handler ke directly files deliver hoti hain → **fast**

### 3️⃣ HOW — Setup

```
Step 1: Create a folder (e.g., "public")
Step 2: Put static files inside
Step 3: app.use(express.static("public"))
Step 4: Files accessible at http://localhost:3000/filename
```

```
URL Mapping:

File Location              URL
─────────────              ───
public/index.html    →    /index.html  (or just /)
public/style.css     →    /style.css
public/images/cat.jpg →   /images/cat.jpg
public/js/app.js     →    /js/app.js
```

### 4️⃣ CODE

```javascript
const express = require("express");
const path = require("path");
const app = express();

// ═══════════════════════════════════════════
// 1. BASIC STATIC SERVING
// ═══════════════════════════════════════════
// "public" folder ki saari files directly serve hongi
app.use(express.static("public"));

// Now accessible:
// http://localhost:3000/index.html
// http://localhost:3000/style.css
// http://localhost:3000/images/logo.png

// ═══════════════════════════════════════════
// 2. VIRTUAL PATH PREFIX
// ═══════════════════════════════════════════
// Files "public" mein hain, but URL mein /static prefix lagega
app.use("/static", express.static("public"));

// Now accessible:
// http://localhost:3000/static/style.css
// http://localhost:3000/static/images/logo.png

// ═══════════════════════════════════════════
// 3. MULTIPLE STATIC FOLDERS
// ═══════════════════════════════════════════
app.use(express.static("public"));
app.use(express.static("uploads"));  // User uploaded files
app.use("/docs", express.static("documentation"));

// ═══════════════════════════════════════════
// 4. ABSOLUTE PATH (Production Best Practice)
// ═══════════════════════════════════════════
// __dirname = current file ka directory
const publicPath = path.join(__dirname, "public");
app.use(express.static(publicPath));

// Why? Relative paths break when you run from different directories

// ═══════════════════════════════════════════
// 5. WITH OPTIONS (Caching, Max Age)
// ═══════════════════════════════════════════
app.use(express.static("public", {
    maxAge: "1d",           // Cache for 1 day
    etag: true,             // Enable ETag for caching
    dotfiles: "ignore",     // Ignore .hidden files
    index: "index.html",    // Default file
    extensions: ["html"],   // Try .html extension
}));

// ═══════════════════════════════════════════
// 6. COMPLETE EXAMPLE
// ═══════════════════════════════════════════
const PORT = 3000;

// Static files
app.use(express.static(path.join(__dirname, "public")));

// API routes
app.get("/api/users", (req, res) => {
    res.json([{ id: 1, name: "Kanishk" }]);
});

// SPA Fallback (React/Angular/Vue ke liye)
// Agar koi route match nahi hua → index.html bhejo
app.get("*", (req, res) => {
    res.sendFile(path.join(__dirname, "public", "index.html"));
});

app.listen(PORT, () => {
    console.log(`Server + Static files at http://localhost:${PORT}`);
});
```

```
📁 Final Project Structure:
my-app/
├── public/                ← Static files
│   ├── index.html
│   ├── style.css
│   ├── app.js
│   └── images/
│       └── logo.png
├── uploads/               ← User uploads
│   └── avatar.jpg
├── server.js
├── package.json
└── .gitignore
```

### 5️⃣ INTERVIEW ANSWER

> *"Express provides a **built-in middleware** called `express.static()` to serve static files like HTML, CSS, JavaScript, and images. I pass the directory name — preferably using `path.join(__dirname, 'public')` for absolute paths — and all files in that directory become accessible via their relative URL. I can also add a **virtual path prefix** like `/static` for cleaner URLs, serve **multiple directories**, and configure options like **caching with maxAge**. For Single Page Applications, I add a catch-all route `app.get('*')` that sends `index.html` for any unmatched route, letting the frontend router handle navigation."*

📌 **One-liner:** `express.static("public")` = folder ki files directly URL pe serve | Built-in middleware

---

---

## ⚡ Lightning Revision Sheet (5 Min Before Interview)

| # | Topic | One-Line Answer |
|---|-------|----------------|
| 1 | **Node.js** | JS runtime on V8, non-blocking I/O, event-driven, server-side JS |
| 2 | **npm & package.json** | npm = JS package manager | package.json = project metadata + dependencies |
| 3 | **Middleware** | Function between req & res — `req, res, next` — logging, auth, parsing |
| 4 | **Express Server** | `express()` → middleware → routes → `app.listen(PORT)` |
| 5 | **GET vs POST** | GET = read (URL, cacheable) | POST = create (body, secure) |
| 6 | **Env Variables** | `.env` + `dotenv` + `process.env.KEY` + `.gitignore` |
| 7 | **next()** | "Aage bhejo" | `next()` = continue | `next(err)` = error handler |
| 8 | **Static Files** | `express.static("public")` = serve HTML/CSS/JS/images directly |

---

## 🔗 Node.js Concept Map

```
Node.js Runtime
    │
    ├── V8 Engine (JS → Machine Code)
    ├── Event Loop (Non-blocking I/O)
    ├── npm (Package Management)
    │     └── package.json (Dependencies)
    │
    └── Express.js (Web Framework)
          │
          ├── app.get/post/put/delete (Routes)
          ├── Middleware Chain
          │     ├── Built-in (express.json, express.static)
          │     ├── Custom (auth, logger)
          │     ├── Third-party (cors, helmet)
          │     └── Error Handler (4 args)
          ├── next() (Chain Control)
          ├── Environment Variables (dotenv)
          └── Static Files (express.static)
```

