---
layer: 03_software_engineering
type: engineering
tool: typescript
status: evergreen
tags: []
created: 2026-03-02
---

# TypeScript Unions and Intersections

## Purpose

Unions (`A | B`) and intersections (`A & B`) are the primary tools for composing types in TypeScript. They allow you to model "this OR that" and "this AND that" respectively, and together with literal types they enable precise value-level constraints without the overhead of enums.

## Implementation Notes

### Union Types

A union allows a value to be one of several types. Use the `|` operator:

```ts
let userId: string | number;
userId = "user_42"; // OK
userId = 42;        // OK
```

TypeScript **narrows** the type inside conditionals:

```ts
function safeSquare(val: string | number): number {
  if (typeof val === "string") {
    val = parseInt(val, 10); // val is string here
  }
  return val * val; // val is number here
}
```

### Literal Types

A specific value can be used as a type, narrowing a `string` or `number` to a single allowed value:

```ts
function move(direction: "north" | "south" | "east" | "west") { ... }

// Extract as a reusable alias
type Direction = "north" | "south" | "east" | "west";
function move(direction: Direction) { ... }
```

### Optional Parameters

Mark parameters optional with `?`. Optional params have `undefined` automatically unioned in:

```ts
function greet(name: string, title?: string): string {
  // title is `string | undefined` inside
  if (title) return `Hello, ${title} ${name}!`;
  return `Hello, ${name}!`;
}
greet("Gandalf");           // OK
greet("Gandalf", "Wizard"); // OK
```

Rules: optional params must come **after** all required params.

### Default Parameters

Default values make a parameter implicitly optional — no `?` needed. Type is inferred from the default:

```ts
function newCharacter(name: string, role = "warrior"): string {
  return `${name} is a ${role}`;
}
// role is inferred as string
```

### Template Literal Types

Combine string unions using template literal syntax to generate all combinations:

```ts
type Class = "wizard" | "warrior" | "rogue";
type Race  = "elf" | "human" | "dwarf";
type Hero  = `${Race} ${Class}`;
// "elf wizard" | "elf warrior" | ... (9 members)
```

Pattern enforcement:

```ts
type LogRecord = `${string}: ${number}`;
const entry: LogRecord = "CRITICAL: 42"; // OK
const bad: LogRecord   = "CRITICAL 42";  // Error
```

### Super-Set Unions (autocomplete hints)

Adding literal members to a wider type preserves the set of allowed values but surfaces common values in editor autocomplete:

```ts
type ErrorCodes = 200 | 404 | 500 | number;
// Any number is valid, but 200/404/500 appear as suggestions
```

### Giant Unions — when to avoid

Template literal types can explode combinatorially:

```ts
type Distance = 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9;
type Class = "Warrior" | "Rogue" | "Mage" | /* ... */ "Shaman";
// type MoveMessage = `The ${Class} moves ${Distance}...` → thousands of members → compiler error
```

TypeScript will error with "Union type too complex to represent." When the combination space is large, fall back to a plain `string`.

### Intersection Types

An intersection requires a value to satisfy **all** component types simultaneously. Use `&`:

```ts
type IndividualContributor = { id: number; name: string; tasks: string[] };
type Manager = { directReports: number[] };

type GoodManager = IndividualContributor & Manager;
// Must have: id, name, tasks, directReports

const hunter: GoodManager = {
  id: 1,
  name: "Hunter",
  tasks: ["review PRs"],
  directReports: [2, 3],
};
```

Object properties merge; shared properties must be compatible:

```ts
type Point2D = { x: number; y: number };
type Point3D = Point2D & { z: number };
// Equivalent to { x: number; y: number; z: number }
```

### Intersecting Incompatible Types → `never`

When the same property is required to satisfy two mutually exclusive types, the property (and often the whole type) becomes `never`:

```ts
type Saiyan = { name: "goku" | "vegeta"; powerLevel: number };
type Human  = { name: "krillin" | "yamcha"; age: number };

type SaiyanHuman = Saiyan & Human;
// name: never → SaiyanHuman is never
```

This is TypeScript telling you the type is impossible to construct.

### The `never` Type

`never` represents a value that can never exist. It arises naturally from:
- Exhausted union narrowing
- Intersecting incompatible types
- Functions that never return (infinite loop or always throw)

Use it for exhaustive union checks — see [[ts_type_system]].

### Unions vs Intersections

| | Union `\|` | Intersection `&` |
|---|---|---|
| Operator | `\|` (OR) | `&` (AND) |
| Effect | **Widens** the type | **Narrows** the type |
| Use case | "This OR that" | "This AND that" |
| Example | `string \| number` | `Serializable & Loggable` |

## Trade-offs

- **Unions** model mutually exclusive states (HTTP method, direction, status). Prefer over enums for lightweight cases.
- **Intersections** add required properties to existing types — but use `interface extends` for inheritance chains (better compiler performance and error messages).
- **Optional `?`** communicates "this field may not exist". Don't overuse it — require fields that should always be present.
- **Template literals** are powerful but can cause compilation timeouts when the combination space exceeds a few thousand members. Prefer `string` in those cases.
- Intersecting incompatible primitive types is a common mistake: `string & number` is `never`.

## References

- [TypeScript Handbook — Union Types](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#union-types)
- [TypeScript Handbook — Intersection Types](https://www.typescriptlang.org/docs/handbook/2/objects.html#intersection-types)
- [TypeScript Handbook — Literal Types](https://www.typescriptlang.org/docs/handbook/literal-types.html)
- [TypeScript Handbook — Template Literal Types](https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html)
- [TypeScript Handbook — never](https://www.typescriptlang.org/docs/handbook/2/never-type.html)

## Links

- [[ts_type_system]]
- [[ts_objects_and_interfaces]]
- [[ts_classes]]
