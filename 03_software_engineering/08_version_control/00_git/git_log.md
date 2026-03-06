---
layer: 03_software_engineering
type: engineering
tool: git
status: growing
tags:
  - git
created: 2026-03-02
---

# Git Log

## Purpose

`git log` displays the commit history of a repository. It is the primary tool for understanding what changed, when, and why — and for locating specific commits.

## Architecture

Each commit in the log contains:
- **Commit hash (SHA-1)** — unique 40-character identifier; typically abbreviated to 7 characters
- **Author** — name + email of who wrote the change
- **Date** — when the commit was authored
- **Commit message** — subject line + optional body

Git walks the commit graph from `HEAD` backwards through parent pointers to produce the log.

## Implementation Notes

**Basic log:**
```bash
git log                                # full log, oldest last
git log --oneline                      # one line per commit: short hash + subject
```

**Visualise branch structure:**
```bash
git log --oneline --decorate --graph          # ASCII branch graph
git log --oneline --decorate --graph --all    # include all branches
```

**Useful flags:**

| Flag | Description |
|------|-------------|
| `--oneline` | Short hash + subject only |
| `--decorate` | Show branch/tag refs next to commits |
| `--graph` | ASCII graph of branch topology |
| `--all` | Include all branches and remotes |
| `--parents` | Show parent commit hashes |
| `-n <N>` | Limit to last N commits |
| `--author=<name>` | Filter by author |
| `--since=<date>` | Filter by date (e.g. `--since="2 weeks ago"`) |
| `--grep=<pattern>` | Filter by commit message |
| `--follow <file>` | Follow file renames |
| `-p` | Show diff for each commit (patch) |
| `--stat` | Show file change statistics per commit |

**Log of a remote branch:**
```bash
git log origin/main
```

**Search commit content:**
```bash
git log -S "function_name"             # commits that added/removed string (pickaxe)
git log -G "regex"                     # commits where diff matches regex
```

**Referencing commits:**
```bash
HEAD~1        # one commit before HEAD
HEAD~3        # three commits before HEAD
<hash>        # any commit by (abbreviated) SHA
```

## Trade-offs

- `--graph --all --oneline --decorate` is verbose but invaluable for understanding complex merge/rebase histories.
- `git log -p` produces a lot of output; pipe to `less` or use `git show <hash>` to inspect a single commit.
- `--follow` is necessary when a file was renamed; without it, log stops at the rename.

## References

- [git-log documentation](https://git-scm.com/docs/git-log)

## Links
- [[git_internals|Git Internals]]
- [[git_staging|Git Staging Area]]
- [[git_reset|Git Reset]]
