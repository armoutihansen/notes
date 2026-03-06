---
layer: 03_software_engineering
type: engineering
tool: Hypothesis
status: growing
tags: [algorithm]
created: 2026-03-05
---

# Property-Based Testing

## Purpose

Reference for property-based testing (PBT) using the Hypothesis library. Instead of
specifying example inputs and expected outputs, PBT specifies invariants — properties
that must hold for *all* valid inputs. Hypothesis generates hundreds of inputs
automatically, including edge cases the author would never think to write by hand.

## Architecture

### Core Concept: Invariants vs Examples

```python
# Example-based: manually chosen inputs, manually chosen expected outputs
def test_sort_example():
    assert sorted([3, 1, 2]) == [1, 2, 3]
    assert sorted([]) == []
    assert sorted([1]) == [1]
    # What about: all duplicates? negative numbers? floats? NaN?

# Property-based: describe what MUST be true for any input
from hypothesis import given
from hypothesis import strategies as st

@given(st.lists(st.integers()))
def test_sort_properties(lst):
    result = sorted(lst)
    assert len(result) == len(lst)               # same length
    assert set(result) == set(lst)               # same elements
    assert all(result[i] <= result[i+1]          # non-decreasing
               for i in range(len(result) - 1))
    assert sorted(result) == result              # idempotent
```

Hypothesis found inputs like `[0, 0]`, `[-1, -2147483648]`, `[]`, `[1]` automatically.

---

### @given and Strategies

```python
from hypothesis import given, settings, assume, note
from hypothesis import strategies as st

# Primitive strategies
@given(st.integers())
def test_int(n): ...

@given(st.integers(min_value=0, max_value=100))
def test_bounded_int(n): ...

@given(st.floats(allow_nan=False, allow_infinity=False))
def test_float(x): ...

@given(st.text())
def test_text(s): ...

@given(st.text(alphabet=st.characters(blacklist_categories=("Cs",))))
def test_valid_unicode(s): ...  # exclude surrogates

@given(st.binary())
def test_bytes(b): ...

@given(st.booleans())
def test_bool(b): ...

# Collection strategies
@given(st.lists(st.integers(), min_size=1, max_size=20))
def test_nonempty_list(lst): ...

@given(st.sets(st.text(max_size=10), min_size=2))
def test_set(s): ...

@given(st.dictionaries(st.text(), st.integers()))
def test_dict(d): ...

@given(st.tuples(st.integers(), st.text()))
def test_tuple(t): ...

# Sampling from a fixed pool
@given(st.sampled_from(["read", "write", "admin"]))
def test_role(role): ...

# One-of: choose between strategies
@given(st.one_of(st.integers(), st.text(), st.none()))
def test_any_value(v): ...
```

---

### Building Complex Strategies

```python
from hypothesis import given
from hypothesis import strategies as st
from dataclasses import dataclass

@dataclass
class Order:
    customer_id: int
    amount: float
    currency: str

# st.builds: construct arbitrary objects
order_strategy = st.builds(
    Order,
    customer_id=st.integers(min_value=1),
    amount=st.floats(min_value=0.01, max_value=1_000_000, allow_nan=False),
    currency=st.sampled_from(["USD", "EUR", "SEK", "GBP"]),
)

@given(order_strategy)
def test_order_vat_is_non_negative(order):
    vat = calculate_vat(order)
    assert vat >= 0

# st.fixed_dictionaries: dict with typed fields
config_strategy = st.fixed_dictionaries({
    "host": st.text(min_size=1, max_size=50),
    "port": st.integers(min_value=1024, max_value=65535),
    "timeout": st.floats(min_value=0.1, max_value=30.0),
})

# Recursive / composite structures
json_value = st.recursive(
    st.none() | st.booleans() | st.integers() | st.text(),
    lambda children: st.lists(children) | st.dictionaries(st.text(), children),
    max_leaves=10,
)

@given(json_value)
def test_json_roundtrip(obj):
    import json
    assert json.loads(json.dumps(obj)) == obj
```

---

### @settings and Controlling Exploration

```python
from hypothesis import given, settings, HealthCheck

# Increase the number of test cases (default: 100)
@settings(max_examples=500)
@given(st.integers())
def test_intensive(n): ...

# Suppress slow-test health check (for expensive fixtures)
@settings(suppress_health_check=[HealthCheck.too_slow])
@given(st.lists(st.text(), min_size=100))
def test_large_inputs(lst): ...

# Deadline: fail if any example takes longer than N ms
@settings(deadline=200)   # 200ms per example
@given(st.integers())
def test_fast_enough(n): ...

# Disable deadline entirely
@settings(deadline=None)
@given(st.text())
def test_no_deadline(s): ...

# Phases: control what Hypothesis does
from hypothesis import Phase
@settings(phases=[Phase.generate, Phase.shrink])  # skip database replay
@given(st.integers())
def test_phases(n): ...
```

---

### Shrinking

When Hypothesis finds a failing example, it automatically **shrinks** it to the
smallest/simplest input that still fails. This is one of Hypothesis's most powerful
features — you see the minimal counterexample, not a random 200-element list.

```
# Found failing example: lst = [9, -3, 7, 0, 15, -1, 4, 2]
# After shrinking: lst = [0, -1]    ← minimal counterexample
```

Hypothesis stores the failing example in a `.hypothesis/examples` database and
replays it first on subsequent runs — failure is reproducible.

```python
# Force a known failing example for debugging
from hypothesis import reproduce_failure
@reproduce_failure("6.100.1", b"...")  # from output of a failing run
@given(st.lists(st.integers()))
def test_known_failure(lst): ...
```

---

### assume() — Preconditions

Use `assume()` to discard inputs that don't meet preconditions. Hypothesis adjusts
its search to find valid inputs — but excessive filtering slows generation.

```python
from hypothesis import given, assume
from hypothesis import strategies as st

@given(st.integers(), st.integers())
def test_integer_division(a, b):
    assume(b != 0)          # discard b=0; don't fail on ZeroDivisionError
    assert a // b == int(a / b) or abs(a / b - a // b) < 1
```

**Better pattern for complex preconditions:** use `st.filter()` on the strategy directly.
```python
nonzero = st.integers().filter(lambda x: x != 0)

@given(st.integers(), nonzero)
def test_division(a, b):
    assert isinstance(a // b, int)
```

---

### Stateful Testing with RuleBasedStateMachine

For testing stateful systems (data structures, APIs, protocol implementations),
`RuleBasedStateMachine` defines rules (actions) and invariants that must hold
after any sequence of actions.

```python
from hypothesis.stateful import RuleBasedStateMachine, rule, invariant, initialize

class ShoppingCartMachine(RuleBasedStateMachine):
    def __init__(self):
        super().__init__()
        self.cart = ShoppingCart()
        self.model = []  # parallel reference model

    @initialize(item=st.sampled_from(["apple", "banana", "cherry"]), qty=st.integers(1, 10))
    def add_initial_item(self, item, qty):
        self.cart.add(item, qty)
        self.model.append((item, qty))

    @rule(item=st.sampled_from(["apple", "banana", "cherry"]), qty=st.integers(1, 5))
    def add_item(self, item, qty):
        self.cart.add(item, qty)
        self.model.append((item, qty))

    @rule()
    def clear(self):
        self.cart.clear()
        self.model.clear()

    @invariant()
    def item_count_matches(self):
        assert self.cart.total_items() == sum(q for _, q in self.model)

    @invariant()
    def cart_is_never_negative(self):
        assert self.cart.total_items() >= 0

TestCart = ShoppingCartMachine.TestCase  # pytest discovers this
```

Hypothesis generates arbitrary sequences of `add_item` and `clear` calls and checks
invariants after each step.

---

### Applications: Data Pipeline Invariants

```python
import pandas as pd
from hypothesis import given
from hypothesis import strategies as st
from hypothesis.extra.pandas import column, data_frames

# Generate DataFrames with typed columns
@given(data_frames([
    column("age",    dtype=int,   elements=st.integers(0, 120)),
    column("income", dtype=float, elements=st.floats(0, 1e7, allow_nan=False)),
    column("active", dtype=bool),
]))
def test_preprocess_preserves_dtypes(df):
    result = preprocess(df)
    assert result["age"].dtype == int
    assert result["income"].dtype == float

# Serialisation round-trip
@given(st.lists(st.dictionaries(
    st.text(min_size=1, max_size=20),
    st.one_of(st.integers(), st.text(), st.floats(allow_nan=False)),
    min_size=1, max_size=10,
)))
def test_csv_roundtrip(records):
    df = pd.DataFrame(records)
    buf = io.StringIO()
    df.to_csv(buf, index=False)
    buf.seek(0)
    recovered = pd.read_csv(buf)
    # Column names preserved (types may differ after CSV round-trip — expected)
    assert list(recovered.columns) == list(df.columns)

# Model output invariants
@given(st.arrays(
    dtype=np.float32,
    shape=st.tuples(st.integers(1, 64), st.just(FEATURE_DIM)),
    elements=st.floats(-10, 10, allow_nan=False),
))
def test_classifier_output_is_valid_distribution(X):
    probs = model.predict_proba(X)
    assert probs.shape == (len(X), N_CLASSES)
    np.testing.assert_allclose(probs.sum(axis=1), 1.0, atol=1e-5)
    assert (probs >= 0).all() and (probs <= 1).all()
```

## Implementation Notes

### Hypothesis Database

Hypothesis saves failing examples to `.hypothesis/examples/` (local) and replays
them first on the next run. Commit this directory to version control to share
discovered failures with the team.

```bash
# .gitignore — do NOT ignore the examples database
# DO ignore the internal directory (large, auto-managed)
.hypothesis/unicode_data/
# Keep:
# .hypothesis/examples/
```

### Integration with pytest

```bash
pip install hypothesis

# Run with verbose output to see generated examples
pytest --hypothesis-show-statistics

# Profile which strategies are slowest
pytest --hypothesis-seed=0   # reproducible run with fixed seed
```

### Combining with Example-Based Tests

PBT complements, not replaces, example-based tests. Use examples for:
- Regression tests (specific bugs)
- Documentation of canonical behaviour
- Fast sanity checks

Use PBT for:
- Algorithmic correctness (sort, parse, encode/decode)
- Data pipeline invariants
- API contract validation
- Stateful system exploration

## Trade-offs

| Aspect | Property-Based | Example-Based |
|---|---|---|
| Edge case coverage | Excellent — systematic | Limited by author creativity |
| Readability | Requires understanding invariants | Immediately clear |
| Speed | Slower (100+ examples per test) | Fast |
| Debugging | Shrinking gives minimal failing case | Example is the case |
| Test writing effort | High (formulate invariants) | Low (just write the case) |
| Best for | Algorithms, parsers, pipelines | Business logic, regression |

**Diminishing returns warning**: not all code has clean invariants. Forcing PBT on
CRUD business logic often produces unhelpful tests. Reserve for code where invariants
are natural — encoding, validation, data transformations, mathematical operations.

## References

- [Hypothesis documentation](https://hypothesis.readthedocs.io/)
- [Hypothesis — strategies reference](https://hypothesis.readthedocs.io/en/latest/data.html)
- *Property-Based Testing with PropEr, Erlang, and Elixir* — Fred Hébert (concepts
  transfer well to Hypothesis)
- [John Hughes — How to Specify It (paper)](https://www.cs.tufts.edu/~nr/cs257/archive/john-hughes/specify.pdf)
- [Hypothesis — stateful testing docs](https://hypothesis.readthedocs.io/en/latest/stateful.html)

## Links
- [[testing_strategies|Testing Strategies]]
- [[test_driven_development|TDD]]
