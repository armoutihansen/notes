---
layer: 03_software_engineering
type: engineering
tool: Python
status: growing
tags: [pattern]
created: 2026-03-05
---

# Python Async

## Purpose

Python's `asyncio` library and `async`/`await` syntax provide cooperative multitasking for **I/O-bound** concurrent workloads within a single thread. This note covers the event loop, coroutines, tasks, common `asyncio` primitives, and the decision between async, threading, and multiprocessing.

---

## Architecture

```
┌─────────────────────────────────────────────────┐
│                 Event Loop                      │
│                                                 │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐       │
│  │ Task A   │  │ Task B   │  │ Task C   │       │
│  │ (coro)   │  │ (coro)   │  │ (coro)   │       │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘       │
│       │ await        │ await        │ await     │
│       ▼              ▼              ▼           │
│            I/O selector (epoll/kqueue)          │
└─────────────────────────────────────────────────┘
```

**Key concepts:**
- **Event loop** — single-threaded loop that polls for I/O readiness and schedules coroutines
- **Coroutine** — a function defined with `async def`, which returns a coroutine object when called (not yet executed)
- **Task** — wraps a coroutine and schedules it on the event loop; tasks are concurrent
- **Awaitable** — anything that can be `await`ed: coroutines, `Task`, `Future`
- **Future** — low-level object representing a result not yet computed

---

## Implementation Notes

### Basic `async`/`await`

```python
import asyncio

async def fetch(url: str) -> str:
    # await suspends this coroutine until the awaited thing completes
    # giving the event loop the opportunity to run other tasks
    await asyncio.sleep(1)   # simulates network latency
    return f"Response from {url}"

async def main():
    result = await fetch("https://api.example.com")
    print(result)

asyncio.run(main())   # creates event loop, runs, closes
```

A coroutine is **not started** by calling it; calling `fetch(url)` returns a coroutine object. It runs only when awaited or wrapped in a task.

---

### `asyncio.gather` — Concurrent Execution

`asyncio.gather` runs multiple awaitables **concurrently** (not in parallel — still one thread, but I/O waits overlap).

```python
import asyncio
import time

async def fetch(url: str, delay: float) -> str:
    await asyncio.sleep(delay)
    return f"Got {url}"

async def main():
    start = time.perf_counter()

    # Sequential — total time ≈ 1 + 2 + 3 = 6s
    r1 = await fetch("url1", 1)
    r2 = await fetch("url2", 2)
    r3 = await fetch("url3", 3)

    # Concurrent — total time ≈ max(1, 2, 3) = 3s
    results = await asyncio.gather(
        fetch("url1", 1),
        fetch("url2", 2),
        fetch("url3", 3),
    )

    print(f"Elapsed: {time.perf_counter() - start:.2f}s")
    print(results)   # ["Got url1", "Got url2", "Got url3"]

asyncio.run(main())
```

`gather` returns results in input order, regardless of completion order. Exceptions in any coroutine are re-raised at the `gather` site (unless `return_exceptions=True`).

---

### `asyncio.create_task` — Fire and Continue

Creates a task that runs concurrently without waiting for it immediately.

```python
async def background_job(name: str) -> None:
    print(f"Starting {name}")
    await asyncio.sleep(2)
    print(f"Done {name}")

async def main():
    # Fire off tasks without awaiting immediately
    task_a = asyncio.create_task(background_job("A"))
    task_b = asyncio.create_task(background_job("B"))

    print("Tasks created, doing other work...")
    await asyncio.sleep(0.5)

    # Now wait for both
    await task_a
    await task_b

asyncio.run(main())
```

`create_task` vs `gather`: use `create_task` when you need handles to individual tasks (to cancel, check status, await independently). Use `gather` when you want to wait for a fixed set and collect results.

---

### Async Iterators and Context Managers

The async data model mirrors the sync one with `__aiter__`, `__anext__`, `__aenter__`, `__aexit__`:

```python
import asyncio
from contextlib import asynccontextmanager

# Async context manager
@asynccontextmanager
async def async_connection(url: str):
    conn = await connect(url)   # async connect
    try:
        yield conn
    finally:
        await conn.close()

async with async_connection("db://host") as conn:
    await conn.execute("SELECT 1")

# Async generator / async iterator
async def stream_records(limit: int):
    for i in range(limit):
        await asyncio.sleep(0)   # yield control to event loop
        yield {"id": i, "value": i * 2}

async def consume():
    async for record in stream_records(100):
        print(record)
```

---

### Exception Handling in Async Code

```python
async def risky() -> str:
    await asyncio.sleep(0.1)
    raise ValueError("Something went wrong")

async def main():
    # Handle per-task exceptions with return_exceptions
    results = await asyncio.gather(
        risky(),
        asyncio.sleep(1),
        return_exceptions=True,
    )
    for r in results:
        if isinstance(r, Exception):
            print(f"Error: {r}")
        else:
            print(f"Result: {r}")

    # TaskGroup (Python 3.11+) — better structured concurrency
    try:
        async with asyncio.TaskGroup() as tg:
            t1 = tg.create_task(risky())
            t2 = tg.create_task(asyncio.sleep(1))
    except* ValueError as eg:
        print(f"Errors: {eg.exceptions}")
```

`asyncio.TaskGroup` (3.11+) is the preferred structured concurrency primitive — it cancels all sibling tasks if any fails, preventing resource leaks.

---

### Timeouts and Cancellation

```python
async def slow_operation() -> str:
    await asyncio.sleep(10)
    return "result"

async def main():
    # asyncio.timeout (Python 3.11+)
    try:
        async with asyncio.timeout(2.0):
            result = await slow_operation()
    except TimeoutError:
        print("Timed out!")

    # asyncio.wait_for (earlier versions)
    try:
        result = await asyncio.wait_for(slow_operation(), timeout=2.0)
    except asyncio.TimeoutError:
        print("Timed out!")

    # Manual cancellation
    task = asyncio.create_task(slow_operation())
    await asyncio.sleep(1)
    task.cancel()
    try:
        await task
    except asyncio.CancelledError:
        print("Task was cancelled")
```

---

### Running Blocking Code in Async Context

CPU-bound or blocking I/O code blocks the event loop. Offload to a thread or process pool:

```python
import asyncio
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor

def blocking_io(path: str) -> str:
    with open(path) as f:
        return f.read()

def cpu_intensive(n: int) -> int:
    return sum(i * i for i in range(n))

async def main():
    loop = asyncio.get_event_loop()

    # Run blocking I/O in thread pool (threads share GIL but release on I/O)
    content = await loop.run_in_executor(None, blocking_io, "/etc/hosts")

    # Run CPU work in process pool (bypasses GIL)
    with ProcessPoolExecutor() as pool:
        result = await loop.run_in_executor(pool, cpu_intensive, 10_000_000)
```

---

## Trade-offs

### When to Use Async vs Threading vs Multiprocessing

| Scenario | Best choice | Reason |
|----------|-------------|--------|
| Many concurrent HTTP requests | `asyncio` | I/O wait dominates; single thread sufficient |
| Calling blocking APIs (DB drivers without async support) | `threading` | Releases GIL during I/O; simple to wrap |
| CPU-bound work (data processing, compression) | `multiprocessing` | Bypasses GIL; true parallelism |
| Mixed I/O + some CPU | `asyncio` + `ProcessPoolExecutor` for CPU parts | Best of both |
| Simplicity + occasional concurrency | `threading.Thread` or `concurrent.futures.ThreadPoolExecutor` | Simpler mental model for low-scale needs |

```
Decision tree:
Is work I/O-bound?
  Yes → Is it all async-compatible?
          Yes → asyncio
          No  → ThreadPoolExecutor
  No  → Is it CPU-bound?
          Yes → ProcessPoolExecutor / multiprocessing
```

### Common Pitfalls

| Pitfall | Description | Fix |
|---------|-------------|-----|
| Forgetting `await` | Coroutine object created but never executed; no error raised | Enable `asyncio.get_event_loop().set_debug(True)` |
| Blocking the event loop | Synchronous `time.sleep`, file I/O, `requests.get` in async code | Use `asyncio.sleep`, `aiofiles`, `aiohttp` or `run_in_executor` |
| Mixing sync/async improperly | Calling `asyncio.run` inside a running loop | Use `await` or `asyncio.create_task` from within async context |
| Not handling `CancelledError` | Cleanup code not reached on cancellation | Use `finally` blocks; don't swallow `CancelledError` |
| Over-concurrencing | Too many tasks created without backpressure | Use `asyncio.Semaphore` to limit concurrency |

---

### Concurrency vs Parallelism

- **Concurrency** (asyncio, threading): structure for dealing with many things at once — tasks interleave
- **Parallelism** (multiprocessing): literally doing many things simultaneously on multiple CPU cores

asyncio achieves concurrency, not parallelism. For a single long-running computation, asyncio provides no speedup.

---

## References

- [asyncio documentation](https://docs.python.org/3/library/asyncio.html)
- [PEP 492 — Coroutines with async and await](https://peps.python.org/pep-0492/)
- [PEP 654 — Exception Groups and except*](https://peps.python.org/pep-0654/)
- *Python Concurrency with asyncio* — Matthew Fowler (2022)

---

## Links
- [[python_core_language|Core Language]] — generators underlie coroutines; context managers underlie `async with`
- [[python_data_model|Data Model]] — `__aiter__`, `__anext__`, `__aenter__`, `__aexit__` async protocol methods
