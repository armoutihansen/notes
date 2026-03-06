---
layer: 03_software_engineering
type: engineering
tool: git
status: growing
tags:
  - git
created: 2026-03-02
---

# Git Ignore

## Purpose

`.gitignore` tells Git which files and directories to leave untracked — preventing secrets, build artifacts, editor files, and OS junk from being committed accidentally.

## Architecture

`.gitignore` is a plain text file in a repository directory. Git checks it when determining the untracked status of files. Multiple `.gitignore` files can exist: one at the repo root (typically committed and shared) and additional ones in subdirectories (rules apply only within that subtree).

**Priority (highest to lowest):**
1. Patterns in `.git/info/exclude` (local, never committed)
2. File specified by `core.excludesFile` in global Git config (e.g. `~/.gitignore_global`)
3. `.gitignore` files in the repo directory tree (applied from nearest to root)

## Implementation Notes

**Pattern syntax:**

| Pattern | Matches |
|---------|---------|
| `*.log` | Any `.log` file anywhere in the repo |
| `build/` | The `build/` directory |
| `/config.env` | Only `config.env` at repo root (rooted pattern) |
| `!important.log` | Un-ignore (negate) `important.log` |
| `**/temp` | `temp` directory at any depth |
| `*.py[cod]` | `.pyc`, `.pyo`, `.pyd` files |

**Common ignore rules:**
```
# Build artifacts
dist/
build/
*.o
*.a

# Python
__pycache__/
*.pyc
*.egg-info/
.venv/

# Node
node_modules/
.npm/

# Editor / OS
.DS_Store
.idea/
.vscode/
*.swp
Thumbs.db

# Secrets
.env
*.pem
*.key
```

**Stop tracking an already-committed file:**
```bash
git rm --cached <file>          # remove from index, keep on disk
# then add pattern to .gitignore and commit
```

**Check why a file is ignored:**
```bash
git check-ignore -v <file>      # shows which rule in which file is ignoring it
```

**Temporarily ignore changes to a tracked file** (not `.gitignore` — this is index-level):
```bash
git update-index --assume-unchanged <file>
```

## Trade-offs

- `.gitignore` only works on **untracked** files. If a file was committed first, it must be explicitly removed from the index with `git rm --cached` before `.gitignore` takes effect.
- Negation patterns (`!`) must come after the matching pattern, and parent directories cannot be re-included once ignored.
- For sensitive files (secrets, keys), `.gitignore` is a safety net — never rely on it alone. Rotate any credentials that were committed, even briefly.

## References

- [git-ignore documentation](https://git-scm.com/docs/gitignore)
- [gitignore.io](https://www.toptal.com/developers/gitignore) — generate `.gitignore` for any language/tool

## Links
- [[03_software_engineering/version_control/git/git_staging|Git Staging Area]]
- [[03_software_engineering/version_control/git/git_workflow|Git Workflow]]
