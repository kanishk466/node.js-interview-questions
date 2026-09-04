# 🚀 JavaScript & Programming Fundamentals — Interview Prep Guide

> **Level:** Basic → Intermediate
> **Format:** What → Why → How → Example → Interview Answer
> **Language:** Hinglish (for easy understanding) + English (for interview delivery)

---

## 📑 Table of Contents

| # | Topic | Difficulty | Frequency |
|---|-------|-----------|-----------|
| 1 | [Data Types in JavaScript](#1-data-types-in-javascript) | ⭐ | 🔥🔥🔥 |
| 2 | [`var` vs `let` vs `const`](#2-var-vs-let-vs-const) | ⭐ | 🔥🔥🔥🔥🔥 |
| 3 | [`==` vs `===`](#3--vs-) | ⭐ | 🔥🔥🔥🔥 |
| 4 | [Arrow Functions vs Regular Functions](#4-arrow-functions-vs-regular-functions) | ⭐⭐ | 🔥🔥🔥🔥 |
| 5 | [Callback Functions](#5-callback-functions) | ⭐⭐ | 🔥🔥🔥 |
| 6 | [`async/await`](#6-asyncawait) | ⭐⭐ | 🔥🔥🔥🔥 |
| 7 | [Promises](#7-promises) | ⭐⭐⭐ | 🔥🔥🔥🔥🔥 |
| 8 | [Closures](#8-closures) | ⭐⭐⭐ | 🔥🔥🔥🔥🔥 |

---

## 🧠 Answer Framework (Har Question Ke Liye)

Interview mein **ratta mat maaro**, ye 5-step structure follow karo:

```
1️⃣  WHAT  →  "Yeh kya hai?" (1-2 lines definition)
2️⃣  WHY   →  "Iski zaroorat kyun hai?" (problem it solves)
3️⃣  HOW   →  "Yeh kaam kaise karta hai?" (mechanism)
4️⃣  CODE  →  "Ek chhota example dikhata hoon..." (live code)
5️⃣  USE   →  "Practical mein main isse tab use karta hoon jab..." (real-world)
```

> 💡 **Pro Tip:** Ye structure aapko kabhi blank nahi hone dega. Exact words yaad nahi? Koi baat nahi — flow yaad rakho.

---

---

## 1. Data Types in JavaScript

### 1️⃣ WHAT

JavaScript mein data types **2 categories** mein aate hain:

| Category | Types | Yaad Rakhne Ka Trick |
|----------|-------|---------------------|
| **Primitive** (7) | `String`, `Number`, `Boolean`, `Undefined`, `Null`, `BigInt`, `Symbol` | **S**ome **N**erds **B**elieve **U**nicorns **N**ever **B**ecome **S**mart |
| **Reference** (1) | `Object` (includes `Array`, `Function`, `Date`, etc.) | Sab kuch object hai! |

### 2️⃣ WHY

Data type batata hai ki:
- Variable ke andar **kis type ka data** stored hai
- Us data ke saath **kya operations** perform kar sakte hain

> JavaScript **dynamically typed** hai → variable ka type explicitly declare nahi karna padta.

### 3️⃣ HOW

Runtime pe value ke basis par type decide hota hai:

```javascript
let x = 10;
console.log(typeof x);  // "number"

x = "hello";
console.log(typeof x);  // "string"  ← same variable, different type!
```

### 4️⃣ CODE

```javascript
// ── Primitives ──
let name      = "Kanishk";     // String
let age       = 25;            // Number
let isActive  = true;          // Boolean
let address;                   // Undefined (declared, no value)
let salary    = null;          // Null (intentionally empty)
let bigNum    = 123456789n;    // BigInt
let id        = Symbol("id");  // Symbol (unique identifier)

// ── Reference ──
let user = { name: "Kanishk", age: 25 };  // Object
let nums = [1, 2, 3];                      // Array (technically object)
let greet = function() {};                 // Function (technically object)
```

### 5️⃣ INTERVIEW ANSWER

> *"JavaScript has **seven primitive data types** — String, Number, Boolean, Undefined, Null, BigInt, and Symbol. Apart from these, **Object** is the only reference type, which includes arrays, functions, and dates. JavaScript is **dynamically typed**, meaning the type is determined at runtime based on the assigned value."*

📌 **One-liner:** Primitive = actual value stored | Reference = memory address stored

---

---

## 2. `var` vs `let` vs `const`

> ⚠️ **Most asked question in JS interviews!**

### 1️⃣ WHAT

Teeno JavaScript mein **variable declare** karne ke tarike hain — lekin alag-alag rules ke saath.

### 2️⃣ WHY

`var` purana hai aur **unpredictable behavior** deta hai (function scope, hoisting issues). ES6 ne `let` aur `const` introduce kiye for **block scope** and **safer code**.

### 3️⃣ HOW — The 3 Pillars of Difference

```
Yaad rakho:  SCOPE  +  REDECLARATION  +  REASSIGNMENT
```

| Feature | `var` 🟡 | `let` 🟢 | `const` 🔴 |
|---------|---------|---------|-----------|
| **Scope** | Function | Block `{}` | Block `{}` |
| **Reassign** | ✅ Yes | ✅ Yes | ❌ No |
| **Redeclare** | ✅ Yes | ❌ No | ❌ No |
| **Hoisted** | ✅ Yes (with `undefined`) | ✅ Yes (TDZ*) | ✅ Yes (TDZ*) |
| **Use Case** | Avoid ❌ | Changing values | Default choice ✅ |

> \* **TDZ (Temporal Dead Zone):** Variable declared hai but initialize nahi hua — access karne pe `ReferenceError`.

### 4️⃣ CODE

```javascript
// ── SCOPE ──
function testScope() {
    if (true) {
        var a = 10;   // function scope → bahar bhi dikhega
        let b = 20;   // block scope → sirf if ke andar
    }
    console.log(a);   // 10 ✅
    console.log(b);   // ReferenceError ❌
}

// ── REASSIGNMENT ──
let x = 5;
x = 10;              // ✅ Allowed

const y = 5;
y = 10;              // ❌ TypeError: Assignment to constant

// ── ⚠️ INTERVIEW TRAP: const + object ──
const user = { name: "Kanishk" };
user.name = "Rahul";  // ✅ Allowed! (property change ≠ reassignment)
user = {};            // ❌ TypeError (reassignment not allowed)
```

### 5️⃣ INTERVIEW ANSWER

> *"The main differences are **scope, redeclaration, and reassignment**. `var` is **function-scoped** and allows both redeclaration and reassignment. `let` and `const` are **block-scoped**. `let` allows reassignment but not redeclaration, while `const` allows neither. One important nuance: `const` prevents **reassignment**, not **mutation** — so object properties can still be modified. In modern JavaScript, I use `const` by default and `let` only when reassignment is needed."*

📌 **One-liner:** `var` → function | `let` → change OK | `const` → reassign NOT OK

---

---

## 3. `==` vs `===`

### 1️⃣ WHAT

Dono **comparison operators** hain, but ek critical difference hai:

| Operator | Name | Behavior |
|----------|------|----------|
| `==` | Loose Equality | Value compare karta hai **type convert karke** |
| `===` | Strict Equality | Value **+ Type** dono compare karta hai |

### 2️⃣ WHY

`==` **type coercion** karta hai — matlab compare karne se pehle types ko match karne ki koshish karta hai. Isse **unexpected results** aa sakte hain.

### 3️⃣ HOW

```javascript
// ── == (Loose) ──
5 == "5"        // true  ← "5" ko number mein convert kiya
false == 0      // true  ← false → 0
null == undefined // true ← special rule
"" == 0         // true  ← "" → 0

// ── === (Strict) ──
5 === "5"       // false ← number ≠ string
false === 0     // false ← boolean ≠ number
null === undefined // false ← different types
```

### 4️⃣ CODE

```javascript
// Real-world confusion with ==
let userInput = "";

if (userInput == 0) {
    console.log("This runs!");  // 😱 Wait, what?
}

if (userInput === 0) {
    console.log("This won't");  // ✅ Predictable
}
```

### 5️⃣ INTERVIEW ANSWER

> *"`==` performs **loose equality** with type coercion — it converts types before comparing. `===` performs **strict equality** — it checks both **value and type** without any conversion. Because `==` can produce unexpected results like `"" == 0` being `true`, I always prefer `===` in production code for predictability."*

📌 **One-liner:** `==` → value after conversion | `===` → value + type (no conversion)

---

---

## 4. Arrow Functions vs Regular Functions

### 1️⃣ WHAT

Arrow function ES6 ka **shorter syntax** hai function likhne ka.

```javascript
// Regular
function add(a, b) { return a + b; }

// Arrow
const add = (a, b) => a + b;
```

### 2️⃣ WHY

- ✅ Concise syntax (especially for callbacks)
- ✅ **Lexical `this`** — apna `this` nahi banata (most important!)

### 3️⃣ HOW — Key Differences

| Feature | Regular Function | Arrow Function |
|---------|-----------------|----------------|
| **Syntax** | Verbose | Concise |
| **Own `this`** | ✅ Yes (depends on call) | ❌ No (inherits from parent) |
| **`arguments` object** | ✅ Yes | ❌ No |
| **`new` keyword** | ✅ Constructor ban sakta hai | ❌ No |
| **Hoisting** | ✅ Yes (function declaration) | ❌ No (behaves like `let`/`const`) |

### 4️⃣ CODE

```javascript
// ── The `this` Trap (Interview Favorite!) ──
const user = {
    name: "Kanishk",

    regularFn: function() {
        console.log(this.name);  // "Kanishk" ✅ (this = user)
    },

    arrowFn: () => {
        console.log(this.name);  // undefined ❌ (this = window/global)
    }
};

user.regularFn();  // "Kanishk"
user.arrowFn();    // undefined

// ── Perfect for callbacks ──
const numbers = [1, 2, 3];
const doubled = numbers.map(n => n * 2);  // [2, 4, 6]
```

### 5️⃣ INTERVIEW ANSWER

> *"Arrow functions provide a **concise syntax** for writing functions. The most critical difference is how `this` works: a regular function creates its **own `this`** based on how it's called, while an arrow function **lexically inherits `this`** from its surrounding scope. This makes arrow functions ideal for callbacks but unsuitable for object methods or constructors."*

📌 **One-liner:** Arrow = short syntax + no own `this`

---

---

## 5. Callback Functions

### 1️⃣ WHAT

> **Callback = ek function jo doosre function ko argument ke roop mein pass kiya jata hai, aur baad mein execute hota hai.**

### 2️⃣ WHY

Jab humein kehna ho: *"Ye kaam hone ke **baad** ye karo"* — tab callback use hota hai.

**Use cases:** Async operations, event handling, array methods.

### 3️⃣ HOW — Real-World Analogy

```
🍔 Restaurant Analogy:

1. Aapne order diya (function call)
2. Kitchen mein food ban raha hai (async operation)
3. Food ready → Waiter aapko notify karta hai (CALLBACK!)

"Jab food ready ho, mujhe bata dena" = Callback
```

### 4️⃣ CODE

```javascript
// ── Simple Sync Callback ──
function greet(name, callback) {
    console.log("Hello " + name);
    callback();  // ← callback execute hua
}

greet("Kanishk", () => console.log("Goodbye!"));
// Hello Kanishk
// Goodbye!

// ── Async Callback (setTimeout) ──
console.log("Start");
setTimeout(() => {
    console.log("2 sec baad!");  // ← callback
}, 2000);
console.log("End");
// Output: Start → End → 2 sec baad!

// ── Array Method Callback ──
const nums = [1, 2, 3];
const doubled = nums.map(n => n * 2);  // n => n*2 is callback
```

### 5️⃣ INTERVIEW ANSWER

> *"A callback is a **function passed as an argument** to another function, to be executed later — either synchronously or asynchronously. They're commonly used in event handling, array methods like `map` and `filter`, and async operations like `setTimeout`. However, deeply nested callbacks lead to **callback hell**, which is why Promises and async/await were introduced."*

📌 **One-liner:** Function ko function mein pass karo → Callback

---

---

## 6. `async/await`

### 1️⃣ WHAT

`async/await` **Promises ke upar syntactic sugar** hai — async code ko **synchronous-looking** banata hai.

### 2️⃣ WHY

Promise chaining mushkil hoti hai:

```javascript
// ❌ Promise Chaining (messy)
getUser()
    .then(user => getOrders(user.id))
    .then(orders => getDetails(orders[0]))
    .then(details => console.log(details))
    .catch(err => console.log(err));

// ✅ async/await (clean!)
const user = await getUser();
const orders = await getOrders(user.id);
const details = await getDetails(orders[0]);
console.log(details);
```

### 3️⃣ HOW

- `async` keyword function ke aage lagao → function **Promise return** karega
- `await` keyword Promise ke aage lagao → **result ka wait** karega

> ⚠️ **Critical Interview Point:** `await` JavaScript thread ko **BLOCK nahi karta**. It only pauses the current `async` function. Event loop free rehta hai.

### 4️⃣ CODE

```javascript
function fetchUser() {
    return new Promise(resolve =>
        setTimeout(() => resolve({ id: 1, name: "Kanishk" }), 1000)
    );
}

async function getData() {
    try {
        const user = await fetchUser();   // 1 sec wait
        console.log(user.name);           // "Kanishk"
    } catch (error) {
        console.log("Error:", error);     // Error handling
    }
}

getData();
```

### 5️⃣ INTERVIEW ANSWER

> *"`async/await` is **syntactic sugar over Promises** that makes asynchronous code look and behave more like synchronous code. The `await` keyword pauses the execution of the **current async function** until the Promise resolves, but it **does not block the main thread or event loop**. I always pair it with `try/catch` for clean error handling."*

📌 **One-liner:** Promise ka clean syntax — `await` pauses function, NOT the thread

---

---

## 7. Promises

### 1️⃣ WHAT

Promise ek **object** hai jo represent karta hai **future mein milne wala result** — success ya failure.

```
         ┌──────────┐
         │ PENDING  │  (initial state)
         └────┬─────┘
              │
       ┌──────┴──────┐
       ▼             ▼
  ✅ FULFILLED   ❌ REJECTED
   (resolve)      (reject)
```

### 2️⃣ WHY

Bina Promises ke → **Callback Hell** 😱

```javascript
getUser(id, function(user) {
    getOrders(user.id, function(orders) {
        getProducts(orders[0], function(products) {
            // 🤯 Pyramid of Doom
        });
    });
});
```

Promises isko **flat aur manageable** banate hain.

### 3️⃣ HOW

```javascript
// ── Create ──
const myPromise = new Promise((resolve, reject) => {
    const success = true;
    success ? resolve("Done!") : reject("Failed!");
});

// ── Consume ──
myPromise
    .then(result => console.log(result))   // "Done!"
    .catch(error => console.log(error));
```

### 4️⃣ CODE — Promise Combinators (Interview Follow-up!)

| Method | Behavior | Kab Use Karein? |
|--------|----------|----------------|
| `Promise.all([])` | Sab resolve hone ka wait. **Ek fail → sab fail** | Parallel API calls |
| `Promise.allSettled([])` | Sabka result, **chahe fail ho** | Logging / reporting |
| `Promise.race([])` | **Sabse pehle** jo settle ho | Timeout logic |
| `Promise.any([])` | **Sabse pehle** jo resolve ho | Fallback servers |

```javascript
// Example: Promise.all
const [users, products] = await Promise.all([
    fetchUsers(),
    fetchProducts()
]);
```

### 5️⃣ INTERVIEW ANSWER

> *"A Promise is an object representing the **eventual completion or failure** of an asynchronous operation. It has three states: **pending, fulfilled, and rejected**. We handle results with `.then()` and errors with `.catch()`. Promises solved the **callback hell** problem and form the foundation for `async/await`. For multiple promises, I use combinators like `Promise.all()` for parallel execution."*

📌 **One-liner:** Promise = future result | Pending → Fulfilled / Rejected

---

---

## 8. Closures

> 🏆 **Most conceptual question — understanding check hoti hai!**

### 1️⃣ WHAT

> **Jab ek inner function apne outer function ke variables ko yaad rakhta hai — even after outer function execute ho chuka ho — usse closure kehte hain.**

```
Closure = Inner Function + Outer Scope Variables + Retained Reference
```

### 2️⃣ WHY

Closures enable:
- 🔒 **Data Privacy** (private variables)
- 💾 **State Persistence** (counters, caches)
- 🏭 **Function Factories**
- 🎯 **Callbacks & Event Handlers**

### 3️⃣ HOW — Step by Step

```javascript
function outer() {
    let count = 0;          // ← outer scope variable

    function inner() {
        count++;            // ← inner function "remembers" count
        console.log(count);
    }

    return inner;           // ← return the function (not call it!)
}

const counter = outer();    // outer() done. count should be destroyed? NO!

counter();  // 1  ← closure ne count ko zinda rakha
counter();  // 2
counter();  // 3
```

**Mental Model:**

```
outer() executed & finished
        ↓
   count = 0  ← normally garbage collected
        ↓
   BUT inner() has a "backpack" 🎒
        ↓
   backpack mein count ka reference hai
        ↓
   count survives! = CLOSURE
```

### 4️⃣ CODE — Practical Example: Private Bank Account

```javascript
function createBankAccount(initialBalance) {
    let balance = initialBalance;  // 🔒 PRIVATE variable

    return {
        deposit(amount) {
            balance += amount;
            console.log(`Deposited ₹${amount}. Balance: ₹${balance}`);
        },
        getBalance() {
            return balance;
        }
    };
}

const myAccount = createBankAccount(1000);

myAccount.deposit(500);        // Deposited ₹500. Balance: ₹1500
console.log(myAccount.getBalance());  // 1500

console.log(myAccount.balance);       // undefined ❌ (truly private!)
```

### 5️⃣ INTERVIEW ANSWER

> *"A closure is created when a function **retains access to variables from its lexical scope** even after the outer function has finished executing. Internally, the inner function maintains a reference to the outer scope's variables. Closures are practically useful for **data privacy, maintaining state**, and creating **function factories**. A classic example is a counter function where the count variable remains private but persists across multiple calls."*

📌 **One-liner:** Inner function + outer variables + outer function gone = Closure

---

---

## ⚡ Lightning Revision Sheet (5 Min Before Interview)

| # | Topic | One-Line Answer |
|---|-------|----------------|
| 1 | **Data Types** | 7 primitives (String, Number, Boolean, Undefined, Null, BigInt, Symbol) + Object |
| 2 | **var/let/const** | Scope (function vs block), redeclaration, reassignment — `const` by default |
| 3 | **== vs ===** | `==` coerces types, `===` checks value + type — always use `===` |
| 4 | **Arrow Functions** | Short syntax, no own `this` (lexical), no `arguments`, no `new` |
| 5 | **Callbacks** | Function passed as argument, executed later — sync or async |
| 6 | **async/await** | Syntactic sugar over Promises, pauses function not thread, use try/catch |
| 7 | **Promises** | Future result object: Pending → Fulfilled/Rejected, avoids callback hell |
| 8 | **Closures** | Inner function remembers outer scope variables even after outer returns |

---

## 🎯 Final Tips

| Do ✅ | Don't ❌ |
|-------|---------|
| 5-step framework use karo (What→Why→How→Code→Use) | Definition rat ke mat bolo |
| Code example zaroor do | Sirf theory mat batao |
| "I prefer..." se personal touch do | Robot jaisa answer mat do |
| Follow-up questions ke liye ready raho | Ek line bolke chup mat ho jao |
| `const` trap, `this` trap, `==` trap yaad rakho | Surface-level answer mat do |
