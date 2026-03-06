---
layer: 03_software_engineering
type: engineering
tool: git
status: growing
tags:
  - git
created: 2026-03-02
---

# Git Staging Area

## Purpose

The staging area (index) is Git's intermediate layer between your working directory and the repository. It allows you to craft commits precisely — selecting exactly which changes go into the next commit — rather than committing everything at once.

## Architecture

Git maintains three distinct areas:

| Area | Location | Description |
|------|----------|-------------|
| Working tree | Your filesystem | Files as they exist on disk |
| Index (staging area) | `.git/index` | Snapshot of what will go into the next commit |
| Repository | `.git/objects/` | All committed snapshots |

A typical change moves: **working tree → index → repository**.

## Implementation Notes

**Check current state:**
```bash
git status              # show staged, unstaged, and untracked changes
git diff                # diff: working tree vs index (unstaged changes)
git diff --staged       # diff: index vs last commit (staged changes)
```

**Stage changes:**
```bash
git add <file>          # stage a specific file
git add .               # stage all changes in current directory
git add -p              # interactively stage hunks (partial file staging)
```

**Commit staged changes:**
```bash
git commit -m "message"              # commit with inline message
git commit                           # open editor for message
git commit --amend                   # amend the most recent commit (rewrites history)
git commit --amend --no-edit         # amend without changing the message
```

**Unstage / discard:**
```bash
git restore --staged <file>          # unstage a file (keep working tree changes)
git restore <file>                   # discard working tree changes (irreversible)
git rm --cached <file>               # remove from index only (stop tracking)
```

## Trade-offs

- `git add -p` is powerful for keeping commits atomic and focused, but requires discipline.
- `git commit --amend` rewrites the last commit's SHA — only use it before pushing; amending a pushed commit forces a push and disrupts collaborators.
- `git restore <file>` discards working tree changes permanently (no undo); prefer `git stash` if you might want them later.

## References

- [git-add documentation](https://git-scm.com/docs/git-add)
- [git-commit documentation](https://git-scm.com/docs/git-commit)

## Links
- [[git_internals|Git Internals]]
- [[git_stash|Git Stash]]
- [[git_log|Git Log]]
