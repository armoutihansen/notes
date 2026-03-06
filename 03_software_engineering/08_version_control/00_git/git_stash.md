---
layer: 03_software_engineering
type: engineering
tool: git
status: growing
tags:
  - git
created: 2026-03-02
---

# Git Stash

## Purpose

`git stash` temporarily shelves (stashes) uncommitted changes so you can switch context — e.g., check out a different branch to fix a bug — without committing half-finished work.

## Architecture

The stash is a stack of WIP (work-in-progress) entries stored in the ref `refs/stash`. Each entry records:
- A commit for the index state
- A commit for the working tree state
- An optional commit for untracked files (with `--include-untracked`)

Popping restores the saved state and removes the entry from the stack; applying restores without removing.

## Implementation Notes

**Save and restore:**
```bash
git stash                               # stash tracked modified files
git stash -u                            # also stash untracked files
git stash -m "message"                  # stash with descriptive name

git stash pop                           # apply most recent stash and remove it
git stash apply                         # apply most recent stash, keep it in stack
git stash apply stash@{2}               # apply a specific stash entry
```

**Inspect the stash stack:**
```bash
git stash list                          # list all stash entries
git stash show                          # show summary of most recent stash
git stash show -p                       # show diff of most recent stash
```

**Clean up:**
```bash
git stash drop                          # remove most recent stash entry
git stash drop stash@{2}               # remove specific entry
git stash clear                         # remove all stash entries
```

**Create a branch from a stash** (useful if context diverged):
```bash
git stash branch <branch-name>          # create branch at stash point, apply stash, pop
```

## Trade-offs

- Stashes are local — they are not pushed to remotes.
- Stash conflicts are possible when `pop`/`apply` is called on a working tree that has diverged from when the stash was created. Resolve them like merge conflicts.
- For longer-lived WIP, consider committing with `git commit --no-verify -m "WIP: ..."` and amending later — commits are more durable and visible than stash entries.
- `git stash clear` is irreversible (stash entries are eventually garbage-collected).

## References

- [git-stash documentation](https://git-scm.com/docs/git-stash)

## Links
- [[03_software_engineering/version_control/git/git_staging|Git Staging Area]]
- [[03_software_engineering/version_control/git/git_branching|Git Branching]]
