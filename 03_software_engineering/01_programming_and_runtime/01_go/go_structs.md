---
layer: 03_software_engineering
type: engineering
tool: go
status: evergreen
tags: []
created: 2026-03-02
---

# Go Structs

## Purpose

Structs group related fields into a single named type. They are Go's primary mechanism for modelling data — analogous to objects in other languages but without inheritance.

## Architecture

Go is not object-oriented. Structs sit in a **contiguous block of memory** with fields laid out sequentially. Methods are functions with a receiver and are defined outside the struct body.

## Implementation Notes

### Definition & Instantiation

```go
type car struct {
    brand   string
    model   string
    doors   int
    mileage int
}

// Named field initialisation (preferred)
myCar := car{brand: "Toyota", model: "Camry", doors: 4, mileage: 0}

// Zero-value struct
var emptyCar car
```

### Methods — Value vs Pointer Receivers

```go
type rect struct {
    width, height float64
}

// Value receiver — works on a copy, cannot mutate
func (r rect) area() float64 {
    return r.width * r.height
}

// Pointer receiver — can mutate the original
func (r *rect) scale(factor float64) {
    r.width *= factor
    r.height *= factor
}

r := rect{width: 5, height: 10}
r.scale(2)          // Go automatically takes address: (&r).scale(2)
fmt.Println(r.width) // 10
```

Use a pointer receiver when the method must mutate state or the struct is large (to avoid copying). Be consistent within a type — don't mix receivers.

### Embedded Structs

Embedding promotes fields and methods to the outer struct level:

```go
type car struct {
    brand string
    model string
}

type truck struct {
    car           // embedded — truck gets brand & model directly
    bedSize int
}

t := truck{bedSize: 10, car: car{brand: "Toyota", model: "Tundra"}}
fmt.Println(t.brand)  // promoted — no t.car.brand needed
fmt.Println(t.model)
```

### Nested Structs

Fields can be named struct types (no field promotion):

```go
type wheel struct {
    radius   int
    material string
}

type car struct {
    brand      string
    frontWheel wheel
    backWheel  wheel
}

myCar := car{}
myCar.frontWheel.radius = 18
```

### Anonymous Structs

Useful for one-off shapes (e.g. JSON bodies, test data):

```go
myCar := struct {
    brand string
    model string
}{brand: "Toyota", model: "Camry"}
```

### Empty Struct

`struct{}` occupies **zero bytes** of memory:

```go
// Channel signalling (send a signal with no payload)
done := make(chan struct{})
done <- struct{}{}

// Set implementation using map
seen := make(map[string]struct{})
seen["key"] = struct{}{}
```

### Memory Layout & Field Ordering

Go aligns struct fields to their size, potentially inserting padding. Order fields from largest to smallest to minimise wasted space:

```go
// Efficient — no padding between fields
type stats struct {
    Reach    uint16 // 2 bytes
    NumPosts uint8  // 1 byte
    NumLikes uint8  // 1 byte
}

// Inefficient — padding inserted after NumPosts
type stats struct {
    NumPosts uint8  // 1 byte + 1 byte padding
    Reach    uint16 // 2 bytes
    NumLikes uint8  // 1 byte + padding
}
```

## Trade-offs

- Pointer receivers and value receivers behave differently — mixing them on the same type is a bug source.
- Embedding is not inheritance; the embedded type has no knowledge of the outer type.
- Field ordering matters for hot paths / large arrays of structs; irrelevant for most application code.
- Anonymous structs cannot be reused — prefer named structs unless truly one-off.

## References

- [Go spec: Struct types](https://go.dev/ref/spec#Struct_types)
- [Effective Go — Data](https://go.dev/doc/effective_go#composite_literals)
- [The empty struct — Dave Cheney](https://dave.cheney.net/2014/03/25/the-empty-struct)

## Links

- [[go_interfaces]]
- [[go_pointers]]
- [[go_basics]]
