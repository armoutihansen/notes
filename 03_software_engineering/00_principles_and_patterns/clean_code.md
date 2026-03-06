---
layer: 03_software_engineering
type: concept
status: growing
tags: [design, readability, maintainability, principles]
created: 2026-03-05
---

# Clean Code

## Definition

Clean code is code that is easy to read, understand, and modify by any competent developer — including yourself six months from now. It minimises cognitive load through clarity, consistency, and restraint. The term is closely associated with Robert C. Martin's *Clean Code* (2008), but the principles draw on decades of software craft literature.

> "Any fool can write code that a computer can understand. Good programmers write code that humans can understand." — Martin Fowler

Clean code is not:
- Clever code
- The most concise code
- The most performant code (performance is a separate concern)

It *is* code where intent is obvious, structure is logical, and change is low-risk.

---

## Intuition

Think of code as a series of explanations to the next developer. Every name, function, and module is a sentence. Clean code is prose that happens to be executable.

Heuristics:
- If you need a comment to explain *what* something does, the name is wrong
- If you can't summarise a function in one sentence, it does too much
- If you flinch before modifying a module, it's too coupled
- If you read a function top-to-bottom and are surprised at the end, it has side effects

---

## Formal Description

### 1. Meaningful Names

Names are the primary communication channel in code. A good name eliminates the need for comments.

**Principles:**
- Use intention-revealing names (`elapsed_time_in_days` not `d`)
- Avoid disinformation (`account_list` should be a list, not a dict)
- Make meaningful distinctions (`process_user_data` and `handle_user_data` is noise)
- Use pronounceable names (aids code review discussions)
- Use searchable names — avoid magic numbers; name constants
- Avoid encodings (`m_name`, `strName` — unnecessary in typed Python)
- Names for classes: noun or noun phrase (`UserRepository`, `InvoiceParser`)
- Names for functions: verb or verb phrase (`save_to_disk`, `calculate_tax`)
- Names for booleans: predicate form (`is_active`, `has_permission`, `should_retry`)

```python
# Unclear
def calc(d, r):
    return d * (1 - r)

# Clear
def apply_discount(price: float, discount_rate: float) -> float:
    return price * (1 - discount_rate)
```

---

### 2. Small Functions That Do One Thing

Functions should do one thing, do it well, and do it only. "One thing" means one level of abstraction.

**Signs a function does too much:**
- The name contains "and", "or", "then"
- It has many local variables
- It is more than ~20 lines
- It has nested conditionals deeper than 2 levels

```python
# Doing too much
def process_order(order):
    if order["status"] == "pending":
        total = sum(i["price"] * i["qty"] for i in order["items"])
        if total > 100:
            total *= 0.9
        order["total"] = total
        order["status"] = "confirmed"
        db.save(order)
        email.send(order["user"], "Your order is confirmed")

# Split into focused functions
def calculate_total(items: list) -> float:
    subtotal = sum(item["price"] * item["qty"] for item in items)
    return apply_bulk_discount(subtotal)

def apply_bulk_discount(total: float, threshold: float = 100.0) -> float:
    return total * 0.9 if total > threshold else total

def confirm_order(order: dict) -> dict:
    return {**order, "total": calculate_total(order["items"]), "status": "confirmed"}

def process_order(order: dict) -> None:
    if order["status"] != "pending":
        return
    confirmed = confirm_order(order)
    db.save(confirmed)
    email.send(confirmed["user"], "Your order is confirmed")
```

---

### 3. Avoid Deep Nesting

Deeply nested code is hard to follow. Common techniques to flatten:

**Early return (guard clauses):**
```python
# Deep nesting
def process(user):
    if user is not None:
        if user.is_active:
            if user.has_permission("write"):
                do_work(user)

# Flat with guard clauses
def process(user):
    if user is None:
        return
    if not user.is_active:
        return
    if not user.has_permission("write"):
        return
    do_work(user)
```

**Extract conditionals into named predicates:**
```python
def can_write(user) -> bool:
    return user is not None and user.is_active and user.has_permission("write")

def process(user):
    if not can_write(user):
        return
    do_work(user)
```

---

### 4. Avoid Side Effects

A function has a side effect when it modifies state outside its own scope (global state, mutable arguments, I/O) beyond what its name implies. Side effects make code hard to test and reason about.

```python
# Hidden side effect — modifies input in place, surprising to callers
def get_full_name(user: dict) -> str:
    full = f"{user['first']} {user['last']}"
    user["full_name"] = full    # side effect!
    return full

# Pure: same input always produces same output, no mutation
def get_full_name(user: dict) -> str:
    return f"{user['first']} {user['last']}"
```

**Principle of least surprise**: a function should do exactly what its name says and nothing more. If a function must have side effects (e.g., `save_to_db`), the name should make that explicit.

---

### 5. DRY vs DAMP

**DRY (Don't Repeat Yourself):** Every piece of knowledge should have a single, unambiguous, authoritative representation.

**DAMP (Descriptive and Meaningful Phrases):** In tests, some repetition is acceptable to keep each test self-contained and readable.

```python
# DRY: extract shared logic
def format_currency(amount: float, currency: str = "USD") -> str:
    return f"{currency} {amount:,.2f}"

# DAMP in tests: each test stands alone, even if there's some repetition
def test_vip_discount():
    user = User(name="Alice", tier="VIP")    # repeated setup
    order = Order(user=user, total=200.0)
    assert order.discounted_total() == 160.0

def test_standard_discount():
    user = User(name="Bob", tier="standard") # repeated setup — acceptable
    order = Order(user=user, total=200.0)
    assert order.discounted_total() == 190.0
```

**Tension**: DRY is about knowledge, not code. Two identical-looking blocks of code that represent different business rules should *not* be merged — they will evolve independently.

---

### 6. Comments: When and When Not To

**Don't use comments to explain bad code; rewrite the code.**

Comments that add value:
- Legal comments (copyright, licence)
- Explanation of *why* (not what) — non-obvious business rules, algorithmic choices
- Warning of consequences (`# not thread-safe`)
- TODO notes with ticket references

Comments that are noise:
```python
# BAD: restates the obvious
i = i + 1  # increment i

# BAD: commented-out code (use git)
# old_total = sum(prices)

# GOOD: explains non-obvious domain constraint
# VAT is applied before the loyalty discount per tax regulation §14b
total = apply_vat(subtotal)
final = apply_loyalty_discount(total, user.tier)
```

---

### 7. Error Handling

- Use exceptions, not error codes
- Don't return `None` to signal failure when an exception is appropriate
- Don't swallow exceptions silently (`except: pass`)
- Fail fast: validate at system boundaries, not deep in business logic

```python
# Unclear failure mode
def find_user(user_id: int) -> dict | None:
    ...

# Explicit failure contract
class UserNotFoundError(ValueError):
    pass

def find_user(user_id: int) -> dict:
    user = db.get(user_id)
    if user is None:
        raise UserNotFoundError(f"No user with id={user_id}")
    return user
```

---

### 8. Code Organisation and Structure

- **Stepdown rule**: Code should read top-to-bottom like a newspaper — high-level abstractions first, details below
- **Vertical distance**: Related concepts should be near each other; unrelated things separated
- **Horizontal alignment**: Avoid aligning variable assignments — it creates fragile formatting
- **Module cohesion**: A module should contain things that change together

---

## Practical Heuristics (Quick Reference)

| Smell | Fix |
|-------|-----|
| Long function | Extract function |
| Long parameter list | Introduce parameter object or dataclass |
| Duplicate code | Extract common abstraction |
| Dead code | Delete it (git has history) |
| Magic number | Named constant |
| Boolean parameter | Split into two functions |
| Deeply nested ifs | Guard clauses or extract predicate |
| Comment explains what | Rename |
| Comment explains why | Keep the comment |
| Mutable global state | Dependency injection |

---

## Trade-offs

- **Clarity vs brevity**: Python allows extremely terse code (comprehensions, functional chains). Lean toward clarity; terseness helps only for idioms the reader will immediately recognise.
- **Abstraction vs premature design**: Extracting functions and classes too early adds indirection without benefit. Clean code and YAGNI are compatible — extract when duplication appears (Rule of Three).
- **DRY vs flexibility**: Merging similar-but-distinct code to avoid duplication can create tight coupling between unrelated requirements.
- **Small functions vs call-stack depth**: Very small functions increase the call stack and make debugging harder. Aim for functions where the abstraction is real, not just mechanical splitting.

## Links
- [[solid_principles|SOLID Principles]] — Clean code at the function/class level; SOLID at the module/architecture level
- [[design_patterns|Design Patterns]] — Patterns give structure; clean code makes them readable
