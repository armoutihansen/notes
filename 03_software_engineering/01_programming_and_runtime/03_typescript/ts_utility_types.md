---
layer: 03_software_engineering
type: engineering
tool: typescript
status: evergreen
tags: []
created: 2026-03-02
---

# TypeScript Utility Types

## Purpose

Built-in utility types transform existing types into new ones without duplicating type definitions. They embody the **single source of truth** principle: define a type once, then derive variations from it. When the source type changes, all derived types update automatically.

## Implementation Notes

### `Partial<T>` — All Properties Optional (Shallow)

Useful for update/patch functions where only some fields are provided:

```typescript
type User = { id: string; name: string; email: string };

function updateUser(userId: string, userInfo: Partial<User>) {
  // any subset of User is valid
}

updateUser("u1", { name: "Alice" }); // ok — id and email are optional
```

> **Shallow only.** Nested objects are untouched — their required properties remain required unless `preferences` itself is omitted.

```typescript
type Settings = {
  theme: string;
  notifications: boolean;
};
type User = { id: string; preferences: Settings };

type PartialUser = Partial<User>;
// { id?: string; preferences?: Settings }  ← Settings itself is NOT made partial
```

---

### `Required<T>` — All Properties Required (Shallow)

Opposite of `Partial` — strips all `?` modifiers:

```typescript
interface BlogPost {
  title: string;
  content: string;
  tags?: string[];
  publishDate?: Date;
}

type PublishedBlogPost = Required<BlogPost>;
// { title: string; content: string; tags: string[]; publishDate: Date }
```

Like `Partial`, it is shallow — nested optional properties are unaffected.

---

### `Pick<T, K>` — Subset of Properties

Create a leaner type by naming only the properties you need:

```typescript
interface Product {
  id: string;
  name: string;
  price: number;
  description: string;
  images: string[];
}

type ProductSummary = Pick<Product, "id" | "name" | "price">;

const list: ProductSummary[] = [
  { id: "p1", name: "Keyboard", price: 79.99 },
];
```

Great for function parameters that don't need the full object.

---

### `Omit<T, K>` — Exclude Properties

Complement of `Pick` — take everything *except* the listed keys:

```typescript
interface DatabaseUser {
  id: string;
  username: string;
  email: string;
  passwordHash: string;
  createdAt: Date;
}

// Safe representation for API responses
type PublicUser = Omit<DatabaseUser, "passwordHash">;

function getUserProfile(userId: string): PublicUser {
  const dbUser = fetchUser(userId);
  const { passwordHash, ...publicUser } = dbUser;
  return publicUser;
}
```

Common use: stripping sensitive fields before returning data to a client.

---

### `Record<K, V>` — Dictionary Type

Creates an object type with keys `K` and values `V`. Especially powerful when `K` is a union — it enforces exhaustiveness:

```typescript
// String key dictionary
type ScoreMap = Record<string, number>;
const scores: ScoreMap = { Alice: 95, Bob: 87 };

// Union key — all members must be present
type PlayerRole = "tank" | "healer" | "dps";
const partySize: Record<PlayerRole, number> = {
  tank: 1,
  healer: 2,
  dps: 3,
  // TypeScript error if any role is missing
};

// Exhaustive lookup table
type HttpStatus = 200 | 201 | 400 | 404 | 500;
const statusMessages: Record<HttpStatus, string> = {
  200: "OK",
  201: "Created",
  400: "Bad Request",
  404: "Not Found",
  500: "Internal Server Error",
};
```

---

### `Readonly<T>` — Immutable Type (Shallow)

Adds `readonly` to every top-level property, preventing reassignment after initialisation:

```typescript
interface Config {
  apiUrl: string;
  timeout: number;
}

const config: Readonly<Config> = { apiUrl: "https://api.example.com", timeout: 5000 };
config.apiUrl = "https://other.com"; // Error: Cannot assign to 'apiUrl' (readonly)
```

Like the other utility types, `Readonly` is shallow — nested objects can still be mutated unless you apply `Readonly` recursively or use `as const`.

---

### Single Source of Truth

The utility types exist to avoid duplicating type definitions. Instead of:

```typescript
interface User { id: string; name: string; email: string }
interface UserWithoutId { name: string; email: string }  // ← duplication
```

Derive:

```typescript
type UserWithoutId = Omit<User, "id">;
// or
type NewUserPayload = Pick<User, "name" | "email">;
```

Any change to `User` automatically propagates.

## Trade-offs

- `Partial` and `Required` are **shallow** — for deep variants, you need a recursive utility type or a custom approach.
- `Pick` vs `Omit`: use `Pick` when you want a small known set; use `Omit` when you want everything except a small known set.
- `Record<string, V>` is essentially an index signature — prefer `Record<LiteralUnion, V>` for exhaustiveness guarantees.
- Over-using utility types to build complex type expressions is a code smell — consider whether a plain `interface` would be clearer.

## References

- [TypeScript Handbook — Utility Types](https://www.typescriptlang.org/docs/handbook/utility-types.html)
- [Wikipedia — Single Source of Truth](https://en.wikipedia.org/wiki/Single_source_of_truth)

## Links

- [[ts_advanced_types]]
- [[ts_objects_and_interfaces]]
- [[ts_type_system]]
