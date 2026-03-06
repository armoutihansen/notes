---
layer: 03_software_engineering
type: engineering
tool: go
status: evergreen
tags: [pattern]
created: 2026-03-02
---

# Go Concurrency

## Purpose

Go was designed for concurrency. The `go` keyword spawns lightweight goroutines; channels provide typed, goroutine-safe communication between them; mutexes protect shared memory when channel-based design isn't a fit.

## Architecture

```
goroutine A ──── ch <- value ────▶ channel ──── value := <-ch ────▶ goroutine B
                                   (buffer)

goroutine C ─┐
             ├── select { case <-chA: ... case <-chB: ... }
goroutine D ─┘

shared map ◀──── mu.Lock() / mu.Unlock() ────── goroutines R/W
```

## Implementation Notes

### Goroutines

A goroutine is a lightweight thread managed by the Go runtime. Spawn one with the `go` keyword:

```go
go doSomething()
go func() { /* anonymous goroutine */ }()
```

### Channels — Basics

Channels are typed, thread-safe queues. Always `make` before use.

```go
ch := make(chan int)   // unbuffered

ch <- 69              // send — blocks until a receiver is ready
v := <-ch             // receive — blocks until a value is available
```

A **deadlock** occurs when all goroutines are blocked waiting; the runtime detects this and panics.

### Buffered Channels

```go
ch := make(chan int, 100)   // buffer of 100
```

- Send blocks only when the buffer is *full*.
- Receive blocks only when the buffer is *empty*.

### Closing Channels

Only a *sender* should close a channel. Sending to a closed channel panics.

```go
close(ch)

// receiver can test closure
v, ok := <-ch   // ok == false when channel is empty and closed
```

### Ranging Over Channels

```go
for v := range ch {
    // blocks each iteration; exits when ch is closed
}
```

### Directional Channel Types

Restrict a channel to read-only or write-only in a function signature to communicate intent and catch misuse at compile time:

```go
func readFrom(ch <-chan int)  { v := <-ch; _ = v }   // read-only
func writeTo(ch chan<- int)   { ch <- 42 }            // write-only
```

### `select` Statement

`select` waits on multiple channels simultaneously, like a switch for I/O:

```go
select {
case i := <-chInts:
    fmt.Println("int:", i)
case s := <-chStrings:
    fmt.Println("string:", s)
}
```

If multiple channels are ready at once, one is chosen at random. Add a `default` to make the select non-blocking:

```go
select {
case v := <-ch:
    use(v)
default:
    // ch not ready; do something else
}
```

### Tickers

`time.NewTicker` returns a ticker whose channel fires on a given interval:

```go
ticker := time.NewTicker(500 * time.Millisecond)
for t := range ticker.C {
    fmt.Println("tick at", t)
}
```

Related helpers: `time.After(d)` — fires once after duration; `time.Sleep(d)` — blocks the current goroutine.

---

### Mutexes

Use `sync.Mutex` when multiple goroutines share memory that at least one goroutine writes.

```go
import "sync"

var mu sync.Mutex

func protected() {
    mu.Lock()
    defer mu.Unlock()
    // only one goroutine at a time reaches here
}
```

`defer mu.Unlock()` ensures the lock is always released, even if the function returns early or panics.

### `sync.RWMutex`

Optimise read-heavy workloads: multiple goroutines may hold `RLock` simultaneously, but `Lock` is exclusive.

```go
var mu sync.RWMutex

// readers
mu.RLock()
defer mu.RUnlock()

// writer
mu.Lock()
defer mu.Unlock()
```

### Maps Are Not Goroutine-Safe

Concurrent reads are safe, but any concurrent write causes a runtime panic:

```
fatal error: concurrent map iteration and map write
```

Always protect map access with a mutex:

```go
mu.Lock()
m[key] = value
mu.Unlock()

// or for read-heavy maps
mu.RLock()
v := m[key]
mu.RUnlock()
```

## Trade-offs

- **Channels vs. mutexes** — prefer channels when goroutines need to *communicate*; prefer mutexes when goroutines share a single piece of state (e.g., a cache map).
- **Deadlocks** — unbuffered channels require a matching sender and receiver to both be ready; forgetting either side stalls the program.
- **Don't send on a closed channel** — results in a panic; the sender owns closing responsibility.
- **RWMutex overhead** — beneficial only when reads significantly outnumber writes; for balanced workloads a plain `Mutex` is simpler.

## References

- [`sync.Mutex`](https://pkg.go.dev/sync#Mutex)
- [`sync.RWMutex`](https://pkg.go.dev/sync#RWMutex)
- [`time.NewTicker`](https://pkg.go.dev/time#NewTicker)
- [Go tour: Concurrency](https://go.dev/tour/concurrency/1)
- [Effective Go: Concurrency](https://go.dev/doc/effective_go#concurrency)

## Links

- [[go_collections]] — maps need mutex protection
- [[go_interfaces]] — channels are typed, interface values can be sent over `chan interface{}`
- [[go_index]]
