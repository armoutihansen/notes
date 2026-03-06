---
layer: 03_software_engineering
type: engineering
tool: typescript
status: evergreen
tags: []
created: 2026-03-02
---

# TypeScript Type Narrowing

## Purpose

Type narrowing makes a broad or union type more specific within a particular code path, allowing TypeScript's type checker to give you better tooling and catch more errors at compile time. The narrower the type, the more the compiler can verify — and the more the code self-documents.

## Implementation Notes

### Conditional Narrowing — `typeof`, `instanceof`, equality checks

TypeScript tracks how types flow through conditionals. Checking a discriminant property narrows the full union to the matching member:

```typescript
type WitcherCharacter = { type: "witcher"; magicPower: boolean };
type StarWarsCharacter = { type: "star-wars"; forceSensitive: boolean };
type Character = WitcherCharacter | StarWarsCharacter;

function fight(a: Character, b: Character) {
  if (a.type === "witcher" && b.type === "witcher") {
    fightWitcher(a, b); // a and b are WitcherCharacter here
  }
}
```

`typeof` narrows primitive types:

```typescript
function process(value: string | number) {
  if (typeof value === "string") {
    return value.toUpperCase(); // string
  }
  return value * 2; // number
}
```

`instanceof` narrows class instances:

```typescript
if (err instanceof TypeError) {
  // err is TypeError here
}
```

### Discriminated Unions Narrowing

A shared literal property ("discriminant") on each union member lets TypeScript narrow by switching on it:

```typescript
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "rect"; width: number; height: number };

function area(s: Shape): number {
  switch (s.kind) {
    case "circle": return Math.PI * s.radius ** 2;
    case "rect":   return s.width * s.height;
  }
}
```

### `in` Operator Narrowing

Use `in` when you don't control the types (e.g. external library) and can't add a discriminant:

```typescript
type TextMessage  = { content: string;  sentAt: Date };
type ImageMessage = { caption: string;  sentAt: Date };
type VideoMessage = { duration: number; sentAt: Date };
type Message = TextMessage | ImageMessage | VideoMessage;

function display(msg: Message) {
  if ("content" in msg) {
    console.log(msg.content); // TextMessage
  } else if ("caption" in msg) {
    console.log(msg.caption); // ImageMessage
  } else {
    console.log(msg.duration); // VideoMessage
  }
}
```

Prefer discriminated unions when you own the types; reach for `in` otherwise.

### Type Predicates

When built-in guards aren't enough, write a **type predicate** — a function whose return type is `value is T`:

```typescript
function isString(value: unknown): value is string {
  return typeof value === "string";
}

// More complex example — overlapping shapes
interface ManagerAdmin { accessLevel: number; numEmployees: number }
interface Admin       { accessLevel: number; payrollDate: Date }
interface Manager     { numEmployees: number }

function isManagerAdmin(boss: ManagerAdmin | Admin | Manager): boss is ManagerAdmin {
  return "numEmployees" in boss && !("payrollDate" in boss);
}

if (isManagerAdmin(boss)) {
  console.log(boss.numEmployees); // ManagerAdmin
}
```

### Guard Clauses (Early Returns)

Guard clauses narrow by eliminating the null/undefined case up front — cleaner than deeply nested `if/else`:

```typescript
function processName(name: string | null | undefined): string {
  if (name == null) {
    throw new Error("Name is required");
  }
  // TypeScript knows name is string here
  return name.toUpperCase();
}
```

### Type Assertions (`as T`) — Use Sparingly

Use `as` when *you* know more than TypeScript does (e.g. typed network responses, framework quirks):

```typescript
const userId = (route.query?.userId as string).toLowerCase();

async function getUser(id: string): Promise<User> {
  const data = await fetch(`/api/users/${id}`).then(r => r.json());
  return data as User; // we own the API contract
}
```

Prefer conditional narrowing over assertions whenever possible — assertions bypass the type checker.

### Double Assertion (`as unknown as T`)

Forces a cast between two types that have no overlap. **Almost never justified in production code**:

```typescript
// Only if you truly know what you're doing
const userId = (42 as unknown as string);
```

### Non-Null Assertion (`!`)

Tells TypeScript a value is neither `null` nor `undefined`:

```typescript
sendText(cleanedText!); // I know this can't be null
user.name!.first;       // I know name is always set in practice
```

Use only when you are certain. Prefer a guard clause if there is any doubt.

### `unknown` Type — Must Narrow Before Use

`unknown` is the safe alternative to `any` for values arriving from the outside world. Unlike `any`, it forces explicit narrowing before you can operate on the value:

```typescript
function processData(data: unknown) {
  if (typeof data === "string") {
    return data.toUpperCase();
  }
  if (typeof data === "number") {
    return data * 2;
  }
  throw new Error("Unexpected type");
}
```

Use `unknown` at I/O boundaries (API responses, user input, `JSON.parse`) and `any` only as a last resort.

### Type Hierarchy

```
unknown / any   ← widest (least known)
  object
    string | number | boolean | …
      "literal" | 42 | true | …
never           ← narrowest (impossible / exhausted)
```

Types lower in the hierarchy are assignable to those above, not the other way around.

### Exhaustive Checks with `never`

When a union is exhaustive, the final branch is unreachable. TypeScript will flag any unhandled members if you assign the value to `never`:

```typescript
type Notif = "email" | "sms" | "push";

function send(notif: Notif): string {
  switch (notif) {
    case "email": return "Sending email";
    case "sms":   return "Sending SMS";
    case "push":  return "Sending push notification";
    default: {
      const _exhaustive: never = notif; // compile error if a case is missing
      return _exhaustive;
    }
  }
}
```

## Trade-offs

| Approach | When to use |
|---|---|
| Conditional / `in` / discriminant | Preferred — type-safe, no runtime risk |
| Type predicates | Complex multi-property checks |
| Guard clauses | Null/undefined elimination at function entry |
| `as T` assertion | You own the contract and TypeScript can't infer it |
| `!` non-null | Very high confidence; value will never be null/undefined |
| `as unknown as T` | Essentially never — desperate last resort |

- Avoid `any`; use `unknown` instead when the type is genuinely unknown.
- Use `readonly` tuples and arrays to prevent mutation-based type confusion.
- Design discriminated unions proactively; they make narrowing trivial.

## References

- [TypeScript Handbook — Narrowing](https://www.typescriptlang.org/docs/handbook/2/narrowing.html)
- [TypeScript Handbook — `unknown` top type](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-0.html#new-unknown-top-type)
- [TypeScript Handbook — Type Predicates](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#using-type-predicates)
- [TypeScript Handbook — `typeof` Types](https://www.typescriptlang.org/docs/handbook/2/typeof-types.html)

## Links

- [[ts_type_system]]
- [[ts_unions_and_intersections]]
- [[ts_advanced_types]]
- [[ts_enums_and_tuples]]
