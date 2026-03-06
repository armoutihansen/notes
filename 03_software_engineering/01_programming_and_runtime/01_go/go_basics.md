---
layer: 03_software_engineering
type: engineering
tool: go
status: evergreen
tags: []
created: 2026-03-02
---

# Go Basics — Variables & Types

## Purpose

Variables and types are the foundation of Go programs. Go is statically typed with type inference, keeping declarations concise while remaining explicit.

## Architecture

Go infers types from values at compile time. Outside functions only `var` or `const` declarations are valid; inside functions the short declaration `:=` is preferred.

## Implementation Notes

### Variable Declaration

```go
// Short declaration (preferred inside functions)
name := "Alice"
age  := 30

// var style (required at package scope or when zero-value is needed)
var count int        // zero value = 0
var active bool      // zero value = false
var label string     // zero value = ""

// Multiple assignment
x, y := 10, 20
```

### Constants

```go
const pi = 3.14159
const greeting = "hello"
// Constants must be primitive types (string, int, float, bool)
// Cannot use := with const
```

### Basic Types

| Type      | Zero value | Notes                              |
|-----------|------------|------------------------------------|
| `bool`    | `false`    | `true` / `false`                   |
| `string`  | `""`       | UTF-8, immutable                   |
| `int`     | `0`        | 32 or 64-bit depending on platform |
| `float64` | `0.0`      | Standard decimal number            |
| `byte`    | `0`        | Alias for `uint8`                  |

### Type Sizes

Use the standard sizes unless you have a specific memory constraint:

```go
// Integers (signed)
int  int8  int16  int32  int64

// Unsigned integers
uint  uint8  uint16  uint32  uint64  uintptr

// Floats
float32  float64   // prefer float64

// Complex (rarely used)
complex64  complex128
```

The number suffix is the bit width. `int` and `uint` map to 32-bit or 64-bit depending on the environment. **Prefer `int`, `uint`, `float64`** unless optimising memory layout.

### Type Conversion

Go has no implicit conversions — cast explicitly:

```go
temperatureFloat := 88.26
temperatureInt   := int64(temperatureFloat) // truncates decimal

var x int  = 42
var y float64 = float64(x)
```

### Formatting Strings

```go
// fmt.Sprintf returns a string; fmt.Printf prints to stdout
name := "Alice"
age  := 30
pi   := 3.14159

fmt.Sprintf("Hello, %s! You are %d years old.", name, age)
fmt.Sprintf("Pi is approximately %.2f", pi)  // 2 decimal places
fmt.Sprintf("Value: %v, Type: %T", age, age) // %v default, %T type name
```

| Verb  | Meaning                     |
|-------|-----------------------------|
| `%v`  | Default representation      |
| `%T`  | Type name                   |
| `%d`  | Integer (decimal)           |
| `%f`  | Float (`%.2f` = 2 decimals) |
| `%s`  | String                      |
| `%t`  | Boolean                     |

## Trade-offs

- `:=` is unavailable at package scope — use `var` there.
- Casting float → int **truncates** (does not round).
- Unused variables cause a compile error; use `_` to discard.
- Go has no implicit type coercion, so mixed arithmetic requires explicit casts.

## References

- [Go Tour: Basics](https://go.dev/tour/basics)
- [fmt package docs](https://pkg.go.dev/fmt)

## Links

- [[go_functions]]
- [[go_control_flow]]
- [[go_structs]]
