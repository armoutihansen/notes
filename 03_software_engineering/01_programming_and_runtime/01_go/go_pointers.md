---
layer: 03_software_engineering
type: engineering
tool: go
status: evergreen
tags: []
created: 2026-03-02
---

# Go Pointers

## Purpose

A pointer holds the memory address of another variable. Pointers allow functions to mutate the caller's data and avoid expensive copies of large values.

## Architecture

Go passes all basic types by value (callee receives a copy). Pointers break that rule intentionally — passing `*T` lets a function read and write the original. Go has no pointer arithmetic (unlike C).

## Implementation Notes

### Pointer Declaration & Address-of

```go
// Declare a nil pointer to int
var p *int

// Take the address of a variable
x := 42
p = &x  // p now holds the memory address of x
```

### Dereferencing

```go
fmt.Println(*p)  // read value at address: 42

*p = 99          // write through pointer
fmt.Println(x)   // 99 — original variable changed
```

### Nil Pointers

A pointer's zero value is `nil`. Dereferencing `nil` causes a **runtime panic**.

```go
var p *int
if p != nil {
    fmt.Println(*p)
}
// Always check before dereferencing an uncertain pointer
```

### Pass by Value vs Pass by Pointer

```go
// By value — caller's x is unchanged
func increment(x int) { x++ }

// By pointer — caller's x is mutated
func increment(x *int) { *x++ }

func main() {
    x := 5
    increment(&x)
    fmt.Println(x) // 6
}
```

### Pointer Receivers on Methods

Pointer receivers allow methods to mutate the struct and avoid copying it on every call:

```go
type car struct{ color string }

// Pointer receiver — mutates original
func (c *car) setColor(color string) {
    c.color = color
}

// Value receiver — works on a copy, no mutation
func (c car) describe() string {
    return "color: " + c.color
}

myCar := car{color: "white"}
myCar.setColor("blue")        // Go auto-takes address: (&myCar).setColor(...)
fmt.Println(myCar.color)      // "blue"
```

**When to use a pointer receiver:**

- The method needs to mutate the receiver.
- The struct is large (avoid copy overhead).
- Consistency — if any method on a type uses a pointer receiver, prefer pointer receivers for all methods on that type.

### Accessing Struct Fields via Pointer

Go automatically dereferences pointer-to-struct in selector expressions:

```go
type point struct{ x, y int }

p := &point{x: 1, y: 2}

// Both are equivalent; the short form is idiomatic
p.x           // auto-dereferenced
(*p).x        // explicit dereference
```

### References Concept

Go does not have a `reference` keyword. "Pass by reference" is achieved by passing a pointer. Slices, maps, and channels are already reference types (they carry an internal pointer), so passing them copies the header, not the underlying data.

## Trade-offs

- Dereferencing a nil pointer **panics at runtime** — always guard uncertain pointers.
- Pointer receivers and value receivers behave differently; mixing them on the same type without reason is a bug source.
- Pointers make code harder to reason about locally; prefer value semantics when the struct is small.
- No pointer arithmetic means Go pointers are safer than C pointers but less flexible.

## References

- [Go Tour: Pointers](https://go.dev/tour/moretypes/1)
- [Effective Go — Pointers vs Values](https://go.dev/doc/effective_go#pointers_vs_values)

## Links

- [[go_structs]]
- [[go_functions]]
- [[go_basics]]
