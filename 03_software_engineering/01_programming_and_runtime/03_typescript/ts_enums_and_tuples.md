---
layer: 03_software_engineering
type: engineering
tool: typescript
status: evergreen
tags: []
created: 2026-03-02
---

# TypeScript Enums and Tuples

## Purpose

**Enums** give named, exhaustive sets of constants — useful when you want symbolic labels backed by a value (numeric or string). **Tuples** are fixed-length, positionally-typed arrays — useful when ordering is semantically significant. Both are TypeScript-specific features that add structure beyond plain JavaScript.

## Implementation Notes

---

### Enums

#### Basic (Numeric) Enum

Values auto-increment from 0 unless overridden:

```typescript
enum Direction {
  North, // 0
  East,  // 1
  South, // 2
  West,  // 3
}

let dir: Direction = Direction.North;
console.log(dir); // 0

// Numeric enums support bidirectional mapping:
const name: string = Direction[2]; // "South"
```

You can explicitly set values — TypeScript will flag duplicates:

```typescript
enum StatusCode {
  OK          = 200,
  Created     = 201,
  BadRequest  = 400,
  NotFound    = 404,
}
```

#### String Enums

Prefer string enums when values will be serialised (JSON, databases, logs) — a raw `7` in an API response is far harder to debug than `"WARN"`:

```typescript
enum LogLevel {
  ERROR = "ERROR",
  WARN  = "WARN",
  INFO  = "INFO",
  DEBUG = "DEBUG",
}

function log(message: string, level: LogLevel) {
  console.log(`[${level}] ${message}`);
}

log("User not found", LogLevel.ERROR);
// [ERROR] User not found
```

String enums do **not** support reverse mapping — compiled output maps name → value only.

#### Const Enums — Inlined at Compile Time

`const enum` is erased entirely during compilation; every use site is replaced by the literal value. No runtime object is emitted:

```typescript
const enum Direction {
  North = "NORTH",
  East  = "EAST",
  South = "SOUTH",
  West  = "WEST",
}

const heading = Direction.North;
// Compiles to:  const heading = "NORTH";
```

Limitations:
- No computed (function-call) initialisers.
- Reverse mapping is impossible (no runtime object exists).
- Can cause issues with `isolatedModules` / Babel pipelines.

Use `const enum` only when bundle size / performance is a specific concern.

#### Enum Compilation Output

A regular numeric enum compiles to a self-executing IIFE that creates a bidirectional object:

```javascript
// TypeScript:  enum Class { Rogue, Mage, Warrior }
var Class;
(function (Class) {
  Class[Class["Rogue"]   = 0] = "Rogue";
  Class[Class["Mage"]    = 1] = "Mage";
  Class[Class["Warrior"] = 2] = "Warrior";
})(Class || (Class = {}));
```

This is the only TypeScript feature that adds non-trivial runtime code. String enums produce the same IIFE but without reverse mapping entries.

#### Enums vs Union Types — Prefer Unions for Simple Cases

```typescript
// Enum — more verbose, ships runtime code
enum CardSuit {
  Hearts   = "Hearts",
  Diamonds = "Diamonds",
  Clubs    = "Clubs",
  Spades   = "Spades",
}

// Union — concise, zero runtime overhead
type CardSuit = "Hearts" | "Diamonds" | "Clubs" | "Spades";
```

Unions are consistent with the rest of the type system, add no runtime cost, and are less verbose. Anders Hejlsberg (TypeScript's creator) has stated enums might not be included if TypeScript were designed today.

**Prefer enums when:** you need reverse mapping from value to name, or you want a namespace (`MyEnum.Value`) for organisational clarity in a large codebase.

---

### Tuples

#### Basic Tuples

A typed fixed-length array. **Always annotate explicitly** — without the type annotation TypeScript infers `(string | number)[]`:

```typescript
const nameAndAge: [string, number] = ["Rose Tyler", 24];

// Without annotation — NOT a tuple:
const inferred = ["Martha Jones", 24]; // (string | number)[]
```

#### Named Tuples

Labels are documentation only — positions still govern destructuring:

```typescript
type UserData = [name: string, age: number, isAdmin: boolean];

function getUser(): UserData {
  return ["Frodo", 33, false];
}

const [name, age, isAdmin] = getUser();
```

#### Optional Tuple Elements

Optional elements must come after all required elements, and are automatically unioned with `undefined`:

```typescript
type HttpResponse = [statusCode: number, data: string, error?: string];

const ok:  HttpResponse = [200, "Success!"];
const err: HttpResponse = [404, "", "Not found"];

const [status, data, maybeError] = ok;
// maybeError: string | undefined
```

#### Readonly Tuples

Tuples are arrays under the hood, so `push` / `pop` are allowed unless you add `readonly`. Always prefer `readonly` tuples:

```typescript
const pair: readonly [string, number] = ["Alice", 42];
pair.push("oops"); // Error: Property 'push' does not exist on type 'readonly [string, number]'
```

`readonly` is enforced at compile time only, like `const`.

#### Tuple Rest Elements

Model a fixed prefix with a variable-length tail:

```typescript
type NameAndScores = [string, ...number[]];

const scores: NameAndScores = ["Edward", 69, 420, 300]; // valid
const minimal: NameAndScores = ["Winry"];                // also valid

// Command pattern — required command, optional args:
type Command = [name: string, ...args: string[]];
const commit: Command = ["git", "commit", "-m", "feat: add types"];
```

#### Destructuring Tuples

Standard array destructuring — positions determine the type:

```typescript
const [lat, lng]: [number, number] = [40.7128, -74.006];
```

#### Tuples vs Objects

Use a **tuple** when ordering is the semantics (a path's segment distances, `[lat, lng]` coordinates). Use an **object** when named access is more natural:

```typescript
// Tuple — ordering matters, no field names needed
type Point = [number, number];

// Object — names carry meaning, ordering is irrelevant
type NamedPoint = { lat: number; lng: number };
```

---

### Arrays in TypeScript

`T[]` and `Array<T>` are interchangeable — prefer `T[]` for brevity:

```typescript
const colors: string[] = ["red", "green", "blue"];
const counts: Array<number> = [1, 2, 3]; // same thing
```

**`readonly T[]`** prevents mutation:

```typescript
function sum(nums: readonly number[]): number {
  return nums.reduce((a, b) => a + b, 0);
}
```

**Heterogeneous arrays** use union types:

```typescript
let mixed: (string | number)[] = [1, "two", 3];
```

**Rest parameters** collect trailing arguments into a typed array:

```typescript
function party(name: string, ...members: string[]): string {
  return `${name}: ${members.join(", ")}`;
}
```

**Evolving `any` pitfall** — empty arrays infer `any[]`, then grow their type as you push items. Explicit annotation prevents this:

```typescript
let inventory: string[] = []; // good — constrained from the start
let bad = [];                  // any[] — will evolve
```

## Trade-offs

| Feature | Guidance |
|---|---|
| Numeric enum | Fine for internal-only use; avoid if values will be serialised |
| String enum | Good for serialised values; prefer union if you control both sides |
| `const enum` | Max performance / bundle size; limited interop |
| Union types | Preferred over enums for most simple cases |
| Tuples | Use `readonly` always; prefer objects for named access |
| `T[]` vs `Array<T>` | Either is fine; `T[]` is idiomatic |

## References

- [TypeScript Handbook — Enums](https://www.typescriptlang.org/docs/handbook/enums.html)
- [TypeScript Handbook — Tuple Types](https://www.typescriptlang.org/docs/handbook/2/objects.html#tuple-types)
- [TypeScript Handbook — Arrays](https://www.typescriptlang.org/docs/handbook/basic-types.html#array)
- [TypeScript Handbook — Rest Parameters](https://www.typescriptlang.org/docs/handbook/2/functions.html#rest-parameters)

## Links

- [[ts_unions_and_intersections]]
- [[ts_type_system]]
- [[ts_generics]]
