# JavaScript — Advanced Interview Topics

## Closures

A closure is a function that **remembers the variables from its outer scope** even after the outer function has returned. This is the single most tested JS concept in interviews.

```javascript
function makeCounter() {
  let count = 0;                    // captured by the inner function
  return {
    increment: () => ++count,
    getCount: () => count,
  };
}

const counter = makeCounter();
counter.increment();
counter.increment();
console.log(counter.getCount()); // 2 — count persists via closure
```

### Classic Closure Trap (The Loop Problem)

```javascript
// ❌ All callbacks print 3
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// prints: 3, 3, 3 — var is function-scoped, shared reference

// ✅ Fix 1: use let (block-scoped, new binding each iteration)
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// prints: 0, 1, 2

// ✅ Fix 2: IIFE to create a new scope
for (var i = 0; i < 3; i++) {
  ((j) => setTimeout(() => console.log(j), 100))(i);
}
```

### Practical Uses

```javascript
// Private state (module pattern)
const auth = (() => {
  let token = null;
  return {
    setToken: (t) => { token = t; },
    getToken: () => token,
  };
})();

// Partial application / currying
const multiply = (a) => (b) => a * b;
const double = multiply(2);
console.log(double(5)); // 10

// Memoization via closure
function memoize(fn) {
  const cache = new Map();
  return function (...args) {
    const key = JSON.stringify(args);
    if (!cache.has(key)) cache.set(key, fn.apply(this, args));
    return cache.get(key);
  };
}
```

---

## The Event Loop

JavaScript is **single-threaded** with an event-driven concurrency model. Understanding the execution order is a top interview question.

### Execution Order

```
Call Stack → Microtask Queue (Promises, queueMicrotask) → Macrotask Queue (setTimeout, setInterval, I/O)
```

```javascript
console.log("1");                          // sync — runs first

setTimeout(() => console.log("2"), 0);     // macrotask — runs last

Promise.resolve().then(() => console.log("3")); // microtask — runs second

console.log("4");                          // sync — runs third

// Output: 1, 4, 3, 2
```

### Key Rules

| Queue | Examples | Priority |
|-------|----------|----------|
| Call stack | Synchronous code | Immediate |
| Microtasks | `Promise.then`, `queueMicrotask`, `MutationObserver` | After current task, before next macrotask |
| Macrotasks | `setTimeout`, `setInterval`, `setImmediate`, I/O | After all microtasks drain |

```javascript
// Microtasks can starve macrotasks
queueMicrotask(function recurse() {
  queueMicrotask(recurse);
  // ⚠️ This blocks setTimeout indefinitely — microtask queue never empties
});
```

---

## Prototypes and Inheritance

Every JS object has a hidden `[[Prototype]]` link. Property lookups walk the **prototype chain**.

```javascript
// Constructor function + prototype
function Animal(name) {
  this.name = name;
}
Animal.prototype.speak = function () {
  return `${this.name} makes a sound`;
};

function Dog(name, breed) {
  Animal.call(this, name);      // call parent constructor
  this.breed = breed;
}
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;
Dog.prototype.bark = function () { return "Woof!"; };

// ES6 class syntax (sugar over prototypes)
class Cat extends Animal {
  purr() { return "Purr..."; }
}

const dog = new Dog("Rex", "Lab");
console.log(dog.speak());  // "Rex makes a sound" — found on Animal.prototype
console.log(dog.bark());   // "Woof!" — found on Dog.prototype
```

### `this` Binding Rules (Most Asked)

| Rule | `this` is... | Example |
|------|-------------|---------|
| Default | `globalThis` (or `undefined` in strict) | `func()` |
| Implicit | The object before the dot | `obj.method()` |
| Explicit | Whatever you pass | `func.call(obj)`, `func.apply(obj)` |
| `new` | The newly created object | `new Func()` |
| Arrow function | Lexically inherited (no own `this`) | `() => this.x` |

```javascript
const obj = {
  name: "Alice",
  greet: function () { return this.name; },
  greetArrow: () => this.name,              // ⚠️ captures outer this
};
console.log(obj.greet());       // "Alice"
console.log(obj.greetArrow());  // undefined (this = global/module)

// .bind() creates a new function with fixed this
const greet = obj.greet.bind(obj);
setTimeout(greet, 100);         // "Alice" — this is locked
```

---

## Promises and Async/Await

### Promise Combinators

```javascript
// Promise.all — fails fast on first rejection
const results = await Promise.all([fetchA(), fetchB(), fetchC()]);

// Promise.allSettled — waits for all, never rejects
const outcomes = await Promise.allSettled([fetchA(), fetchB()]);
// [{ status: "fulfilled", value: ... }, { status: "rejected", reason: ... }]

// Promise.race — first to settle (resolve or reject) wins
const fastest = await Promise.race([fetchA(), timeout(5000)]);

// Promise.any — first to resolve wins (ignores rejections)
const first = await Promise.any([mirror1(), mirror2(), mirror3()]);
```

### Common Patterns

```javascript
// Sequential vs parallel execution
// ❌ Sequential — slow
for (const url of urls) {
  const data = await fetch(url);
}

// ✅ Parallel — fast
const promises = urls.map(url => fetch(url));
const responses = await Promise.all(promises);

// Retry with exponential backoff
async function fetchWithRetry(url, retries = 3) {
  for (let i = 0; i < retries; i++) {
    try {
      return await fetch(url);
    } catch (err) {
      if (i === retries - 1) throw err;
      await new Promise(r => setTimeout(r, 2 ** i * 1000));
    }
  }
}
```

---

## Proxy and Reflect

`Proxy` intercepts operations on objects. Interviewers use it to test deep understanding of the object model.

```javascript
// Validation proxy — reject invalid assignments
const validator = new Proxy({}, {
  set(target, prop, value) {
    if (prop === "age" && (typeof value !== "number" || value < 0)) {
      throw new TypeError("Age must be a positive number");
    }
    target[prop] = value;
    return true;
  }
});

validator.age = 25;       // OK
// validator.age = -1;    // TypeError

// Reactive proxy — track access (Vue 3 reactivity system)
function reactive(obj) {
  return new Proxy(obj, {
    get(target, prop) {
      console.log(`Read ${String(prop)}`);
      return Reflect.get(target, prop);
    },
    set(target, prop, value) {
      console.log(`Write ${String(prop)} = ${value}`);
      return Reflect.set(target, prop, value);
    },
  });
}

// Negative array indices (Python-style)
function negativeArray(arr) {
  return new Proxy(arr, {
    get(target, prop) {
      const index = Number(prop);
      if (Number.isInteger(index) && index < 0)
        return target[target.length + index];
      return Reflect.get(target, prop);
    },
  });
}
const a = negativeArray([1, 2, 3, 4]);
console.log(a[-1]); // 4
```

---

## Generators and Iterators

```javascript
// Generator function — pauses at each yield
function* range(start, end) {
  for (let i = start; i < end; i++) {
    yield i;
  }
}

console.log([...range(0, 5)]); // [0, 1, 2, 3, 4]

// Infinite sequence (lazy evaluation)
function* fibonacci() {
  let a = 0, b = 1;
  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

// Take first n from infinite generator
function take(gen, n) {
  const result = [];
  for (const val of gen) {
    result.push(val);
    if (result.length === n) break;
  }
  return result;
}

console.log(take(fibonacci(), 8)); // [0, 1, 1, 2, 3, 5, 8, 13]

// Async generator — stream data
async function* fetchPages(baseUrl) {
  let page = 1;
  while (true) {
    const resp = await fetch(`${baseUrl}?page=${page}`);
    const data = await resp.json();
    if (data.length === 0) return;
    yield data;
    page++;
  }
}
```

---

## WeakMap and WeakSet

Keys are held **weakly** — if no other reference exists, the entry is garbage collected. Perfect for caching metadata about objects without causing memory leaks.

```javascript
// Private data store (pre-class fields pattern)
const _private = new WeakMap();
class Person {
  constructor(name) {
    _private.set(this, { name });
  }
  getName() {
    return _private.get(this).name;
  }
}
// When Person instance is GC'd, the WeakMap entry is also collected

// DOM element metadata — no memory leaks
const meta = new WeakMap();
function trackElement(el) {
  meta.set(el, { clickCount: 0 });
}
// When el is removed from DOM and dereferenced, metadata is GC'd

// WeakSet — track seen objects (e.g. detect circular references)
function deepClone(obj, seen = new WeakSet()) {
  if (obj === null || typeof obj !== "object") return obj;
  if (seen.has(obj)) throw new Error("Circular reference");
  seen.add(obj);
  const clone = Array.isArray(obj) ? [] : {};
  for (const key of Object.keys(obj)) {
    clone[key] = deepClone(obj[key], seen);
  }
  return clone;
}
```

---

## Destructuring and Spread — Advanced Patterns

```javascript
// Swap without temp
[a, b] = [b, a];

// Rest in destructuring
const { id, ...rest } = user;   // rest = everything except id

// Default values with rename
const { name: userName = "Anonymous", age = 0 } = config;

// Nested destructuring
const { address: { city, zip } } = user;

// Function parameter destructuring
function createUser({ name, role = "viewer", ...meta }) {
  return { name, role, ...meta };
}

// Shallow clone + override (immutable update)
const updated = { ...original, name: "newName" };

// Remove a property immutably
const { password, ...safeUser } = user;
```

---

## Module Patterns

```javascript
// ES Modules (standard)
export const API_URL = "https://api.example.com";
export function fetchData() { }
export default class UserService { }

// Named vs default imports
import UserService, { API_URL, fetchData } from "./service.js";

// Dynamic import (code splitting)
const module = await import("./heavy-module.js");
module.doSomething();

// Re-export (barrel pattern)
// index.js
export { UserService } from "./UserService.js";
export { AuthService } from "./AuthService.js";
```

---

## Key Interview Questions

| Question | Key Point |
|----------|-----------|
| `var` vs `let` vs `const`? | `var` is function-scoped, hoisted; `let`/`const` are block-scoped, TDZ |
| What is hoisting? | Declarations are moved to top of scope; `var` = `undefined`, functions = fully hoisted, `let`/`const` = TDZ |
| `==` vs `===`? | `==` coerces types; `===` checks type + value (always prefer `===`) |
| What is the temporal dead zone? | Period between scope entry and `let`/`const` declaration — accessing throws ReferenceError |
| Event delegation? | Attach one listener to a parent; events bubble up from children |
| `null` vs `undefined`? | `undefined` = not assigned; `null` = intentionally empty |
| What is a pure function? | Same inputs → same output, no side effects |
| Shallow vs deep copy? | Spread/`Object.assign` are shallow; `structuredClone()` is deep (no functions) |
