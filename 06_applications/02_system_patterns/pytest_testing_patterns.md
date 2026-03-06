---
layer: 06_applications
type: application
status: growing
tags: [pytest, testing, fixtures, parametrize, mocking, async, coverage]
created: 2026-03-06
---

# pytest Testing Patterns

## Purpose

Practical pytest patterns for building a robust test suite: fixture composition, parametrize, test doubles (mocker), async test support, and CI coverage integration. Synthesized from: [[testing_strategies|Testing Strategies]], [[test_driven_development|TDD]], and [[property_based_testing|Property-Based Testing]].

### Examples

**Project layout:**
```
src/
    myapp/
        service.py
        models.py
tests/
    conftest.py          ← shared fixtures
    unit/
        test_service.py
    integration/
        test_db.py
pyproject.toml           ← [tool.pytest.ini_options]
```

**`pyproject.toml` pytest config:**
```toml
[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "-v --tb=short --strict-markers -p no:warnings"
markers = [
    "integration: mark as integration test",
    "slow: mark as slow test",
]

[tool.coverage.run]
source = ["src"]
omit = ["tests/*"]

[tool.coverage.report]
fail_under = 80
```

**Fixture hierarchy (`conftest.py`):**
```python
import pytest
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from myapp.db import Base

# Session-scoped: create once per test session
@pytest.fixture(scope="session")
def event_loop_policy():
    return asyncio.DefaultEventLoopPolicy()

# Function-scoped (default): fresh DB per test
@pytest.fixture
async def db_session():
    engine = create_async_engine("sqlite+aiosqlite:///:memory:")
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    async with AsyncSession(engine) as session:
        yield session
    await engine.dispose()

# Dependency injection override
@pytest.fixture
def app(db_session):
    from myapp.main import app
    from myapp.db import get_session
    app.dependency_overrides[get_session] = lambda: db_session
    yield app
    app.dependency_overrides.clear()
```

**Parametrize — table-driven tests:**
```python
import pytest

@pytest.mark.parametrize("input,expected", [
    ("hello world",   2),
    ("",              0),
    ("one",           1),
    ("a  b  c",       3),   # multiple spaces
])
def test_word_count(input, expected):
    assert word_count(input) == expected

# Parametrize across multiple dimensions
@pytest.mark.parametrize("model", ["gpt-4o", "claude-3-5"])
@pytest.mark.parametrize("temperature", [0.0, 0.7])
def test_generation(model, temperature, mock_client):
    ...
```

**Test doubles with `pytest-mock`:**
```python
def test_service_calls_repository(mocker):
    # Patch at the use-site, not the definition site
    mock_repo = mocker.patch("myapp.service.UserRepository")
    mock_repo.return_value.find_by_email.return_value = User(id=1, email="a@b.com")

    svc = UserService()
    result = svc.get_by_email("a@b.com")

    mock_repo.return_value.find_by_email.assert_called_once_with("a@b.com")
    assert result.id == 1
```

**Async tests (pytest-asyncio):**
```python
import pytest

@pytest.mark.asyncio
async def test_async_endpoint(app):
    async with AsyncClient(app=app, base_url="http://test") as client:
        resp = await client.get("/health")
    assert resp.status_code == 200
```

**`pytest-asyncio` mode config (pyproject.toml):**
```toml
[tool.pytest.ini_options]
asyncio_mode = "auto"   # auto-detect async tests; no @pytest.mark.asyncio needed
```

**Coverage in CI (GitHub Actions):**
```yaml
- name: Run tests with coverage
  run: pytest --cov=src --cov-report=xml --cov-report=term-missing

- uses: codecov/codecov-action@v4
  with:
    token: ${{ secrets.CODECOV_TOKEN }}
    fail_ci_if_error: true
```

## Architecture

```
Tests
├── conftest.py        ← shared fixtures (session / module / function scope)
├── unit/              ← fast, isolated, no I/O
│   └── test_*.py
├── integration/       ← real DB or external service
│   └── test_*.py
└── e2e/               ← full stack (optional, slowest)
    └── test_*.py

Coverage gate → CI: must be ≥ 80% before merge
```

**Fixture scope hierarchy:** `session` > `module` > `class` > `function`. Use the narrowest scope that permits the test to be deterministic. Session-scoped fixtures (heavy DB setup, loaded models) amortize cost across the test suite.

**Marker strategy:**
- `@pytest.mark.integration` — skip in fast unit CI run; run in full CI and pre-merge
- `@pytest.mark.slow` — skip by default in dev (`pytest -m "not slow"`)
- `@pytest.mark.parametrize` — prefer over copy-paste; keep fixture setup outside the parametrized loop

## Links
- [[testing_strategies|Testing Strategies]]
- [[test_driven_development|TDD]]
- [[property_based_testing|Property-Based Testing]]
- [[cicd_pipelines|CI/CD Pipelines]]
- [[github_actions|GitHub Actions]]
