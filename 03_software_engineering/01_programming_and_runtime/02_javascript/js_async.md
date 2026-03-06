---
layer: 03_software_engineering
type: engineering
tool: javascript
status: evergreen
tags: [javascript, async, promises, event-loop]
created: 2026-03-02
---

# JavaScript — Async and the Event Loop

## Purpose

Asynchronous programming is the defining characteristic of JavaScript's execution model. Understanding the event loop, task queues, and Promises is essential for writing correct Node.js servers, browser code, and anything that touches I/O.

## Architecture

### The Event Loop Mental Model

```
┌─────────────────────────────────────────────────────────────┐
│  JavaScript Runtime (single-threaded)                       │
│                                                             │
│  ┌───────────────┐    drains first    ┌─────────────────┐   │
│  │  Call Stack   │ ◄── then ────────  │ Microtask Queue │   │
│  │  (LIFO)       │                    │ Promise .then   │   │
│  │               │                    │ async/await     │   │
│  └───────────────┘                    └─────────────────┘   │
│          ▲  one task                                        │
│          │  per loop tick             ┌─────────────────┐   │
│          └──────────────────────────  │  Task Queue     │   │
│                                       │  (Macrotasks)   │   │
│                                       │  setTimeout     │   │
│                                       │  setInterval    │   │
│                                       └─────────────────┘   │
└─────────────────────────────────────────────────────────────┘
           ▲                    push callbacks when done
           │
┌──────────────────────────────┐
│  External APIs (multi-thread)│
│  setTimeout, fetch, fs.read  │
│  addEventListener, I/O       │
└──────────────────────────────┘
```

**Event loop tick:**
1. Execute everything on the call stack until empty
2. Drain the entire microtask queue (including any microtasks added during draining)
3. Take **one** task from the task queue → go to step 1

---

## Implementation Notes

### Single-Threaded, Non-Blocking

JavaScript has **one thread** — only one piece of code runs at a time. But I/O operations (network, file, timers) are delegated to external APIs (browser APIs or Node.js C++ bindings) that run on separate threads. When done, they push their callback into the task or microtask queue.

This makes JavaScript excellent for **I/O-bound concurrency** (many simultaneous connections with little CPU work) and poor for **CPU-bound computation** (heavy calculation blocks the single thread).

```
Node.js server vs multi-threaded Python/PHP:
✓ Node wins on concurrent I/O (hundreds of open connections)
✗ Node loses on heavy computation per request (blocks the loop)
```

---

### The Call Stack

A LIFO stack of execution frames. Each function call pushes a frame; `return` pops it.

```js
function startJob() {
  console.log("start");
  workOnJob();
}
function workOnJob() {
  console.log("work");
  finishJob();
}
function finishJob() {
  console.log("done");
}
startJob();
// Stack: startJob → workOnJob → finishJob → (returns) → (returns) → (returns)
// Output: start, work, done
```

---

### Task Queue (Macrotask Queue)

Callbacks from `setTimeout`, `setInterval`, and I/O operations are queued here. The event loop takes **one task at a time** when the call stack is empty.

```js
function main() {
  setTimeout(() => console.log("async!"), 0);
  console.log("sync");
}
main();
// Output: sync
//         async!
// Even with delay=0, the callback runs after the call stack drains
```

---

### Microtask Queue

Promise `.then`/`.catch` callbacks and `async/await` continuations go here. **All microtasks are drained before the next macrotask runs**, and microtasks can enqueue more microtasks.

```js
function main() {
  console.log("start");

  setTimeout(() => console.log("macrotask"), 0);

  Promise.resolve()
    .then(() => console.log("microtask 1"))
    .then(() => console.log("microtask 2"));

  console.log("end");
}
main();
// Output:
// start
// end
// microtask 1
// microtask 2
// macrotask
```

---

### Synchronous vs Asynchronous Code

```js
// Synchronous — sequential, predictable
console.log("first");
console.log("second");
console.log("third");

// Asynchronous — "second" prints before "third" despite callback coming before it
console.log("first");
setTimeout(() => console.log("third"), 100);
console.log("second");
```

Async is necessary when waiting for network responses, file reads, or timers — blocking the thread would freeze the UI or stall the Node.js server.

---

### Promises

```js
// Creating
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    if (Math.random() > 0.5) {
      resolve("success!");
    } else {
      reject(new Error("failed"));
    }
  }, 1000);
});

// Consuming with .then / .catch / .finally
promise
  .then(result => console.log(result))
  .catch(err => console.error(err.message))
  .finally(() => console.log("always runs"));

// Chaining — each .then receives the return value of the previous
fetchUser(id)
  .then(user => fetchProfile(user.id))
  .then(profile => renderProfile(profile))
  .catch(err => showError(err));

// Parallel execution
const [user, posts] = await Promise.all([fetchUser(id), fetchPosts(id)]);
```

Promise states: **pending** → **fulfilled** (resolve called) or **rejected** (reject called). Once settled, state never changes.

---

### `async` / `await`

Syntactic sugar over Promises. An `async` function always returns a Promise; `await` pauses execution *within* the async function until the Promise resolves.

```js
// .then chain equivalent
async function loadUser(id) {
  const user = await fetchUser(id);
  const profile = await fetchProfile(user.id);
  return profile;
}

// Error handling with try/catch
async function loadUser(id) {
  try {
    const user = await fetchUser(id);
    return await fetchProfile(user.id);
  } catch (err) {
    console.error("Failed:", err.message);
    throw err;
  } finally {
    console.log("done loading");
  }
}

// IIFE for top-level await (pre-ESM)
(async () => {
  const result = await loadUser(42);
  console.log(result);
})();

// Top-level await — works natively in ES modules
const result = await loadUser(42);
```

---

### `then` vs `await` Trade-offs

| | `.then` chaining | `async/await` |
|---|---|---|
| Readability | Moderate (callback style) | High (looks synchronous) |
| Error handling | `.catch()` | `try/catch` |
| Parallel requests | `Promise.all([...])` | `await Promise.all([...])` |
| Debugging | Stack traces less clear | Better stack traces |
| Legacy support | Works everywhere | Needs ES2017+ (or Babel) |

**Prefer `async/await`** for new code. Reserve `.then` for when you need to fire-and-forget or build composable promise chains.

## Trade-offs

- **Don't block the event loop**: CPU-intensive operations (image processing, crypto, large JSON parse) should be offloaded to a Worker thread or a child process in Node.js.
- **Unhandled rejections**: Always attach `.catch()` or wrap in `try/catch`. In Node.js, unhandled rejections terminate the process in recent versions.
- **`await` in loops**: `await` inside a `for...of` loop is sequential. For parallel execution use `Promise.all()`.
- **Microtask starvation**: A microtask that continuously queues more microtasks can starve macrotasks (rare in practice).
- **`async` is contagious**: Once you `await`, all callers up the chain must also be `async` — plan the async boundary early.

## References

- [MDN: Event Loop](https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop)
- [MDN: Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- [MDN: async function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)
- [MDN: await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await)
- [MDN: Microtask guide](https://developer.mozilla.org/en-US/docs/Web/API/HTML_DOM_API/Microtask_guide)
- [MDN: Promise.all()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all)

## Links

- [[js_functions]]
- [[js_modules]]
- [[js_runtimes]]
- [[03_software_engineering/01_programming_and_runtime/index|index]]
