---
layer: 03_software_engineering
type: engineering
tool: go
status: evergreen
tags: []
created: 2026-03-02
---

# Go Generics

## Purpose

Generics (added in Go 1.18) allow functions and types to be written once and reused across multiple concrete types, eliminating repetitive type-specific implementations while keeping full static type safety.

## Implementation Notes

### Type Parameters

A type parameter is declared in square brackets after the function or type name:

```go
func splitAnySlice[T any](s []T) ([]T, []T) {
    mid := len(s) / 2
    return s[:mid], s[mid:]
}

// usage — type argument inferred
firstInts, secondInts := splitAnySlice([]int{0, 1, 2, 3})
firstStrs, secondStrs := splitAnySlice([]string{"a", "b", "c", "d"})
```

`T` is the conventional name for a single type parameter; use descriptive names (`K`, `V`, `E`) when there are multiple.

### The `any` Constraint

`any` is an alias for `interface{}` — the type parameter can be *anything*. Use it when the function body makes no assumptions about the type.

### Type Constraints via Interfaces

When the function body needs to call methods or use operators, declare a more specific constraint using an interface:

```go
// constraint: type must have a String() method
type stringer interface {
    String() string
}

func concat[T stringer](vals []T) string {
    result := ""
    for _, v := range vals {
        result += v.String()
    }
    return result
}
```

**Union constraints** restrict to a set of underlying types using `~` (includes all types whose *underlying* type matches):

```go
type Number interface {
    ~int | ~int64 | ~float64
}

func Sum[T Number](nums []T) T {
    var total T
    for _, n := range nums {
        total += n
    }
    return total
}
```

### Parametric Constraints

Interfaces used as constraints can themselves take type parameters, enabling generic relationships between types:

```go
type product interface {
    Price() float64
    Name() string
}

type store[P product] interface {
    Sell(P)
}

// sellProducts works for any store/product combination
func sellProducts[P product](s store[P], products []P) {
    for _, p := range products {
        s.Sell(p)
    }
}
```

### Generic Types

Type parameters can also appear on struct definitions:

```go
type Stack[T any] struct {
    items []T
}

func (s *Stack[T]) Push(item T) {
    s.items = append(s.items, item)
}

func (s *Stack[T]) Pop() (T, bool) {
    if len(s.items) == 0 {
        var zero T
        return zero, false
    }
    top := s.items[len(s.items)-1]
    s.items = s.items[:len(s.items)-1]
    return top, true
}
```

### Generics vs. Interfaces

| Scenario | Prefer |
|---|---|
| Behaviour varies by type, resolved at runtime | `interface` |
| Algorithm is identical across types, zero-value/operator needed | generics |
| Writing reusable data structures (stacks, queues, sets) | generics |
| Single concrete type | neither — just use the type |

## Trade-offs

- **Compile-time monomorphisation** — the compiler generates type-specific code; binary size can grow with many instantiations.
- **Not a replacement for interfaces** — generics are best for *structural* reuse (same algorithm, different types); interfaces are better for *behavioural* polymorphism (different algorithms, same call site).
- **Type inference is limited** — the compiler can infer type arguments from function arguments, but not always from return types; explicit `func[T](...)` annotation may be required.
- **Avoid premature generification** — Go's community ethos values simplicity; reach for generics only when duplication is tangible.

## References

- [Go blog: An Introduction to Generics](https://go.dev/blog/intro-generics)
- [Go tour: Generics](https://go.dev/tour/generics/1)
- [Go spec: Type parameters](https://go.dev/ref/spec#Type_parameter_declarations)

## Links

- [[go_interfaces]] — constraints are defined as interfaces
- [[go_collections]] — generic functions over slices and maps
- [[go_index]]
