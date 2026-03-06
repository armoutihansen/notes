---
layer: 03_software_engineering
type: engineering
tool: pytest
status: growing
tags: [testing, pytest, test-doubles, coverage, ci, quality]
created: 2026-03-05
---

# Testing Strategies

## Purpose

Comprehensive reference for software testing approaches, test doubles, fixture management,
and coverage strategies. Effective testing is not about maximising coverage metrics — it is
about building a fast, reliable safety net that enables confident refactoring and deployment.

## Architecture

### Testing Pyramid

```
        ┌─────────────────────┐
        │         E2E         │  Few, slow, expensive, high confidence
        ├─────────────────────┤
        │     Integration     │  Moderate — test component boundaries
        ├─────────────────────┤
        │        Unit         │  Many, fast, isolated, cheap
        └─────────────────────┘
```

**Unit tests:** Exercise a single function or class in isolation. Dependencies are replaced
with test doubles. Should be milliseconds per test.

**Integration tests:** Exercise two or more components together (e.g., service + database,
HTTP handler + middleware). Use real or in-memory dependencies. Slower than unit tests.

**End-to-end tests:** Exercise the full system from the user's perspective (browser/API
client through to the database). Slowest, most fragile, highest confidence.

**Guideline**: Invest heavily in unit tests; write integration tests at component seams;
use E2E tests sparingly for critical user journeys.

---

### FIRST Properties of Good Tests

| Property | Meaning |
|---|---|
| **F**ast | Milliseconds per test; the suite runs in seconds |
| **I**solated | No shared mutable state between tests; order-independent |
| **R**epeatable | Same result every run regardless of environment or time |
| **S**elf-validating | Pass/fail is unambiguous — no manual inspection |
| **T**imely | Written at the same time as (or before) production code |

Violations of **Isolated** are the most common source of flaky tests.

---

### Test Doubles

```python
import pytest
from unittest.mock import MagicMock, patch, call

# STUB: returns fixed data; used to eliminate dependencies
def test_order_total_with_stub():
    pricing_service = MagicMock()
    pricing_service.get_price.return_value = 9.99   # stub behaviour
    order = Order(pricing_service)
    assert order.calculate_total(qty=3) == 29.97

# MOCK: stub + assertion on how it was called
def test_email_sent_on_order_completion():
    email_service = MagicMock()
    order = Order(email_service=email_service)
    order.complete()
    email_service.send.assert_called_once_with(
        to="customer@example.com", template="order_confirmation"
    )

# FAKE: working implementation with simplified internals
class InMemoryUserRepository:
    def __init__(self):
        self._store = {}
    def save(self, user): self._store[user.id] = user
    def get(self, id): return self._store.get(id)

def test_user_service_with_fake():
    repo = InMemoryUserRepository()
    svc = UserService(repo)
    svc.register("ada@example.com")
    assert repo.get(1).email == "ada@example.com"

# SPY: real object that records calls; verify interactions after the fact
# In Python, use MagicMock(wraps=real_object)
real_notifier = EmailNotifier()
spy = MagicMock(wraps=real_notifier)
order = Order(notifier=spy)
order.complete()
assert spy.notify.call_count == 1
```

**Prefer fakes over mocks** when the interface is stable and a fake is straightforward —
fakes test real behaviour; mocks test implementation details.

---

### pytest Fixtures

```python
import pytest

# Session-scoped fixture: created once for entire test run (expensive setup)
@pytest.fixture(scope="session")
def db_engine():
    engine = create_engine("postgresql://localhost/test_db")
    yield engine
    engine.dispose()

# Function-scoped (default): fresh instance per test
@pytest.fixture
def db_session(db_engine):
    connection = db_engine.connect()
    transaction = connection.begin()
    session = Session(bind=connection)
    yield session
    session.close()
    transaction.rollback()  # isolate each test — no committed state bleeds through
    connection.close()

# Parametrised fixtures: run test with multiple fixture variants
@pytest.fixture(params=["sqlite", "postgresql"])
def db(request):
    if request.param == "sqlite":
        return create_engine("sqlite:///:memory:")
    else:
        return create_engine("postgresql://localhost/test")

# Factory fixture: returns a callable that creates objects
@pytest.fixture
def make_user():
    created = []
    def _make(email="test@example.com", role="user"):
        user = User(email=email, role=role)
        db.session.add(user)
        db.session.flush()
        created.append(user)
        return user
    yield _make

# conftest.py: shared fixtures across test files (auto-discovered by pytest)
```

**Fixture scopes:** `function` (default) → `class` → `module` → `package` → `session`.
Use the widest scope that still guarantees isolation.

---

### Test Data Management

```python
# Option 1: Factory Boy — object factories for test data
import factory

class UserFactory(factory.Factory):
    class Meta:
        model = User
    email = factory.Sequence(lambda n: f"user{n}@example.com")
    role = "user"
    is_active = True

class AdminFactory(UserFactory):
    role = "admin"

user = UserFactory()
admin = AdminFactory(email="custom@example.com")
users = UserFactory.create_batch(10)

# Option 2: pytest-factoryboy — integrates factories as fixtures
from pytest_factoryboy import register
register(UserFactory)

def test_user(user):  # fixture injected automatically
    assert user.is_active

# Option 3: Faker — realistic fake data
from faker import Faker
fake = Faker()
user = User(email=fake.email(), name=fake.name())
```

---

### Coverage

```bash
# Run with coverage
pytest --cov=mypackage --cov-report=term-missing --cov-report=html

# Branch coverage (catches uncovered if/else branches)
pytest --cov=mypackage --cov-branch

# Fail if coverage drops below threshold
pytest --cov=mypackage --cov-fail-under=85
```

**Line coverage** counts executed lines. **Branch coverage** also tracks whether
both True and False paths of every conditional are exercised. Branch coverage is
strictly stronger.

**Mutation testing** (mutmut, cosmic-ray): automatically mutates production code
(e.g., flips `>` to `>=`, removes `not`) and checks if tests fail. A mutation that
survives means a test that doesn't actually verify the logic it claims to.

```bash
pip install mutmut
mutmut run --paths-to-mutate src/
mutmut results
```

---

### Property-Based vs Example-Based Testing

**Example-based** (the norm): `assert add(2, 3) == 5` — specific inputs and outputs.
Limited by the author's imagination for edge cases.

**Property-based**: specify invariants that must hold for *all* inputs in a domain.
The framework generates hundreds of inputs automatically, including edge cases.

```python
from hypothesis import given, strategies as st

@given(st.lists(st.integers()))
def test_sort_idempotent(lst):
    assert sorted(sorted(lst)) == sorted(lst)

@given(st.text(), st.text())
def test_concat_length(a, b):
    assert len(a + b) == len(a) + len(b)
```

See [[property_based_testing|Property Based Testing]] for full Hypothesis patterns.

---

### Testing in CI

```yaml
# .github/workflows/test.yml (GitHub Actions example)
- name: Run tests
  run: pytest --cov=src --cov-report=xml --cov-fail-under=80 -x -q

- name: Upload coverage
  uses: codecov/codecov-action@v4
```

**CI testing best practices:**
- Use `-x` (`--exitfirst`) to fail fast on first error during development.
- Use `-n auto` (pytest-xdist) to parallelise across CPU cores.
- Cache test dependencies to keep CI fast.
- Separate fast unit tests from slow integration tests using markers:

```python
# Mark slow tests
@pytest.mark.slow
def test_full_pipeline():
    ...

# Run only fast tests
# pytest -m "not slow"
# Run all
# pytest
```

## Implementation Notes

### pytest.ini / pyproject.toml Configuration

```toml
[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py", "*_test.py"]
python_classes = ["Test*"]
python_functions = ["test_*"]
markers = [
    "slow: marks tests as slow (deselect with '-m \"not slow\"')",
    "integration: marks integration tests",
    "unit: marks pure unit tests",
]
addopts = "-ra -q --strict-markers"
```

### Directory Layout

```
project/
├── src/
│   └── mypackage/
│       ├── __init__.py
│       ├── models.py
│       └── services.py
└── tests/
    ├── conftest.py          # shared fixtures
    ├── unit/
    │   ├── test_models.py
    │   └── test_services.py
    └── integration/
        └── test_api.py
```

## Trade-offs

| Approach | Upside | Downside |
|---|---|---|
| High mock count | Fast, isolated | Tests implementation not behaviour; brittle on refactor |
| Real dependencies (fakes) | Tests behaviour | Slower setup; fakes must stay in sync |
| High coverage target | Forces test writing | Can incentivise shallow tests for metric sake |
| E2E focused | High confidence | Slow, flaky; expensive to maintain |
| TDD | Design pressure, docs | Requires discipline; hard on unfamiliar domains |

## References

- [pytest documentation](https://docs.pytest.org/)
- [Factory Boy documentation](https://factoryboy.readthedocs.io/)
- *Growing Object-Oriented Software Guided by Tests* — Freeman & Pryce
- *The Art of Unit Testing* — Roy Osherove
- [Test Doubles — Martin Fowler](https://martinfowler.com/bliki/TestDouble.html)

## Links
- [[test_driven_development|TDD]]
- [[property_based_testing|Property Testing]]
- [[cicd_pipelines|CI/CD Pipelines]]
- [[04_ml_engineering/00_principles_and_lifecycle/ml_testing_principles|ML Testing Principles]]
