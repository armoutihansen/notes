---
layer: 03_software_engineering
type: engineering
tool: typescript
status: evergreen
tags: []
created: 2026-03-02
---

# TypeScript Generics

## Purpose

Generics let you write **reusable logic that preserves type information** across many concrete types — without falling back to `any`. Instead of writing `NumberQueue`, `StringQueue`, `UserQueue`, you write one `Queue<T>` that works with any `T` while keeping full type safety at each usage site. TypeScript's own utility types (`Partial<T>`, `Record<K,V>`, `Promise<T>`) are all generics.

## Implementation Notes

### Generic Functions

Add a type parameter `<T>` before the parameter list:

```ts
async function fetchFromApi<T>(url: string): Promise<T | undefined> {
  try {
    const response = await fetch(url);
    if (!response.ok) throw new Error("Network response was not ok");
    return await response.json() as T;
  } catch (error) {
    console.error("Error fetching data:", error);
    return undefined;
  }
}

// Caller specifies what they expect:
const user    = await fetchFromApi<User>("https://api.example.com/user/1");
const posts   = await fetchFromApi<Post[]>("https://api.example.com/posts");
const comments = await fetchFromApi<Comment[]>("https://api.example.com/comments");
```

Without `<T>`, the function would return `Promise<any>` — all downstream type information is lost.

### Generic Type Inference

TypeScript infers type parameters from the arguments passed — explicit annotation is usually unnecessary:

```ts
function transform<InputType, OutputType>(
  inputs: InputType[],
  update: (item: InputType) => OutputType,
): OutputType[] {
  return inputs.map(update);
}

type Human = { name: string; age: number };
const humans: Human[] = [
  { name: "Eren", age: 15 },
  { name: "Mikasa", age: 16 },
];

const toName = (h: Human): string => `${h.name} is a titan!`;

// TypeScript infers <Human, string> from the arguments:
const names = transform(humans, toName);
// names: string[]

// Explicit form works too, but is redundant here:
const names2 = transform<Human, string>(humans, toName);
```

### Multiple Type Parameters

Use multiple parameters when the function operates on more than one independent type:

```ts
function transform<InputType, OutputType>(
  inputs: InputType[],
  update: (item: InputType) => OutputType,
): OutputType[] {
  const outputs: OutputType[] = [];
  for (const input of inputs) {
    outputs.push(update(input));
  }
  return outputs;
}

const numbers = [1, 2, 3, 4, 5];
const doubled = transform(numbers, (n) => n * 2); // number[]

const humans: Human[] = [{ name: "Armin", age: 15 }];
const names = transform(humans, (h) => h.name);   // string[]
```

Convention: short capital letters — `T`, `U`, `V`, `K`, `V` — are standard, though descriptive names like `InputType`/`OutputType` improve readability in public APIs.

### Generic Constraints

Use `extends` to require that a type parameter satisfies a minimum shape:

```ts
interface HasCost {
  cost: number;
}

function applyDiscount<T extends HasCost>(items: T[], discount: number): T[] {
  return items.map((item) => ({ ...item, cost: item.cost * (1 - discount) }));
}

const shoes = [{ size: 12, cost: 120 }, { size: 11, cost: 110 }];
const discountedShoes = applyDiscount(shoes, 0.3); // still { size, cost }[]

const people = [{ name: "Lane" }];
// Error: '{ name: string }' does not satisfy constraint 'HasCost'
applyDiscount(people, 0.2);
```

The constraint allows the function to access `item.cost` safely, while still preserving the full `T` type in the return value.

### Generic Type Aliases and Interfaces

Type parameters work on `type` and `interface` too:

```ts
interface Store<T> {
  get(id: string): T;
  save(id: string, item: T): void;
  list(): T[];
}

// A function that works with any Store<T>:
function addAndGetItems<T>(store: Store<T>, id: string, item: T): T[] {
  store.save(id, item);
  return store.list();
}
```

Consume with a concrete type:

```ts
type Product = { name: string; price: number };

const productStore: Store<Product> = {
  products: {} as Record<string, Product>,
  get(id) { return this.products[id]; },
  save(id, item) { this.products[id] = item; },
  list() { return Object.values(this.products); },
};

const items = addAndGetItems(productStore, "laptop", { name: "Laptop", price: 999 });
// items: Product[]
```

### Generic Classes

Classes accept type parameters in the same position:

```ts
interface Repository<T> {
  getAll(): T[];
  getById(id: string): T | undefined;
  save(item: T): void;
}

class InMemoryRepository<T extends { id: string }> implements Repository<T> {
  private items: T[] = [];

  getAll(): T[] { return [...this.items]; }

  getById(id: string): T | undefined {
    return this.items.find((item) => item.id === id);
  }

  save(item: T): void {
    const index = this.items.findIndex((i) => i.id === item.id);
    if (index >= 0) {
      this.items[index] = item;
    } else {
      this.items.push(item);
    }
  }
}

interface User { id: string; name: string }

const userRepo = new InMemoryRepository<User>();
userRepo.save({ id: "1", name: "Alice" });
userRepo.save({ id: "2", name: "Bob" });
console.log(userRepo.getAll()); // [{ id: "1", name: "Alice" }, { id: "2", name: "Bob" }]
console.log(userRepo.getById("1")); // { id: "1", name: "Alice" }
```

Attempting to use a type without `id` causes a compile error:

```ts
interface Post { title: string }
// Error: Type 'Post' does not satisfy the constraint '{ id: string }'
const postRepo = new InMemoryRepository<Post>();
```

## Trade-offs

- **Generics vs `any`**: always prefer generics — `any` silently discards type information that generics preserve through the entire call chain.
- **Over-constraining**: adding too many `extends` constraints makes a generic function rigid. Only constrain what the function actually needs to access.
- **Type inference vs explicit annotation**: let TypeScript infer whenever possible. Explicit type args (`fn<MyType>(...)`) are needed when inference fails or for documentation at call sites.
- **Too many type parameters**: beyond 3–4 parameters, generics become hard to read. Consider whether a simpler design (dedicated function or wrapper type) would be clearer.
- **Generic classes vs factory functions**: generic classes carry all class overhead (instances, `this`, prototype). Generic factory functions or plain generic types are often simpler in functional-style code.

## References

- [TypeScript Handbook — Generics](https://www.typescriptlang.org/docs/handbook/2/generics.html)
- [TypeScript Handbook — Generic Functions](https://www.typescriptlang.org/docs/handbook/2/generics.html#generic-functions)
- [TypeScript Handbook — Generic Constraints](https://www.typescriptlang.org/docs/handbook/2/generics.html#generic-constraints)

## Links

- [[ts_type_system]]
- [[ts_objects_and_interfaces]]
- [[ts_classes]]
