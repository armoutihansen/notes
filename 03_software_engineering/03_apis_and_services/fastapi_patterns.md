---
layer: 03_software_engineering
type: engineering
tool: FastAPI
status: growing
tags: [fastapi, python, api, async, pydantic, dependency-injection, ml-serving]
created: 2026-03-05
---

# FastAPI Patterns

## Purpose

FastAPI is a modern Python web framework for building APIs with automatic OpenAPI documentation, type-driven request/response validation via Pydantic, and native async support. It is the go-to framework for ML model serving, data-intensive APIs, and any Python backend where developer speed and performance matter. FastAPI's design philosophy: use Python type hints everywhere; the framework derives validation, serialization, and documentation from those annotations.

## Architecture

### Core Framework Design

FastAPI is built on two foundational libraries:
- **Starlette**: ASGI web framework handling routing, middleware, WebSockets, background tasks
- **Pydantic**: data validation and serialization library; v2 (Rust-core) is the current default

The ASGI (Asynchronous Server Gateway Interface) foundation means FastAPI runs on async-capable servers: **Uvicorn** (single-process) or **Gunicorn + Uvicorn workers** (multi-process production).

**Request lifecycle**:
```
Client → Uvicorn → Middleware stack → Router → Dependencies → Route handler → Response model serialization → Client
```

### Path and Query Parameters

```python
from fastapi import FastAPI, Path, Query
from typing import Annotated

app = FastAPI()

@app.get("/users/{user_id}/posts")
async def list_user_posts(
    user_id: Annotated[int, Path(gt=0, description="The user ID")],
    limit: Annotated[int, Query(ge=1, le=100)] = 20,
    cursor: str | None = None,
):
    ...
```

- Path parameters: part of the URL path; always required
- Query parameters: after `?`; optional if they have a default value
- `Annotated[type, Field(...)]` pattern keeps validation metadata co-located with type
- FastAPI validates types, ranges, and patterns before the function is called; returns `422` on failure

### Pydantic Models

```python
from pydantic import BaseModel, Field, EmailStr, model_validator
from datetime import datetime
from typing import Literal

class UserCreate(BaseModel):
    email: EmailStr
    name: str = Field(min_length=1, max_length=100)
    role: Literal["admin", "user", "viewer"] = "user"

class UserResponse(BaseModel):
    id: int
    email: EmailStr
    name: str
    created_at: datetime

    model_config = {"from_attributes": True}  # enables ORM mode

@app.post("/users", response_model=UserResponse, status_code=201)
async def create_user(body: UserCreate):
    ...
```

- Input models (`UserCreate`) validate and parse incoming JSON
- Response models (`UserResponse`) filter and serialize outgoing data — fields not in the model are excluded
- `from_attributes=True` allows Pydantic to read attributes from ORM objects (SQLAlchemy rows)
- Use separate models for input vs output to avoid accidentally exposing internal fields (e.g., password hash)

**Nested models and validation**:
```python
class Address(BaseModel):
    street: str
    city: str
    country: str = Field(min_length=2, max_length=2)  # ISO 3166-1 alpha-2

class OrderCreate(BaseModel):
    items: list[ItemCreate] = Field(min_length=1)
    shipping_address: Address

    @model_validator(mode="after")
    def validate_total(self) -> "OrderCreate":
        if sum(i.quantity for i in self.items) > 1000:
            raise ValueError("Order exceeds maximum item count")
        return self
```

## Implementation Notes

### Dependency Injection

FastAPI's DI system is one of its most powerful features. Dependencies are declared as function parameters with `Depends(...)` and are automatically resolved, cached, and composed.

```python
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials

security = HTTPBearer()

async def get_current_user(
    credentials: Annotated[HTTPAuthorizationCredentials, Depends(security)],
    db: Annotated[AsyncSession, Depends(get_db)],
) -> User:
    token = credentials.credentials
    user_id = verify_jwt(token)  # raises if invalid
    user = await db.get(User, user_id)
    if not user:
        raise HTTPException(status_code=status.HTTP_401_UNAUTHORIZED)
    return user

@app.get("/me", response_model=UserResponse)
async def get_me(current_user: Annotated[User, Depends(get_current_user)]):
    return current_user
```

**Database session dependency** (SQLAlchemy async):
```python
from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine, async_sessionmaker

engine = create_async_engine(settings.DATABASE_URL, pool_size=10, max_overflow=20)
AsyncSessionLocal = async_sessionmaker(engine, expire_on_commit=False)

async def get_db() -> AsyncGenerator[AsyncSession, None]:
    async with AsyncSessionLocal() as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise
```

**Dependency scoping**: by default, a dependency is called once per request and cached within the request. Pass `use_cache=False` to `Depends` to call it fresh each time.

**Class-based dependencies**:
```python
class Pagination:
    def __init__(self, limit: int = 20, cursor: str | None = None):
        self.limit = min(limit, 100)
        self.cursor = cursor

@app.get("/posts")
async def list_posts(pagination: Annotated[Pagination, Depends()]):
    ...
```

### Async Request Handlers

- Use `async def` for I/O-bound handlers (database queries, HTTP calls, file reads)
- Use `def` for CPU-bound handlers — FastAPI runs these in a thread pool to avoid blocking the event loop
- Never call blocking I/O (synchronous `requests`, `time.sleep`, synchronous ORM queries) inside `async def` — use `asyncio.to_thread` or run in executor

```python
import asyncio
import httpx

@app.get("/external-data")
async def fetch_external(client: Annotated[httpx.AsyncClient, Depends(get_http_client)]):
    response = await client.get("https://api.example.com/data")
    return response.json()
```

### Background Tasks

```python
from fastapi import BackgroundTasks

def send_welcome_email(email: str, name: str):
    # Runs after response is sent; does not block the response
    email_client.send(to=email, subject=f"Welcome, {name}!")

@app.post("/users", status_code=201)
async def create_user(body: UserCreate, background_tasks: BackgroundTasks):
    user = await user_service.create(body)
    background_tasks.add_task(send_welcome_email, user.email, user.name)
    return user
```

For heavier background work (retries, durability, distributed): use Celery + Redis/RabbitMQ or ARQ.

### Lifespan Events

Replaced deprecated `@app.on_event("startup")` in FastAPI 0.93+:

```python
from contextlib import asynccontextmanager

@asynccontextmanager
async def lifespan(app: FastAPI):
    # Startup: runs before first request
    await database.connect()
    ml_model = load_model("model.pkl")
    app.state.model = ml_model
    yield
    # Shutdown: runs after last request
    await database.disconnect()

app = FastAPI(lifespan=lifespan)
```

Use `app.state` to share objects (models, clients, connection pools) across requests.

### Middleware

```python
from fastapi.middleware.cors import CORSMiddleware
from starlette.middleware.base import BaseHTTPMiddleware
import time

app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://app.example.com"],
    allow_methods=["*"],
    allow_headers=["*"],
    allow_credentials=True,
)

class LoggingMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request, call_next):
        start = time.perf_counter()
        response = await call_next(request)
        duration_ms = (time.perf_counter() - start) * 1000
        logger.info(
            "request",
            method=request.method,
            path=request.url.path,
            status=response.status_code,
            duration_ms=round(duration_ms, 2),
        )
        return response

app.add_middleware(LoggingMiddleware)
```

Middleware is applied in LIFO order (last added = first executed on request, last on response).

### Structuring Large FastAPI Apps

```
src/
├── main.py                  # app = FastAPI(lifespan=lifespan); include routers
├── api/
│   ├── v1/
│   │   ├── users.py         # APIRouter for /users
│   │   ├── orders.py        # APIRouter for /orders
│   │   └── __init__.py
│   └── deps.py              # Shared dependencies (get_db, get_current_user)
├── models/                  # SQLAlchemy ORM models
├── schemas/                 # Pydantic request/response models
├── services/                # Business logic layer (no HTTP concerns)
├── repositories/            # Data access layer (DB queries)
└── core/
    ├── config.py            # Settings via pydantic-settings
    └── security.py          # JWT encoding/decoding
```

**Router registration**:
```python
from fastapi import APIRouter
from api.v1 import users, orders

api_v1 = APIRouter(prefix="/api/v1")
api_v1.include_router(users.router, prefix="/users", tags=["users"])
api_v1.include_router(orders.router, prefix="/orders", tags=["orders"])
app.include_router(api_v1)
```

**Layered architecture**:
- Route handler: validate input, call service, return response model
- Service: business logic, orchestration, does not know about HTTP
- Repository: SQL queries, returns domain objects or None

### Testing with TestClient and pytest

```python
from fastapi.testclient import TestClient
from httpx import AsyncClient
import pytest_asyncio

# Synchronous (suitable for most unit/integration tests)
client = TestClient(app)

def test_create_user():
    response = client.post("/api/v1/users", json={"email": "a@b.com", "name": "Alice"})
    assert response.status_code == 201
    assert response.json()["email"] == "a@b.com"

# Async (for tests that need async fixtures or async operations)
@pytest_asyncio.fixture
async def async_client():
    async with AsyncClient(app=app, base_url="http://test") as ac:
        yield ac

async def test_list_users(async_client):
    response = await async_client.get("/api/v1/users")
    assert response.status_code == 200
```

**Override dependencies in tests**:
```python
from main import app
from api.deps import get_db

def get_test_db():
    yield test_session  # use in-memory SQLite or test Postgres

app.dependency_overrides[get_db] = get_test_db
```

### ML Model Serving Patterns

```python
# Load model once at startup, share via app.state
@asynccontextmanager
async def lifespan(app: FastAPI):
    app.state.model = load_model(settings.MODEL_PATH)
    app.state.tokenizer = load_tokenizer(settings.TOKENIZER_PATH)
    yield

class PredictRequest(BaseModel):
    text: str = Field(min_length=1, max_length=10_000)

class PredictResponse(BaseModel):
    label: str
    score: float
    latency_ms: float

@app.post("/predict", response_model=PredictResponse)
async def predict(
    body: PredictRequest,
    request: Request,
):
    model = request.app.state.model
    start = time.perf_counter()
    # CPU-bound: run in thread pool to avoid blocking event loop
    result = await asyncio.to_thread(model.predict, body.text)
    latency_ms = (time.perf_counter() - start) * 1000
    return PredictResponse(label=result["label"], score=result["score"], latency_ms=latency_ms)
```

**Batching** for throughput: collect requests for 10–50 ms, process as a batch, return results. Implement with `asyncio.Queue` + a background consumer task.

**Model versioning**: expose multiple model versions via path params (`/v1/predict`, `/v2/predict`) or a `model_version` query param; load models into a dict keyed by version.

## Trade-offs

| Choice | Pro | Con |
|--------|-----|-----|
| `async def` handlers | High concurrency for I/O-bound work | Blocking calls inside `async def` stall entire event loop |
| Pydantic v2 | 5–50× faster than v1, strict validation | Some API changes from v1; Rust dependency |
| Single-process Uvicorn | Simple | Does not use multiple CPU cores; use Gunicorn + workers in prod |
| `BackgroundTasks` | Simple, no extra infrastructure | No durability, retries, or distributed execution |
| Dependency injection | Testable, composable | Slight indirection; can be confusing for newcomers |

## References

- FastAPI docs: https://fastapi.tiangolo.com
- Pydantic v2 docs: https://docs.pydantic.dev/latest/
- SQLAlchemy async: https://docs.sqlalchemy.org/en/20/orm/extensions/asyncio.html
- Starlette docs: https://www.starlette.io

## Links
- [[rest_api_design|REST Design]]
- [[grpc_and_protobuf|gRPC and Protobuf]]
- [[docker_patterns|Docker Patterns]]
- [[testing_strategies|Testing Strategies]]
- [[04_ml_engineering/05_deployment_and_serving/serving_patterns|Model Serving Patterns]]
