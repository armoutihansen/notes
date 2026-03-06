---
layer: 03_software_engineering
type: concept
status: growing
tags: [design, patterns, oop, architecture]
created: 2026-03-05
---

# Design Patterns

## Definition

Design patterns are reusable solutions to commonly occurring problems in software design. Documented by the "Gang of Four" (Gamma, Helm, Johnson, Vlissides) in *Design Patterns: Elements of Reusable Object-Oriented Software* (1994). They are not copy-paste code — they are descriptions of relationships and responsibilities between classes.

Three families:

| Category | Concern | Patterns |
|----------|---------|---------|
| **Creational** | Object creation mechanisms | Factory Method, Abstract Factory, Builder, Prototype, Singleton |
| **Structural** | Composing objects and classes | Adapter, Bridge, Composite, Decorator, Facade, Flyweight, Proxy |
| **Behavioural** | Communication between objects | Strategy, Observer, Command, Iterator, Template Method, State, Chain of Responsibility, Visitor, Mediator, Memento, Interpreter |

---

## Intuition

Patterns solve recurring design problems in a way that is:
- **Named** — gives vocabulary for design discussions
- **Composable** — can be combined (e.g., a Decorator wrapping a Proxy)
- **Language-agnostic** — the structure transcends any specific language, though idiomatic implementations vary greatly

In Python, many patterns are either built into the language (Iterator, Decorator, Context Manager) or trivially implemented with first-class functions and duck typing.

---

## Formal Description

Each pattern has four essential elements:
1. **Name** — vocabulary handle
2. **Problem** — when to apply
3. **Solution** — arrangement of classes/objects
4. **Consequences** — trade-offs and results

---

## Applications

### Creational Patterns

**Problem they solve**: Decouple client code from the concrete classes it instantiates, enabling flexibility in what gets created and how.

---

#### Factory Method

Define an interface for creating an object, but let subclasses decide which class to instantiate.

```python
from abc import ABC, abstractmethod

class Serializer(ABC):
    @abstractmethod
    def serialize(self, data: dict) -> str: ...

class JSONSerializer(Serializer):
    def serialize(self, data: dict) -> str:
        import json
        return json.dumps(data)

class YAMLSerializer(Serializer):
    def serialize(self, data: dict) -> str:
        import yaml
        return yaml.dump(data)

def get_serializer(fmt: str) -> Serializer:
    """Factory function — Pythonic factory method equivalent."""
    registry = {"json": JSONSerializer, "yaml": YAMLSerializer}
    cls = registry.get(fmt)
    if cls is None:
        raise ValueError(f"Unknown format: {fmt}")
    return cls()

# Usage — client doesn't know which class it gets
s = get_serializer("json")
print(s.serialize({"key": "value"}))
```

Use when: the exact type to create is determined at runtime; when a class wants its subclasses to specify the objects it creates.

---

#### Builder

Separate the construction of a complex object from its representation, letting the same construction process create different representations.

```python
from dataclasses import dataclass, field
from typing import List

@dataclass
class Query:
    table: str
    columns: List[str]
    where: str = ""
    limit: int = 0
    order_by: str = ""

class QueryBuilder:
    def __init__(self, table: str):
        self._table = table
        self._columns: List[str] = ["*"]
        self._where = ""
        self._limit = 0
        self._order_by = ""

    def select(self, *cols: str) -> "QueryBuilder":
        self._columns = list(cols)
        return self

    def where(self, condition: str) -> "QueryBuilder":
        self._where = condition
        return self

    def limit(self, n: int) -> "QueryBuilder":
        self._limit = n
        return self

    def order_by(self, col: str) -> "QueryBuilder":
        self._order_by = col
        return self

    def build(self) -> Query:
        return Query(self._table, self._columns,
                     self._where, self._limit, self._order_by)

q = (QueryBuilder("users")
     .select("id", "name")
     .where("active = true")
     .limit(10)
     .order_by("created_at")
     .build())
```

Use when: construction involves many optional parameters; step-by-step construction is needed; you want to create different representations.

---

#### Singleton

Ensure a class has only one instance and provide a global access point.

```python
class Config:
    _instance: "Config | None" = None

    def __new__(cls) -> "Config":
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance._data = {}
        return cls._instance

    def set(self, key: str, value) -> None:
        self._data[key] = value

    def get(self, key: str, default=None):
        return self._data.get(key, default)
```

**Caution**: Singleton is widely considered an anti-pattern in testable code because it introduces global state. Prefer dependency injection of a shared instance. Use only for inherently single resources (e.g., thread pool, logger).

---

### Structural Patterns

**Problem they solve**: Compose objects into larger structures while keeping them flexible and efficient.

---

#### Adapter

Convert the interface of a class into another interface that clients expect. Lets incompatible interfaces work together.

```python
# Existing interface our system expects
class Target(ABC):
    @abstractmethod
    def request(self) -> str: ...

# Incompatible third-party class
class LegacyService:
    def specific_request(self) -> str:
        return "Legacy response"

# Adapter wraps the legacy service
class LegacyAdapter(Target):
    def __init__(self, adaptee: LegacyService):
        self._adaptee = adaptee

    def request(self) -> str:
        return self._adaptee.specific_request()

# Client code uses Target interface
def client(target: Target) -> None:
    print(target.request())

client(LegacyAdapter(LegacyService()))
```

---

#### Decorator

Attach additional responsibilities to an object dynamically. Decorators provide a flexible alternative to subclassing for extending functionality.

```python
from functools import wraps
import time

# Function decorator (Python built-in support)
def timed(fn):
    @wraps(fn)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = fn(*args, **kwargs)
        print(f"{fn.__name__} took {time.perf_counter()-start:.4f}s")
        return result
    return wrapper

# Class-based object decorator
class LoggingRepository:
    def __init__(self, repo):
        self._repo = repo

    def find(self, id_: int):
        print(f"Finding id={id_}")
        result = self._repo.find(id_)
        print(f"Found: {result}")
        return result
```

---

#### Facade

Provide a simplified interface to a complex subsystem. Reduces coupling between clients and subsystem internals.

```python
class VideoEncoder:
    def encode(self, path: str, fmt: str) -> str: ...

class AudioMixer:
    def mix(self, tracks: list) -> bytes: ...

class Uploader:
    def upload(self, data: bytes, dest: str) -> str: ...

# Facade hides the complexity
class VideoPublisher:
    def __init__(self):
        self._encoder = VideoEncoder()
        self._mixer = AudioMixer()
        self._uploader = Uploader()

    def publish(self, video_path: str, audio_tracks: list, dest: str) -> str:
        encoded = self._encoder.encode(video_path, "h264")
        audio = self._mixer.mix(audio_tracks)
        return self._uploader.upload(audio, dest)
```

---

#### Proxy

Provide a surrogate or placeholder for another object to control access to it. Variants: virtual proxy (lazy init), protection proxy (access control), remote proxy (network), caching proxy.

```python
class ExpensiveResource:
    def __init__(self):
        print("Heavy initialisation...")
        self._data = range(10_000_000)

    def compute(self) -> int:
        return sum(self._data)

class LazyProxy:
    """Virtual proxy: defers creation until first use."""
    def __init__(self):
        self._resource: ExpensiveResource | None = None

    def compute(self) -> int:
        if self._resource is None:
            self._resource = ExpensiveResource()
        return self._resource.compute()
```

---

### Behavioural Patterns

**Problem they solve**: Define algorithms, assign responsibilities, and manage communication between objects.

---

#### Strategy

Define a family of algorithms, encapsulate each one, and make them interchangeable. Strategy lets the algorithm vary independently from clients that use it.

```python
from typing import Protocol

class SortStrategy(Protocol):
    def sort(self, data: list) -> list: ...

class QuickSort:
    def sort(self, data: list) -> list:
        return sorted(data)   # simplified

class MergeSort:
    def sort(self, data: list) -> list:
        return sorted(data, key=lambda x: x)   # simplified

class Sorter:
    def __init__(self, strategy: SortStrategy):
        self._strategy = strategy

    def sort(self, data: list) -> list:
        return self._strategy.sort(data)

# Swap strategy at runtime
sorter = Sorter(QuickSort())
result = sorter.sort([3, 1, 2])
```

---

#### Observer

Define a one-to-many dependency so that when one object changes state, all its dependents are notified and updated automatically. Foundation of event-driven systems.

```python
from typing import Callable

class EventEmitter:
    def __init__(self):
        self._listeners: dict[str, list[Callable]] = {}

    def on(self, event: str, fn: Callable) -> None:
        self._listeners.setdefault(event, []).append(fn)

    def emit(self, event: str, *args, **kwargs) -> None:
        for fn in self._listeners.get(event, []):
            fn(*args, **kwargs)

emitter = EventEmitter()
emitter.on("data", lambda x: print(f"Received: {x}"))
emitter.on("data", lambda x: print(f"Logging: {x}"))
emitter.emit("data", {"value": 42})
```

---

#### Command

Encapsulate a request as an object, thereby letting you parameterise clients with different requests, queue or log requests, and support undoable operations.

```python
from abc import ABC, abstractmethod
from collections import deque

class Command(ABC):
    @abstractmethod
    def execute(self) -> None: ...
    @abstractmethod
    def undo(self) -> None: ...

class WriteCommand(Command):
    def __init__(self, doc: list, text: str):
        self._doc = doc
        self._text = text

    def execute(self) -> None:
        self._doc.append(self._text)

    def undo(self) -> None:
        self._doc.pop()

class Editor:
    def __init__(self):
        self._doc: list[str] = []
        self._history: deque[Command] = deque()

    def execute(self, cmd: Command) -> None:
        cmd.execute()
        self._history.append(cmd)

    def undo(self) -> None:
        if self._history:
            self._history.pop().undo()
```

---

#### Iterator

Provide a way to sequentially access elements of an aggregate object without exposing its underlying representation. Python's `__iter__`/`__next__` protocol is a first-class language feature.

```python
class Range:
    def __init__(self, start: int, stop: int, step: int = 1):
        self.start, self.stop, self.step = start, stop, step

    def __iter__(self):
        current = self.start
        while current < self.stop:
            yield current
            current += self.step

for x in Range(0, 10, 2):
    print(x)   # 0, 2, 4, 6, 8
```

---

#### Template Method

Define the skeleton of an algorithm in a base class, deferring some steps to subclasses. Lets subclasses redefine certain steps without changing the algorithm's structure.

```python
class DataPipeline(ABC):
    def run(self) -> None:
        data = self.extract()
        transformed = self.transform(data)
        self.load(transformed)

    @abstractmethod
    def extract(self) -> list: ...

    @abstractmethod
    def transform(self, data: list) -> list: ...

    def load(self, data: list) -> None:
        print(f"Loading {len(data)} records")

class CSVPipeline(DataPipeline):
    def extract(self) -> list:
        return [1, 2, 3]   # read from CSV

    def transform(self, data: list) -> list:
        return [x * 2 for x in data]
```

---

## Trade-offs Between Patterns

| Comparison | Notes |
|-----------|-------|
| **Strategy vs Template Method** | Strategy uses composition (swap algorithm object); Template Method uses inheritance (override steps). Prefer Strategy — composition is more flexible. |
| **Decorator vs Proxy** | Decorator adds behaviour; Proxy controls access. Both wrap an object — intent distinguishes them. |
| **Factory vs Builder** | Factory creates in one step; Builder constructs step-by-step. Use Builder when construction is complex or needs configuration. |
| **Observer vs Command** | Observer is push-based; Command encapsulates intent. Command supports undo; Observer does not. |
| **Facade vs Adapter** | Facade simplifies a subsystem for new clients; Adapter makes an existing interface compatible with another. |

**Over-engineering risk**: Patterns add classes and indirection. A simple function or dataclass often suffices. Apply patterns when the problem they solve is actually present — not pre-emptively.

## Links
- [[solid_principles|SOLID Principles]] — Patterns implement SOLID; Strategy → OCP, Observer → DIP
- [[clean_code|Clean Code]] — Patterns only help if the surrounding code is clean
