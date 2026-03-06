---
layer: 03_software_engineering
type: engineering
tool: Python
status: growing
tags: [workflow]
created: 2026-03-05
---

# Python Tooling

## Purpose

A reference for the modern Python tooling ecosystem: package management, virtual environments, project layout, type checking, linting, formatting, and testing. Includes patterns for ML/DS projects where dependency management and reproducibility are critical.

---

## Architecture

A modern Python project has a clear separation of concerns across tooling layers:

```
Project
├── pyproject.toml          ← single config file for all tools
├── src/
│   └── mypackage/
│       ├── __init__.py
│       └── ...
├── tests/
│   ├── conftest.py
│   └── test_*.py
├── .venv/                  ← local virtual environment (git-ignored)
└── uv.lock / poetry.lock   ← pinned dependency versions
```

---

## Implementation Notes

### Package Management

#### pip (standard library)
```bash
pip install requests
pip install -r requirements.txt
pip install -e .          # editable install (for development)
pip freeze > requirements.txt
```

#### uv (recommended for speed)
[uv](https://github.com/astral-sh/uv) is a Rust-based pip/virtualenv replacement, 10-100× faster. Drop-in compatible.

```bash
# Install uv
curl -LsSf https://astral.sh/uv/install.sh | sh

# Create and sync environment from pyproject.toml
uv sync

# Add a dependency
uv add requests
uv add --dev pytest ruff mypy

# Run in the project env without activating
uv run python script.py
uv run pytest

# Lock file for reproducibility
uv lock                   # generates uv.lock
uv sync --frozen          # install exactly what's in the lock file
```

#### poetry (full project management)
```bash
poetry new myproject      # creates project scaffold
poetry add requests       # adds to [tool.poetry.dependencies]
poetry add --group dev pytest mypy ruff
poetry install            # creates .venv and installs
poetry run pytest         # run in poetry env
poetry build              # build wheel + sdist
poetry publish            # publish to PyPI
```

---

### Virtual Environments

Isolate project dependencies from the system Python. Never install packages into the system interpreter.

```bash
# stdlib venv
python -m venv .venv
source .venv/bin/activate    # Linux/Mac
.venv\Scripts\activate       # Windows
deactivate

# uv (creates .venv automatically)
uv sync

# conda (for ML projects with non-Python dependencies like CUDA libs)
conda create -n myenv python=3.11
conda activate myenv
conda install pytorch torchvision -c pytorch
```

---

### Project Structure: src Layout

The `src/` layout prevents importing the package directly from the repo root during development, ensuring the installed version is always tested.

```
myproject/
├── pyproject.toml
├── README.md
├── src/
│   └── mypackage/
│       ├── __init__.py
│       ├── models.py
│       └── utils.py
└── tests/
    ├── conftest.py
    ├── test_models.py
    └── test_utils.py
```

`pyproject.toml` minimum for src layout:
```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.hatch.build.targets.wheel]
packages = ["src/mypackage"]
```

---

### pyproject.toml — Unified Configuration

```toml
[project]
name = "mypackage"
version = "0.1.0"
description = "Example project"
requires-python = ">=3.11"
dependencies = [
    "requests>=2.31",
    "pydantic>=2.0",
]

[project.optional-dependencies]
dev = ["pytest", "pytest-cov", "ruff", "mypy"]
ml  = ["torch", "numpy", "pandas"]

# ── Type checking ─────────────────────────────────
[tool.mypy]
python_version = "3.11"
strict = true
ignore_missing_imports = true

# ── Linting + formatting ──────────────────────────
[tool.ruff]
line-length = 88
target-version = "py311"

[tool.ruff.lint]
select = ["E", "F", "I", "N", "UP", "ANN", "B", "SIM"]
ignore = ["ANN101", "ANN102"]

[tool.ruff.format]
quote-style = "double"

# ── Testing ───────────────────────────────────────
[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "--strict-markers -q"

[tool.coverage.report]
exclude_lines = ["if TYPE_CHECKING:", "pragma: no cover"]
```

---

### Type Checking with mypy

```bash
mypy src/mypackage/          # check package
mypy --strict src/mypackage/ # strictest settings
mypy --ignore-missing-imports src/  # skip untyped third-party libs
```

Key mypy settings:
```toml
[tool.mypy]
strict = true              # enables all strict flags
# Equivalent to:
#   disallow_untyped_defs = true
#   disallow_any_generics = true
#   warn_return_any = true
#   no_implicit_optional = true
#   warn_unused_ignores = true
```

Common patterns:
```python
from __future__ import annotations   # PEP 563: postponed evaluation of annotations
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from mypackage.models import User   # avoid circular imports at runtime

# Type narrowing
def process(value: int | str) -> str:
    if isinstance(value, int):
        return str(value)   # mypy knows value is int here
    return value            # mypy knows value is str here

# TypedDict for structured dicts
from typing import TypedDict

class Config(TypedDict):
    host: str
    port: int
    debug: bool
```

---

### Linting with ruff

`ruff` replaces flake8, isort, pyupgrade, and many other tools in a single fast binary.

```bash
ruff check .                 # lint
ruff check --fix .           # auto-fix safe issues
ruff format .                # format (replaces black)
ruff format --check .        # check formatting without changing files
```

Most useful rule sets to enable:

| Code | Plugin | What it catches |
|------|--------|----------------|
| `E`, `W` | pycodestyle | PEP 8 style |
| `F` | Pyflakes | unused imports, undefined names |
| `I` | isort | import order |
| `N` | pep8-naming | naming conventions |
| `UP` | pyupgrade | deprecated syntax |
| `B` | flake8-bugbear | likely bugs |
| `SIM` | flake8-simplify | simplifiable code |
| `ANN` | flake8-annotations | missing type annotations |

---

### Formatting with black (or ruff format)

`ruff format` is now the recommended alternative to `black` — same style, faster.

```bash
black src/ tests/       # format in place
black --check src/      # check only
```

black/ruff-format is intentionally opinionated: minimal configuration, consistent output. This eliminates formatting debates in code review.

---

### Testing with pytest

```bash
pytest                          # run all tests
pytest tests/test_models.py     # single file
pytest -k "test_create"         # filter by name
pytest -x                       # stop on first failure
pytest --cov=src --cov-report=html   # coverage report
pytest -v                       # verbose
pytest --tb=short               # shorter tracebacks
```

**Test structure:**
```python
# tests/conftest.py — shared fixtures
import pytest
from mypackage.database import Database

@pytest.fixture(scope="session")
def db():
    database = Database(":memory:")
    database.create_tables()
    yield database
    database.close()

@pytest.fixture
def clean_db(db):
    yield db
    db.rollback()   # reset state after each test
```

```python
# tests/test_models.py
import pytest
from mypackage.models import User, UserNotFoundError

class TestUserCreation:
    def test_creates_with_valid_data(self, clean_db):
        user = User.create(clean_db, name="Alice", email="alice@example.com")
        assert user.id is not None
        assert user.name == "Alice"

    def test_raises_on_duplicate_email(self, clean_db):
        User.create(clean_db, name="Alice", email="alice@example.com")
        with pytest.raises(ValueError, match="email already exists"):
            User.create(clean_db, name="Alice2", email="alice@example.com")

    @pytest.mark.parametrize("name,email", [
        ("", "a@b.com"),
        ("Alice", "not-an-email"),
        ("a" * 300, "a@b.com"),   # name too long
    ])
    def test_rejects_invalid_data(self, clean_db, name, email):
        with pytest.raises(ValueError):
            User.create(clean_db, name=name, email=email)
```

**Mocking:**
```python
from unittest.mock import AsyncMock, MagicMock, patch

def test_sends_email_on_confirm(clean_db):
    with patch("mypackage.services.email_client") as mock_email:
        order = Order.create(clean_db, items=[...])
        order.confirm()
        mock_email.send.assert_called_once_with(
            order.user.email, subject="Order confirmed"
        )

@pytest.mark.asyncio   # requires pytest-asyncio
async def test_async_fetch():
    mock_client = AsyncMock()
    mock_client.get.return_value = {"data": 42}
    result = await fetch_data(mock_client, "key")
    assert result == 42
```

---

### ML/DS Python Project Patterns

**Dependency separation:**
```toml
[project.optional-dependencies]
train = ["torch", "transformers", "accelerate", "datasets"]
serve = ["fastapi", "uvicorn", "onnxruntime"]
dev   = ["pytest", "ruff", "mypy", "jupyter"]
```

**Reproducibility:**
```bash
# Pin all dependencies for training reproducibility
uv lock                       # lock exact versions
uv sync --frozen              # install exactly the lock file (CI)

# Or use conda for CUDA version pinning
conda env export > environment.yml
conda env create -f environment.yml
```

**Typical src layout for ML projects:**
```
src/myproject/
├── __init__.py
├── config.py          # pydantic Settings
├── data/
│   ├── dataset.py
│   └── transforms.py
├── models/
│   ├── base.py
│   └── transformer.py
├── training/
│   ├── trainer.py
│   └── callbacks.py
├── evaluation/
│   └── metrics.py
└── serving/
    └── api.py
```

**Configuration with pydantic-settings:**
```python
from pydantic_settings import BaseSettings

class TrainingConfig(BaseSettings):
    model_name: str = "bert-base-uncased"
    batch_size: int = 32
    learning_rate: float = 3e-5
    max_epochs: int = 10
    device: str = "cuda"

    class Config:
        env_prefix = "TRAIN_"   # reads TRAIN_BATCH_SIZE etc from env

cfg = TrainingConfig()           # from defaults
cfg = TrainingConfig(batch_size=16)  # override programmatically
# Or: TRAIN_BATCH_SIZE=16 python train.py
```

---

## Trade-offs

| Tool choice | Advantage | Caveat |
|-------------|-----------|--------|
| `uv` over `pip` | 10-100× faster resolution and install | Newer, smaller ecosystem of integrations |
| `poetry` over `uv` | Richer publishing workflow | Slower; poetry lock format is different |
| `ruff format` over `black` | Faster, single tool | Slightly different output in edge cases |
| `src/` layout | Prevents import confusion; accurate testing | Slightly more setup; less familiar to beginners |
| `strict` mypy | Catches many bugs at type-check time | Requires complete annotations; can slow initial development |
| `pytest` fixtures | Composable, scope-controlled setup/teardown | Learning curve; easy to over-fixture simple tests |
| conda over venv | Manages non-Python deps (CUDA, MKL) | Heavier; slower than venv for pure-Python projects |

---

## References

- [uv documentation](https://docs.astral.sh/uv/)
- [ruff documentation](https://docs.astral.sh/ruff/)
- [pytest documentation](https://docs.pytest.org/)
- [mypy documentation](https://mypy.readthedocs.io/)
- [pyproject.toml specification](https://packaging.python.org/en/latest/specifications/pyproject-toml/)
- [Hypermodern Python](https://cjolowicz.github.io/posts/hypermodern-python-01-setup/) — Claudio Jolowicz

---

## Links
- [[python_core_language|Core Language]] — understanding the language is prerequisite to tooling well
