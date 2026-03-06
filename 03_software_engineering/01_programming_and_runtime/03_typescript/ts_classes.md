---
layer: 03_software_engineering
type: engineering
tool: typescript
status: evergreen
tags: []
created: 2026-03-02
---

# TypeScript Classes

## Purpose

Classes in TypeScript extend JavaScript's ES2015 class syntax with static type checking: property type declarations, access modifiers, abstract members, and the `implements` clause. They're used when you need encapsulation, inheritance, shared method implementations, or private state — capabilities that plain `type` / `interface` aliases do not have.

## Implementation Notes

### Basic Class with Type Annotations

Properties must be declared with their types at the class level:

```ts
class Hero {
  name: string;
  health: number;

  constructor(name: string, health: number) {
    this.name = name;
    this.health = health;
  }

  attack(damage: number): void {
    console.log(`${this.name} attacks for ${damage} damage!`);
  }

  getHealth() {
    return this.health; // return type inferred as number
  }
}

const geralt = new Hero("Geralt", 100);
geralt.attack(25); // "Geralt attacks for 25 damage!"
```

### Access Modifiers

TypeScript adds `public`, `private`, and `protected` modifiers:

| Modifier | Accessible from |
|---|---|
| `public` (default) | Anywhere |
| `private` | Inside the class only |
| `protected` | Inside the class and subclasses |

```ts
class Character {
  protected health: number;

  constructor(health: number) {
    this.health = health;
  }

  protected takeDamage(amount: number): void {
    this.health = Math.max(0, this.health - amount);
  }
}

class Fighter extends Character {
  public fight(damage: number): void {
    this.takeDamage(damage);             // OK — protected access
    console.log(`Health: ${this.health}`); // OK — protected access
  }
}

const f = new Fighter(100);
f.fight(30);
f.health;      // Error: protected
f.takeDamage(10); // Error: protected
```

### Constructor Shorthand

Declare and initialise properties in one step by adding a modifier to constructor parameters:

```ts
class SecretAgent {
  constructor(
    private id: string,
    public codeName: string,
    readonly joinedAt: Date,
  ) {}
  // id, codeName, joinedAt are automatically assigned as class properties
}
```

### `readonly`

A `readonly` property can be set in the constructor but not reassigned afterwards:

```ts
class Config {
  readonly apiUrl: string;
  constructor(url: string) {
    this.apiUrl = url;
  }
}
const c = new Config("https://api.example.com");
c.apiUrl = "..."; // Error: read-only property
```

### Abstract Classes and Methods

An `abstract` class cannot be instantiated directly; it serves as a template. Abstract methods have no implementation — subclasses must provide one:

```ts
abstract class Shape {
  size: "small" | "medium" | "large";

  constructor(size: "small" | "medium" | "large") {
    this.size = size;
  }

  abstract calculateArea(): number; // subclass must implement

  displayArea(): void {
    console.log(`Area: ${this.calculateArea()}`); // can call abstract method
  }
}

class Circle extends Shape {
  radius: number;

  constructor(size: "small" | "medium" | "large") {
    super(size);
    this.radius = size === "small" ? 5 : size === "medium" ? 10 : 15;
  }

  calculateArea(): number {
    return Math.PI * this.radius ** 2;
  }
}

const circle = new Circle("medium");
circle.displayArea(); // "Area: 314.159..."
// new Shape("small"); // Error: cannot instantiate abstract class
```

`abstract` is TypeScript-only and is erased at compile time.

### Implementing Interfaces

The `implements` clause enforces that a class conforms to one or more interfaces:

```ts
interface Vehicle { make: string; model: string }
interface Drivable { drive(distance: number): void }

class ElectricCar implements Vehicle, Drivable {
  make: string;
  model: string;
  private isRunning = false;

  constructor(make: string, model: string) {
    this.make = make;
    this.model = model;
  }

  drive(distance: number): void {
    this.isRunning = true;
    console.log(`Driving ${distance} miles`);
  }
}

const myCar = new ElectricCar("Tesla", "Model S");

function testDrive(vehicle: Vehicle) {
  console.log(`Testing ${vehicle.make} ${vehicle.model}`);
}
testDrive(myCar); // "Testing Tesla Model S"
```

Classes can add extra properties beyond what the interface requires.

### Classes vs Interfaces vs Types

```ts
class Hero    { name: string; health: number; }  // runtime value + type
interface Hero { name: string; health: number; } // type only (erased)
type Hero =   { name: string; health: number; }  // type only (erased)
```

Use **classes** when you need:
- Private / protected / abstract members
- Constructors with initialisation logic
- Shared method implementations on instances
- Inheritance hierarchies

Use **`type` / `interface`** when you only need to describe a shape (no runtime overhead).

### The `this` Type

Inside a class, `this` is automatically typed as the class instance. You can make it explicit for clarity or to prevent unsafe detachment:

```ts
class Counter {
  private count = 0;

  // this parameter is erased at runtime; it's only for type checking
  increment(this: Counter, n: number): void {
    this.count += n;
  }

  getCount(this: Counter): number {
    return this.count;
  }
}

const counter = new Counter();
counter.increment(5);
console.log(counter.getCount()); // 5
```

### Private Fields: `#` vs `private`

**`#` (native JS, ES2022)** — enforced at runtime and compile time:

```ts
class SecretAgent {
  #id: string;

  constructor(id: string) { this.#id = id; }

  getCodeName(): string { return this.#id === "007" ? "James Bond" : "Unknown"; }
}

const bond = new SecretAgent("007");
bond.#id; // Error at compile time AND runtime
```

**`private` (TypeScript-only)** — enforced only at compile time; compiled to a plain property at runtime:

```ts
class SecretAgent {
  private id: string;
  constructor(id: string) { this.id = id; }
}
bond.id; // Compile error only — accessible at runtime via JS
```

**Recommendation**: prefer `#` because it's the JavaScript standard and provides true runtime privacy. Use `private` only when targeting environments that don't support ES2022 private fields.

### Static Members

Static properties and methods belong to the class itself, not instances:

```ts
class MathHelper {
  static readonly PI = 3.14159;

  static circleArea(radius: number): number {
    return MathHelper.PI * radius ** 2;
  }
}

MathHelper.circleArea(5); // 78.53...
```

## Trade-offs

- **Classes vs plain objects**: classes add runtime overhead (prototype chain) and coupling. Prefer `type` / `interface` + plain functions for pure data shapes.
- **`protected` is TypeScript-only**: it disappears at runtime. If you need runtime privacy, use `#`.
- **`abstract` classes vs interfaces**: abstract classes can provide default implementations; interfaces cannot. Use abstract classes when subclasses should share implementation, not just contract.
- **`implements` does not inherit**: it only type-checks the contract. Behaviour must still be implemented on the class.
- **Constructor shorthand** (`constructor(private x: string)`) reduces boilerplate but can be surprising in code reviews — document or use only in simple cases.

## References

- [TypeScript Handbook — Classes](https://www.typescriptlang.org/docs/handbook/2/classes.html)
- [TypeScript Handbook — Abstract Classes](https://www.typescriptlang.org/docs/handbook/2/classes.html#abstract-classes-and-members)
- [TypeScript Handbook — `this` in functions](https://www.typescriptlang.org/docs/handbook/2/functions.html#declaring-this-in-a-function)

## Links

- [[ts_type_system]]
- [[ts_objects_and_interfaces]]
- [[ts_generics]]
