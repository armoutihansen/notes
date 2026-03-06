---
layer: 03_software_engineering
type: engineering
tool: javascript
status: evergreen
tags: []
created: 2026-03-02
---

# JavaScript — Core Language

## Purpose

Cover the fundamental building blocks of JavaScript: its type system, variable declarations, and the comparison/control-flow mechanics that make JS distinctive (and occasionally surprising).

## Implementation Notes

### Type System

JavaScript is **dynamically typed** (types are resolved at runtime, not compile time) and **weakly typed** (it silently coerces types when mixing them):

```js
let n = 42;
let s = "42";
console.log(n + s); // "4242"  — number coerced to string
```

**Primitive types** (`typeof` values in parentheses):

| Type        | Example               | Notes                                    |
|-------------|----------------------|------------------------------------------|
| `boolean`   | `true`, `false`      |                                          |
| `string`    | `"hello"`            | UTF-16 encoded; emojis take 2 code units |
| `number`    | `42`, `3.14`, `-5`   | Single IEEE-754 float type for all nums  |
| `null`      | `null`               | Explicit "empty"; `typeof` returns `"object"` (historical quirk) |
| `undefined` | `undefined`          | Default value of uninitialised variables |
| `symbol`    | `Symbol("id")`       | Unique identifier primitives             |
| `bigint`    | `9007199254740991n`  | Arbitrary-precision integers             |

Objects and arrays are reference types (not primitives).

---

### Variable Declarations

Prefer `const` by default; use `let` when reassignment is needed. **Never use `var`.**

```js
const PI = 3.14;          // block-scoped, can't be reassigned
let count = 0;            // block-scoped, can be reassigned
count = 1;                // OK
PI = 3;                   // TypeError: Assignment to constant variable

// var is function-scoped — leaks out of if/for blocks
if (true) {
  var leak = "oops";
}
console.log(leak); // "oops" — unexpected
```

`const` on objects/arrays only prevents reassignment of the binding — the contents remain mutable:

```js
const arr = [1, 2];
arr.push(3);   // OK — [1, 2, 3]
arr = [];      // TypeError
```

Multiple declarations on one line are valid but rarely clearer:

```js
let x = 1, y = 2;
```

---

### `null` vs `undefined` vs Undeclared

```js
let a;              // undefined — declared but not assigned
let b = null;       // null — explicitly "empty"
console.log(c);     // ReferenceError — undeclared (confusingly says "not defined")

typeof undefined;   // "undefined"
typeof null;        // "object"  ← historical bug, can't be fixed
```

Prefer `undefined` as your "no value" sentinel in most code; use `null` only when the distinction matters or an external API requires it.

---

### Truthy & Falsy

In JavaScript any value can appear in a boolean context. **Falsy** values (everything else is truthy):

```
false   0   ""   null   undefined   NaN
```

Notable **truthy** values that trip people up:

```
[]   {}   "0"   "false"
```

```js
if ([]) console.log("truthy");   // prints — empty array is truthy
if (0)  console.log("truthy");   // doesn't print — 0 is falsy
```

---

### Comparison Operators

Always use **strict equality** (`===` / `!==`), which checks both value *and* type:

```js
5 === "5";   // false — different types
5 == "5";    // true  — loose equality coerces "5" to 5 (avoid this)

null === undefined;   // false
null ==  undefined;   // true  — another coercion surprise
```

---

### Nullish Coalescing `??`

Returns the right-hand value only when the left is `null` or `undefined` (unlike `||`, which also triggers on `0` and `""`):

```js
const name = userInput ?? "Anonymous";   // "Anonymous" if null/undefined
const port = config.port ?? 3000;        // keeps 0 if explicitly set
const port2 = config.port || 3000;       // replaces 0 with 3000 — often wrong
```

---

### Template Literals

```js
const shade = 101;
console.log(`The shade is ${shade}`);   // "The shade is 101"

// Multi-line
const msg = `Line one
Line two`;
```

---

### Numbers

All numeric values share a single `number` type (IEEE-754 double-precision float):

```js
let x = 2;      // integer — really a float
x = 5.69;       // fractional
x = -5.42;      // negative

// Arithmetic
2 + 3;   // 5
6 / 4;   // 1.5 — no integer division
6 % 4;   // 2   — remainder

// Increment / decrement
let i = 0;
i++;    // 1
i--;    // 0
i += 5; // 5
```

---

### Strings

```js
const s = "Hello";
s[0];             // 'H'
s.length;         // 5
s.includes("ell"); // true

// UTF-16: emojis occupy 2 code units
const e = "🐸";
e.length;   // 2
```

---

### Control Flow

```js
// if / else if / else
if (score > 90) {
  grade = "A";
} else if (score > 70) {
  grade = "B";
} else {
  grade = "C";
}

// ternary — good for simple one-liners, avoid nesting
const label = isMember ? "$2" : "$10";

// switch — always add break to prevent fall-through
switch (os) {
  case "mac":   creator = "Apple"; break;
  case "linux": creator = "Torvalds"; break;
  default:      creator = "Unknown"; break;
}

// logical operators
true && false;   // false
true || false;   // true
!true;           // false
```

## Trade-offs

- **`==` vs `===`**: Use `===` everywhere. Loose equality's coercion rules are complex and error-prone.
- **`null` vs `undefined`**: Pick one convention and stick to it; mixing them forces consumers to check both. Prefer `undefined` unless forced otherwise.
- **`??` vs `||`**: `??` is safer when `0` or `""` are valid values.
- **`var`**: Avoid entirely — function scope instead of block scope causes subtle bugs with loops and closures.
- **Floating point**: `0.1 + 0.2 !== 0.3` — use `Math.round` or integer arithmetic when precision matters.

## References

- [MDN: JavaScript data types and data structures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures)
- [MDN: let](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let)
- [MDN: const](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const)
- [MDN: Equality comparisons](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Equality_comparisons_and_sameness)
- [MDN: Nullish coalescing](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing)
- [MDN: Template literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals)

## Links

- [[js_functions]]
- [[js_objects_and_classes]]
- [[03_software_engineering/01_programming_and_runtime/02_javascript/index]]
