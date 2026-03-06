---
layer: 03_software_engineering
type: engineering
tool: typescript
status: evergreen
tags: []
created: 2026-03-02
---

# TypeScript Objects and Interfaces

## Purpose

Object types are the cornerstone of TypeScript — they describe the **shape** of data structures. TypeScript provides two syntaxes for defining object types (`type` and `interface`), plus a suite of modifiers and utilities that let you express optional fields, immutability, dynamic keys, and pattern constraints.

## Implementation Notes

### Object Literal Types

Inline type annotation on a function parameter:

```ts
function logSaiyan(saiyan: { name: string; power: number }) {
  console.log(`${saiyan.name}: ${saiyan.power}`);
}
```

More commonly, extract to a named alias:

```ts
type Saiyan = {
  name: string;
  power: number;
};

function logSaiyan(saiyan: Saiyan) { ... }
```

### Optional Properties

Mark a property optional with `?`. Its type is automatically `T | undefined`:

```ts
type Superhero = {
  name: string;
  strength: number;
  cape?: boolean; // boolean | undefined
};
```

Don't overuse — only mark fields that genuinely may be absent.

### `readonly` Modifier

Prevents reassignment after initialisation (compile-time only):

```ts
type Point = {
  readonly x: number;
  y: number;
};

const p: Point = { x: 10, y: 20 };
p.y = 30; // OK
p.x = 15; // Error: Cannot assign to 'x' because it is a read-only property
```

### Interfaces

Interfaces describe object shapes using a dedicated `interface` keyword:

```ts
interface Superhero {
  name: string;
  powers: string[];
  isAvenger: boolean;
}
```

In most daily usage, `type` and `interface` are interchangeable for object types.

### `type` vs `interface` — When to Use Which

| Feature | `type` | `interface` |
|---|---|---|
| Object shape | ✅ | ✅ |
| Union / intersection | ✅ | ❌ |
| Extending (inheritance) | via `&` | via `extends` (preferred) |
| Declaration merging | ❌ (error) | ✅ (automatic) |
| Generic constraints | ✅ | ✅ |

**Default recommendation**: use `type`. Use `interface` when:
1. You need declaration merging (e.g., extending the global `Window` type)
2. You're writing a library and want consumers to be able to augment your types

### Declaration Merging (Interface-Only)

Declaring the same interface name multiple times merges all declarations:

```ts
interface Spaceship { name: string }
interface Spaceship { engines: number }
// Equivalent to: { name: string; engines: number }
```

This is usually a footgun — `type` raises a "Duplicate identifier" error instead, which is safer.

### Extending Interfaces

Single extension:

```ts
interface Character { name: string; level: number }

interface Wizard extends Character {
  spellbook: string[];
  mana: number;
}
// Wizard has: name, level, spellbook, mana
```

Multiple extension:

```ts
interface BattleMage extends Character, Magical, Physical {
  combineAttacks(): void;
}
```

**Why `extends` beats `&` for inheritance**: interfaces detect property conflicts and cache type relationships, giving faster compilation and clearer error messages than intersection types.

### Overriding Interface Properties

A subtype can narrow (but not widen) an inherited property:

```ts
interface Character { rank: string | number }

interface Wizard extends Character {
  rank: number; // OK — number is assignable to string | number
}

// Error — number is NOT assignable to string:
// interface Wizard extends Character { rank: number } // if base is rank: string
```

### Discriminated Unions (Tagged Unions)

Give each variant a literal `kind` field so TypeScript can narrow inside `switch`/`if`:

```ts
type MultipleChoiceLesson = {
  kind: "multiple-choice";
  question: string;
  studentAnswer: string;
  correctAnswer: string;
};

type CodingLesson = {
  kind: "coding";
  studentCode: string;
  solutionCode: string;
};

type Lesson = MultipleChoiceLesson | CodingLesson;

function isCorrect(lesson: Lesson): boolean {
  switch (lesson.kind) {
    case "multiple-choice":
      return lesson.studentAnswer === lesson.correctAnswer;
    case "coding":
      return lesson.studentCode === lesson.solutionCode;
  }
}
```

Adding a new variant to `Lesson` causes a compile error if `isCorrect` doesn't handle it.

### Dynamic Keys (Index Signatures)

When keys aren't known ahead of time:

```ts
type UserMetrics = {
  [key: string]: number;
};

const metrics: UserMetrics = { wordsPerMinute: 50, errors: 2 };
metrics["refreshRate"] = 60; // OK
metrics["theme"] = "dark";   // Error: string not assignable to number
```

### `Record<K, V>`

Shorthand for an object with keys of type `K` and values of type `V`:

```ts
const scores: Record<string, number> = { alice: 95, bob: 87 };
const codeName: Record<"007" | "006", string> = { "007": "Bond", "006": "Trevelyan" };
```

### `PropertyKey`

Built-in type for all valid JS property key types:

```ts
type PropertyKey = string | number | symbol;

type Tags = { [key: PropertyKey]: any };

const server: Tags = {
  name: "prod-1",
  1: 420,
  [Symbol("role")]: "Admin",
};
```

### Required vs Optional Dynamic Keys

Mix an index signature with required fields:

```ts
type FormData = {
  [field: string]: string | number | boolean; // any extra field
  email: string;    // required
  password: string; // required
  age: number;      // required
};
```

### Excess Property Checking

TypeScript applies stricter checking when you pass an **object literal** directly:

```ts
type Spaceship = { name: string; speed: number };

const falcon = { name: "Falcon", speed: 75, weapons: 4 };
pilot(falcon); // OK — variable assignment bypasses excess-property check

// Error: 'weapons' does not exist in type 'Spaceship'
pilot({ name: "Falcon", speed: 75, weapons: 4 });
```

### `satisfies` Operator

Validates a value matches a type without losing its inferred literal types:

```ts
type ColorMap = { red: string; green: string; blue: string; yellow: string };

const colors = {
  red: "#FF0000",
  green: "#00FF00",
  blue: "#0000FF",
  yellow: "#FFFF00",
  // yelow: "#FFFF00", // Error: not in ColorMap
} satisfies ColorMap;

type RedHex = typeof colors.red; // "#FF0000" (literal), not string
```

Without `satisfies`, an explicit `ColorMap` annotation would widen `red` to `string`.

### `as const` and `Object.freeze()`

`as const` makes the entire value deeply `readonly` with literal types:

```ts
const config = {
  apiUrl: "https://api.example.com",
  admins: { johnny: "lawrence" },
  features: ["no mercy"],
} as const;

config.apiUrl = "...";           // Error: read-only property
config.admins.johnny = "kreese"; // Error: read-only property
config.features.push("...");     // Error: readonly array
```

`Object.freeze()` is a **runtime** JavaScript operation that prevents top-level mutations. TypeScript recognises it and marks top-level properties readonly, but **nested properties are not frozen**:

```ts
const frozen = Object.freeze({ apiUrl: "https://api.example.com", admins: { johnny: "lawrence" } });
frozen.apiUrl = "...";           // Compile error (top-level)
frozen.admins.johnny = "kreese"; // Allowed — nested not frozen
```

Use `as const` for compile-time deep immutability; `Object.freeze()` for runtime enforcement of top-level only.

### Maps and Sets

```ts
const scores = new Map<string, number>();
scores.set("Alice", 95);
scores.get("Alice"); // 95
scores.has("Bob");   // false
scores.delete("Alice");

for (const [name, score] of scores) { ... }

const unique = new Set<string>(["a", "b", "a"]); // Set { "a", "b" }
unique.add("c");
unique.has("a"); // true
unique.size;     // 3
```

### Optional Chaining

Works naturally with object types — TypeScript narrows the type after each `?.`:

```ts
type User = { profile?: { address?: { city: string } } };
const city = user?.profile?.address?.city; // string | undefined
```

## Trade-offs

- **`type` vs `interface`**: default to `type`; use `interface` for extension-heavy APIs and declaration merging scenarios.
- **`readonly` vs `as const`**: `readonly` on a property is shallow; `as const` is deep. For config objects, `as const` is more thorough.
- **`satisfies` vs explicit annotation**: `satisfies` preserves literal type information — use it when you need both validation and inference precision.
- **Index signatures** vs `Record<K,V>`: `Record` is cleaner for full dynamic-key objects; index signatures are better when you need to mix required fields with dynamic ones.
- **Discriminated unions** scale well: adding a new variant causes compile-time errors at all unhandled `switch` statements. Use `kind` as the conventional discriminant field.
- **Excess property checking** only applies to object literals passed directly — not to variables. This can be surprising and is worth documenting in team style guides.

## References

- [TypeScript Handbook — Object Types](https://www.typescriptlang.org/docs/handbook/2/objects.html)
- [TypeScript Handbook — Interfaces](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#interfaces)
- [TypeScript Handbook — `satisfies` operator](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-9.html#the-satisfies-operator)
- [TypeScript Handbook — `as const`](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#literal-inference)
- [Microsoft TS Wiki — Prefer interfaces over intersections](https://github.com/microsoft/TypeScript/wiki/Performance#preferring-interfaces-over-intersections)

## Links

- [[ts_type_system]]
- [[ts_unions_and_intersections]]
- [[ts_classes]]
