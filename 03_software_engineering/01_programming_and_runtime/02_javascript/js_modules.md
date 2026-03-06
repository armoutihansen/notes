---
layer: 03_software_engineering
type: engineering
tool: javascript
status: evergreen
tags: []
created: 2026-03-02
---

# JavaScript — Modules

## Purpose

Modules let you split JavaScript code across multiple files and control what is public. Every serious JS project uses a module system; understanding the two systems (CommonJS and ES Modules) and when each applies is essential for Node.js development.

## Implementation Notes

### Why Modules Exist

Early JavaScript was small browser scripts. As applications grew, global scope pollution and dependency management became unmanageable. Modules solve:

- **Encapsulation**: each file has its own scope; nothing is global unless explicitly exported
- **Dependency management**: explicit imports make relationships between files clear
- **Code splitting / lazy loading**: bundlers can deliver only the code each page needs

---

### CommonJS (CJS)

Created for Node.js before ES modules. Still widely used in Node.js projects, especially older ones.

```js
// math.js — exporting
const add = (a, b) => a + b;
const subtract = (a, b) => a - b;

module.exports = { add, subtract };

// Or export one thing as the entire module value
module.exports = function add(a, b) { return a + b; };
```

```js
// main.js — importing
const { add, subtract } = require("./math.js");
const add = require("./math.js"); // if module.exports is a function

console.log(add(1, 2)); // 3
```

**Key characteristics:**
- **Synchronous** — `require()` blocks until the file is loaded (fine for server startup, not for browsers)
- **Dynamic** — `require()` can be called anywhere, including inside conditions
- **Node.js only** — needs a bundler to run in browsers

---

### ES Modules (ESM)

The modern standard, built into the language. Works natively in browsers and Node.js.

```js
// math.js — named exports
export const add = (a, b) => a + b;
export const subtract = (a, b) => a - b;

// or export at the bottom
const multiply = (a, b) => a * b;
export { multiply };

// Default export — one per file
export default function divide(a, b) { return a / b; }
```

```js
// main.js — importing
import { add, subtract } from "./math.js";          // named
import divide from "./math.js";                      // default
import { add as sum } from "./math.js";              // rename
import * as MathUtils from "./math.js";              // namespace

import divide, { add, subtract } from "./math.js";  // mixed
```

**Key characteristics:**
- **Static** — `import` statements are parsed at load time (enables tree-shaking)
- **Asynchronous** — loads don't block execution
- **Always strict mode** — `"use strict"` is implicit
- **Top-level `await`** — available natively

---

### Enabling ESM in Node.js

By default Node.js treats `.js` files as CommonJS. To use ESM:

```json
// package.json
{
  "type": "module"
}
```

Or rename files to `.mjs` (ESM) / `.cjs` (CommonJS) to mix both in one project.

```js
// With "type": "module", these are ES modules
import { readFile } from "fs/promises";

const content = await readFile("./data.txt", "utf8");
```

---

### Browser Modules

```html
<!-- Without type="module": scripts share global scope, run in order -->
<script src="utils.js"></script>
<script src="app.js"></script>

<!-- With type="module": separate scope, deferred, always strict -->
<script type="module" src="app.js"></script>
```

Module scripts are **deferred by default** — they run after the document has parsed, so you rarely need `defer` explicitly.

---

### Node.js Module Resolution

When you write `import { x } from "lodash"`:

1. Node checks `node_modules/lodash/` for `package.json` → `"exports"` or `"main"` field
2. Falls back to `index.js` if no explicit entry point

Relative imports (starting with `./` or `../`) are resolved from the current file. Bare specifiers (`"lodash"`) always resolve from `node_modules`.

---

### Strict Mode

In non-module scripts, strict mode must be opted into:

```js
"use strict";
// now in strict mode
```

**ES modules are always in strict mode** — no declaration needed. This means:
- Undeclared variables throw `ReferenceError` (instead of creating globals)
- `this` in global scope is `undefined`
- Duplicate parameter names are forbidden

---

### Bundlers

Bundlers process your source files and produce optimised output for deployment.

| Bundler | Strengths | Common Use |
|---------|-----------|------------|
| **Webpack** | Mature, rich ecosystem, extensive config | Large apps, legacy projects |
| **Vite** | Fast (ESM dev server, esbuild), simple config | Modern React/Vue/Svelte projects |
| **Rollup** | Excellent tree-shaking, clean output | Libraries |
| **esbuild** | Extremely fast, Go-based | Build tool backbone (used by Vite) |

What bundlers do beyond bundling:
- **Minification**: remove whitespace, shorten variable names
- **Tree shaking**: eliminate unused exports (ESM only — requires static imports)
- **Code splitting**: lazy-load per-route or per-feature chunks
- **Asset processing**: optimise images, inline CSS, generate source maps

---

### Named vs Default Exports

```js
// Named — multiple per file, imported by exact name (or renamed)
export const PI = 3.14;
export function area(r) { return PI * r * r; }

// Default — one per file, imported with any name
export default class Circle { ... }
```

**Recommendation**: prefer named exports. They make auto-imports in editors unambiguous and are more tree-shake-friendly.

## Trade-offs

- **CJS vs ESM**: Use ESM for new projects. CJS is still common in older Node.js codebases and is required when consuming packages that don't ship ESM. Mixing them requires care (`.mjs`/`.cjs` extensions or `"type"` field).
- **Dynamic `require()` vs static `import`**: CJS's dynamic `require` is flexible but opaque to bundlers. ESM's static `import` enables tree-shaking and faster startup via analysis.
- **Default vs named exports**: Default exports are convenient but lead to inconsistent naming across the codebase. Named exports are explicit.
- **Bundlers**: Avoid adding a bundler to a backend Node.js project unless there's a specific reason — Node.js resolves modules natively. Bundlers pay off for frontend delivery.

## References

- [MDN: JavaScript modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)
- [Node.js: ECMAScript modules](https://nodejs.org/api/esm.html)
- [MDN: import](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import)
- [MDN: export](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/export)
- [MDN: Strict mode](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode)

## Links

- [[js_runtimes]]
- [[js_async]]
- [[03_software_engineering/01_programming_and_runtime/02_javascript/index]]
