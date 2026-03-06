---
layer: 03_software_engineering
type: engineering
tool: go
status: evergreen
tags: []
created: 2026-03-02
---

# Go Errors

## Purpose

Go treats errors as ordinary values rather than exceptions. Functions signal failure by returning an `error` as their last return value; callers check it explicitly. Go also uses `iota`-based const blocks as a lightweight substitute for enums, often combined with error types.

## Implementation Notes

### The `error` Interface

```go
type error interface {
    Error() string
}
```

Any type that implements `Error() string` satisfies the interface.

### Idiomatic Error Handling

Return `error` as the last value; return zero values for all other results on failure:

```go
i, err := strconv.Atoi("42b")
if err != nil {
    fmt.Println("couldn't convert:", err)
    return
}
// i is valid here
```

### Creating Errors

```go
// simple string error
err := errors.New("something went wrong")

// formatted error
err := fmt.Errorf("user %s not found", name)

// wrapping — %w preserves the original error for errors.Is / errors.As
err := fmt.Errorf("failed to get user: %w", originalErr)
```

### Custom Error Types

Implement the `error` interface with a struct to carry structured data:

```go
type userError struct {
    name string
}

func (e userError) Error() string {
    return fmt.Sprintf("%v has a problem with their account", e.name)
}

func sendSMS(msg, userName string) error {
    if !canSendToUser(userName) {
        return userError{name: userName}
    }
    // ...
    return nil
}
```

### Iota — Pseudo-Enum Constants

Go has no native enum type. The closest idiom combines a type definition with a `const` block using `iota`:

```go
type sendingChannel int

const (
    Email sendingChannel = iota   // 0
    SMS                           // 1
    Phone                         // 2
)
```

`iota` resets to 0 at the start of each `const` block and increments by 1 per constant. For string-based pseudo-enums:

```go
type sendingChannel string

const (
    Email sendingChannel = "email"
    SMS   sendingChannel = "sms"
    Phone sendingChannel = "phone"
)

func sendNotification(ch sendingChannel, message string) { /* ... */ }
```

The type definition prevents accidental use of a plain `string` (the compiler rejects untyped string variables), though explicit conversion and string literals are still accepted.

## Trade-offs

- **No exhaustiveness checking** — unlike Rust's `Result` or TypeScript union types, Go does not force callers to handle errors. A careless caller can ignore the error return entirely.
- **Wrapping vs. hiding** — use `%w` when callers may need to inspect the underlying error with `errors.Is`/`errors.As`; use `%v` (or `errors.New`) when the original error is an implementation detail.
- **Iota is just integers** — `iota` constants are not sealed; any value of the underlying type can be cast to the pseudo-enum type, so invalid values are possible at runtime.

## References

- [Go blog: Error handling and Go](https://blog.golang.org/error-handling-and-go)
- [`errors` package](https://pkg.go.dev/errors)
- [`fmt.Errorf`](https://pkg.go.dev/fmt#Errorf)
- [Go spec: Iota](https://go.dev/ref/spec#Iota)

## Links

- [[go_interfaces]] — the `error` interface is a standard Go interface
- [[go_functions]] — multiple return values
- [[go_index]]
