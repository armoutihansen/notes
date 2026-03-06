---
layer: 03_software_engineering
type: engineering
tool: Python
status: growing
tags: [pattern]
created: 2026-03-05
---

# Python Data Model

## Purpose

Python's data model is the set of interfaces (special methods, a.k.a. "dunder" methods) that objects implement to participate in the language's core operations: arithmetic, comparison, iteration, attribute access, context management, callable invocation, and more. Understanding the data model is what separates idiomatic Python from Python-that-looks-like-Java.

> "The Python data model formalizes the interfaces of the building blocks of the language itself." — Luciano Ramalho, *Fluent Python*

---

## Architecture

The model is built on **protocols** — informal interfaces defined purely by the presence of specific dunder methods. Unlike Java/C# interfaces, there is no explicit `implements` declaration; an object satisfies a protocol simply by having the right methods. From Python 3.8+, `typing.Protocol` makes structural subtyping explicit and checkable.

When you call `len(obj)`, Python calls `obj.__len__()`. When you write `a + b`, Python calls `a.__add__(b)` (and potentially `b.__radd__(a)`). Built-in functions and operators are thin wrappers around the data model.

---

## Implementation Notes

### `__repr__` and `__str__`

```python
class Vector:
    def __init__(self, x: float, y: float):
        self.x = x
        self.y = y

    def __repr__(self) -> str:
        # For developers: should be unambiguous, ideally eval()-able
        return f"Vector({self.x!r}, {self.y!r})"

    def __str__(self) -> str:
        # For end-users: readable presentation
        return f"({self.x}, {self.y})"

v = Vector(1, 2)
repr(v)    # "Vector(1, 2)"
str(v)     # "(1, 2)"
print(v)   # "(1, 2)"  — uses __str__
f"{v!r}"   # "Vector(1, 2)"  — forces __repr__
```

If only `__repr__` is defined, it is used as a fallback for `__str__`. Always implement `__repr__`; implement `__str__` only when a user-facing presentation differs.

---

### `__len__` and `__bool__`

```python
class Playlist:
    def __init__(self, tracks: list):
        self._tracks = tracks

    def __len__(self) -> int:
        return len(self._tracks)

    def __bool__(self) -> bool:
        return len(self) > 0    # or just: return bool(self._tracks)

p = Playlist([])
len(p)      # 0
bool(p)     # False
if not p:
    print("Empty playlist")
```

If `__bool__` is absent, Python falls back to `__len__` (truthy iff non-zero). If neither is defined, objects are always truthy.

---

### `__getitem__`, `__setitem__`, `__delitem__`, `__contains__`

Implementing `__getitem__` is enough to make an object iterable (Python will call it with indices 0, 1, 2... until `IndexError`). Implementing it fully enables slicing and the `in` operator.

```python
class TimeSeries:
    def __init__(self, values: list):
        self._values = values

    def __getitem__(self, index):
        return self._values[index]   # supports slicing automatically

    def __setitem__(self, index, value):
        self._values[index] = value

    def __len__(self) -> int:
        return len(self._values)

    def __contains__(self, item) -> bool:
        return item in self._values

ts = TimeSeries([1.0, 2.5, 3.1, 4.0])
ts[0]          # 1.0
ts[-1]         # 4.0
ts[1:3]        # [2.5, 3.1]  — slice works because list handles it
3.1 in ts      # True
for v in ts:   # iterable via __getitem__
    print(v)
```

---

### `__iter__` and `__next__`

`__iter__` makes an object **iterable** (usable in `for` loops, `list()`, `zip()`, etc.).
`__next__` makes it an **iterator** (stateful, single-pass).

```python
class CountUp:
    """An iterator that counts from start to stop."""
    def __init__(self, start: int, stop: int):
        self.current = start
        self.stop = stop

    def __iter__(self) -> "CountUp":
        return self   # iterators return themselves

    def __next__(self) -> int:
        if self.current >= self.stop:
            raise StopIteration
        value = self.current
        self.current += 1
        return value

list(CountUp(0, 5))   # [0, 1, 2, 3, 4]
```

**Iterable vs iterator distinction:**
```python
# An iterable returns a fresh iterator each time
class EvenNumbers:
    def __init__(self, limit):
        self.limit = limit

    def __iter__(self):
        return (x for x in range(0, self.limit, 2))   # new generator each call

evens = EvenNumbers(10)
list(evens)   # [0, 2, 4, 6, 8]
list(evens)   # [0, 2, 4, 6, 8]  — works again; iterable, not exhausted
```

---

### `__enter__` and `__exit__`

Enable the `with` statement (context manager protocol).

```python
class DatabaseTransaction:
    def __init__(self, connection):
        self._conn = connection
        self._tx = None

    def __enter__(self) -> "DatabaseTransaction":
        self._tx = self._conn.begin()
        return self

    def __exit__(self, exc_type, exc_val, exc_tb) -> bool:
        if exc_type is None:
            self._tx.commit()
        else:
            self._tx.rollback()
        return False   # False = do not suppress the exception

with DatabaseTransaction(conn) as tx:
    tx.execute("INSERT INTO ...")
```

`__exit__` signature:
- `exc_type`: exception class if an exception occurred, else `None`
- `exc_val`: exception instance
- `exc_tb`: traceback object
- Return `True` to suppress the exception; `False` (or `None`) to let it propagate

---

### `__call__`

Makes an instance callable like a function. Enables stateful callables — objects that behave like functions but carry state.

```python
class Throttle:
    """Callable that rate-limits invocations."""
    import time

    def __init__(self, fn, min_interval: float):
        self._fn = fn
        self._min_interval = min_interval
        self._last_called = 0.0

    def __call__(self, *args, **kwargs):
        import time
        now = time.monotonic()
        if now - self._last_called < self._min_interval:
            raise RuntimeError("Throttled")
        self._last_called = now
        return self._fn(*args, **kwargs)

def fetch(url): ...

fetch_throttled = Throttle(fetch, min_interval=1.0)
fetch_throttled("https://api.example.com")   # works
fetch_throttled("https://api.example.com")   # raises RuntimeError if called too quickly
```

`callable(obj)` returns `True` iff `obj` has `__call__`.

---

### Arithmetic and Comparison Operators

| Dunder | Operator | Reflected |
|--------|----------|-----------|
| `__add__` | `a + b` | `__radd__` |
| `__mul__` | `a * b` | `__rmul__` |
| `__eq__` | `a == b` | — |
| `__lt__` | `a < b` | `__gt__` |
| `__hash__` | `hash(a)` | — |

```python
class Vector:
    def __init__(self, x, y):
        self.x, self.y = x, y

    def __add__(self, other: "Vector") -> "Vector":
        return Vector(self.x + other.x, self.y + other.y)

    def __mul__(self, scalar: float) -> "Vector":
        return Vector(self.x * scalar, self.y * scalar)

    def __rmul__(self, scalar: float) -> "Vector":
        return self.__mul__(scalar)   # enables 3.0 * v

    def __eq__(self, other: object) -> bool:
        if not isinstance(other, Vector):
            return NotImplemented
        return self.x == other.x and self.y == other.y

    def __hash__(self) -> int:
        return hash((self.x, self.y))   # must define hash when __eq__ is defined

v1 = Vector(1, 2)
v2 = Vector(3, 4)
v1 + v2        # Vector(4, 6)
3.0 * v1       # Vector(3.0, 6.0) — via __rmul__
v1 == v1       # True
{v1, v2}       # works because __hash__ is defined
```

**Rule**: if you define `__eq__`, you must define `__hash__` (or set `__hash__ = None` to make it unhashable). Python sets `__hash__ = None` automatically when `__eq__` is defined without `__hash__`.

---

### Protocols via `typing.Protocol`

Structural subtyping: a class satisfies a `Protocol` if it has the required methods, regardless of inheritance.

```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class Drawable(Protocol):
    def draw(self) -> None: ...

class Circle:
    def draw(self) -> None:
        print("Drawing circle")

class Square:
    def draw(self) -> None:
        print("Drawing square")

def render(shape: Drawable) -> None:
    shape.draw()

# No inheritance needed; both satisfy Drawable
render(Circle())
render(Square())
isinstance(Circle(), Drawable)   # True (with @runtime_checkable)
```

**Key standard protocols:**

| Protocol | Required methods | Used by |
|----------|-----------------|---------|
| `Iterable[T]` | `__iter__` | `for`, `list()`, `zip()` |
| `Iterator[T]` | `__iter__`, `__next__` | `next()` |
| `Sequence[T]` | `__getitem__`, `__len__` | slicing, indexing |
| `Mapping[K, V]` | `__getitem__`, `__len__`, `__iter__` | `dict`-like access |
| `Callable[..., T]` | `__call__` | function calls |
| `ContextManager[T]` | `__enter__`, `__exit__` | `with` statement |
| `Hashable` | `__hash__` | dict keys, set elements |

---

### `__slots__`

By default, instances use a `__dict__` for attribute storage. `__slots__` replaces this with a fixed-size array, saving memory and speeding attribute access.

```python
class Point:
    __slots__ = ("x", "y")

    def __init__(self, x: float, y: float):
        self.x = x
        self.y = y

# ~40% less memory per instance vs __dict__-based class
# Cannot add arbitrary attributes at runtime
p = Point(1.0, 2.0)
p.z = 3.0   # AttributeError
```

Use `__slots__` when creating millions of small instances (e.g., nodes in a graph, data records).

---

## Trade-offs

| Decision | Advantage | Cost |
|----------|-----------|------|
| Implement `__eq__` without `__hash__` | Equality works | Object becomes unhashable — cannot be a dict key or set member |
| `__getitem__` only (no `__iter__`) | Simpler implementation | Python's fallback iteration works but is O(n) index probing |
| `__slots__` | Memory and speed | Cannot add arbitrary attrs; multiple inheritance with slots is tricky |
| `@runtime_checkable` Protocol | `isinstance()` checks work | Checks only method presence, not signatures — false positives possible |

---

## References

- [Python Data Model docs](https://docs.python.org/3/reference/datamodel.html)
- *Fluent Python* — Ramalho, 2nd ed. — Part I (Data Model) covers chapters 1–3 in depth
- [PEP 544 — Protocols: Structural subtyping](https://peps.python.org/pep-0544/)

---

## Links
- [[python_core_language|Core Language]] — generators, context managers, decorators all use data model protocols
- [[python_async|Async Python]] — `__aiter__`, `__anext__`, `__aenter__`, `__aexit__` are async variants of these protocols
