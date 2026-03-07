---
layer: 04_software_engineering
type: engineering
tool: python
status: evergreen
tags: [pattern]
created: 2026-03-02
---

# Filesystem Sandboxing

## Purpose

Restrict file-system operations (read, write, execute) to a designated working directory. Prevents **directory traversal attacks**, where a caller supplies a path like `../../etc/passwd` to escape the intended boundary.

Common context: tool functions exposed to an LLM agent, where the model (or a malicious prompt injection) could supply arbitrary paths.

## Architecture

```
User-supplied path  (e.g., "../../secrets/key.txt")
        │
        ▼
abs_working_dir = os.path.abspath(working_directory)
full_path       = os.path.normpath(os.path.join(abs_working_dir, user_path))
full_path_abs   = os.path.abspath(full_path)
        │
        ▼
if os.path.commonpath([abs_working_dir, full_path_abs]) != abs_working_dir:
    raise ValueError("Path is outside permitted working directory")
        │
        ▼
  Safe to use full_path_abs
```

## Implementation Notes

**Canonical pattern**

```python
import os

def resolve_safe_path(working_directory: str, user_path: str) -> str:
    """Return absolute path if inside working_directory, else raise."""
    abs_wd = os.path.abspath(working_directory)
    candidate = os.path.abspath(
        os.path.normpath(os.path.join(abs_wd, user_path))
    )
    if os.path.commonpath([abs_wd, candidate]) != abs_wd:
        raise ValueError(
            f'Path "{user_path}" is outside the permitted working directory'
        )
    return candidate
```

**Why both `normpath` and `abspath`?**

- `normpath` collapses `../` sequences and redundant separators in the string representation.
- `abspath` resolves remaining relative segments against the real CWD, producing a fully canonical path.
- Using only one leaves edge cases: `normpath` alone doesn't resolve CWD-relative paths; `abspath` alone doesn't collapse embedded `../`.

**`commonpath` vs. `startswith`**

```python
# Fragile: "/tmp/work2/evil" starts with "/tmp/work"
if candidate.startswith(abs_wd):       # BUG

# Correct: commonpath returns the longest shared ancestor
if os.path.commonpath([abs_wd, candidate]) == abs_wd:  # OK
```

**Additional guards per operation**

| Operation | Extra check |
|---|---|
| Read | `os.path.isfile(path)` |
| Write | `not os.path.isdir(path)` |
| Execute | `path.endswith(".py")` + file existence |
| List | `os.path.isdir(path)` |

**Return errors to callers, don't raise (LLM tool context)**

When used inside an LLM tool function, catch `ValueError` and return a structured error dict so the model can reason about the failure:

```python
try:
    safe = resolve_safe_path(wd, user_path)
except ValueError as e:
    return {"error": str(e)}
```

## Trade-offs

| Approach | Pro | Con |
|---|---|---|
| `commonpath` (above) | Correct; pure Python; no privileges needed | Application-level only; symlinks can bypass |
| `chroot` / `seccomp` (OS-level) | Kernel-enforced; strongest isolation | Requires elevated privileges; complex setup |
| Container / VM sandbox | Full process isolation | Heavy; impractical for lightweight tool functions |
| Allowlist of permitted paths | Explicit control | Maintenance burden; too restrictive for dynamic use |

## References

- Python `os.path` docs: [`commonpath`](https://docs.python.org/3/library/os.path.html#os.path.commonpath)
- OWASP: [Path Traversal](https://owasp.org/www-community/attacks/Path_Traversal)

## Links

- [[function_calling]] — context: sandboxing LLM tool functions
