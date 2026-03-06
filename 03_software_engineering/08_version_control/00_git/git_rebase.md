---
layer: 03_software_engineering
type: engineering
tool: git
status: growing
tags:
  - git
created: 2026-03-02
---

# Git Rebase

## Purpose

Rebase re-applies commits from one branch on top of another, producing a **linear history** without merge commits. It is commonly used to keep a feature branch up to date with `main`, or to clean up commits before merging.

## Architecture

Given:
```
A - B - C         main
   \
    D - E         feature
```

`git rebase main` (run on `feature`) replays `D` and `E` as new commits on top of `C`:
```
A - B - C         main
         \
          D'- E'  feature
```

`D'` and `E'` are **new commits** with new SHAs — the original `D` and `E` are discarded. The content is the same; the parent lineage is rewritten.

Rebase proceeds commit-by-commit. If a conflict arises at any step, Git pauses and lets you resolve it before continuing.

## Implementation Notes

**Rebase current branch onto another:**
```bash
git rebase <base>               # e.g. git rebase main
```

**Interactive rebase** (rewrite, squash, reorder, drop commits):
```bash
git rebase -i HEAD~<n>          # rebase last n commits interactively
git rebase -i <commit-hash>     # rebase from a specific commit
```
Interactive rebase options per commit:
- `pick` — keep as-is
- `reword` — keep, edit message
- `squash` / `fixup` — combine with previous commit
- `drop` — remove commit

**Handle conflicts during rebase:**
```bash
# resolve conflict in file, then:
git add <file>
git rebase --continue

# or abort entirely:
git rebase --abort
```

**Configure pull to rebase instead of merge:**
```bash
git config --global pull.rebase true
```

## Trade-offs

- **Linear history** is easier to read with `git log` and `git bisect`.
- **Never rebase a public/shared branch** (e.g. `main`). Rewriting SHAs that others have pulled causes divergent histories and requires force-pushing, which breaks collaborators' repos.
- Rebase is safe on your own local or private feature branches.
- Interactive rebase is powerful for polishing commit history before a PR, but rewriting too many commits increases conflict risk.
- If in doubt, use merge — it's always safe and preserves true history.

## References

- [git-rebase documentation](https://git-scm.com/docs/git-rebase)
- [Merging vs. Rebasing (Atlassian)](https://www.atlassian.com/git/tutorials/merging-vs-rebasing)

## Links
- [[03_software_engineering/version_control/git/git_merge|Git Merge]]
- [[03_software_engineering/version_control/git/git_branching|Git Branching]]
- [[03_software_engineering/version_control/git/git_workflow|Git Workflow]]
