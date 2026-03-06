---
layer: 03_software_engineering
type: engineering
tool: javascript
status: evergreen
tags: []
created: 2026-03-02
---

# JavaScript — Collections

## Purpose

JavaScript provides three primary collection types — arrays, maps, and sets — plus several loop constructs for iterating over them. Knowing which to use and how to iterate efficiently is everyday backend and frontend work.

## Implementation Notes

### Arrays

```js
// Literal syntax — mixed types allowed
const numbers = [1, 2, 3, 4, 5];
const mixed = [true, 7, "text"];

// Indexing
numbers[0];            // 1
numbers[numbers.length - 1]; // 5

// Push to end
const drinks = [];
drinks.push("lemonade");   // ["lemonade"]
```

**`const` arrays are mutable** — `const` only prevents rebinding:

```js
const items = ["a"];
items.push("b");   // OK — ["a", "b"]
items = ["c"];     // TypeError
```

---

### Array Spread

Creates a new array by expanding elements:

```js
const a = [1, 2, 3];
const b = [...a, 4, 5, 6];   // [1, 2, 3, 4, 5, 6]

// Copy
const copy = [...a];

// Merge
const merged = [...a, ...b];
```

---

### Array Destructuring

```js
const [x, y, z] = [1, 2, 3];
console.log(x, y, z); // 1 2 3

// Skip elements
const [first, , third] = [10, 20, 30];

// Rest operator — captures remaining elements
const [head, ...tail] = [1, 2, 3, 4];
// head = 1, tail = [2, 3, 4]

// Over-destructuring gives undefined (no error)
const [a, b, c, d] = [1, 2, 3];
typeof d; // "undefined"
```

---

### `slice`

Returns a new sub-array (non-mutating). Supports negative indices:

```js
const animals = ["ant", "bison", "camel", "duck", "elephant"];

animals.slice(2);       // ["camel", "duck", "elephant"]
animals.slice(2, 4);    // ["camel", "duck"]
animals.slice(-2);      // ["duck", "elephant"]
animals.slice(2, -1);   // ["camel", "duck"]
animals.slice();        // full copy
```

---

### `includes`

```js
["apple", "orange"].includes("orange");  // true
"Hello, world!".includes("world");       // true — works on strings too
```

---

### Iteration Methods

```js
const nums = [1, 2, 3, 4, 5];

// forEach — side effects only, no return value
nums.forEach(n => console.log(n));

// map — transform each element, returns new array
const doubled = nums.map(n => n * 2);     // [2, 4, 6, 8, 10]

// filter — keep elements matching predicate
const evens = nums.filter(n => n % 2 === 0); // [2, 4]

// reduce — fold to single value
const sum = nums.reduce((acc, n) => acc + n, 0); // 15

// find / findIndex
const first = nums.find(n => n > 3);      // 4
const idx   = nums.findIndex(n => n > 3); // 3

// flat / flatMap
[[1, 2], [3, 4]].flat();               // [1, 2, 3, 4]
[[1, 2], [3]].flatMap(a => a.map(x => x * 2)); // [2, 4, 6]
```

---

### Loops

```js
// Classic for — use when index matters
for (let i = 0; i < 5; i++) {
  console.log(i);
}

// for...of — iterate values of any iterable (array, string, Map, Set)
const woods = ["oak", "pine", "maple"];
for (const wood of woods) {
  console.log(wood);
}

// for...in — iterate keys of a plain object (not for arrays)
const user = { name: "Lane", age: 30 };
for (const key in user) {
  console.log(`${key}: ${user[key]}`);
}

// while
let n = 10;
while (n > 0) {
  n -= 3;
}

// break — exit loop immediately
for (let i = 0; i < 10; i++) {
  if (i === 3) break;
}

// continue — skip current iteration
for (let i = 0; i < 10; i++) {
  if (i % 2 === 0) continue;
  console.log(i); // 1 3 5 7 9
}
```

> **Gotcha**: `for...of` iterates *values*; `for...in` iterates *keys*. They're easily confused.

---

### Maps

An ordered key-value store where keys can be any type:

```js
const map = new Map();
map.set("alice", 100);
map.set("bob", 200);
map.get("alice");    // 100
map.has("alice");    // true
map.delete("bob");
map.size;            // 1

// Initialise from array of pairs
const m2 = new Map([["a", 1], ["b", 2]]);

// Iterate
for (const [key, value] of map) {
  console.log(key, value);
}

// Keys must be the *same reference* for object/array keys
const key = ["hello"];
map.set(key, "value");
map.get(key);           // "value"
map.get(["hello"]);     // undefined — different reference
```

---

### Map vs Plain Object

| Feature            | `Map`                      | Plain Object                  |
|--------------------|---------------------------|-------------------------------|
| Key ordering       | Insertion order guaranteed | Less predictable              |
| Key types          | Any value                  | String / Symbol only          |
| Iterable           | Directly via `for...of`    | Needs `Object.keys()` etc.    |
| Performance        | Better for frequent add/del | Fine for mostly-static data   |
| Extra properties   | None                       | Inherits `__proto__` etc.     |

Use a `Map` when key ordering matters, keys are non-strings, or you're doing many insertions/deletions.

---

### Sets

An unordered collection of **unique** values:

```js
const set = new Set([1, 2, 3, 3, 3]);
console.log(set); // Set { 1, 2, 3 }

set.add(4);
set.delete(1);
set.has(2);       // true
set.size;         // 3

// Iterate
for (const val of set) {
  console.log(val);
}

// De-duplicate an array
const unique = [...new Set([1, 1, 2, 3, 2])]; // [1, 2, 3]
```

**Set composition** (native methods, no spread needed):

```js
const a = new Set([1, 2, 3, 4]);
const b = new Set([3, 4, 5, 6]);

a.intersection(b);  // Set { 3, 4 }
a.difference(b);    // Set { 1, 2 }
a.union(b);         // Set { 1, 2, 3, 4, 5, 6 }
```

Alternatively, using spread (wider compatibility):

```js
const union        = new Set([...a, ...b]);
const intersection = new Set([...a].filter(x => b.has(x)));
const difference   = new Set([...a].filter(x => !b.has(x)));
```

## Trade-offs

- **`for...of` vs `forEach`**: `for...of` supports `break`/`continue`; `forEach` does not. Prefer `for...of` when you need early exit.
- **`for...in` on arrays**: Avoid — it iterates inherited prototype keys and can behave unexpectedly. Use `for...of` or `.forEach` instead.
- **`map` / `filter` / `reduce` chains**: Readable but each creates a new array; for tight loops over large data, a single `for` loop is faster.
- **Map vs Object**: Objects are simpler to write (`{}`); Maps are better for dynamic key sets and ordered iteration.
- **Set**: Best for O(1) membership tests and de-duplication. Not a replacement for arrays when order or duplicates matter.

## References

- [MDN: Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)
- [MDN: Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)
- [MDN: Set](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set)
- [MDN: for...of](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...of)
- [MDN: Destructuring assignment](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)

## Links

- [[js_core_language]]
- [[js_objects_and_classes]]
- [[js_async]]
- [[03_software_engineering/01_programming_and_runtime/02_javascript/index]]
