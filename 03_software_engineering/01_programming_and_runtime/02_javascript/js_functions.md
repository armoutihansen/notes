---
layer: 03_software_engineering
type: engineering
tool: javascript
status: evergreen
tags: []
created: 2026-03-02
---

# JavaScript — Functions

## Purpose

Functions are first-class values in JavaScript. Understanding declarations vs expressions, arrow functions, `this` binding, closures, and scope is essential to writing predictable JS — especially for callbacks and event-driven code.

## Implementation Notes

### Declarations vs Expressions

```js
// Declaration — hoisted, callable before definition in file
function add(x, y) {
  return x + y;
}

// Expression assigned to variable — NOT hoisted
const add = function(x, y) {
  return x + y;
};

// Arrow function expression — most concise, differences in `this`
const add = (x, y) => x + y;          // implicit return when body is single expression
const add = (x, y) => { return x + y; }; // explicit return with braces
```

Only function **declarations** are hoisted — expressions and arrow functions assigned to `const`/`let` are not:

```js
console.log(greet("World")); // OK — hoisted
function greet(name) { return `Hello ${name}`; }

console.log(hi("World")); // ReferenceError — not hoisted
const hi = (name) => `Hi ${name}`;
```

---

### Arrow Functions and `this`

Arrow functions **do not have their own `this`**. They inherit `this` from the enclosing lexical scope. Regular functions bind `this` to the call-site object.

```js
const user = {
  name: "Lane",

  // Regular method — `this` bound to the object at call time
  greetRegular() {
    return `Hi, I'm ${this.name}`;
  },

  // Arrow function — `this` refers to outer scope (not the object)
  greetArrow: () => {
    return `Hi, I'm ${this.name}`; // this.name is undefined
  },
};

user.greetRegular(); // "Hi, I'm Lane"
user.greetArrow();   // "Hi, I'm undefined"
```

Arrow functions also lack `arguments` and can't be used as constructors.

---

### `this` in Different Contexts

```js
// Global context
console.log(this); // window (browser) or {} (Node.js)

// Strict mode
"use strict";
console.log(this); // undefined in global scope

// Method context — this = the object
const obj = {
  value: 42,
  get() { return this.value; },
};
obj.get(); // 42

// Detached method — this is lost
const fn = obj.get;
fn(); // undefined (or TypeError in strict mode)

// Fix with .bind()
const bound = obj.get.bind(obj);
bound(); // 42
```

**Methods are not bound by default**. Passing a method as a callback breaks `this`:

```js
const user = {
  firstName: "Lane",
  getFullName() { return this.firstName; },
};

function call(cb) { return cb(); }

call(user.getFullName);           // TypeError — this is undefined
call(user.getFullName.bind(user)); // "Lane"
```

---

### Closures and Lexical Scope

A closure is a function that captures variables from its enclosing scope. Variables remain accessible even after the outer function returns.

```js
function makeCounter(start = 0) {
  let count = start; // captured by the returned function
  return () => ++count;
}

const counter = makeCounter(10);
counter(); // 11
counter(); // 12
```

Scope levels (highest → lowest priority):

1. **Global** — `window` (browser) / `global` (Node.js)
2. **Module** — top-level of an ES module file
3. **Function** — inside a `function` body
4. **Block** — inside `{}` when using `let`/`const`

---

### Default Parameters

```js
function greet(email, name = "there") {
  return `Hello ${name}, your email is ${email}`;
}

greet("a@b.com", "Lane"); // "Hello Lane..."
greet("a@b.com");          // "Hello there..."
```

Defaults must come after required parameters.

---

### Functions as First-Class Values

Functions can be assigned, passed, and returned like any value:

```js
function add(x, y) { return x + y; }
function mul(x, y) { return x * y; }

// Higher-order function — takes a function as argument
function aggregate(a, b, c, op) {
  return op(op(a, b), c);
}

aggregate(2, 3, 4, add); // 9
aggregate(2, 3, 4, mul); // 24
```

**Anonymous functions** (inline, no name) are useful for single-use callbacks:

```js
[1, 2, 3].map(function(n) { return n * 2; }); // [2, 4, 6]
[1, 2, 3].map(n => n * 2);                     // same, arrow syntax
```

---

### IIFE (Immediately Invoked Function Expression)

Creates its own scope on the spot — useful for initialisation that shouldn't pollute the outer scope:

```js
const result = (function(a, b) {
  return a + b;
})(1, 2);
// result = 3
```

---

### Passing by Value

Primitives are passed **by value** (the function gets a copy):

```js
let x = 5;
function increment(n) { n++; }
increment(x);
console.log(x); // still 5
```

Objects and arrays are passed **by reference** (mutations inside the function affect the original).

---

### Error Handling

```js
// Throw an Error object (not a plain string)
throw new Error("something went wrong");

// try / catch / finally
try {
  const result = riskyOperation();
} catch (err) {
  console.error(err.message); // err.message is the human-readable string
} finally {
  cleanup(); // always runs, even if catch throws
}
```

`finally` is essential for releasing resources (file handles, DB connections) regardless of success or failure.

## Trade-offs

- **Declarations vs arrow functions for methods**: Use regular methods (shorthand syntax) inside object literals and classes — they bind `this` correctly. Use arrow functions when you need to preserve `this` from an outer scope (e.g., inside `setTimeout` callbacks on a class instance).
- **`.bind()` vs arrow function**: Both solve the detached-method problem. Arrow functions at definition time are cleaner; `.bind()` is useful when you receive a method from outside.
- **Closures**: Powerful but can cause memory leaks if large objects are held in closure scope unnecessarily.
- **IIFE**: Mostly superseded by ES modules (which have their own scope), but still useful in scripts or for async top-level patterns.
- **Error types**: Always throw `Error` objects, not plain strings — error objects carry a stack trace; strings don't.

## References

- [MDN: Functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Functions)
- [MDN: Arrow functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)
- [MDN: this](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this)
- [MDN: Closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures)
- [MDN: Function.prototype.bind](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)
- [MDN: try...catch](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/try...catch)

## Links

- [[js_core_language]]
- [[js_objects_and_classes]]
- [[js_async]]
- [[03_software_engineering/01_programming_and_runtime/02_javascript/index]]
