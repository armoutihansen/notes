---
layer: 03_software_engineering
type: engineering
tool: git
status: growing
tags:
  - git
created: 2026-03-02
---

# Git Branching

## Purpose

Branches let you isolate independent lines of work — a feature, a bug fix, an experiment — without affecting the main codebase. Because a branch is just a lightweight pointer to a commit, creating and switching branches is nearly instantaneous.

## Architecture

A **branch** is a named ref (a file in `.git/refs/heads/`) that points to a specific commit — the *tip* of that branch. When you commit on a branch, the ref automatically advances to the new commit.

**HEAD** is a special ref that points to the currently checked-out branch (or directly to a commit in "detached HEAD" state). Moving HEAD is what `git switch` does.

```
    D - E   feature
   /
A - B - C   main  ← HEAD
```

Creating a branch from `main` at `C` gives both branches the same history up to `C`; commits after that diverge independently.

## Implementation Notes

**View branches:**
```bash
git branch              # list local branches (* = current)
git branch -a           # list local + remote-tracking branches
```

**Create and switch:**
```bash
git switch -c <name>            # create and switch (preferred)
git switch <name>               # switch to existing branch
git branch <name>               # create without switching

# Legacy equivalent:
git checkout -b <name>
git checkout <name>
```

**Rename:**
```bash
git branch -m <old> <new>
```

**Delete:**
```bash
git branch -d <name>            # delete (safe — refuses if unmerged)
git branch -D <name>            # force delete
```

**Default branch:** `main` is the modern convention (GitHub default); older repos use `master`. Rename with:
```bash
git branch -m master main
```

## Trade-offs

- Branches are cheap — create them freely. The cost is in merging and keeping them up to date with `main`.
- Short-lived feature branches (hours to a few days) minimize merge conflicts; long-lived branches accumulate drift and make merging painful.
- Prefer `git switch` over `git checkout` for branch operations — it's clearer and was designed specifically for this purpose.

## References

- [git-branch documentation](https://git-scm.com/docs/git-branch)
- [git-switch documentation](https://git-scm.com/docs/git-switch)

## Links
- [[git_merge|Git Merge]]
- [[git_rebase|Git Rebase]]
- [[git_workflow|Git Workflow]]
