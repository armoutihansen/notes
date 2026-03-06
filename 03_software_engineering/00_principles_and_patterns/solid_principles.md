---
layer: 03_software_engineering
type: concept
status: growing
tags: [design, oop, principles]
created: 2026-03-05
---

# SOLID Principles

## Definition

SOLID is an acronym for five object-oriented design principles introduced by Robert C. Martin. Together they guide the construction of software that is easy to maintain, extend, and understand.

| Letter | Principle | Core Idea |
|--------|-----------|-----------|
| S | Single Responsibility | A class should have only one reason to change |
| O | Open/Closed | Open for extension, closed for modification |
| L | Liskov Substitution | Subtypes must be substitutable for their base types |
| I | Interface Segregation | Clients should not depend on interfaces they don't use |
| D | Dependency Inversion | Depend on abstractions, not concretions |

---

### S — Single Responsibility Principle (SRP)

A module, class, or function should be responsible for exactly one actor — one stakeholder whose change requests could require modifying it.

```python
# Violates SRP: mixes data access + business logic + formatting
class Order:
    def calculate_total(self): ...
    def save_to_db(self): ...
    def generate_invoice_pdf(self): ...

# Respects SRP
class Order:
    def calculate_total(self): ...

class OrderRepository:
    def save(self, order: Order): ...

class InvoiceRenderer:
    def render_pdf(self, order: Order): ...
```

The key heuristic: **"a class should have only one reason to change."** If two different teams (e.g., finance and infrastructure) could independently request changes to the same class, it has two responsibilities.

---

### O — Open/Closed Principle (OCP)

Software entities should be **open for extension** (you can add new behaviour) but **closed for modification** (existing code stays unchanged). Typically achieved through abstraction and polymorphism.

```python
from abc import ABC, abstractmethod

class Discount(ABC):
    @abstractmethod
    def apply(self, price: float) -> float: ...

class NoDiscount(Discount):
    def apply(self, price: float) -> float:
        return price

class PercentDiscount(Discount):
    def __init__(self, pct: float):
        self.pct = pct
    def apply(self, price: float) -> float:
        return price * (1 - self.pct)

# Adding a new discount type never touches existing classes
class BuyOneGetOne(Discount):
    def apply(self, price: float) -> float:
        return price / 2
```

The strategy pattern, plugin systems, and decorator chains are common OCP enablers.

---

### L — Liskov Substitution Principle (LSP)

If `S` is a subtype of `T`, then objects of type `T` may be replaced by objects of type `S` without altering the correctness of the program. Formally: a subtype must honour the **contract** (preconditions, postconditions, invariants) of its supertype.

**Classic violation: the Square/Rectangle problem**

```python
class Rectangle:
    def set_width(self, w): self.width = w
    def set_height(self, h): self.height = h
    def area(self): return self.width * self.height

class Square(Rectangle):
    def set_width(self, w):
        self.width = w
        self.height = w   # breaks Rectangle's implicit contract
    def set_height(self, h):
        self.width = h
        self.height = h
```

Code that assumes `set_width` does not change height will break when given a `Square`. The fix: do not inherit; model them as separate types or use composition.

**Practical signals of LSP violation:**
- `isinstance` checks to decide behaviour
- Subclass raises `NotImplementedError` for inherited methods
- Subclass weakens postconditions or strengthens preconditions

---

### I — Interface Segregation Principle (ISP)

Clients should not be forced to depend on methods they do not use. Prefer many small, focused interfaces over one large "fat" interface.

```python
# Fat interface
class Worker(ABC):
    @abstractmethod
    def work(self): ...
    @abstractmethod
    def eat(self): ...   # Robots don't eat

# Segregated
class Workable(ABC):
    @abstractmethod
    def work(self): ...

class Feedable(ABC):
    @abstractmethod
    def eat(self): ...

class HumanWorker(Workable, Feedable):
    def work(self): ...
    def eat(self): ...

class RobotWorker(Workable):
    def work(self): ...
```

In Python, ISP is often satisfied by **structural subtyping** (duck typing / `Protocol`) rather than explicit inheritance, making it lighter-weight than in Java/C++.

---

### D — Dependency Inversion Principle (DIP)

1. High-level modules should not depend on low-level modules; both should depend on abstractions.
2. Abstractions should not depend on details; details should depend on abstractions.

```python
from abc import ABC, abstractmethod

# Abstraction
class MessageSender(ABC):
    @abstractmethod
    def send(self, msg: str) -> None: ...

# Low-level detail
class EmailSender(MessageSender):
    def send(self, msg: str) -> None:
        print(f"Email: {msg}")

class SMSSender(MessageSender):
    def send(self, msg: str) -> None:
        print(f"SMS: {msg}")

# High-level module depends on abstraction, not EmailSender directly
class NotificationService:
    def __init__(self, sender: MessageSender):
        self._sender = sender          # injected
    def notify(self, msg: str):
        self._sender.send(msg)
```

Dependency injection (constructor, method, or property injection) is the primary mechanism for DIP. Frameworks like `pytest` fixtures and `FastAPI` dependency injection implement this at the framework level.

## Intuition

- **SRP**: A class is like a job description — give it one job.
- **OCP**: Write code so new features are added by writing new code, not editing old code.
- **LSP**: Don't lie about your type. If you say you're a `Duck`, you must quack.
- **ISP**: Don't force clients to import what they don't need.
- **DIP**: Depend on the menu (interface), not the chef (implementation).

## Formal Description

The principles are grounded in:
- **Coupling** — degree of interdependence between modules. SOLID reduces coupling.
- **Cohesion** — degree to which elements belong together. SRP maximises cohesion.
- **Contracts** — pre/postconditions and invariants formalised in Hoare logic. LSP is a contract rule.
- **Stable Dependencies Principle** — depend in the direction of stability; DIP is a corollary.

## Applications

| Principle | Most relevant contexts |
|-----------|----------------------|
| SRP | Domain models, services, controller/view separation |
| OCP | Plugin architectures, strategy implementations, serialisers |
| LSP | Class hierarchies, collections of polymorphic objects |
| ISP | Python `Protocol` types, API surface design |
| DIP | Service wiring, testing with mocks, framework design |

In ML engineering: DIP is critical for swapping model backends (PyTorch vs ONNX vs TFLite) behind a common `Predictor` interface. SRP separates data loading, feature engineering, training, and evaluation.

## Trade-offs

- **Over-application**: Premature abstraction leads to unnecessary indirection. Applying OCP everywhere creates interface explosion.
- **SRP vs cohesion**: Splitting too aggressively fragments related logic across many files.
- **LSP vs pragmatism**: Strict LSP can prohibit natural hierarchies; sometimes composition is simply the right tool.
- **ISP in Python**: Python duck typing often makes explicit interface segregation redundant; `Protocol` classes provide opt-in segregation without inheritance overhead.
- **DIP vs simplicity**: Not every dependency needs inversion. Simple scripts and utilities don't need DI containers.

**When principles are in tension:**
- SRP and DIP can push in opposite directions when a class that "owns" behaviour also needs to construct its collaborators. Resolution: use factories or DI containers.
- OCP and YAGNI ("you ain't gonna need it") tension: don't abstract for hypothetical future variation.

## Links
- [[design_patterns|Design Patterns]] — Patterns are mechanisms for implementing SOLID (Strategy → OCP/DIP, Decorator → OCP)
- [[clean_code|Clean Code]] — Naming and structure practices that support SRP
