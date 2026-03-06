---
layer: 03_software_engineering
type: engineering
tool: javascript
status: evergreen
tags: []
created: 2026-03-02
---

# JavaScript — Runtimes

## Purpose

A runtime is the environment that executes JavaScript code. It determines which APIs are available, how modules are resolved, and what platform features your code can use. Understanding the runtime landscape is important when choosing a backend stack or debugging environment-specific issues.

## Implementation Notes

### What is a Runtime?

A JavaScript runtime provides:
- **The JS engine** — parses and executes JavaScript (V8, SpiderMonkey, JavaScriptCore)
- **Platform APIs** — what's available beyond the language itself (`fetch`, `fs`, `Deno.readFile`, etc.)
- **Module system** — how files are resolved and loaded
- **Event loop implementation** — the I/O event loop (libuv in Node.js)

The language specification (ECMAScript) defines the language; the runtime defines the environment.

---

### Runtime Comparison

| Feature | Browser | Node.js | Deno | Bun |
|---------|---------|---------|------|-----|
| **Engine** | V8 / SpiderMonkey / JSC | V8 | V8 | JavaScriptCore |
| **Primary use** | Frontend / UI | Backend / scripts | Backend / scripts | Backend / scripts |
| **Module default** | ESM (with `type="module"`) | CommonJS (ESM opt-in) | ESM | ESM |
| **TypeScript** | Via bundler/transpiler | Via ts-node / bundler | Native | Native |
| **Package manager** | — | npm / yarn / pnpm | Built-in (URL imports + npm) | Built-in (`bun`) |
| **Security model** | Sandboxed | Full system access | Explicit permissions | Full system access |
| **Maturity** | Established | Established | Growing | Early-stage |
| **Speed** | N/A | Fast | Fast | Fastest startup |

**For new backend projects**: Node.js is the safe default (largest ecosystem, most documentation). Bun is worth watching for performance-sensitive services. Deno is a good choice if native TypeScript and security permissions matter.

---

### Node.js

```bash
# Install (via nvm — preferred)
nvm install 20
nvm use 20

# Run a script
node main.js

# REPL
node
```

Node.js brought JavaScript to the server in 2009. It uses **libuv** for its event loop (enabling non-blocking I/O across platforms) and **V8** for execution. It excels at I/O-bound workloads and has the largest ecosystem of any backend runtime.

---

### NPM (Node Package Manager)

```bash
npm init -y                      # create package.json
npm install lodash               # add dependency
npm install --save-dev jest      # add dev dependency
npm install                      # install from package.json
npm run start                    # run "start" script
npm run test                     # run "test" script
```

**`package.json`** — the project manifest:

```json
{
  "name": "my-app",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "start": "node src/index.js",
    "test": "jest"
  },
  "dependencies": {
    "express": "^4.18.2"
  },
  "devDependencies": {
    "jest": "^29.0.0"
  }
}
```

**Semantic versioning** in `package.json`:

| Specifier | Meaning |
|-----------|---------|
| `"4.18.2"` | Exact version |
| `"^4.18.2"` | Compatible with 4.x.x (most common) |
| `"~4.18.2"` | Compatible with 4.18.x (patch updates only) |
| `"*"` | Any version (dangerous) |

**`node_modules/`** — where installed packages live. Never commit to git (add to `.gitignore`). **`package-lock.json`** (or `yarn.lock`) records exact resolved versions — commit this to ensure reproducible installs.

---

### Polyfills and Transpilers

**The problem**: JavaScript runs in environments you don't control (users' browsers, older Node.js versions). New language features may not be available everywhere.

**Polyfill** — runtime code that adds missing built-in functionality:

```js
// Example: polyfilling Array.prototype.flat for IE11
if (!Array.prototype.flat) {
  Array.prototype.flat = function(depth = 1) { /* implementation */ };
}
```

**Transpiler** — converts modern JS syntax to older, widely-supported JS at build time:

| Tool | What it does |
|------|-------------|
| **Babel** | Compiles ES2015+ syntax (arrow functions, classes, `async/await`) down to ES5 |
| **TypeScript compiler (`tsc`)** | Strips types and optionally targets an older JS version |
| **esbuild** | Extremely fast JS/TS transpilation (used by Vite, Bun) |

```bash
# Babel example
npx babel src --out-dir dist --presets @babel/preset-env
```

**When to transpile**: frontend projects targeting older browsers still need it. Modern Node.js (18+) supports all current JS features natively — transpilation is usually unnecessary for pure backend code.

## Trade-offs

- **Node.js vs Bun**: Bun is significantly faster at startup and package installation, but its ecosystem maturity and edge-case compatibility with npm packages is still catching up. For production workloads, Node.js is lower risk.
- **npm vs yarn vs pnpm**: npm is the default and fully capable. pnpm uses hard links to save disk space and is faster for monorepos. yarn has workspaces and parallel installs. All three are interchangeable for most projects.
- **Polyfills**: Add bundle size. Use tools like `browserslist` + `@babel/preset-env` to only polyfill what your target browsers actually need.
- **Transpilers**: Add build complexity and can obscure stack traces. Minimise transpilation for server code; use source maps when you need it.

## References

- [Node.js documentation](https://nodejs.org/en/docs/)
- [npm documentation](https://docs.npmjs.com/)
- [Deno documentation](https://deno.land/manual)
- [Bun documentation](https://bun.sh/docs)
- [Babel](https://babeljs.io/docs/)
- [MDN: Polyfill](https://developer.mozilla.org/en-US/docs/Glossary/Polyfill)

## Links

- [[js_modules]]
- [[js_async]]
- [[03_software_engineering/01_programming_and_runtime/02_javascript/index]]
