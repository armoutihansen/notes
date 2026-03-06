---
layer: 03_software_engineering
type: engineering
tool: javascript
status: evergreen
tags: []
created: 2026-03-02
---

# JavaScript — Objects and Classes

## Purpose

JavaScript's object model is the foundation of everything in the language. Objects are used as plain key-value stores, as records, and as class instances. Classes are syntactic sugar over prototypal inheritance, which remains the underlying mechanism.

## Implementation Notes

### Object Literals

```js
const apple = {
  name: "Apple",
  radius: 2,
  color: "red",
};

// Property access
apple.name;       // "Apple"
apple["name"];    // "Apple" — bracket notation for dynamic keys

const key = "color";
apple[key];       // "red"

// Accessing a missing property returns undefined (no error)
apple.weight;     // undefined
```

---

### Property Shorthand

When a variable name matches the property name, you can omit the value:

```js
const name = "Lane";
const age = 30;

const user = { name, age };   // same as { name: name, age: age }
```

---

### Spread Syntax

Shallow-copies or merges object properties. Later properties overwrite earlier ones:

```js
const defaults = { theme: "dark", lang: "en" };
const overrides = { lang: "fr", fontSize: 14 };

const config = { ...defaults, ...overrides };
// { theme: "dark", lang: "fr", fontSize: 14 }
```

Useful for creating modified copies without mutating the original:

```js
const updated = { ...user, age: 31 };  // user is unchanged
```

---

### Destructuring

Unpack object properties into local variables:

```js
const { radius, color } = apple;

// Rename during destructuring
const { radius: r, color: c } = apple;

// Default value if property is missing
const { weight = 0 } = apple;

// From a function return value
function getApple() { return { radius: 2, color: "red" }; }
const { radius, color } = getApple();
```

Return multiple values from a function by returning an object:

```js
function minMax(arr) {
  return { min: Math.min(...arr), max: Math.max(...arr) };
}
const { min, max } = minMax([3, 1, 4, 1, 5]);
```

---

### Optional Chaining `?.`

Safely access nested properties — returns `undefined` instead of throwing when an intermediate property is `null`/`undefined`:

```js
const h = tournament.referee?.height;         // undefined, no error
const city = user?.address?.city;             // safe deep access
const name = getUserById(id)?.profile?.name;  // safe method + property chain
```

---

### Object Methods and `this`

```js
const person = {
  firstName: "Lane",
  lastName: "Wagner",
  getFullName() {           // shorthand method syntax
    return `${this.firstName} ${this.lastName}`;
  },
};

person.getFullName(); // "Lane Wagner"
```

Methods can mutate properties. **`const` does not protect object contents:**

```js
const tree = {
  height: 256,
  cut() { this.height /= 2; },
};
tree.cut();          // tree.height is now 128
tree = {};           // TypeError — binding is const, but contents changed fine
```

---

### Prototypal Inheritance

Every object has a prototype. `Object.create()` sets the prototype explicitly:

```js
const animal = {
  speak() { console.log("..."); },
};

const dog = Object.create(animal);
dog.name = "Rex";
dog.speak();  // inherited from animal prototype
```

Property lookup walks the **prototype chain** until `null` is reached:

```
dog → animal → Object.prototype → null
```

```js
Object.getPrototypeOf(dog) === animal;          // true
Object.getPrototypeOf(animal) === Object.prototype; // true
```

---

### Classes (Syntactic Sugar over Prototypes)

```js
class User {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }

  greet() {
    return `Hi, I'm ${this.name}`;
  }
}

const u = new User("Lane", 30);
u.greet(); // "Hi, I'm Lane"
```

---

### Getters and Setters

```js
class User {
  constructor(name, age) {
    this._name = name;  // convention: _ prefix to avoid name collision with getter
    this._age = age;
  }

  get name() {
    return this._name.toUpperCase();
  }

  get age() { return this._age; }
  set age(value) {
    if (value < 0) throw new Error("Age can't be negative");
    this._age = value;
  }
}

const u = new User("lane", 29);
u.name;      // "LANE"
u.age = -1;  // throws
```

---

### Inheritance with `extends` and `super`

```js
class Animal {
  constructor(name) {
    this.name = name;
  }
  toString() { return `Animal: ${this.name}`; }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name);         // must call super() before using this
    this.breed = breed;
  }
  toString() {
    return `${super.toString()}, Breed: ${this.breed}`;
  }
  bark() { return "Woof!"; }
}

const d = new Dog("Rex", "Husky");
d.toString(); // "Animal: Rex, Breed: Husky"
d.bark();     // "Woof!"
```

---

### Private Fields (`#`)

Declare at the top of the class body; accessible only within the class:

```js
class BankAccount {
  #balance;                     // must be declared here

  constructor(initial) {
    this.#balance = initial;
  }

  deposit(n) { this.#balance += n; }
  get balance() { return this.#balance; }
}

const acc = new BankAccount(100);
acc.#balance; // SyntaxError — not accessible outside class
acc.balance;  // 100 — via getter
```

---

### Static Methods and Properties

Belong to the class itself, not instances:

```js
class MathUtils {
  static PI = 3.14159;

  static circleArea(r) {
    return MathUtils.PI * r * r;
  }
}

MathUtils.circleArea(5);  // 78.54
new MathUtils().circleArea(5); // TypeError — not on instances
```

## Trade-offs

- **Object literals vs classes**: Use object literals for simple one-off data bags; use classes when you need multiple instances, inheritance, or encapsulation.
- **Prototypal vs class syntax**: Classes are clearer for team code and mirror familiar OOP patterns, but they're still prototypes underneath. Understanding the prototype chain matters for debugging.
- **Private fields `#` vs convention `_`**: `#` is enforced by the engine (SyntaxError if accessed outside); `_` is merely a social convention.
- **`const` object mutation**: `const` only prevents rebinding the variable — the object contents are freely mutable. Use `Object.freeze()` if you want true immutability.
- **Optional chaining**: Don't chain so deeply that legitimate `undefined` errors become silent; `?.` should guard genuinely optional paths, not hide bugs.

## References

- [MDN: Working with objects](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Working_with_objects)
- [MDN: Classes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes)
- [MDN: Inheritance and the prototype chain](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)
- [MDN: Object.create()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create)
- [MDN: Optional chaining](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Optional_chaining)
- [MDN: Private class features](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Private_class_fields)

## Links

- [[js_core_language]]
- [[js_functions]]
- [[js_collections]]
- [[03_software_engineering/01_programming_and_runtime/02_javascript/index]]
