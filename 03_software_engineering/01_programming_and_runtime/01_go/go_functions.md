---
layer: 03_software_engineering
type: engineering
tool: go
status: evergreen
tags: []
created: 2026-03-02
---

# Go Functions

## Purpose

Functions are first-class values in Go. They are the primary unit of code organisation and support multiple return values, closures, variadic arguments, and deferred execution.

## Architecture

Go functions are block-scoped. Variables declared inside a block (function, loop, if, switch) are inaccessible outside it. Go passes all basic types **by value** — the callee receives a copy unless a pointer is passed explicitly.

## Implementation Notes

### Signature — type comes after name

```go
func sub(x int, y int) int {
    return x - y
}

// Consecutive same-type params can share the type annotation
func add(x, y int) int {
    return x + y
}
```

### Multiple Return Values

```go
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}

result, err := divide(10, 2)
```

### Named Return Values

Named returns document intent and allow bare `return`:

```go
func calculator(a, b int) (mul, div int, err error) {
    if b == 0 {
        return 0, 0, errors.New("can't divide by zero")
    }
    mul = a * b
    div = a / b
    return // naked return — returns mul, div, err
}
```

Use naked returns only in short functions; they hurt readability in longer ones.

### Ignoring Return Values

```go
x, _ := getPoint() // discard the second return value
```

The compiler rejects unused variables, so `_` is required when you don't need a value.

### Variadic Functions

```go
func sum(nums ...int) int {
    total := 0
    for _, n := range nums {
        total += n
    }
    return total
}

sum(1, 2, 3)          // 6
nums := []int{1,2,3}
sum(nums...)           // spread a slice
```

### Functions as First-Class Values

```go
func aggregate(a, b, c int, op func(int, int) int) int {
    return op(op(a, b), c)
}

func add(x, y int) int { return x + y }

aggregate(2, 3, 4, add) // 9
```

### Anonymous Functions & Closures

```go
double := func(n int) int { return n * 2 }

// Closure captures outer variable
adder := func(base int) func(int) int {
    return func(n int) int { return base + n }
}
add5 := adder(5)
add5(3) // 8
```

### Defer

`defer` schedules a function call to execute just before the enclosing function returns. Arguments are evaluated immediately; execution is deferred. Multiple defers run in **LIFO** order.

```go
func readFile(path string) error {
    f, err := os.Open(path)
    if err != nil {
        return err
    }
    defer f.Close() // runs when readFile returns, regardless of which return

    // ... use f ...
    return nil
}
```

Use `defer` for cleanup: closing files, releasing locks, logging exit.

### Early Returns / Guard Clauses

Prefer early returns over nested conditionals:

```go
func divide(dividend, divisor int) (int, error) {
    if divisor == 0 {
        return 0, errors.New("can't divide by zero")
    }
    return dividend / divisor, nil
}
```

### Pass by Value

```go
func increment(x int) { x++ }   // does NOT affect caller

func main() {
    x := 5
    increment(x)
    fmt.Println(x) // 5
}
```

Use [[go_pointers]] when the function needs to mutate the caller's variable.

## Trade-offs

- Naked returns reduce boilerplate but harm readability in long functions — avoid them there.
- `defer` overhead is small but measurable in hot paths.
- Closures capture variables by reference; mutating a captured variable in a goroutine requires care.
- Variadic functions accept a slice spread with `...`, but cannot mix variadic and individual args after the spread.

## References

- [Effective Go — Functions](https://go.dev/doc/effective_go#functions)
- [Go Tour: Functions](https://go.dev/tour/basics/4)

## Links

- [[go_basics]]
- [[go_pointers]]
- [[go_control_flow]]
- [[go_interfaces]]
