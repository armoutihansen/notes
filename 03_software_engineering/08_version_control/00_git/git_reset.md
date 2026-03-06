---
layer: 03_software_engineering
type: engineering
tool: git
status: growing
tags:
  - git
created: 2026-03-02
---

# Git Reset

## Purpose

`git reset` moves the `HEAD` (and current branch tip) to a specified commit, optionally changing the state of the index and working tree. It is the primary tool for undoing commits and unstaging changes.

## Architecture

`git reset` can operate at three levels:

| Mode | Branch tip | Index (staging) | Working tree |
|------|-----------|-----------------|--------------|
| `--soft` | ✅ moved | unchanged | unchanged |
| `--mixed` (default) | ✅ moved | ✅ reset | unchanged |
| `--hard` | ✅ moved | ✅ reset | ✅ reset |

- **`--soft`**: Commits are undone, but all changes remain staged — ready to recommit differently.
- **`--mixed`**: Commits are undone and changes are unstaged — they appear as modified files in the working tree.
- **`--hard`**: Commits are undone and changes are **permanently discarded** from both index and working tree.

## Implementation Notes

**Syntax:**
```bash
git reset [--soft | --mixed | --hard] <commit>
```

**Common patterns:**

```bash
# Undo last commit, keep changes staged:
git reset --soft HEAD~1

# Undo last commit, keep changes unstaged (default):
git reset HEAD~1

# Undo last 3 commits, discard all changes (destructive):
git reset --hard HEAD~3

# Reset to a specific commit:
git reset --hard 5ba786f

# Unstage a file (without touching the commit):
git restore --staged <file>     # modern equivalent of: git reset HEAD <file>
```

**Recover a hard-reset commit:** Commits are not immediately deleted. Use `git reflog` to find the old hash and reset back to it — within the reflog expiry window (default 90 days).

```bash
git reflog                              # find the lost commit hash
git reset --hard <recovered-hash>
```

## Trade-offs

- `--hard` is irreversible from a UX perspective (though technically recoverable via reflog). Be certain before using it.
- `git reset` rewrites branch history — **do not reset commits that have been pushed** to a shared remote. Use `git revert` instead, which creates a new commit that undoes changes without rewriting history.
- `--soft` is useful for squashing several messy commits into one clean commit.

## References

- [git-reset documentation](https://git-scm.com/docs/git-reset)
- [git-reflog documentation](https://git-scm.com/docs/git-reflog)

## Links
- [[git_staging|Git Staging Area]]
- [[git_log|Git Log]]
- [[git_rebase|Git Rebase]]
