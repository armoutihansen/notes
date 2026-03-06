---
layer: 03_software_engineering
type: engineering
tool: go
status: evergreen
tags: []
created: 2026-03-02
---

# Go Collections

## Purpose

Go provides three built-in collection types: arrays (fixed-size, rarely used directly), slices (dynamic views over arrays, the workhorse of ordered lists), and maps (key→value hash tables). Understanding slice and map internals prevents subtle sharing bugs.

## Implementation Notes

### Arrays

Fixed size, determined at compile time. Rarely used directly — slices are almost always preferred.

```go
var myInts [10]int                        // zero-valued array
primes := [6]int{2, 3, 5, 7, 11, 13}    // literal
```

### Slices

A slice is a *dynamically-sized, flexible view* of an underlying array. The zero value of a slice is `nil`.

**Creating slices**

```go
// literal
s := []string{"I", "love", "go"}

// make([]T, len, cap) — filled with zero values
s := make([]int, 5)       // len=5, cap=5
s := make([]int, 3, 8)    // len=3, cap=8
```

**Slice expressions** — `lowIndex` is inclusive, `highIndex` is exclusive:

```go
primes := [6]int{2, 3, 5, 7, 11, 13}
s := primes[1:4]   // [3 5 7]
s = primes[2:]     // from index 2 to end
s = primes[:3]     // first three elements
s = primes[:]      // entire array
```

**Length and capacity**

```go
fmt.Println(len(s), cap(s))
```

**`append`** — always assign back to the same variable:

```go
s = append(s, 4)
s = append(s, 1, 2, 3)
s = append(s, otherSlice...)   // variadic spread
```

If the underlying array has insufficient capacity, `append` allocates a new one. This is safe when appending to the *same* variable; appending to a *different* variable from a shared base can cause aliasing (see Trade-offs).

**Range iteration** — the element is a copy:

```go
fruits := []string{"apple", "banana", "grape"}
for i, fruit := range fruits {
    fmt.Println(i, fruit)
}
```

**Slices of slices** (2D):

```go
rows := [][]int{}
```

**Variadic functions** accept `...T` and receive arguments as a slice:

```go
func concat(strs ...string) string {
    result := ""
    for _, s := range strs {
        result += s
    }
    return result
}

names := []string{"a", "b", "c"}
concat(names...)   // spread a slice into variadic args
```

---

### Maps

A map provides key→value mapping. The zero value is `nil`; a nil map *panics on write*.

**Creating maps**

```go
// make
ages := make(map[string]int)
ages["John"] = 37

// literal
ages := map[string]int{
    "John": 37,
    "Mary": 21,
}
```

**Reading** — missing keys return the zero value of the value type:

```go
v := ages["unknown"]   // 0, no panic
```

**Comma-ok idiom** — distinguish a missing key from a zero-value entry:

```go
v, ok := ages["John"]
if !ok {
    // key not present
}
```

**Mutations**

```go
ages["Alice"] = 30          // insert / update
delete(ages, "Alice")       // delete (safe even if key absent)
fmt.Println(len(ages))      // number of key/value pairs
```

**Nested maps**

```go
hits := make(map[string]map[string]int)
// or use a struct key to avoid inner-map initialisation:
type Key struct{ Path, Country string }
hits2 := make(map[Key]int)
hits2[Key{"/", "au"}]++
```

**Key type constraints** — keys must be *comparable* (bool, numeric, string, pointer, channel, interface, or structs/arrays of those). Slices, maps, and functions cannot be map keys.

## Trade-offs

**Shared backing array (slices)** — when two slices share the same underlying array and there is still spare capacity, an `append` to one will silently overwrite the other:

```go
i := make([]int, 3, 8)
j := append(i, 4)   // j and i share array; cap not yet exhausted
g := append(i, 5)   // g overwrites index 3 — j[3] is now 5!
```

Rule: always append to *the same variable*: `s = append(s, v)`.

**Maps are not goroutine-safe** — concurrent read/write causes a runtime panic (`fatal error: concurrent map iteration and map write`). Protect with a `sync.Mutex` or `sync.RWMutex`. See [[go_concurrency]].

**Maps pass by reference** — passing a map to a function mutates the caller's map; no copy is made.

## References

- [Go blog: Slices](https://go.dev/blog/slices-intro)
- [Go blog: Maps](https://go.dev/blog/maps)
- [Effective Go: Maps](https://go.dev/doc/effective_go#maps)
- [`builtin.make`](https://pkg.go.dev/builtin#make)
- [`builtin.append`](https://pkg.go.dev/builtin#append)

## Links

- [[go_concurrency]] — mutex protection for maps
- [[go_generics]] — generic functions over slices
- [[go_index]]
