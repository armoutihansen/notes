---
layer: 03_software_engineering
type: engineering
tool: go
status: evergreen
tags: []
created: 2026-03-02
---

# Go Packages and Modules

## Purpose

Go's package system is the unit of code organisation and visibility. Modules are collections of packages released together and declared in a `go.mod` file. The toolchain commands `go run`, `go build`, and `go install` drive the development workflow.

## Implementation Notes

### Package Declaration

Every Go source file begins with a package declaration:

```go
package main      // executable entry point — must define func main()
package mylib     // library package — exports functions, types, etc.
```

Identifiers starting with an **uppercase letter** are exported (public); lowercase identifiers are package-private.

### Import Syntax

```go
// single import
import "fmt"

// grouped import (preferred)
import (
    "fmt"
    "math/rand"
    "github.com/some/external"
)
```

By convention the package name is the last path element (`math/rand` → `rand`).

### Library vs. Main Packages

| | `package main` | library package |
|---|---|---|
| Entry point | `func main()` required | none |
| Output | executable binary | importable API |
| Export functions | not needed | yes — uppercase names |

### Module System

A module is declared by a `go.mod` file at the repository root:

```
module github.com/myorg/myproject

go 1.23.0

require github.com/google/somepackage v1.3.0
```

Initialise a new module:

```bash
go mod init github.com/myorg/myproject
```

The module path also tells the `go` command *where to download* the module (`go get` consults the URL derived from the module path via the module proxy).

### Directory Structure

```
myproject/             ← one module (one go.mod)
  go.mod
  main.go              ← package main
  internal/
    auth/
      auth.go          ← package auth (import path: github.com/myorg/myproject/internal/auth)
```

- One module per repository (typically).
- One package per directory; all `.go` files in a directory share the same package name.

### Toolchain Commands

```bash
go run main.go          # compile + run (no binary saved; for local dev/testing)
go build ./...          # compile → statically linked binary in working directory
go install ./...        # compile + install binary to $GOBIN
go get github.com/x/y  # add/upgrade a dependency; updates go.mod and go.sum
```

`go build` output is a single statically linked binary — no Go runtime needed on the target machine.

### Remote Packages

Dependencies are fetched through the [module proxy](https://proxy.golang.org) by default. After `go get`, the dependency appears in `go.mod` and a checksum in `go.sum`:

```bash
go get github.com/some/pkg@v1.2.3
```

### Package Naming Conventions

- Short, lowercase, no underscores or mixed caps: `rand`, `http`, `strconv`.
- Name = last element of import path: `math/rand` → `package rand`.
- One package per directory; avoid `package main` in library directories.

### Clean Package Design

- **Single responsibility** — expose only what callers need; keep internal logic unexported.
- **Stable API** — don't change exported function signatures without a major version bump.
- **No cyclic imports** — Go forbids import cycles at compile time.
- **Packages shouldn't know about dependents** — a library must not import the application that uses it.

```go
// Only ClassifyImage is exported; internal helpers are lowercase
package classifier

func ClassifyImage(image []byte) string {
    if hasHotdogShape(image) && hasHotdogColors(image) {
        return "hotdog"
    }
    return "not hotdog"
}

func hasHotdogShape(image []byte) bool  { return true }
func hasHotdogColors(image []byte) bool { return true }
```

## Trade-offs

- **`go run` is not for production** — it recompiles on every invocation and does not produce a deployable artifact; always use `go build` for production binaries.
- **`GOPATH` is legacy** — modern Go uses modules; avoid working in `$GOPATH/src`.
- **`replace` directive** — useful for local development (`replace github.com/x/y => ../y`), but must be removed before publishing; downstream consumers cannot resolve local paths.

## References

- [Go doc: Code organisation](https://golang.org/doc/code)
- [Go command reference](https://pkg.go.dev/cmd/go)
- [Effective Go: Package names](https://go.dev/doc/effective_go#package-names)
- [Module proxy](https://proxy.golang.org)

## Links

- [[go_functions]] — exported vs. unexported function visibility
- [[go_interfaces]] — interfaces defined in one package, implemented in another
- [[go_index]]
