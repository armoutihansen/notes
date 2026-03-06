---
layer: 03_software_engineering
type: engineering
tool: typescript
status: evergreen
tags: []
created: 2026-03-02
---

# TypeScript Type System

## Purpose

TypeScript adds **static type checking on top of JavaScript**. Its primary goal is to catch type errors at compile time rather than at runtime. All JavaScript is valid TypeScript — the type system is purely additive. When you compile plain JS with `tsc`, everything is implicitly `any`. The workflow for migrating a JS codebase: rename `.js` → `.ts`, get `tsc` running (usually works via `any`), then progressively replace `any` with specific types.

## Implementation Notes

### Basic Types

```ts
const message: string = "hello";
const port: number = 3000;
const active: boolean = true;
const noValue: null = null;
const notDefined: undefined = undefined;
```

### Type Inference

TypeScript infers types from values — explicit annotations on simple assignments are redundant and add noise:

```ts
// Prefer this (inferred as string)
const bootupLog = "Starting servers...";

// Over this (explicit but unnecessary)
const bootupLog: string = "Starting servers...";
```

Inference works for variables, return types, and destructured values.

### Explicit Annotations — When They Matter

Function parameters are **not** inferred from call sites, so annotate them explicitly:

```ts
function createMessage(name: string, a: number, b: number): string {
  return `${name} scored ${a + b}`;
}

// Arrow function syntax
const createMessage = (name: string, a: number, b: number): string => {
  return `${name} scored ${a + b}`;
};
```

Return types can usually be inferred:

```ts
function divide(a: number, b: number) {
  return a / b; // inferred as number
}
```

### Type Aliases

Use the `type` keyword to name any type for reuse:

```ts
type UserId = string | number;
type LoggerCallback = (s1: string, s2: string) => string;

function setLoggerTimeout(loggerCallback: LoggerCallback, delay: number) {
  // ...
}
```

### Function Type Syntax

A function's type captures its parameter types and return type:

```ts
// Type: (a: number, b: number) => number
const add = (a: number, b: number) => a + b;
const subtract = (a: number, b: number) => a - b;
```

### Special Types

#### `any` — escape hatch, avoid in production code

```ts
let x: any = "hello";
x = 42; // no error — type checking is disabled
```

Useful when migrating JS to TS; replace progressively with specific types.

#### `unknown` — type-safe alternative to `any`

`unknown` accepts any value but **requires narrowing before use**:

```ts
function process(val: unknown) {
  if (typeof val === "string") {
    console.log(val.toUpperCase()); // safe: narrowed to string
  }
}
```

#### `never` — unreachable code and exhaustive checks

`never` represents values that can never occur. Use it to enforce exhaustive handling:

```ts
function handleStatusCode(code: 200 | 404 | 500) {
  if (code === 200) { console.log("OK"); return; }
  if (code === 404) { console.log("Not Found"); return; }
  if (code === 500) { console.log("Internal Server Error"); return; }
  // TypeScript narrows code to never here — all cases handled
  const exhaustive: never = code;
  return exhaustive;
}
```

If you forget a case, the assignment to `never` produces a compile error.

#### `void` — function returns nothing

```ts
function logMessage(message: string): void {
  console.log(message);
  // no return statement — intent is explicit
}
```

`void` is more communicative than `undefined`: it signals the caller should not use the return value.

### Importing Types

Use `import type` to keep type imports explicit and tree-shakeable:

```ts
import type { User, Post } from "./models";
// or inline:
import { type User, type Post } from "./models";
```

## Trade-offs

| Approach | When to use |
|---|---|
| Type inference | Default — for variables, return types |
| Explicit annotation | Function params, public API boundaries, complex cases |
| `any` | JS migration only; replace ASAP |
| `unknown` | When input type is truly unknown (API responses, user input) |
| `void` return | Functions whose return value is intentionally unused |

- **Never leave `any` in production code** — it silently disables the type checker for downstream usages.
- **Prefer inferred return types** unless the function is part of a public API where the return type serves as documentation.
- The `never` exhaustive-check pattern (`const _: never = value`) is the idiomatic way to enforce all union cases are handled.

## References

- [TypeScript Handbook — Basic Types](https://www.typescriptlang.org/docs/handbook/basic-types.html)
- [TypeScript Handbook — Type Inference](https://www.typescriptlang.org/docs/handbook/type-inference.html)
- [TypeScript Handbook — More on Functions](https://www.typescriptlang.org/docs/handbook/2/functions.html)
- [TypeScript Handbook — Narrowing](https://www.typescriptlang.org/docs/handbook/2/narrowing.html)

## Links

- [[ts_unions_and_intersections]]
- [[ts_objects_and_interfaces]]
- [[ts_generics]]
