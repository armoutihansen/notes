---
layer: 03_software_engineering
type: engineering
tool: git
status: growing
tags:
  - git
created: 2026-03-02
---

# Git Merge

## Purpose

Merge integrates the history of one branch into another, combining divergent work. It is the standard way to bring a completed feature branch back into `main`.

## Architecture

Git finds the **merge base** — the best common ancestor commit of the two branch tips — then applies the changes from both branches relative to that base.

**Three-way merge** (branches have diverged):
```
A - B - C - F    main
   \       /
    D - E        feature
```
`F` is a **merge commit** with two parents (`C` and `E`). It preserves the full branching history.

**Fast-forward merge** (no divergence — one branch is a direct ancestor of the other):
```
Before:          After:
A - B     main   A - B - C   main
     \                        feature (deleted)
      C   feature
```
No merge commit is created; `main` simply advances to `C`.

## Implementation Notes

**Merge a branch into the current branch:**
```bash
git merge <branch>
```

**Force a merge commit even if fast-forward is possible:**
```bash
git merge --no-ff <branch>
```

**Abort an in-progress merge (e.g. after conflicts):**
```bash
git merge --abort
```

**Merge a remote branch:**
```bash
git merge origin/<branch>
```

**Resolving conflicts:** When Git cannot auto-merge, it marks conflict regions in the file:
```
<<<<<<< HEAD
current branch version
=======
incoming branch version
>>>>>>> feature
```
Edit the file, then stage and commit:
```bash
git add <resolved-file>
git commit
```

## Trade-offs

- Merge preserves the true branching history — useful for traceability but can make `git log` noisy.
- Fast-forward merges produce a clean linear history without extra commits.
- `--no-ff` is useful when you want a clear record that a feature branch existed, even if it could fast-forward.
- For very large teams, squash merges (`git merge --squash`) collapse feature branch commits into one, keeping `main` history tidy.

## References

- [git-merge documentation](https://git-scm.com/docs/git-merge)

## Links
- [[03_software_engineering/version_control/git/git_branching|Git Branching]]
- [[03_software_engineering/version_control/git/git_rebase|Git Rebase]]
- [[03_software_engineering/version_control/git/git_workflow|Git Workflow]]
- [[03_software_engineering/version_control/git/git_pull_requests|Pull Requests]]
