# 🚀 JavaScript & Programming Fundamentals — Advanced Level

> **Level:** Advanced
> **Format:** What → Why → How → Example → Interview Answer
> **Prerequisite:** Basic + Intermediate Level concepts

---

## 📑 Table of Contents

| # | Topic | Difficulty | Frequency |
|---|-------|-----------|-----------|
| 1 | [Debouncing & Throttling](#1-debouncing--throttling) | ⭐⭐⭐ | 🔥🔥🔥🔥🔥 |
| 2 | [Web Workers](#2-web-workers) | ⭐⭐⭐ | 🔥🔥🔥 |
| 3 | [Garbage Collection](#3-garbage-collection) | ⭐⭐⭐⭐ | 🔥🔥🔥 |
| 4 | [Currying](#4-currying) | ⭐⭐⭐ | 🔥🔥🔥🔥 |
| 5 | [Generators & Iterators](#5-generators--iterators) | ⭐⭐⭐⭐ | 🔥🔥🔥 |
| 6 | [Deep Clone](#6-deep-clone-of-an-object) | ⭐⭐⭐ | 🔥🔥🔥🔥 |
| 7 | [Memory Leaks](#7-memory-leaks) | ⭐⭐⭐⭐ | 🔥🔥🔥🔥 |
| 8 | [Microtasks vs Macrotasks](#8-microtasks-vs-macrotasks) | ⭐⭐⭐ | 🔥🔥🔥🔥 |

---

## 🧠 Answer Framework (Reminder)

```
1️⃣  WHAT  →  "Yeh kya hai?" (crisp definition)
2️⃣  WHY   →  "Iski zaroorat kyun hai?" (problem it solves)
3️⃣  HOW   →  "Yeh kaam kaise karta hai?" (mechanism + visuals)
4️⃣  CODE  →  "Ek example dikhata hoon..." (production-grade code)
5️⃣  USE   →  "Practical mein main isse tab use karta hoon jab..." (real-world)
```

---

---

## 1. Debouncing & Throttling

> 🏆 **Most asked performance optimization question!**

### 1️⃣ WHAT

Dono **rate-limiting techniques** hain jo function calls ko control karti hain:

| Technique | Core Idea | Analogy |
|-----------|-----------|---------|
| **Debounce** | "Ruk jao, **last call** ke baad X ms wait karo, phir execute karo" | Elevator ka door — jab tak log aa rahe hain, door khula rehta hai |
| **Throttle** | "Har X ms mein **sirf ek baar** execute karo, chahe kitni bhi calls aayein" | Machine gun — trigger dabaye raho, but bullets fixed rate pe nikalti hain |

### 2️⃣ WHY

User events (typing, scrolling, resizing) **seconds mein hundreds of calls** fire kar sakte hain:

```
User types "JavaScript" in search box:

Without debounce:
J → API call
Ja → API call
Jav → API call
Java → API call
... (10 API calls for 10 characters! 💸)

With debounce (300ms):
User types "JavaScript" → waits 300ms → 1 API call ✅
```

### 3️⃣ HOW — Visual Timeline

```
User Events:   🔴🔴🔴  🔴🔴  🔴🔴🔴🔴
Time:          0ms      200ms    500ms

── DEBOUNCE (wait 300ms after LAST event) ──
               ❌❌❌  ❌❌  ❌❌❌✅
               (reset timer on every event, execute only after gap)

── THROTTLE (execute once every 300ms) ──
               ✅❌❌  ✅❌  ✅❌❌❌
               (execute first, ignore until cooldown ends)
```

### 4️⃣ CODE

```javascript
// ═══════════════════════════════════════════
// DEBOUNCE IMPLEMENTATION
// ═══════════════════════════════════════════
function debounce(fn, delay) {
    let timerId;

    return function (...args) {
        // Har nayi call pe purana timer cancel karo
        clearTimeout(timerId);

        // Naya timer start karo
        timerId = setTimeout(() => {
            fn.apply(this, args);  // 'this' preserve karo!
        }, delay);
    };
}

// Usage: Search Input
const searchInput = document.getElementById("search");

const handleSearch = debounce((e) => {
    console.log("API Call:", e.target.value);
    // fetch(`/api/search?q=${e.target.value}`)
}, 300);

searchInput.addEventListener("input", handleSearch);

// ═══════════════════════════════════════════
// THROTTLE IMPLEMENTATION
// ═══════════════════════════════════════════
function throttle(fn, limit) {
    let inThrottle = false;
    let lastArgs = null;

    return function (...args) {
        if (!inThrottle) {
            fn.apply(this, args);   // Pehli call → execute immediately
            inThrottle = true;

            setTimeout(() => {
                inThrottle = false;
                // Optional: trailing call
                if (lastArgs) {
                    fn.apply(this, lastArgs);
                    lastArgs = null;
                }
            }, limit);
        } else {
            lastArgs = args;  // Store latest for trailing call
        }
    };
}

// Usage: Scroll Event
const handleScroll = throttle(() => {
    console.log("Scroll position:", window.scrollY);
    // Check if user reached bottom → load more
}, 200);

window.addEventListener("scroll", handleScroll);

// ═══════════════════════════════════════════
// ADVANCED: Debounce with Leading Edge
// ═══════════════════════════════════════════
function debounceAdvanced(fn, delay, immediate = false) {
    let timerId;

    return function (...args) {
        const callNow = immediate && !timerId;

        clearTimeout(timerId);

        timerId = setTimeout(() => {
            timerId = null;
            if (!immediate) fn.apply(this, args);
        }, delay);

        if (callNow) fn.apply(this, args);  // Execute immediately first time
    };
}
```

### When to Use What?

| Scenario | Use | Why |
|----------|-----|-----|
| Search input | **Debounce** | Wait until user stops typing |
| Window resize | **Debounce** | Wait until resize ends |
| Scroll position | **Throttle** | Regular updates during scroll |
| Button click (API) | **Debounce** | Prevent double-clicks |
| Mouse move (game) | **Throttle** | Consistent frame rate |
| Auto-save | **Debounce** | Save after user pauses |

### 5️⃣ INTERVIEW ANSWER

> *"Debouncing and throttling are **rate-limiting techniques** for performance optimization. **Debounce** delays execution until a specified time has passed since the **last invocation** — ideal for search inputs where you want to wait until the user stops typing. **Throttle** ensures execution at most **once per specified interval** — ideal for scroll or resize events where you want regular but controlled updates. Both techniques prevent excessive function calls that could degrade performance or overwhelm APIs. I implement debounce using `clearTimeout`/`setTimeout` and throttle using a boolean flag with a cooldown timer."*

📌 **One-liner:** Debounce = "wait for silence" | Throttle = "fixed rate, no matter what"

---

---

## 2. Web Workers

### 1️⃣ WHAT

Web Workers JavaScript ko **multi-threaded** banate hain by running code in a **separate background thread**, independent from the main UI thread.

```
Without Worker:                    With Worker:
┌──────────────┐                   ┌──────────────┐  ┌──────────────┐
│  Main Thread │                   │  Main Thread │  │   Worker     │
│  ─ UI        │                   │  ─ UI (free!)│  │  Thread      │
│  ─ JS        │                   │  ─ JS        │  │  ─ Heavy     │
│  ─ Heavy     │ ← BLOCKS UI! 😱  │              │  │    Computation│
│    Computation                   │              │  │    (background)
└──────────────┘                   └──────────────┘  └──────────────┘
```

### 2️⃣ WHY

JavaScript **single-threaded** hai. Heavy computation (data processing, image manipulation, crypto) main thread ko **block** kar deta hai → UI **freeze** ho jata hai → user frustrated.

Web Workers solve this by offloading heavy work to a **parallel thread**.

### 3️⃣ HOW — Architecture

```
Main Thread                          Worker Thread
───────────                          ─────────────
                                     
const worker = new Worker('w.js') ──→  [Worker starts]
                                     
worker.postMessage(data) ──────────→  onmessage = (e) => {
                                        // heavy work
                                        postMessage(result)
                                      }
                                     
worker.onmessage = (e) ←──────────  // result received
  console.log(e.data)               
                                     
worker.terminate() ──────────────→   [Worker destroyed]
```

**Key Constraints:**
- ❌ Workers **DOM access nahi** kar sakte
- ❌ `window`, `document` available nahi
- ✅ `fetch`, `setTimeout`, `IndexedDB` available hai
- ✅ Communication sirf **message passing** se (data copy hota hai)

### 4️⃣ CODE

```javascript
// ═══════════════════════════════════════════
// MAIN THREAD (main.js)
// ═══════════════════════════════════════════
const worker = new Worker("heavy-worker.js");

// Data bhejo worker ko
const largeArray = Array.from({ length: 10_000_000 }, (_, i) => i);

console.log("Sending data to worker...");
worker.postMessage({ numbers: largeArray, operation: "sum" });

// Result receive karo
worker.onmessage = (e) => {
    console.log("Result from worker:", e.data);  // 49999995000000
    console.log("UI was never blocked! ✅");
};

// Error handling
worker.onerror = (error) => {
    console.error("Worker error:", error.message);
};

// Cleanup when done
// worker.terminate();

// ═══════════════════════════════════════════
// WORKER THREAD (heavy-worker.js)
// ═══════════════════════════════════════════
self.onmessage = (e) => {
    const { numbers, operation } = e.data;

    let result;

    if (operation === "sum") {
        // Heavy computation — main thread ko block nahi karega
        result = numbers.reduce((acc, num) => acc + num, 0);
    }

    // Result wapas bhejo
    self.postMessage(result);
};

// ═══════════════════════════════════════════
// INLINE WORKER (no separate file needed!)
// ═══════════════════════════════════════════
const workerCode = `
    self.onmessage = (e) => {
        const result = e.data * e.data;
        self.postMessage(result);
    };
`;

const blob = new Blob([workerCode], { type: "application/javascript" });
const inlineWorker = new Worker(URL.createObjectURL(blob));

inlineWorker.postMessage(5);
inlineWorker.onmessage = (e) => console.log(e.data);  // 25

// ═══════════════════════════════════════════
// TRANSFERABLE OBJECTS (Zero-copy for large data!)
// ═══════════════════════════════════════════
const buffer = new ArrayBuffer(1024 * 1024 * 100);  // 100MB

// Normal: data COPY hota hai (slow for large data)
// worker.postMessage(buffer);

// Transferable: data MOVE hota hai (instant!)
worker.postMessage(buffer, [buffer]);
// ⚠️ buffer ab main thread mein usable nahi hai!
```

### When to Use Web Workers?

| Use Case | Worker Type |
|----------|-------------|
| Image/Video processing | Dedicated Worker |
| Large data sorting/filtering | Dedicated Worker |
| Cryptography/Encryption | Dedicated Worker |
| Real-time data (WebSocket) | Shared Worker |
| Complex math (3D, physics) | Dedicated Worker |
| PWA background sync | Service Worker |

### 5️⃣ INTERVIEW ANSWER

> *"Web Workers allow JavaScript to run **computationally intensive tasks in a background thread** without blocking the main UI thread. Communication happens via **message passing** using `postMessage` and `onmessage`. Workers don't have access to the DOM, `window`, or `document`, but can use `fetch`, timers, and IndexedDB. For large data transfers, I use **Transferable Objects** like `ArrayBuffer` to avoid expensive copying. Common use cases include image processing, data crunching, and cryptography. In modern apps, I also use **Service Workers** for caching and offline support."*

📌 **One-liner:** Web Worker = background thread for heavy tasks | No DOM | Message passing only

---

---

## 3. Garbage Collection

### 1️⃣ WHAT

Garbage Collection (GC) JavaScript engine ka **automatic memory management** system hai jo **unreachable objects** ko identify karke unki memory free karta hai.

> **Developer ko manually memory allocate/free nahi karni padti** (unlike C/C++).

### 2️⃣ WHY

- Har variable, object, closure → **memory consume** karta hai
- Agar unused data memory mein pada rahe → **memory leak** → app slow → crash
- GC automatically "kooda" saaf karta hai

### 3️⃣ HOW — V8 Engine (Chrome/Node) GC Algorithm

V8 uses **Generational Garbage Collection** with two main spaces:

```
┌─────────────────────────────────────────────────┐
│              V8 HEAP MEMORY                      │
│                                                  │
│  ┌─────────────────┐  ┌──────────────────────┐  │
│  │  Young Generation│  │   Old Generation     │  │
│  │  (New Space)     │  │   (Old Space)        │  │
│  │                  │  │                      │  │
│  │  • Short-lived   │  │  • Long-lived        │  │
│  │  • Frequent GC   │  │  • Infrequent GC     │  │
│  │  • Scavenger     │  │  • Mark-and-Sweep    │  │
│  │    Algorithm     │  │    + Compaction      │  │
│  │  • Fast! ⚡      │  │  • Slower 🐢         │  │
│  └────────┬────────┘  └──────────────────────┘  │
│           │ (survive 2 GC cycles)                │
│           └──→ Promoted to Old Generation        │
└─────────────────────────────────────────────────┘
```

**Mark-and-Sweep Algorithm (Core):**

```
Step 1: MARK Phase
        ┌──────────┐
        │  GC Root │ (global, stack variables, active closures)
        └────┬─────┘
             │ traverse
        ┌────▼─────┐     ┌──────────┐
        │ Object A │────→│ Object B │  ← Reachable ✅ (Marked)
        └──────────┘     └──────────┘
                         
        ┌──────────┐
        │ Object C │  ← No root path → Unreachable ❌ (Not marked)
        └──────────┘

Step 2: SWEEP Phase
        Object C → Memory freed! 🗑️
        Object A, B → Kept alive ✅
```

**Key Concept: Reachability**

```
GC Roots (always alive):
  ├── Global variables (window.x)
  ├── Current function's local variables & parameters
  ├── Variables in outer scope (closures!)
  └── Active DOM nodes

Reachable = GC Root se koi reference path exist karta hai
Unreachable = Koi path nahi → Garbage!
```

### 4️⃣ CODE — GC in Action

```javascript
// ── Example 1: Simple GC ──
function createUser() {
    let user = { name: "Kanishk", data: new Array(1000000) };
    return user.name;  // Only string returned
}

createUser();
// user object ab unreachable hai → GC will free it 🗑️

// ── Example 2: Closures prevent GC ──
function createCounter() {
    let count = 0;
    let hugeData = new Array(1000000);  // ⚠️ Stays in memory!

    return function increment() {
        count++;
        // hugeData is in closure scope → NOT garbage collected!
    };
}

const counter = createCounter();
// hugeData will stay alive as long as counter exists

// ── Example 3: Breaking references ──
let obj1 = { name: "A" };
let obj2 = { name: "B" };

obj1.ref = obj2;
obj2.ref = obj1;  // Circular reference!

obj1 = null;
obj2 = null;
// Modern GC (Mark-and-Sweep) handles this ✅
// Old GC (Reference Counting) would leak ❌

// ── Example 4: WeakRef & FinalizationRegistry (ES2021) ──
let target = { data: "important" };
const weakRef = new WeakRef(target);

console.log(weakRef.deref()?.data);  // "important"

target = null;
// GC can now collect target
// weakRef.deref() may return undefined later

// ── Example 5: WeakMap for cache (GC-friendly) ──
const cache = new WeakMap();

function processUser(user) {
    if (cache.has(user)) return cache.get(user);

    const result = /* heavy computation */ user.name.toUpperCase();
    cache.set(user, result);  // When user is GC'd, cache entry auto-removed!
    return result;
}
```

### 5️⃣ INTERVIEW ANSWER

> *"JavaScript uses **automatic garbage collection**, primarily the **Mark-and-Sweep** algorithm in V8. The GC starts from **GC roots** (global scope, active stack frames, closures) and traverses all reachable objects. Objects that are **not reachable** from any root are considered garbage and their memory is freed. V8 uses a **generational approach** — short-lived objects are collected frequently in the Young Generation, while long-lived objects are promoted to the Old Generation. A key point: **closures can prevent GC** by keeping references alive. To help the GC, I explicitly nullify references when they're no longer needed and use `WeakMap`/`WeakRef` for caches."*

📌 **One-liner:** GC = Mark reachable from roots → Sweep unreachable → Free memory

---

---

## 4. Currying

### 1️⃣ WHAT

Currying ek **functional programming technique** hai jismein ek function jo **multiple arguments** leta hai, usse **chain of functions** mein convert kar diya jata hai — har function **ek argument** leta hai.

```
Normal:    add(a, b, c)        →  add(1, 2, 3)
Curried:   add(a)(b)(c)        →  add(1)(2)(3)
```

### 2️⃣ WHY

- **Partial Application** — kuch arguments pehle fix karo, baaki baad mein
- **Function Composition** — reusable, composable functions
- **Configuration** — ek baar config set karo, baar-baar use karo
- **Readability** — intent clear hota hai

### 3️⃣ HOW — Transformation

```
Step 1: Normal Function
        multiply(a, b, c) → a * b * c

Step 2: Curried Version
        multiply(a) → returns function(b) → returns function(c) → a * b * c

Step 3: Usage
        multiply(2)(3)(4)  → 24
        
        const double = multiply(2);        // Partially applied!
        const doubleOf3 = double(3);       // Still waiting for c
        doubleOf3(4)                       → 24
```

### 4️⃣ CODE

```javascript
// ═══════════════════════════════════════════
// BASIC CURRYING
// ═══════════════════════════════════════════
// Manual currying
const add = (a) => (b) => (c) => a + b + c;

console.log(add(1)(2)(3));  // 6

// Partial application
const add5 = add(5);
const add5and3 = add5(3);
console.log(add5and3(2));   // 10

// ═══════════════════════════════════════════
// GENERIC CURRY FUNCTION (Interview Favorite!)
// ═══════════════════════════════════════════
function curry(fn) {
    return function curried(...args) {
        // Agar sufficient arguments mil gaye → execute
        if (args.length >= fn.length) {
            return fn.apply(this, args);
        }
        // Warna → aur arguments ka wait karo
        return function (...nextArgs) {
            return curried.apply(this, [...args, ...nextArgs]);
        };
    };
}

// Usage
function multiply(a, b, c) {
    return a * b * c;
}

const curriedMultiply = curry(multiply);

curriedMultiply(2)(3)(4);     // 24
curriedMultiply(2, 3)(4);     // 24  ← flexible!
curriedMultiply(2)(3, 4);     // 24
curriedMultiply(2, 3, 4);     // 24

// ═══════════════════════════════════════════
// PRACTICAL USE CASE 1: Logging
// ═══════════════════════════════════════════
const log = curry((level, module, message) => {
    console.log(`[${level}] [${module}] ${message}`);
});

const errorLog = log("ERROR");
const dbErrorLog = errorLog("Database");

dbErrorLog("Connection failed");
// [ERROR] [Database] Connection failed

dbErrorLog("Timeout");
// [ERROR] [Database] Timeout

// ═══════════════════════════════════════════
// PRACTICAL USE CASE 2: Validation
// ═══════════════════════════════════════════
const validate = curry((minLength, maxLength, value) => {
    return value.length >= minLength && value.length <= maxLength;
});

const validateUsername = validate(3, 20);
const validatePassword = validate(8, 50);

validateUsername("Kan");       // true
validateUsername("Ka");        // false
validatePassword("secret123"); // true

// ═══════════════════════════════════════════
// PRACTICAL USE CASE 3: API Helper
// ═══════════════════════════════════════════
const apiRequest = curry((method, baseUrl, endpoint, data) => {
    return fetch(`${baseUrl}${endpoint}`, {
        method,
        headers: { "Content-Type": "application/json" },
        body: data ? JSON.stringify(data) : undefined,
    });
});

const getFromMyAPI = apiRequest("GET", "https://api.mysite.com");
const postToMyAPI = apiRequest("POST", "https://api.mysite.com");

// Now use cleanly:
getFromMyAPI("/users");
postToMyAPI("/users", { name: "Kanishk" });
```

### 5️⃣ INTERVIEW ANSWER

> *"Currying is a functional programming technique where a function with **multiple arguments** is transformed into a **sequence of functions**, each taking a single argument. This enables **partial application** — fixing some arguments upfront and supplying the rest later. I implement a generic curry function that checks if enough arguments have been provided; if not, it returns a new function that collects more arguments. Practical use cases include creating **specialized logging functions, reusable validators, and pre-configured API helpers**. Currying promotes code reusability and composability."*

📌 **One-liner:** `f(a, b, c)` → `f(a)(b)(c)` | Partial application = pre-fill arguments

---

---

## 5. Generators & Iterators

### 1️⃣ WHAT

| Concept | Definition |
|---------|-----------|
| **Iterator** | Ek object jo `next()` method provide karta hai, jo `{ value, done }` return karta hai |
| **Iterable** | Koi bhi object jismein `Symbol.iterator` method ho (Array, String, Map, Set) |
| **Generator** | Ek **special function** (`function*`) jo execution **pause** (`yield`) aur **resume** kar sakta hai |

> **Generator = Iterator Factory** 🏭

### 2️⃣ WHY

- **Lazy Evaluation** — data tab generate karo jab zaroorat ho (memory efficient)
- **Infinite Sequences** — bina memory crash ke infinite data
- **Custom Iteration** — apne objects ko `for...of` compatible banao
- **Async Flow Control** — `async generators` for streams
- **State Machines** — complex state management

### 3️⃣ HOW — Iterator Protocol

```
Iterator Protocol:
┌─────────────────────────────────────┐
│  const iterator = obj[Symbol.iterator]()  │
│                                     │
│  iterator.next() → { value: 1, done: false } │
│  iterator.next() → { value: 2, done: false } │
│  iterator.next() → { value: undefined, done: true } │
└─────────────────────────────────────┘

Generator Flow:
┌─────────────────────────────────────┐
│  function* gen() {                  │
│      yield 1;  ← PAUSE (return 1)   │
│      yield 2;  ← PAUSE (return 2)   │
│      return 3; ← DONE               │
│  }                                  │
│                                     │
│  const g = gen();                   │
│  g.next() → { value: 1, done: false }│
│  g.next() → { value: 2, done: false }│
│  g.next() → { value: 3, done: true } │
└─────────────────────────────────────┘
```

### 4️⃣ CODE

```javascript
// ═══════════════════════════════════════════
// ITERATOR (Manual)
// ═══════════════════════════════════════════
const myIterable = {
    data: [10, 20, 30],

    [Symbol.iterator]() {
        let index = 0;
        const data = this.data;

        return {
            next() {
                if (index < data.length) {
                    return { value: data[index++], done: false };
                }
                return { value: undefined, done: true };
            }
        };
    }
};

for (const val of myIterable) {
    console.log(val);  // 10, 20, 30
}

// ═══════════════════════════════════════════
// GENERATOR (Simple)
// ═══════════════════════════════════════════
function* countToThree() {
    console.log("Start");
    yield 1;
    console.log("After 1");
    yield 2;
    console.log("After 2");
    yield 3;
    console.log("Done");
}

const counter = countToThree();

counter.next();  // "Start"         → { value: 1, done: false }
counter.next();  // "After 1"       → { value: 2, done: false }
counter.next();  // "After 2"       → { value: 3, done: false }
counter.next();  // "Done"          → { value: undefined, done: true }

// ═══════════════════════════════════════════
// GENERATOR: Two-Way Communication
// ═══════════════════════════════════════════
function* chatBot() {
    const name = yield "What's your name?";
    const age = yield `Nice, ${name}! How old are you?`;
    yield `${name}, age ${age}. Got it!`;
}

const bot = chatBot();

bot.next();            // { value: "What's your name?" }
bot.next("Kanishk");   // { value: "Nice, Kanishk! How old are you?" }
bot.next(25);          // { value: "Kanishk, age 25. Got it!" }

// ═══════════════════════════════════════════
// PRACTICAL: Infinite Sequence (Lazy!)
// ═══════════════════════════════════════════
function* fibonacci() {
    let a = 0, b = 1;
    while (true) {           // ♾️ Infinite loop — but safe!
        yield a;
        [a, b] = [b, a + b];
    }
}

const fib = fibonacci();
console.log(fib.next().value);  // 0
console.log(fib.next().value);  // 1
console.log(fib.next().value);  // 1
console.log(fib.next().value);  // 2
console.log(fib.next().value);  // 3
// Memory: O(1) — sirf 2 variables! No array of millions!

// ═══════════════════════════════════════════
// PRACTICAL: Range Generator
// ═══════════════════════════════════════════
function* range(start, end, step = 1) {
    for (let i = start; i <= end; i += step) {
        yield i;
    }
}

for (const num of range(1, 5)) {
    console.log(num);  // 1, 2, 3, 4, 5
}

// ═══════════════════════════════════════════
// PRACTICAL: Async Generator (Data Streaming)
// ═══════════════════════════════════════════
async function* fetchPages(baseUrl) {
    let page = 1;
    while (true) {
        const response = await fetch(`${baseUrl}?page=${page}`);
        const data = await response.json();

        if (data.length === 0) return;  // No more data

        yield data;
        page++;
    }
}

// Usage:
// for await (const pageData of fetchPages("/api/users")) {
//     console.log("Got page:", pageData);
// }
```

### 5️⃣ INTERVIEW ANSWER

> *"An **iterator** is an object with a `next()` method that returns `{ value, done }`. An **iterable** is any object implementing `Symbol.iterator`. A **generator** is a special function declared with `function*` that can **pause execution** using `yield` and **resume** later. Generators are essentially **iterator factories**. They enable **lazy evaluation** — values are computed only when requested — making them ideal for infinite sequences like Fibonacci or data streaming. Generators also support **two-way communication** via `next(value)`. In modern JavaScript, **async generators** combined with `for await...of` are powerful for processing data streams."*

📌 **One-liner:** Generator = `function*` + `yield` = pausable function = lazy iterator factory

---

---

## 6. Deep Clone of an Object

### 1️⃣ WHAT

Deep clone ka matlab hai object ka **complete independent copy** banana — nested objects samet — taaki original aur copy **koi reference share na karein**.

```
Shallow Copy:                     Deep Clone:
obj ──→ { a: 1, b: { c: 2 } }    obj  ──→ { a: 1, b: { c: 2 } }
copy ──→ { a: 1, b: ↗ same! }    copy ──→ { a: 1, b: { c: 2 } } ← NEW object!
         (b is shared!)                    (completely independent)
```

### 2️⃣ WHY

- Shallow copy (`...spread`, `Object.assign`) sirf **top level** copy karta hai
- Nested objects **reference share** karte hain → copy modify karo → original bhi change!
- State management (Redux), undo/redo, form data → deep clone zaroori hai

### 3️⃣ HOW — Methods Comparison

| Method | Deep? | Handles Date? | Handles RegExp? | Handles Circular? | Handles Functions? | Speed |
|--------|-------|--------------|----------------|-------------------|-------------------|-------|
| `JSON.parse(JSON.stringify())` | ✅ | ❌ (→ string) | ❌ (→ {}) | ❌ (crash) | ❌ (skip) | Fast |
| `structuredClone()` | ✅ | ✅ | ✅ | ✅ | ❌ | Fast |
| Custom Recursive | ✅ | ✅ (if coded) | ✅ (if coded) | ✅ (if coded) | ✅ (if coded) | Medium |
| Lodash `_.cloneDeep()` | ✅ | ✅ | ✅ | ✅ | ✅ | Medium |

### 4️⃣ CODE

```javascript
// ═══════════════════════════════════════════
// METHOD 1: JSON (Quick & Dirty — has limitations!)
// ═══════════════════════════════════════════
const original = {
    name: "Kanishk",
    address: { city: "Delhi" },
    date: new Date(),
    regex: /test/g,
    fn: () => "hello"
};

const jsonClone = JSON.parse(JSON.stringify(original));

console.log(jsonClone.address === original.address);  // false ✅ (deep)
console.log(jsonClone.date);    // "2024-01-01T..." ❌ (string, not Date!)
console.log(jsonClone.regex);   // {} ❌ (lost!)
console.log(jsonClone.fn);      // undefined ❌ (functions skipped!)

// ═══════════════════════════════════════════
// METHOD 2: structuredClone() (Modern — Best Built-in!)
// ═══════════════════════════════════════════
const original2 = {
    name: "Kanishk",
    address: { city: "Delhi" },
    date: new Date(),
    regex: /test/g,
    set: new Set([1, 2, 3]),
    map: new Map([["key", "value"]]),
};

// Add circular reference
original2.self = original2;

const structured = structuredClone(original2);

console.log(structured.address === original2.address);  // false ✅
console.log(structured.date instanceof Date);            // true ✅
console.log(structured.regex instanceof RegExp);         // true ✅
console.log(structured.set instanceof Set);              // true ✅
console.log(structured.self === structured);             // true ✅ (circular preserved!)
// ❌ Functions still not cloned

// ═══════════════════════════════════════════
// METHOD 3: Custom Recursive Deep Clone (Interview!)
// ═══════════════════════════════════════════
function deepClone(obj, seen = new WeakMap()) {
    // Primitives & functions → return as-is
    if (obj === null || typeof obj !== "object") {
        return obj;
    }

    // Handle circular references
    if (seen.has(obj)) {
        return seen.get(obj);
    }

    // Handle Date
    if (obj instanceof Date) {
        return new Date(obj.getTime());
    }

    // Handle RegExp
    if (obj instanceof RegExp) {
        return new RegExp(obj.source, obj.flags);
    }

    // Handle Map
    if (obj instanceof Map) {
        const mapClone = new Map();
        seen.set(obj, mapClone);
        obj.forEach((value, key) => {
            mapClone.set(deepClone(key, seen), deepClone(value, seen));
        });
        return mapClone;
    }

    // Handle Set
    if (obj instanceof Set) {
        const setClone = new Set();
        seen.set(obj, setClone);
        obj.forEach(value => {
            setClone.add(deepClone(value, seen));
        });
        return setClone;
    }

    // Handle Array
    if (Array.isArray(obj)) {
        const arrClone = [];
        seen.set(obj, arrClone);
        obj.forEach((item, index) => {
            arrClone[index] = deepClone(item, seen);
        });
        return arrClone;
    }

    // Handle plain Object
    const objClone = Object.create(Object.getPrototypeOf(obj));
    seen.set(obj, objClone);

    for (const key of Reflect.ownKeys(obj)) {
        objClone[key] = deepClone(obj[key], seen);
    }

    return objClone;
}

// ═══════════════════════════════════════════
// TEST
// ═══════════════════════════════════════════
const complex = {
    name: "Kanishk",
    nested: { scores: [90, 85, 92] },
    date: new Date("2024-01-01"),
    pattern: /hello/gi,
};
complex.circular = complex;  // Circular reference!

const cloned = deepClone(complex);

console.log(cloned.nested === complex.nested);       // false ✅
console.log(cloned.nested.scores === complex.nested.scores); // false ✅
console.log(cloned.date instanceof Date);             // true ✅
console.log(cloned.circular === cloned);              // true ✅
```

### 5️⃣ INTERVIEW ANSWER

> *"Deep cloning creates a **completely independent copy** of an object, including all nested structures. The simplest approach is `JSON.parse(JSON.stringify())`, but it **fails** with Dates, RegExps, functions, and circular references. The modern built-in solution is **`structuredClone()`**, which handles most types including circular references. For full control — especially in interviews — I implement a **recursive deep clone** that handles Date, RegExp, Map, Set, Arrays, and circular references using a `WeakMap` to track already-cloned objects. In production, I'd use `structuredClone()` or Lodash's `_.cloneDeep()`."*

📌 **One-liner:** JSON = quick but limited | `structuredClone()` = modern best | Recursive = interview answer

---

---

## 7. Memory Leaks

### 1️⃣ WHAT

Memory leak tab hota hai jab **objects memory mein reh jaate hain** even though they are **no longer needed** — kyunki kuch reference unhe GC se bacha raha hai.

```
Normal:    Object created → Used → Reference removed → GC frees memory ✅
Leak:      Object created → Used → Reference NOT removed → Memory stuck! ❌
```

### 2️⃣ WHY

- JavaScript GC **automatic** hai, but **foolproof nahi**
- Agar koi reference **accidentally alive** hai → GC usse collect nahi kar sakta
- Long-running apps (SPAs, dashboards) mein leaks accumulate → **performance degradation → crash**

### 3️⃣ HOW — 5 Most Common Causes

```
┌─────────────────────────────────────────────────────┐
│           TOP 5 MEMORY LEAK CAUSES                   │
│                                                      │
│  1️⃣  Accidental Global Variables                    │
│  2️⃣  Forgotten Timers (setInterval)                 │
│  3️⃣  Detached DOM References                        │
│  4️⃣  Closures Holding Large Data                    │
│  5️⃣  Forgotten Event Listeners                      │
└─────────────────────────────────────────────────────┘
```

### 4️⃣ CODE — Each Cause + Fix

```javascript
// ═══════════════════════════════════════════
// LEAK 1: Accidental Globals
// ═══════════════════════════════════════════
function processData() {
    // ❌ LEAK: 'data' becomes global (no let/const/var!)
    data = new Array(1000000).fill("x");
}

// ✅ FIX: Always use let/const
function processDataFixed() {
    const data = new Array(1000000).fill("x");
    // data is garbage collected when function exits
}

// ═══════════════════════════════════════════
// LEAK 2: Forgotten Timers
// ═══════════════════════════════════════════
function startPolling() {
    const hugeData = new Array(1000000);

    // ❌ LEAK: Interval keeps running, hugeData stays alive
    setInterval(() => {
        console.log(hugeData.length);
    }, 1000);
}

// ✅ FIX: Store reference and clear when done
function startPollingFixed() {
    const hugeData = new Array(1000000);
    const timerId = setInterval(() => {
        console.log(hugeData.length);
    }, 1000);

    // Cleanup when component unmounts / no longer needed
    return () => clearInterval(timerId);
}

// ═══════════════════════════════════════════
// LEAK 3: Detached DOM References
// ═══════════════════════════════════════════
let cachedElement;

function createElement() {
    const div = document.createElement("div");
    div.innerHTML = "<p>Heavy content...</p>";
    cachedElement = div;  // ❌ LEAK: Reference persists even after DOM removal

    document.body.appendChild(div);
}

function removeElement() {
    const div = document.querySelector("div");
    div.remove();  // DOM se hat gaya, but cachedElement abhi bhi reference rakhta hai!
}

// ✅ FIX: Nullify the reference
function removeElementFixed() {
    const div = document.querySelector("div");
    div.remove();
    cachedElement = null;  // Now GC can collect it
}

// ═══════════════════════════════════════════
// LEAK 4: Closures Holding Large Data
// ═══════════════════════════════════════════
function createHandler() {
    const largeData = new Array(1000000).fill("x");  // 1M elements!

    return function handler() {
        // ❌ LEAK: Even if we only need 'name', largeData stays alive
        const name = "Kanishk";
        console.log(name);
        // largeData is in closure scope → NOT garbage collected!
    };
}

// ✅ FIX: Nullify or restructure
function createHandlerFixed() {
    let largeData = new Array(1000000).fill("x");
    // Process what you need
    const summary = largeData.length;
    largeData = null;  // ✅ Release memory!

    return function handler() {
        console.log("Items:", summary);  // Only summary retained
    };
}

// ═══════════════════════════════════════════
// LEAK 5: Forgotten Event Listeners
// ═══════════════════════════════════════════
class ChatWidget {
    constructor() {
        this.messages = [];

        // ❌ LEAK: Even if widget is destroyed, listener persists
        window.addEventListener("resize", this.handleResize.bind(this));
    }

    handleResize() {
        console.log("Resize:", this.messages.length);
    }

    destroy() {
        // Forgot to remove listener! 😱
    }
}

// ✅ FIX: Remove listener on cleanup
class ChatWidgetFixed {
    constructor() {
        this.messages = [];
        this._boundResize = this.handleResize.bind(this);
        window.addEventListener("resize", this._boundResize);
    }

    handleResize() {
        console.log("Resize:", this.messages.length);
    }

    destroy() {
        window.removeEventListener("resize", this._boundResize);  // ✅
        this.messages = null;
    }
}

// ═══════════════════════════════════════════
// DETECTION: Chrome DevTools
// ═══════════════════════════════════════════
/*
1. Open DevTools → Memory tab
2. Take Heap Snapshot (before action)
3. Perform action (navigate, open modal, etc.)
4. Take Heap Snapshot (after action)
5. Compare snapshots → look for growing objects
6. Use "Allocation Timeline" for real-time tracking

Key indicators:
- Detached DOM trees growing
- Event listener count increasing
- JS Heap size continuously growing
*/
```

### 5️⃣ INTERVIEW ANSWER

> *"Memory leaks in JavaScript occur when objects are **no longer needed but still referenced**, preventing the garbage collector from freeing them. The five most common causes are: **accidental global variables, forgotten timers (`setInterval`), detached DOM references, closures retaining large data, and unremoved event listeners**. To prevent leaks, I always use `let`/`const`, clear timers on cleanup, nullify DOM references after removal, minimize closure scope, and remove event listeners in component teardown. For detection, I use **Chrome DevTools Memory tab** — taking heap snapshots before and after operations to identify growing object counts."*

📌 **One-liner:** Leak = needed nahi but referenced hai | Fix = nullify + cleanup + DevTools

---

---

## 8. Microtasks vs Macrotasks

### 1️⃣ WHAT

Event Loop ke andar **2 types ki queues** hoti hain:

| | Microtask Queue | Macrotask Queue (Task Queue) |
|---|----------------|----------------------------|
| **Priority** | 🔴 **HIGH** | 🟡 **LOW** |
| **When** | Har macrotask ke **baad**, stack empty hone pe | Event loop ke **har iteration** mein ek-ek |
| **Sources** | `Promise.then/catch/finally`, `queueMicrotask()`, `MutationObserver` | `setTimeout`, `setInterval`, `setImmediate` (Node), I/O, UI rendering, `requestAnimationFrame` |
| **Drain** | **Poora queue** ek baar mein drain hota hai | **Ek task** per iteration |

### 2️⃣ WHY

Ye samajhna zaroori hai kyunki:
- Code ka **execution order** ispe depend karta hai
- Interview mein **output prediction** questions aate hain
- Performance optimization — microtasks UI render se pehle run hote hain

### 3️⃣ HOW — Execution Order Visual

```
Event Loop ka ek cycle:

┌─────────────────────────────────────────────────┐
│  1. Call Stack se synchronous code execute       │
│     ↓                                            │
│  2. Stack empty?                                 │
│     ↓ YES                                        │
│  3. 🔴 MICROTASK QUEUE check                     │
│     → Poora drain karo (ALL microtasks!)         │
│     → Agar microtask ne naya microtask add kiya  │
│       → Woh bhi isi cycle mein run hoga!         │
│     ↓                                            │
│  4. 🎨 UI Render (if needed)                     │
│     ↓                                            │
│  5. 🟡 MACROTASK QUEUE check                     │
│     → Sirf EK macrotask uthao                    │
│     ↓                                            │
│  6. Loop back to Step 1 🔄                       │
└─────────────────────────────────────────────────┘
```

### 4️⃣ CODE — Output Prediction (Interview Favorite!)

```javascript
// ═══════════════════════════════════════════
// EXAMPLE 1: Basic Order
// ═══════════════════════════════════════════
console.log("1: Sync");

setTimeout(() => console.log("2: setTimeout"), 0);

Promise.resolve().then(() => console.log("3: Promise"));

queueMicrotask(() => console.log("4: queueMicrotask"));

console.log("5: Sync");

// Output:
// 1: Sync              ← Call Stack (sync)
// 5: Sync              ← Call Stack (sync)
// 3: Promise           ← Microtask Queue (HIGH)
// 4: queueMicrotask    ← Microtask Queue (HIGH)
// 2: setTimeout        ← Macrotask Queue (LOW)

// ═══════════════════════════════════════════
// EXAMPLE 2: Nested Microtasks (Tricky!)
// ═══════════════════════════════════════════
console.log("A");

setTimeout(() => console.log("B"), 0);

Promise.resolve()
    .then(() => {
        console.log("C");
        // Nested microtask!
        Promise.resolve().then(() => console.log("D"));
    })
    .then(() => console.log("E"));

setTimeout(() => console.log("F"), 0);

// Output:
// A    ← Sync
// C    ← Microtask (first .then)
// D    ← Microtask (nested — SAME cycle! poora drain hota hai)
// E    ← Microtask (second .then)
// B    ← Macrotask (first setTimeout)
// F    ← Macrotask (second setTimeout)

// ═══════════════════════════════════════════
// EXAMPLE 3: The Ultimate Test
// ═══════════════════════════════════════════
console.log("1");

setTimeout(() => {
    console.log("2");
    Promise.resolve().then(() => console.log("3"));
}, 0);

new Promise((resolve) => {
    console.log("4");       // Constructor is SYNC!
    resolve();
}).then(() => console.log("5"));

queueMicrotask(() => console.log("6"));

setTimeout(() => console.log("7"), 0);

// Step-by-step:
// Sync:    1, 4 (Promise constructor runs immediately!)
// Micro:   5, 6 (drain all microtasks)
// Macro 1: 2 (first setTimeout)
//   → Micro from inside: 3
// Macro 2: 7 (second setTimeout)

// Output: 1, 4, 5, 6, 2, 3, 7

// ═══════════════════════════════════════════
// EXAMPLE 4: async/await (Microtasks!)
// ═══════════════════════════════════════════
async function asyncFn() {
    console.log("A");
    await Promise.resolve();  // Everything after await = microtask!
    console.log("B");
}

console.log("C");
asyncFn();
console.log("D");

// Output:
// C    ← Sync
// A    ← Sync (before await)
// D    ← Sync
// B    ← Microtask (after await)
```

### Quick Reference Table

| API | Queue | Priority |
|-----|-------|----------|
| `console.log()` | Call Stack (Sync) | ⚡ Highest |
| `Promise.then()` | Microtask | 🔴 High |
| `queueMicrotask()` | Microtask | 🔴 High |
| `MutationObserver` | Microtask | 🔴 High |
| `await` (after) | Microtask | 🔴 High |
| `setTimeout()` | Macrotask | 🟡 Low |
| `setInterval()` | Macrotask | 🟡 Low |
| `setImmediate()` (Node) | Macrotask | 🟡 Low |
| `requestAnimationFrame` | Macrotask (before render) | 🟡 Low |
| I/O, UI events | Macrotask | 🟡 Low |

### 5️⃣ INTERVIEW ANSWER

> *"The Event Loop manages two types of task queues: **Microtasks** and **Macrotasks**. Microtasks — like `Promise.then`, `queueMicrotask`, and `MutationObserver` — have **higher priority** and the **entire queue is drained** after each macrotask. Macrotasks — like `setTimeout`, `setInterval`, and I/O — are processed **one per event loop iteration**. A critical nuance: if a microtask schedules another microtask, it runs in the **same cycle**, which can potentially starve macrotasks. Also, `async/await` after the `await` keyword behaves as a microtask. Understanding this order is essential for debugging execution sequence and optimizing performance."*

📌 **One-liner:** Microtask (Promises) = HIGH priority, drain ALL | Macrotask (setTimeout) = LOW, one per cycle

---

---

## ⚡ Lightning Revision Sheet (5 Min Before Interview)

| # | Topic | One-Line Answer |
|---|-------|----------------|
| 1 | **Debounce/Throttle** | Debounce = wait for silence | Throttle = fixed rate limit |
| 2 | **Web Workers** | Background thread for heavy tasks, no DOM, message passing |
| 3 | **Garbage Collection** | Mark-and-Sweep from roots; unreachable = freed; V8 generational |
| 4 | **Currying** | `f(a,b,c)` → `f(a)(b)(c)`; partial application; reusable configs |
| 5 | **Generators** | `function*` + `yield` = pausable function = lazy iterator factory |
| 6 | **Deep Clone** | JSON (limited) → `structuredClone()` (modern) → recursive (full control) |
| 7 | **Memory Leaks** | Globals, timers, DOM refs, closures, listeners → nullify + cleanup |
| 8 | **Micro vs Macro** | Micro (Promise) = HIGH, drain all | Macro (setTimeout) = LOW, one per loop |

---

## 🔗 Full Connection Map (Basic → Intermediate → Advanced)

```
BASIC                    INTERMEDIATE              ADVANCED
─────                    ────────────              ────────
Data Types ──────────→ null vs undefined ──────→ Deep Clone (type handling)
var/let/const ──────→ this keyword ──────────→ Memory Leaks (globals)
Arrow Functions ────→ this (lexical) ────────→ Currying (arrow chains)
Callbacks ──────────→ Event Bubbling ────────→ Debounce/Throttle (events)
Callbacks ──────────→ HOFs ─────────────────→ Currying (HOF pattern)
Promises ───────────→ Event Loop ────────────→ Micro vs Macrotasks
async/await ────────→ Error Handling ────────→ Async Generators
Closures ───────────→ Prototypal Inheritance → Memory Leaks (closure refs)
                     map/filter/reduce ─────→ Generators (lazy iteration)
                                              Web Workers (offload main thread)
                                              Garbage Collection (how memory freed)
```

---
