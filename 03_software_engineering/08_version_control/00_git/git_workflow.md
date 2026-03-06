---
layer: 03_software_engineering
type: engineering
tool: git
status: growing
tags:
  - git
created: 2026-03-02
---

# Git Workflow

## Purpose

A Git workflow defines the conventions a team (or solo developer) uses to structure branches, commits, and merges. Choosing a consistent workflow reduces integration friction and keeps history navigable.

## Architecture

### Solo workflow

```
main  ──A──B──C──D──  (all commits directly on main)
```

Simple: commit directly to `main`. Appropriate for personal projects with no collaborators.

### Feature-branch workflow (standard team workflow)

```
main       ──A──────────────F──
                \          /
feature/x        B──C──D──E
```

1. Branch from `main` for every piece of work
2. Work on the feature branch; commit freely
3. Open a PR; get it reviewed
4. Merge (or squash-merge) into `main`; delete feature branch

This keeps `main` always in a stable, releasable state.

### Rebase-on-pull workflow

Instead of `git pull` (which merges), configure Git to rebase:
```bash
git config --global pull.rebase true
```

This keeps individual branches linear and avoids noisy "Merge branch 'main'" commits in history.

## Implementation Notes

**Typical feature-branch cycle:**
```bash
git switch main && git pull             # start from latest main
git switch -c feature/my-feature        # create feature branch

# ... do work, commit frequently ...

git fetch origin                        # get latest main
git rebase origin/main                  # rebase feature onto latest main (optional, before PR)
git push -u origin feature/my-feature  # push feature branch
# open PR on GitHub/GitLab
```

**After PR is merged, clean up:**
```bash
git switch main && git pull
git branch -d feature/my-feature
```

**Commit message convention** (Conventional Commits style):
```
feat: add login page
fix: correct null pointer in auth handler
docs: update API reference
refactor: extract validation helper
```
Format: `<type>: <short imperative description>`

## Trade-offs

- The feature-branch workflow adds overhead for solo projects but pays off when collaborating.
- Long-lived branches diverge from `main` and make merging painful — keep branches short-lived.
- Squash-merging (`git merge --squash`) keeps `main` history clean but loses individual commit context; use when feature commits are noisy.
- Trunk-based development (commit directly to `main` with feature flags) is preferred at high-velocity teams but requires strong CI and test coverage.

## References

- [Atlassian Git Workflow Guide](https://www.atlassian.com/git/tutorials/comparing-workflows)
- [Conventional Commits](https://www.conventionalcommits.org/)

## Links
- [[03_software_engineering/version_control/git/git_branching|Git Branching]]
- [[03_software_engineering/version_control/git/git_merge|Git Merge]]
- [[03_software_engineering/version_control/git/git_rebase|Git Rebase]]
- [[03_software_engineering/version_control/git/git_remotes|Git Remotes]]
- [[03_software_engineering/version_control/git/git_pull_requests|Pull Requests]]
