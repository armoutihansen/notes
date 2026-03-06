---
layer: 03_software_engineering
type: engineering
tool: go
status: evergreen
tags: []
created: 2026-03-02
---

# Go Control Flow

## Purpose

Go provides `if`/`else`, `switch`, and `for` (the only loop keyword) for controlling execution flow. The design is intentionally minimal: no `while`, no `do-while`, no parentheses on conditions.

## Implementation Notes

### if / else if / else

No parentheses around the condition; braces are mandatory.

```go
if height > 6 {
    fmt.Println("super tall")
} else if height > 4 {
    fmt.Println("tall enough")
} else {
    fmt.Println("not tall enough")
}
```

### Short Variable Declaration in if

Declare a variable scoped to the `if` block only:

```go
if length := getLength(email); length < 1 {
    fmt.Println("email is invalid")
}
// length is not accessible here
```

This limits scope and is idiomatic for error checks:

```go
if err := doSomething(); err != nil {
    return err
}
```

### switch

No `break` needed — cases do **not** fall through by default. Use `fallthrough` explicitly when needed.

```go
switch os {
case "linux":
    fmt.Println("Linus Torvalds")
case "mac":
    fmt.Println("A Steve")
default:
    fmt.Println("Unknown")
}
```

**No-condition switch** (acts like an if/else chain):

```go
switch {
case score >= 90:
    grade = "A"
case score >= 80:
    grade = "B"
default:
    grade = "F"
}
```

**Explicit fallthrough**:

```go
case "macOS":
    fallthrough
case "Mac OS X":
    fallthrough
case "mac":
    creator = "A Steve"
```

### for — Go's Only Loop

Standard C-style:

```go
for i := 0; i < 10; i++ {
    fmt.Println(i)
}
```

**While-style** (omit init and post):

```go
n := 1
for n < 100 {
    n *= 2
}
```

**Infinite loop** (omit all three sections):

```go
for {
    // runs forever; use break to exit
}
```

### for range

Iterate over slices, maps, strings, and channels:

```go
// slice — index + value
for i, v := range []string{"a", "b", "c"} {
    fmt.Println(i, v)
}

// map — key + value
for k, v := range myMap {
    fmt.Println(k, v)
}

// discard index
for _, v := range items {
    fmt.Println(v)
}
```

### continue & break

```go
for i := 0; i < 10; i++ {
    if i%2 == 0 {
        continue // skip even numbers
    }
    if i == 7 {
        break // stop at 7
    }
    fmt.Println(i) // 1, 3, 5
}
```

`continue` and `break` can be used inside `for range` the same way.

## Trade-offs

- There is **no `while`** keyword — `for condition {}` is the equivalent.
- Switch cases do **not** fall through by default (opposite of C/Java); this avoids a common bug class.
- The short init statement in `if` keeps the scope tight but may surprise readers unfamiliar with Go.
- `for range` over a map has non-deterministic iteration order.

## References

- [Go Tour: Flow control](https://go.dev/tour/flowcontrol)
- [Effective Go — Control structures](https://go.dev/doc/effective_go#control-structures)

## Links

- [[go_basics]]
- [[go_functions]]
- [[go_collections]]
