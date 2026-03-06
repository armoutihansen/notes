---
layer: 03_software_engineering
type: engineering
tool: go
status: evergreen
tags: []
created: 2026-03-02
---

# Go Interfaces

## Purpose

Interfaces define behaviour as a set of method signatures. They enable polymorphism and decoupled design without explicit inheritance or `implements` declarations.

## Architecture

Implementation is **implicit** — a type automatically satisfies an interface if it has all the required methods. This means interfaces can be defined after the fact, enabling loose coupling between packages.

## Implementation Notes

### Interface Definition

```go
type shape interface {
    area()      float64
    perimeter() float64
}
```

Any type with `area()` and `perimeter()` methods satisfying those signatures implements `shape` automatically.

### Implicit Implementation

```go
type rect struct{ width, height float64 }

func (r rect) area() float64      { return r.width * r.height }
func (r rect) perimeter() float64 { return 2*r.width + 2*r.height }

// rect now implements shape — no declaration needed
func printShapeData(s shape) {
    fmt.Printf("Area: %v, Perimeter: %v\n", s.area(), s.perimeter())
}
```

### Empty Interface — `interface{}` / `any`

Every type implements the empty interface. Use `any` (alias introduced in Go 1.18):

```go
func printAnything(v any) {
    fmt.Printf("%v (%T)\n", v, v)
}
```

Avoid overusing `any`; it bypasses compile-time type safety.

### Multiple Interfaces

A type can satisfy multiple interfaces simultaneously:

```go
type stringer interface{ String() string }
type sizer   interface{ Size() int }

// A type implementing both:
type file struct{ name string; size int }
func (f file) String() string { return f.name }
func (f file) Size() int      { return f.size }
```

Interfaces can embed other interfaces:

```go
type firetruck interface {
    car           // embeds car interface
    HoseLength() int
}
```

### Type Assertions

Extract the underlying concrete type from an interface value:

```go
// Safe assertion — ok is false if s is not a circle
c, ok := s.(circle)
if !ok {
    log.Fatal("s is not a circle")
}
fmt.Println(c.radius)
```

Unsafe assertion (panics if wrong type): `c := s.(circle)`

### Type Switches

Branch on the underlying type of an interface value:

```go
func describe(v interface{}) {
    switch t := v.(type) {
    case int:
        fmt.Printf("int: %d\n", t)
    case string:
        fmt.Printf("string: %q\n", t)
    default:
        fmt.Printf("other: %T\n", t)
    }
}
```

### Clean Interface Design

1. **Keep interfaces small** — the smaller, the more reusable. `io.Reader` (one method) is the canonical example.
2. **Accept interfaces, return concrete types** — functions should accept the minimal interface they need; callers should receive concrete values they can use freely.
3. **Interfaces are not classes** — no constructors, no hierarchy, no shared implementation. Each satisfying type defines its own behaviour.
4. **Don't encode type knowledge in the interface** — avoid `IsX() bool` methods. Use type assertions or sub-interfaces instead.

## Trade-offs

- Implicit implementation is powerful but can cause accidental satisfaction — name your interfaces clearly.
- `any` (empty interface) loses type safety; prefer typed interfaces or generics.
- Type assertions at runtime add a small cost and can panic; prefer the `v, ok` form.
- Small interfaces compose well; large interfaces are rigid and hard to mock in tests.

## References

- [Go Tour: Interfaces](https://go.dev/tour/methods/9)
- [Effective Go — Interfaces](https://go.dev/doc/effective_go#interfaces)
- [Best practices for Go interfaces](https://blog.boot.dev/golang/golang-interfaces/)

## Links

- [[go_structs]]
- [[go_functions]]
- [[go_pointers]]
- [[go_errors]]
