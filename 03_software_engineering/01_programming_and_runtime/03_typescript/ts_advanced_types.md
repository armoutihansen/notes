---
layer: 03_software_engineering
type: engineering
tool: typescript
status: evergreen
tags: []
created: 2026-03-02
---

# TypeScript Advanced Types

## Purpose

> **Note:** These are advanced features. They are primarily useful in **library code** that must be flexible, abstract, and reusable. Everyday application code rarely needs them — prefer plain object types, unions, and generics for day-to-day work.

Conditional types and mapped types give you a meta-level over the type system: you can compute types from other types, filter union members, reshape objects, and extract type information from generics.

## Implementation Notes

### Conditional Types

Conditional types follow ternary syntax at the type level:

```typescript
type NewType = SomeType extends OtherType ? TrueType : FalseType;
```

Simple example:

```typescript
type IsString<T> = T extends string ? true : false;

type R1 = IsString<"hello">; // true
type R2 = IsString<42>;      // false
```

### Built-in Conditional Types

TypeScript ships three commonly-used conditional utility types:

```typescript
// Keeps only the members of T that are assignable to U
type Extract<T, U>      = T extends U ? T : never;

// Removes members of T that are assignable to U
type Exclude<T, U>      = T extends U ? never : T;

// Removes null and undefined from T
type NonNullable<T>     = T extends null | undefined ? never : T;
```

Practical use — dynamically extracting mouse-related events:

```typescript
type ClickEvent     = { type: "click";     x: number; y: number };
type KeyEvent       = { type: "key";       key: string };
type MouseMoveEvent = { type: "mousemove"; x: number; y: number };
type FormEvent      = { type: "submit";    formId: string };
type Event = ClickEvent | KeyEvent | MouseMoveEvent | FormEvent;

// Automatically includes any future event that has x and y
type MouseRelatedEvents = Extract<Event, { x: number; y: number }>;
// = ClickEvent | MouseMoveEvent
```

### `infer` — Extracting Types from Generics

`infer` declares a type variable *inside* a conditional type to capture part of a matched type:

```typescript
type GetReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

function greet() { return "Hello!"; }
function add(a: number, b: number) { return a + b; }

type GreetReturn = GetReturnType<typeof greet>; // string
type AddReturn   = GetReturnType<typeof add>;   // number
```

The `infer R` says: "if `T` matches a function shape, capture its return type into `R`". TypeScript's built-in `ReturnType<T>` is implemented exactly this way.

### Mapped Types

Mapped types create new object types by iterating over the keys of an existing type:

```typescript
type Soldier = {
  name:   string;
  age:    number;
  branch: "garrison" | "military police" | "survey corps";
};

// Make all properties optional (same as Partial<T>)
type OptionalSoldier = {
  [K in keyof Soldier]?: Soldier[K];
};

// Make all values strings
type StringifiedSoldier = {
  [K in keyof Soldier]: string;
};
```

The pattern `[K in keyof T]` is the core idiom. `keyof T` produces the union of all keys; `in` iterates them; `T[K]` retrieves the value type.

### Mapped Types with Conditionals

Combine mapped and conditional types to filter or transform based on property type:

```typescript
// Keep only string-typed properties; set others to never
type FilteredSoldier = {
  [K in keyof Soldier]: Soldier[K] extends string ? Soldier[K] : never;
};
// Result:
// { name: string; age: never; branch: "garrison" | "military police" | "survey corps" }
```

Because `Soldier[K]` is used in the conditional, the precise type of `branch` is preserved — it isn't widened to `string`.

### Extracting / Filtering Keys from Types

Use a two-step pattern to get a *union of key names* that satisfy a condition:

```typescript
// Step 1: map each key to itself (if condition met) or never
type StringKeys<T> = {
  [K in keyof T]: T[K] extends string ? K : never;
};

// Step 2: index into that map to collapse the union
type StringKeyUnion<T> = StringKeys<T>[keyof T];

type Keys = StringKeyUnion<Soldier>;
// "name" | "branch"   (age is excluded because it's number)
```

This pattern is used in utility type implementations throughout the TS standard library.

## Trade-offs

- **Readability cost is real.** Conditional mapped types can become very hard to follow. Leave a comment explaining the intent.
- **Use built-ins first.** `Partial`, `Required`, `Pick`, `Omit`, `Extract`, `Exclude`, `NonNullable` — these cover the vast majority of real-world needs.
- **`infer` is powerful but rare.** Only reach for it when you genuinely need to capture a sub-type from a generic structure.
- **Application code should stay simple.** If you're writing this in an app rather than a shared library, step back and ask whether a plain type alias would be clearer.

## References

- [TypeScript Handbook — Conditional Types](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html)
- [TypeScript Handbook — Mapped Types](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html)
- [TypeScript Handbook — Utility Types](https://www.typescriptlang.org/docs/handbook/utility-types.html)

## Links

- [[ts_utility_types]]
- [[ts_generics]]
- [[ts_type_narrowing]]
- [[ts_unions_and_intersections]]
