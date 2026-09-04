# 🚀 JavaScript & Programming Fundamentals — Intermediate Level

> **Level:** Intermediate
> **Format:** What → Why → How → Example → Interview Answer
> **Prerequisite:** Basic Level concepts (Data Types, Closures, Promises, etc.)

---

## 📑 Table of Contents

| # | Topic | Difficulty | Frequency |
|---|-------|-----------|-----------|
| 1 | [Event Bubbling & Capturing](#1-event-bubbling--event-capturing) | ⭐⭐ | 🔥🔥🔥🔥 |
| 2 | [Event Loop & Call Stack](#2-event-loop--call-stack) | ⭐⭐⭐ | 🔥🔥🔥🔥🔥 |
| 3 | [`null` vs `undefined`](#3-null-vs-undefined) | ⭐⭐ | 🔥🔥🔥🔥 |
| 4 | [Prototypal Inheritance](#4-prototypal-inheritance) | ⭐⭐⭐ | 🔥🔥🔥🔥 |
| 5 | [Higher-Order Functions](#5-higher-order-functions) | ⭐⭐ | 🔥🔥🔥 |
| 6 | [`map()` vs `filter()` vs `reduce()`](#6-map-vs-filter-vs-reduce) | ⭐⭐ | 🔥🔥🔥🔥🔥 |
| 7 | [`this` Keyword](#7-this-keyword) | ⭐⭐⭐ | 🔥🔥🔥🔥🔥 |
| 8 | [Error Handling in `async/await`](#8-error-handling-in-asyncawait) | ⭐⭐ | 🔥🔥🔥 |

---

## 🧠 Answer Framework (Reminder)

```
1️⃣  WHAT  →  "Yeh kya hai?"
2️⃣  WHY   →  "Iski zaroorat kyun hai?"
3️⃣  HOW   →  "Yeh kaam kaise karta hai?"
4️⃣  CODE  →  "Ek example dikhata hoon..."
5️⃣  USE   →  "Practical mein main isse tab use karta hoon jab..."
```

---

---

## 1. Event Bubbling & Event Capturing

### 1️⃣ WHAT

Jab DOM mein koi event trigger hota hai (jaise click), toh woh event **DOM tree mein travel** karta hai. Ye travel **2 directions** mein ho sakta hai:

| Phase | Direction | Alias |
|-------|-----------|-------|
| **Capturing** (Trickling) | ⬇️ Top → Bottom (Parent → Child) | "Pehle bade log" |
| **Target** | 🎯 Jis element pe event hua | "Actual target" |
| **Bubbling** | ⬆️ Bottom → Top (Child → Parent) | "Pehle chhote log" |

> **Default behavior = Bubbling** (bottom-up)

### 2️⃣ WHY

Real-world DOM nested hota hai:

```html
<div id="grandparent">
    <div id="parent">
        <button id="child">Click Me</button>
    </div>
</div>
```

Agar aap button click karo, toh **teenon elements** ko pata chalna chahiye ki click hua. Event propagation decide karta hai ki **kis order** mein pata chalega.

### 3️⃣ HOW — Visual Flow

```
Capturing Phase (⬇️)          Bubbling Phase (⬆️)
─────────────────────          ──────────────────
  document                        document
      ↓                               ↑
    html                            html
      ↓                               ↑
    body                            body
      ↓                               ↑
   #grandparent  ──→  TARGET  ──→  #grandparent
      ↓                 ↑               ↑
    #parent             ↑            #parent
      ↓                 ↑               ↑
   #child (button) ─────┘           #child
```

### 4️⃣ CODE

```javascript
const grandparent = document.getElementById("grandparent");
const parent = document.getElementById("parent");
const child = document.getElementById("child");

// ── BUBBLING (default — 3rd argument false or omitted) ──
grandparent.addEventListener("click", () => {
    console.log("Grandparent clicked");  // 3rd
});

parent.addEventListener("click", () => {
    console.log("Parent clicked");       // 2nd
});

child.addEventListener("click", () => {
    console.log("Child clicked");        // 1st
});

// Click button → Output:
// Child clicked
// Parent clicked
// Grandparent clicked

// ── CAPTURING (3rd argument = true) ──
grandparent.addEventListener("click", () => {
    console.log("Grandparent (capture)");  // 1st
}, true);

parent.addEventListener("click", () => {
    console.log("Parent (capture)");       // 2nd
}, true);

// ── STOP PROPAGATION ──
child.addEventListener("click", (e) => {
    e.stopPropagation();  // Event yahan ruk jayega, upar nahi jayega
    console.log("Child only");
});
```

### 5️⃣ INTERVIEW ANSWER

> *"Event propagation in the DOM happens in three phases: **capturing** (top-down from document to target), **target**, and **bubbling** (bottom-up from target back to document). By default, event listeners fire during the **bubbling phase**. We can enable capturing by passing `true` as the third argument to `addEventListener`. We can stop propagation using `event.stopPropagation()`. In practice, I mostly rely on bubbling and use **event delegation** — attaching a single listener to a parent to handle events from multiple children."*

📌 **One-liner:** Bubbling = ⬆️ Child→Parent (default) | Capturing = ⬇️ Parent→Child

---

---

## 2. Event Loop & Call Stack

> 🏆 **Most important JS interview question at intermediate level!**

### 1️⃣ WHAT

JavaScript **single-threaded** hai (ek time pe ek hi kaam). Phir bhi async kaam kaise hota hai? Jawab: **Event Loop**.

| Component | Role |
|-----------|------|
| **Call Stack** | Synchronous code execute karta hai (LIFO) |
| **Web APIs** | Browser async kaam handle karta hai (setTimeout, fetch, DOM) |
| **Callback Queue** (Task Queue) | Async callbacks yahan wait karte hain |
| **Microtask Queue** | Promises, `queueMicrotask` yahan jaate hain (**higher priority**) |
| **Event Loop** | Constantly check karta hai: Stack empty? → Queue se uthao |

### 2️⃣ WHY

Agar JS synchronous hota toh:
- `fetch()` call pe pura page **freeze** ho jata
- `setTimeout` pe browser **ruk** jata
- User **kuch click nahi** kar pata

Event Loop JS ko **non-blocking** banata hai despite being single-threaded.

### 3️⃣ HOW — Step-by-Step Flow

```
┌─────────────────────────────────────────────────┐
│                  BROWSER                         │
│                                                  │
│   ┌──────────┐    ┌────────────┐                │
│   │Call Stack│    │  Web APIs  │                │
│   │  (LIFO)  │───→│ setTimeout │                │
│   │          │    │   fetch    │                │
│   │          │    │   DOM      │                │
│   └────┬─────┘    └─────┬──────┘                │
│        │                │ (done)                 │
│        │                ▼                        │
│        │    ┌───────────────────────┐            │
│        │    │  Microtask Queue      │ ← Promises │
│        │    │  (HIGH PRIORITY)      │            │
│        │    ├───────────────────────┤            │
│        │    │  Callback Queue       │ ← setTimeout│
│        │    │  (LOW PRIORITY)       │            │
│        │    └───────────┬───────────┘            │
│        │                │                        │
│        │         ┌──────┴──────┐                 │
│        └─────────│ EVENT LOOP  │                 │
│                  │ "Stack empty │                 │
│                  │  hai? Queue  │                 │
│                  │  se uthao!"  │                 │
│                  └─────────────┘                 │
└─────────────────────────────────────────────────┘
```

**Event Loop ka Rule:**
```
1. Call Stack mein code push karo → execute karo → pop karo
2. Async kaam Web APIs ko bhejo
3. Jab Web API ka kaam ho jaye → callback Queue mein daalo
4. Event Loop check karta hai:
   → Stack EMPTY hai?
     → Microtask Queue check (Promises pehle!)
     → Phir Callback Queue (setTimeout etc.)
     → Callback ko Stack mein push karo
5. Repeat forever 🔄
```

### 4️⃣ CODE — Classic Interview Output Question

```javascript
console.log("1: Start");

setTimeout(() => {
    console.log("2: setTimeout");
}, 0);

Promise.resolve().then(() => {
    console.log("3: Promise");
});

console.log("4: End");

// Output:
// 1: Start        ← Call Stack (sync)
// 4: End          ← Call Stack (sync)
// 3: Promise      ← Microtask Queue (HIGH priority)
// 2: setTimeout   ← Callback Queue (LOW priority)
```

**Why this order?**

```
Step 1: "1: Start" → Stack → Execute → Pop
Step 2: setTimeout → Web API → Callback Queue mein wait
Step 3: Promise → Microtask Queue mein wait
Step 4: "4: End" → Stack → Execute → Pop
Step 5: Stack empty!
        → Microtask Queue check → "3: Promise" execute
        → Callback Queue check → "2: setTimeout" execute
```

### 5️⃣ INTERVIEW ANSWER

> *"JavaScript is **single-threaded** but handles async operations through the **Event Loop**. Synchronous code runs on the **Call Stack**. Async operations like `setTimeout` are offloaded to **Web APIs**, and their callbacks are placed in the **Callback Queue** once complete. Promises go to the **Microtask Queue**, which has **higher priority**. The Event Loop continuously checks: if the Call Stack is empty, it first drains the Microtask Queue, then picks from the Callback Queue. This is why a `Promise.resolve().then()` runs before a `setTimeout(fn, 0)`."*

📌 **One-liner:** Stack (sync) → Microtask (Promises) → Callback Queue (setTimeout) | Event Loop orchestrates all

---

---

## 3. `null` vs `undefined`

### 1️⃣ WHAT

Dono "empty" lagte hain, but **meaning alag** hai:

| | `undefined` | `null` |
|---|------------|--------|
| **Meaning** | Variable declared hai, but **value assign nahi** hui | **Intentionally** empty value assign ki hai |
| **Type** | `"undefined"` | `"object"` (JS ka famous bug 🐛) |
| **Set by** | JavaScript engine (automatic) | Developer (manual) |
| **Analogy** | Khali dabba — kuch daala hi nahi | Khali dabba — jaan-bujh ke khaali rakha |

### 2️⃣ WHY

Ye distinction zaroori hai kyunki:
- `undefined` = **"Mujhe abhi pata nahi"** (absence of assignment)
- `null` = **"Mujhe pata hai, aur value kuch nahi hai"** (intentional absence)

### 3️⃣ HOW

```javascript
// ── undefined scenarios ──
let x;
console.log(x);          // undefined (declared, no value)

function greet(name) {
    console.log(name);
}
greet();                 // undefined (argument nahi diya)

const obj = { a: 1 };
console.log(obj.b);      // undefined (property exist nahi karti)

// ── null scenario ──
let user = null;         // Developer ne intentionally khaali rakha
console.log(user);       // null
```

### 4️⃣ CODE — Tricky Comparisons

```javascript
// ── Type Check ──
typeof undefined   // "undefined"
typeof null        // "object"  ← 🐛 JS ka oldest bug (1995 se!)

// ── Equality ──
null == undefined   // true  (loose equality — dono "empty" hain)
null === undefined  // false (strict — types alag hain)

// ── Arithmetic ──
null + 5       // 5     (null → 0)
undefined + 5  // NaN   (undefined → NaN)

// ── Boolean ──
Boolean(null)       // false
Boolean(undefined)  // false
// Dono falsy hain
```

### 5️⃣ INTERVIEW ANSWER

> *"`undefined` means a variable has been **declared but not assigned** a value — it's JavaScript's default. `null` is an **intentional assignment** representing the absence of any value. Interestingly, `typeof null` returns `'object'`, which is a well-known **legacy bug** in JavaScript. In loose equality, `null == undefined` is `true`, but in strict equality they are `false` because their types differ. In practice, I use `null` when I want to explicitly indicate 'no value' and let `undefined` remain as JavaScript's default for uninitialized variables."*

📌 **One-liner:** `undefined` = not assigned (JS default) | `null` = intentionally empty (developer choice)

---

---

## 4. Prototypal Inheritance

### 1️⃣ WHAT

JavaScript mein **classes nahi hoti** (ES6 `class` syntax sugar hai). Real inheritance **prototype chain** se hoti hai:

> **Har JavaScript object ke paas ek hidden `[[Prototype]]` link hota hai jo doosre object ki taraf point karta hai. Jab koi property milni nahi aati, JS us prototype pe jaake dhundhta hai.**

### 2️⃣ WHY

- Memory efficient — shared methods **ek jagah** rehti hain (har instance pe copy nahi)
- Dynamic — runtime pe bhi prototype modify kar sakte ho
- JS ka **core mechanism** — classes, `Object.create`, sab isi pe based hain

### 3️⃣ HOW — Prototype Chain Visual

```
const dog = { bark: function() { return "Woof!"; } };
const myDog = Object.create(dog);
myDog.name = "Tommy";

myDog.name   // "Tommy"  ← own property
myDog.bark() // "Woof!"  ← prototype se aaya!

Chain:
myDog  →  dog  →  Object.prototype  →  null
  ↑          ↑            ↑              ↑
 own      inherited    toString(),     end of
property   property    hasOwn...      chain
```

**Lookup Process:**
```
myDog.bark()
    ↓
myDog ke paas bark hai? → NO
    ↓
myDog.__proto__ (dog) ke paas hai? → YES ✅
    ↓
Return "Woof!"
```

### 4️⃣ CODE

```javascript
// ── Method 1: Object.create() ──
const animal = {
    eat() { return "Eating..."; }
};

const cat = Object.create(animal);
cat.meow = function() { return "Meow!"; };

console.log(cat.eat());   // "Eating..." (inherited)
console.log(cat.meow());  // "Meow!" (own)

// ── Method 2: Constructor Function (pre-ES6) ──
function Person(name) {
    this.name = name;  // instance property
}

Person.prototype.greet = function() {  // shared method
    return `Hi, I'm ${this.name}`;
};

const p1 = new Person("Kanishk");
const p2 = new Person("Rahul");

p1.greet();  // "Hi, I'm Kanishk"
p2.greet();  // "Hi, I'm Rahul"
// greet() ek hi jagah hai — Person.prototype pe!

// ── Method 3: ES6 Class (syntactic sugar!) ──
class Animal {
    constructor(name) { this.name = name; }
    speak() { return `${this.name} makes a sound`; }
}

class Dog extends Animal {
    bark() { return `${this.name} barks!`; }
}

const d = new Dog("Tommy");
d.speak();  // "Tommy makes a sound" (inherited)
d.bark();   // "Tommy barks!" (own)

// Proof: class = prototype sugar
console.log(typeof Animal);  // "function"
```

### 5️⃣ INTERVIEW ANSWER

> *"JavaScript uses **prototypal inheritance** instead of classical inheritance. Every object has an internal `[[Prototype]]` link to another object. When you access a property, JavaScript looks up the **prototype chain** until it finds the property or reaches `null`. This is memory-efficient because methods are stored **once on the prototype** and shared across all instances. ES6 `class` syntax is just **syntactic sugar** over this same prototype mechanism — under the hood, it's still prototypal inheritance."*

📌 **One-liner:** Object → Prototype → Prototype → ... → null (chain lookup)

---

---

## 5. Higher-Order Functions

### 1️⃣ WHAT

> **Higher-Order Function (HOF) = ek function jo ya toh function ko argument mein leta hai, ya function return karta hai, ya dono.**

Simple rule:
```
Function + (Function as Input OR Function as Output) = HOF
```

### 2️⃣ WHY

- **Reusability** — common patterns abstract kar sakte ho
- **Composition** — chhote functions jodke bada logic bana sakte ho
- **Declarative code** — "kya karna hai" batao, "kaise" nahi
- JavaScript mein functions **first-class citizens** hain (variables jaisa treat hote hain)

### 3️⃣ HOW — 3 Types

```
Type 1: Function as ARGUMENT
        ┌──────────────────────┐
        │  HOF(callbackFn)     │
        └──────────────────────┘

Type 2: Function as RETURN VALUE
        ┌──────────────────────┐
        │  HOF() → returns fn  │
        └──────────────────────┘

Type 3: BOTH
        ┌──────────────────────┐
        │  HOF(fn) → returns fn│
        └──────────────────────┘
```

### 4️⃣ CODE

```javascript
// ── Type 1: Function as Argument ──
function greet(name, formatter) {
    console.log(formatter(name));
}

greet("Kanishk", (n) => `Hello, ${n}!`);   // Hello, Kanishk!
greet("Kanishk", (n) => `HEY ${n.toUpperCase()}!`);  // HEY KANISHK!

// ── Type 2: Function as Return Value (Closure + HOF!) ──
function multiplier(factor) {
    return function(number) {
        return number * factor;
    };
}

const double = multiplier(2);
const triple = multiplier(3);

double(5);   // 10
triple(5);   // 15

// ── Type 3: Built-in HOFs ──
const nums = [1, 2, 3, 4, 5];

nums.map(n => n * 2);       // [2, 4, 6, 8, 10]
nums.filter(n => n > 3);    // [4, 5]
nums.reduce((a, b) => a + b, 0);  // 15

// ── Real-World: Custom HOF ──
function withLogging(fn) {
    return function(...args) {
        console.log(`Calling with:`, args);
        const result = fn(...args);
        console.log(`Result:`, result);
        return result;
    };
}

const add = (a, b) => a + b;
const loggedAdd = withLogging(add);

loggedAdd(3, 4);
// Calling with: [3, 4]
// Result: 7
```

### 5️⃣ INTERVIEW ANSWER

> *"A higher-order function is a function that either **takes a function as an argument** or **returns a function** — or both. This is possible because functions in JavaScript are **first-class citizens**. Common built-in examples include `map`, `filter`, and `reduce`. Higher-order functions enable powerful patterns like **function composition, decorators, and closures**. For example, a `multiplier` function can return a new function that multiplies by a specific factor."*

📌 **One-liner:** Function jo function le ya function de = Higher-Order Function

---

---

## 6. `map()` vs `filter()` vs `reduce()`

> ⚠️ **Most asked array method question!**

### 1️⃣ WHAT

Teeno **Higher-Order Array Methods** hain jo original array ko **mutate nahi** karte (immutable):

| Method | Purpose | Returns | Array Size |
|--------|---------|---------|-----------|
| `map()` | Har element ko **transform** karo | New Array | Same size |
| `filter()` | Condition ke basis pe **select** karo | New Array | Same or smaller |
| `reduce()` | Sab elements ko **ek single value** mein combine karo | Single Value | N/A |

### 2️⃣ WHY

Bina inke → `for` loops, `push`, temporary variables = **messy code**

Inke saath → **declarative, chainable, readable** code

### 3️⃣ HOW — Visual Analogy

```
Original Array:  🍎 🍌 🍎 🍊 🍌 🍎

map()     → Har fruit ka juice nikalo
            🧃 🧃 🧃 🧃 🧃 🧃  (same count, transformed)

filter()  → Sirf 🍎 chuno
            🍎 🍎 🍎  (filtered subset)

reduce()  → Sab fruits ka total weight nikalo
            15 kg  (single value)
```

### 4️⃣ CODE

```javascript
const products = [
    { name: "Laptop", price: 50000, inStock: true },
    { name: "Phone", price: 25000, inStock: false },
    { name: "Tablet", price: 30000, inStock: true },
    { name: "Watch", price: 10000, inStock: true },
];

// ── MAP: Transform ──
const names = products.map(p => p.name);
// ["Laptop", "Phone", "Tablet", "Watch"]

const discounted = products.map(p => ({
    ...p,
    price: p.price * 0.9  // 10% off
}));

// ── FILTER: Select ──
const available = products.filter(p => p.inStock);
// [{Laptop}, {Tablet}, {Watch}]

const expensive = products.filter(p => p.price > 20000);
// [{Laptop}, {Phone}, {Tablet}]

// ── REDUCE: Accumulate ──
const total = products.reduce((sum, p) => sum + p.price, 0);
// 115000

// ── 🔥 CHAINING (Real Power!) ──
const totalInStock = products
    .filter(p => p.inStock)          // Step 1: available items
    .map(p => p.price * 0.9)         // Step 2: 10% discount
    .reduce((sum, price) => sum + price, 0);  // Step 3: total
// 81000

// ── REDUCE Advanced: Group By ──
const grouped = products.reduce((acc, p) => {
    const key = p.inStock ? "available" : "outOfStock";
    acc[key] = acc[key] || [];
    acc[key].push(p.name);
    return acc;
}, {});
// { available: ["Laptop", "Tablet", "Watch"], outOfStock: ["Phone"] }
```

### 5️⃣ INTERVIEW ANSWER

> *"`map()`, `filter()`, and `reduce()` are higher-order array methods that don't mutate the original array. **`map`** transforms each element and returns a new array of the **same length**. **`filter`** selects elements based on a condition and returns a **subset**. **`reduce`** accumulates all elements into a **single value** using an accumulator. The real power comes from **chaining** them — for example, filtering in-stock products, mapping to discounted prices, and reducing to a total — all in one readable pipeline."*

📌 **One-liner:** `map` = transform | `filter` = select | `reduce` = accumulate to one value

---

---

## 7. `this` Keyword

> 🏆 **Most confusing yet most asked topic!**

### 1️⃣ WHAT

`this` ek **special keyword** hai jo us **context** ko refer karta hai jismein function execute ho raha hai.

> **`this` ki value function ke likhne se nahi, function ke CALL karne se decide hoti hai!**

### 2️⃣ WHY

`this` ki zaroorat tab padti hai jab:
- Object methods mein current object access karna ho
- Constructor functions mein instance properties set karni ho
- Event handlers mein triggered element chahiye

### 3️⃣ HOW — The 4 Rules of `this`

| # | Rule | `this` Points To | Example |
|---|------|-----------------|---------|
| 1️⃣ | **Default/Global** | `window` (browser) / `global` (Node) | `function fn() { this }` |
| 2️⃣ | **Implicit (Object)** | Object ke **left of dot** | `obj.method()` → `this = obj` |
| 3️⃣ | **Explicit** | Jo tum **force** karo | `call()`, `apply()`, `bind()` |
| 4️⃣ | **`new` (Constructor)** | Naya **instance** | `new Person()` → `this = {}` |
| ⭐ | **Arrow Function** | **Lexical** (surrounding scope) | Parent ka `this` inherit |

### 4️⃣ CODE — All 4 Rules

```javascript
// ── Rule 1: Default (Global) ──
function showThis() {
    console.log(this);
}
showThis();  // window (browser) / global (Node)
// Strict mode mein: undefined

// ── Rule 2: Implicit (Object Method) ──
const user = {
    name: "Kanishk",
    greet() {
        console.log(this.name);
    }
};
user.greet();  // "Kanishk"  ← this = user (left of dot!)

// ⚠️ Trap: Extracting method
const fn = user.greet;
fn();  // undefined  ← this = window (no object left of dot!)

// ── Rule 3: Explicit (call, apply, bind) ──
function introduce(role) {
    console.log(`${this.name} is a ${role}`);
}

const person = { name: "Kanishk" };

introduce.call(person, "Developer");   // "Kanishk is a Developer"
introduce.apply(person, ["Designer"]); // "Kanishk is a Designer"

const boundFn = introduce.bind(person, "Manager");
boundFn();  // "Kanishk is a Manager"

// ── Rule 4: `new` Keyword ──
function Person(name) {
    this.name = name;  // this = newly created {}
}
const p = new Person("Kanishk");
console.log(p.name);  // "Kanishk"

// ── ⭐ Arrow Function (Lexical this) ──
const team = {
    name: "Dev Team",
    members: ["Kanishk", "Rahul"],

    // ❌ Wrong: Arrow as method
    showWrong: () => {
        console.log(this.name);  // undefined (this = window)
    },

    // ✅ Correct: Regular method + arrow inside
    showCorrect() {
        this.members.forEach(member => {
            console.log(`${member} is in ${this.name}`);
            // Arrow inherits this from showCorrect → this = team
        });
    }
};

team.showCorrect();
// Kanishk is in Dev Team
// Rahul is in Dev Team
```

### 5️⃣ INTERVIEW ANSWER

> *"The value of `this` is determined by **how a function is called**, not where it's defined. There are four main rules: **Default** binding points to the global object, **Implicit** binding points to the object before the dot, **Explicit** binding via `call`, `apply`, or `bind` lets you set `this` manually, and the **`new`** keyword binds `this` to the newly created instance. Arrow functions are special — they don't have their own `this` and instead **lexically inherit** it from the enclosing scope. A common pitfall is losing `this` when extracting a method from an object, which I solve using `.bind()` or arrow functions."*

📌 **One-liner:** `this` = call site pe decide hota hai | Arrow = parent ka `this`

---

---

## 8. Error Handling in `async/await`

### 1️⃣ WHAT

`async/await` mein errors handle karne ke **3 main tarike** hain:

| Method | Syntax | Best For |
|--------|--------|----------|
| **try/catch** | `try { await } catch(e) {}` | Single/multiple awaits |
| **Wrapper Function** | Utility function `[error, data]` | Clean, Go-style |
| **.catch() on Promise** | `await fn().catch()` | Single quick catch |

### 2️⃣ WHY

`async/await` synchronous dikhta hai, but **errors still aate hain** (network fail, API 500, invalid data). Agar handle nahi kiya → **unhandled promise rejection** → app crash.

### 3️⃣ HOW — All 3 Methods

```
Method 1: try/catch (Most Common)
┌─────────────────────────────┐
│  try {                      │
│      const data = await fn()│  ← Success path
│  } catch (error) {          │
│      handle(error)          │  ← Error path
│  } finally {                │
│      cleanup()              │  ← Always runs
│  }                          │
└─────────────────────────────┘

Method 2: Wrapper (Go-style)
┌─────────────────────────────┐
│  const [err, data] =        │
│      await to(fetchData())  │
│  if (err) handleError()     │
└─────────────────────────────┘

Method 3: .catch() inline
┌─────────────────────────────┐
│  const data = await fn()    │
│      .catch(e => fallback)  │
└─────────────────────────────┘
```

### 4️⃣ CODE

```javascript
// ── Method 1: try/catch (Standard) ──
async function fetchUser(id) {
    try {
        const response = await fetch(`/api/users/${id}`);

        if (!response.ok) {
            throw new Error(`HTTP ${response.status}`);  // Manual throw
        }

        const user = await response.json();
        return user;

    } catch (error) {
        console.error("Failed to fetch user:", error.message);
        return null;  // Fallback

    } finally {
        console.log("Request completed");  // Always runs
    }
}

// ── Method 2: Wrapper Function (Clean!) ──
function to(promise) {
    return promise
        .then(data => [null, data])
        .catch(err => [err, null]);
}

async function loadData() {
    const [err, user] = await to(fetchUser(1));

    if (err) {
        console.log("Error:", err.message);
        return;
    }

    console.log("User:", user);
}

// ── Method 3: Inline .catch() ──
async function quickFetch() {
    const data = await fetch("/api/data")
        .then(r => r.json())
        .catch(() => ({ fallback: true }));

    console.log(data);
}

// ── ⚠️ Common Mistake ──
async function wrong() {
    const data = await fetchData();  // ❌ No error handling!
    // If fetchData rejects → UnhandledPromiseRejection
}

// ── ✅ Multiple Awaits ──
async function loadDashboard() {
    try {
        const [user, orders, products] = await Promise.all([
            fetchUser(),
            fetchOrders(),
            fetchProducts()
        ]);
        return { user, orders, products };
    } catch (error) {
        console.log("Dashboard load failed:", error);
    }
}
```

### 5️⃣ INTERVIEW ANSWER

> *"In `async/await`, I primarily use **try/catch/finally** blocks for error handling. The `try` block contains the awaited calls, `catch` handles any rejected promises or thrown errors, and `finally` runs cleanup code regardless of outcome. For cleaner code with multiple async calls, I sometimes use a **wrapper utility** that returns an `[error, data]` tuple — inspired by Go's error handling pattern. A critical mistake to avoid is forgetting error handling entirely, which leads to **unhandled promise rejections**. For parallel operations, I combine `Promise.all()` inside a single try/catch."*

📌 **One-liner:** `try/catch` = standard | Wrapper = clean | `.catch()` = quick | Never skip error handling!

---

---

## ⚡ Lightning Revision Sheet (5 Min Before Interview)

| # | Topic | One-Line Answer |
|---|-------|----------------|
| 1 | **Bubbling/Capturing** | Bubbling = ⬆️ Child→Parent (default), Capturing = ⬇️ Parent→Child |
| 2 | **Event Loop** | Stack (sync) → Microtask (Promises) → Callback Queue (setTimeout) |
| 3 | **null vs undefined** | `undefined` = not assigned (JS), `null` = intentionally empty (dev) |
| 4 | **Prototypal Inheritance** | Object → Prototype chain lookup → `null` (ES6 class = sugar) |
| 5 | **Higher-Order Functions** | Function that takes/returns another function (map, filter, etc.) |
| 6 | **map/filter/reduce** | map = transform, filter = select, reduce = accumulate to one value |
| 7 | **`this` Keyword** | Depends on call site: global, object, explicit, new; arrow = lexical |
| 8 | **async/await Errors** | try/catch/finally standard; wrapper for clean; never skip handling |

---

## 🔗 Basic + Intermediate Connection Map

```
Basic Level                          Intermediate Level
─────────────                        ──────────────────
Data Types ──────────────────→  null vs undefined
var/let/const ──────────────→  this keyword (scope matters!)
Arrow Functions ────────────→  this (lexical) + HOFs
Callbacks ──────────────────→  Event Bubbling + HOFs
Promises ───────────────────→  Event Loop + Error Handling
async/await ────────────────→  Error Handling Patterns
Closures ───────────────────→  Prototypal Inheritance + HOFs
== vs === ──────────────────→  null == undefined (true!) trap
```

---
