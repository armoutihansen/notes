---
layer: 03_software_engineering
type: engineering
tool: Python
status: growing
tags: [python, language, types, generators, decorators]
created: 2026-03-05
---

# Python Core Language

## Purpose

A concise reference for Python's core language features: the type system, built-in types, comprehensions, generators, context managers, decorators, and closures. Targeted at developers who know Python but want to reason more precisely about its semantics.

---

## Architecture

Python's execution model:
1. Source is compiled to **bytecode** (`.pyc`) by CPython
2. Bytecode is interpreted by the **CPython VM** (a stack machine)
3. The **GIL** (Global Interpreter Lock) serialises bytecode execution in a single process — relevant for threading
4. Objects are reference-counted; the **cyclic GC** handles reference cycles

Everything in Python is an object, including functions, classes, and modules. The **data model** (dunder methods) defines how objects interact with operators and built-in functions — see [[python_data_model|Data Model]].

---

## Implementation Notes

### Type System

Python is **dynamically typed** (type checked at runtime) and **strongly typed** (no implicit coercion between unrelated types).

Since Python 3.5+ the language has **optional static type annotations**. The interpreter ignores them at runtime; static checkers (mypy, pyright) enforce them.

```python
# Type annotations (PEP 526, 484)
from typing import Optional, Union

def greet(name: str, times: int = 1) -> str:
    return (f"Hello, {name}!\n" * times).strip()

# Union types (Python 3.10+ syntax)
def process(value: int | str) -> str:
    return str(value)

# Optional is equivalent to X | None
def find(key: str) -> Optional[str]:   # str | None
    return None
```

**Generic types:**
```python
from typing import TypeVar, Generic, Sequence

T = TypeVar("T")

class Stack(Generic[T]):
    def __init__(self) -> None:
        self._items: list[T] = []

    def push(self, item: T) -> None:
        self._items.append(item)

    def pop(self) -> T:
        return self._items.pop()
```

---

### Built-in Types

| Type | Notes |
|------|-------|
| `int` | Arbitrary precision |
| `float` | IEEE 754 double (64-bit) |
| `complex` | `3+4j` |
| `str` | Immutable Unicode sequences |
| `bytes` / `bytearray` | Raw bytes; `bytearray` is mutable |
| `list` | Mutable dynamic array |
| `tuple` | Immutable sequence; used for heterogeneous records |
| `dict` | Hash map; insertion-ordered since Python 3.7 |
| `set` / `frozenset` | Hash set; `frozenset` is immutable |
| `bool` | Subtype of `int`; `True == 1`, `False == 0` |
| `NoneType` | Singleton `None` |

**Mutability matters:**
```python
# Lists are mutable — default arg trap
def append_to(val, lst=[]):   # lst is shared across calls!
    lst.append(val)
    return lst

# Fix: use None sentinel
def append_to(val, lst=None):
    if lst is None:
        lst = []
    lst.append(val)
    return lst

# Tuples as hashable records
point = (3, 4)
d = {point: "origin-ish"}  # tuples can be dict keys; lists cannot
```

---

### Comprehensions

Concise, readable alternative to `map`/`filter` and explicit loops.

```python
numbers = range(10)

# List comprehension
squares = [x**2 for x in numbers if x % 2 == 0]

# Dict comprehension
word_lengths = {word: len(word) for word in ["hello", "world"]}

# Set comprehension
unique_mods = {x % 3 for x in numbers}

# Nested comprehension — matrix transpose
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
transposed = [[row[i] for row in matrix] for i in range(3)]

# Generator expression (lazy, no brackets)
total = sum(x**2 for x in numbers)   # no intermediate list allocated
```

**Prefer comprehensions over `map`/`filter`** for readability. Use generator expressions when the full list is not needed — they are memory-efficient for large sequences.

---

### Generators and Iterators

A **generator function** contains `yield` and returns a **generator object** — a lazy iterator that produces values on demand.

```python
from typing import Iterator, Generator

# Generator function
def fibonacci() -> Generator[int, None, None]:
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

# Consume lazily
fib = fibonacci()
first_ten = [next(fib) for _ in range(10)]

# Generator with send() — coroutine-style
def accumulator() -> Generator[float, float, str]:
    total = 0.0
    while True:
        value = yield total
        if value is None:
            break
        total += value
    return f"Final total: {total}"
```

**`yield from`** — delegates to a sub-generator, propagating values and exceptions:
```python
def chain(*iterables):
    for it in iterables:
        yield from it

list(chain([1, 2], [3, 4]))  # [1, 2, 3, 4]
```

**Custom iterator protocol:**
```python
class Countdown:
    def __init__(self, start: int):
        self.current = start

    def __iter__(self) -> "Countdown":
        return self

    def __next__(self) -> int:
        if self.current <= 0:
            raise StopIteration
        self.current -= 1
        return self.current + 1
```

---

### Context Managers

Context managers define setup/teardown logic for `with` statements via `__enter__` and `__exit__`. See [[python_data_model|Data Model]] for protocol details.

```python
# Class-based context manager
class Timer:
    def __enter__(self):
        import time
        self._start = time.perf_counter()
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        import time
        self.elapsed = time.perf_counter() - self._start
        return False   # don't suppress exceptions

with Timer() as t:
    result = sum(range(1_000_000))
print(f"Elapsed: {t.elapsed:.4f}s")
```

**`contextlib.contextmanager`** — simpler generator-based approach:
```python
from contextlib import contextmanager

@contextmanager
def managed_connection(url: str):
    conn = connect(url)
    try:
        yield conn
    finally:
        conn.close()   # always runs, even on exception

with managed_connection("db://localhost") as conn:
    conn.execute("SELECT 1")
```

---

### Decorators

A decorator is a callable that takes a function and returns a replacement function. Python applies `@decorator` syntax as sugar for `fn = decorator(fn)`.

```python
from functools import wraps
import time
import logging

# Decorator factory (takes arguments)
def retry(times: int = 3, exceptions: tuple = (Exception,)):
    def decorator(fn):
        @wraps(fn)
        def wrapper(*args, **kwargs):
            for attempt in range(1, times + 1):
                try:
                    return fn(*args, **kwargs)
                except exceptions as e:
                    if attempt == times:
                        raise
                    logging.warning(f"Attempt {attempt} failed: {e}")
        return wrapper
    return decorator

@retry(times=3, exceptions=(ConnectionError,))
def fetch_data(url: str) -> dict:
    ...
```

**Class-based decorator:**
```python
class Cached:
    def __init__(self, fn):
        self._fn = fn
        self._cache = {}
        wraps(fn)(self)   # copy metadata

    def __call__(self, *args):
        if args not in self._cache:
            self._cache[args] = self._fn(*args)
        return self._cache[args]

@Cached
def slow_compute(n: int) -> int:
    return sum(range(n))
```

Standard library decorators: `@functools.lru_cache`, `@functools.cached_property`, `@staticmethod`, `@classmethod`, `@property`, `@dataclasses.dataclass`.

---

### Closures

A closure is a function that captures variables from its enclosing scope, even after the enclosing function returns.

```python
def make_multiplier(factor: float):
    # `factor` is a free variable captured in the closure
    def multiply(x: float) -> float:
        return x * factor
    return multiply

double = make_multiplier(2.0)
triple = make_multiplier(3.0)
print(double(5))   # 10.0
print(triple(5))   # 15.0
```

**`nonlocal` for mutable closure state:**
```python
def make_counter(start: int = 0):
    count = start
    def increment() -> int:
        nonlocal count
        count += 1
        return count
    return increment

c = make_counter()
c()  # 1
c()  # 2
```

Closures are the mechanism behind partial application, decorators, and callback factories.

---

## Trade-offs

| Feature | Advantage | Caveat |
|---------|-----------|--------|
| Dynamic typing | Fast iteration, duck typing | Runtime errors catch fewer bugs early; mitigate with type annotations + mypy |
| Generators | Memory-efficient, lazy | Single-pass; no random access; harder to debug |
| GIL | Safe reference counting, simpler C extensions | Limits CPU-bound threading — use multiprocessing or async for concurrency |
| Decorators | Clean separation of cross-cutting concerns | Stack of decorators can obscure what a function does; can complicate debugging |
| Mutable defaults | — | Classic footgun; always use `None` sentinel for mutable default args |

---

## References

- [Python Language Reference](https://docs.python.org/3/reference/)
- [PEP 484 — Type Hints](https://peps.python.org/pep-0484/)
- [PEP 255 — Simple Generators](https://peps.python.org/pep-0255/)
- *Fluent Python* — Luciano Ramalho (2nd ed, 2022) — chapters 7 (closures), 14 (iterators), 15 (context managers), 17 (generators)

---

## Links
- [[python_data_model|Data Model]] — dunder methods, protocols, how built-ins delegate to objects
- [[python_async|Async Python]] — coroutines and the event loop build on generator protocol
- [[python_tooling|Python Tooling]] — type checking, linting, project structure
