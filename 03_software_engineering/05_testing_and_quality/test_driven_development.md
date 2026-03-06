---
layer: 03_software_engineering
type: engineering
tool: pytest
status: growing
tags: [tdd, testing, pytest, design, bdd, refactoring]
created: 2026-03-05
---

# Test-Driven Development

## Purpose

Reference for the TDD workflow, its design benefits, the two main schools of thought
(London vs Chicago), and practical application to Python and ML code. TDD is less about
testing and more about design: writing tests first forces you to think about interfaces
before implementations.

## Architecture

### Red-Green-Refactor Cycle

```
  ┌─────────────────────────────────────────────────────┐
  │                                                     │
  │  1. RED      Write a failing test for the next      │
  │              small behaviour. Run it — it must fail. │
  │              (If it passes immediately, the test     │
  │              or the code is wrong.)                  │
  │                    ↓                                 │
  │  2. GREEN    Write the minimal production code to   │
  │              make the test pass. Resist the urge    │
  │              to write more than needed.              │
  │                    ↓                                 │
  │  3. REFACTOR Clean up: remove duplication, improve  │
  │              naming, extract abstractions. Tests     │
  │              stay green throughout.                  │
  │                    │                                 │
  │                    └──────────── repeat ─────────────┘
```

**Key discipline**: only write production code in response to a failing test. Never
skip the Red step — a test that starts green has not verified anything.

---

### Minimal Example in pytest

```python
# Step 1: RED — write test first
def test_fizzbuzz_multiple_of_3():
    assert fizzbuzz(3) == "Fizz"   # NameError: fizzbuzz not defined → RED

# Step 2: GREEN — implement just enough
def fizzbuzz(n: int) -> str:
    if n % 3 == 0:
        return "Fizz"
    return str(n)
# test_fizzbuzz_multiple_of_3 passes → GREEN

# Step 3: RED again — add next behaviour
def test_fizzbuzz_multiple_of_5():
    assert fizzbuzz(5) == "Buzz"   # fails → RED

# GREEN — extend
def fizzbuzz(n: int) -> str:
    if n % 15 == 0: return "FizzBuzz"
    if n % 3 == 0:  return "Fizz"
    if n % 5 == 0:  return "Buzz"
    return str(n)

# REFACTOR — no duplication here; move on
```

---

### Benefits of TDD

**Forced interface design:**  
Writing a test before code requires you to define the public API of the unit. If the
test is awkward to write, the interface is probably awkward. Discovering this before
writing implementation is cheap.

**Documentation by example:**  
The test suite is executable documentation. `test_invoice_total_includes_tax` tells
the next developer exactly what the system does in plain code.

**Regression safety:**  
Every bug fix starts with a test that reproduces the bug. The test ensures the bug
cannot silently reappear.

**Confidence to refactor:**  
A green test suite is a safety net. Without it, refactoring is risky; with it,
large-scale restructuring is routine.

---

### London School (Outside-In / Mockist) TDD

Starts from the outermost layer (e.g., HTTP handler) and works inward. Collaborators
are always mocked; behaviour is verified through interaction assertions.

```python
# Test HTTP handler first; mock the service layer
def test_create_order_returns_201(client, mocker):
    mock_svc = mocker.patch("app.routes.order_service.create_order")
    mock_svc.return_value = Order(id=42, status="pending")

    response = client.post("/orders", json={"product_id": 1, "qty": 2})

    assert response.status_code == 201
    assert response.json()["id"] == 42
    mock_svc.assert_called_once_with(product_id=1, qty=2)
```

**Characteristics:**
- Tests are fast — no real DB or network.
- Drives the design of collaborator interfaces from consumer's perspective.
- Risk: mocking internal implementation means tests break on internal refactoring
  even when behaviour is unchanged. Tests can become tightly coupled to structure.

---

### Chicago / Detroit School (Inside-Out / Classicist) TDD

Starts from the innermost domain logic and builds outward. Real objects are used
wherever feasible; mocks only for external systems (DB, network, clock).

```python
# Test domain model directly; no mocks needed
def test_order_becomes_confirmed_when_payment_succeeds():
    order = Order(items=[Item(sku="A1", qty=1, price=10.00)])
    payment = FakePaymentGateway(always_succeeds=True)

    order.confirm(payment)

    assert order.status == OrderStatus.CONFIRMED
    assert order.confirmed_at is not None
```

**Characteristics:**
- Tests verify observable behaviour, not internal wiring — more resilient to refactor.
- Requires good fakes (in-memory implementations) for dependencies.
- Can be harder to know where to start without an upfront design sketch.

---

### Common TDD Pitfalls

**Over-mocking:**
```python
# BAD: mocking internal helper functions
# If refactored, test breaks — but behaviour may be unchanged
mocker.patch("module._compute_discount")   # too internal

# GOOD: mock external boundaries only
mocker.patch("stripe.Charge.create")       # external service
```

**Testing implementation, not behaviour:**
```python
# BAD: asserts on implementation detail
assert order._discount_calculator.called   # who cares how it's computed?

# GOOD: asserts on observable output
assert order.total == 90.00                # discounted price is correct
```

**Test that is too large (testing too much at once):**  
Each test should verify one behaviour. Long tests with many assertions usually mean
too-large a production increment — break the Red-Green-Refactor cycle into smaller steps.

**Skipping refactor step:**  
If refactoring never happens, TDD produces green tests with messy code. Refactoring is
mandatory, not optional. "Make it work, make it right, make it fast."

---

### Applying TDD to ML Code

ML code is often written exploratorily in notebooks, which makes TDD harder — but
the deterministic parts (data transforms, feature engineering, postprocessing) benefit
greatly from TDD.

```python
# Feature engineering is pure functions → easy to TDD

def test_age_bucket_returns_correct_label():
    assert age_bucket(17) == "under_18"
    assert age_bucket(25) == "18_35"
    assert age_bucket(60) == "36_60"
    assert age_bucket(70) == "60_plus"

def age_bucket(age: int) -> str:
    if age < 18:  return "under_18"
    if age < 36:  return "18_35"
    if age < 61:  return "36_60"
    return "60_plus"

# Model contracts: test input/output shapes and types
def test_model_output_shape_and_range(trained_model):
    x = np.random.rand(16, 10).astype(np.float32)
    preds = trained_model.predict(x)
    assert preds.shape == (16, 1)
    assert (preds >= 0).all() and (preds <= 1).all()  # probabilities

# Data pipeline: test row count invariants
def test_pipeline_preserves_non_null_rows(raw_df):
    result = preprocess(raw_df)
    assert len(result) == raw_df["target"].notna().sum()
    assert result["target"].notna().all()
```

---

### BDD with pytest-bdd

Behaviour-Driven Development extends TDD outward to business language. Scenarios are
written in Gherkin (`Given / When / Then`) and linked to pytest step functions.

```gherkin
# features/order.feature
Feature: Order placement

  Scenario: Customer places a valid order
    Given a customer with a registered account
    And a product "Widget A" costs 10.00
    When the customer orders 3 of "Widget A"
    Then the order total should be 30.00
    And the customer receives a confirmation email
```

```python
# tests/test_order_bdd.py
from pytest_bdd import scenarios, given, when, then, parsers

scenarios("features/order.feature")

@given("a customer with a registered account")
def customer(make_user):
    return make_user(email="bdd@example.com")

@given(parsers.parse('a product "{name}" costs {price:f}'))
def product(name, price):
    return Product(name=name, price=price)

@when(parsers.parse("the customer orders {qty:d} of \"{name}\""))
def place_order(customer, product, qty, name):
    customer.order(product, qty=qty)

@then(parsers.parse("the order total should be {total:f}"))
def check_total(customer, total):
    assert customer.last_order.total == total
```

BDD is most valuable when domain experts (product managers, QA) can read and
contribute to the feature files. Avoid BDD for pure technical behaviour.

## Implementation Notes

### pytest Plugins Useful for TDD

```bash
pip install pytest-watch pytest-xdist pytest-cov pytest-bdd

# pytest-watch: auto-rerun on file change (tight feedback loop)
ptw -- -x -q

# pytest-xdist: parallel test execution
pytest -n auto

# Run only tests that failed last time, then all
pytest --last-failed --last-failed-no-failures all
```

### Test Naming Convention

```python
# Pattern: test_<unit>_<condition>_<expected>
def test_discount_applied_when_coupon_is_valid(): ...
def test_order_raises_when_product_is_out_of_stock(): ...
def test_api_returns_401_when_token_is_expired(): ...
```

Readable names mean test output is self-documenting on failure — no need to open
the test file to understand what broke.

## Trade-offs

| School | Design pressure | Test resilience | Speed |
|---|---|---|---|
| London (mockist) | High — forces explicit interfaces | Brittle to refactoring | Fast |
| Chicago (classicist) | Moderate | Resilient to refactoring | Moderate |

- **Start with Chicago** for domain logic; switch to London at system boundaries
  (HTTP, external APIs).
- **TDD is harder on existing code** — legacy code without tests requires
  characterisation tests and careful seam identification before TDD can take hold.
- **Not all code benefits equally**: algorithmic and domain code → high TDD value;
  glue/infrastructure code → lower ROI.

## References

- *Test-Driven Development by Example* — Kent Beck
- *Growing Object-Oriented Software Guided by Tests* — Freeman & Pryce (London school)
- *Working Effectively with Legacy Code* — Michael Feathers
- [pytest-bdd documentation](https://pytest-bdd.readthedocs.io/)
- [Martin Fowler — Mocks Aren't Stubs](https://martinfowler.com/articles/mocksArentStubs.html)

## Links
- [[testing_strategies|Testing Strategies]]
- [[property_based_testing|Property-Based Testing]]
- [[solid_principles|SOLID Principles]]
