---
layer: 03_software_engineering
type: engineering
tool: git
status: growing
tags:
  - git
created: 2026-03-02
---

# Pull Requests

## Purpose

A pull request (PR) — also called a merge request (MR) on GitLab — is a proposal to merge a branch into another. It serves as a structured code review checkpoint before changes reach the main branch.

## Architecture

A PR exists on the hosting platform (GitHub, GitLab, Bitbucket), not in Git itself. It tracks:
- **Base branch** — the target (usually `main`)
- **Compare branch** — the feature branch with changes
- **Diff** — all commits and file changes in the compare branch not yet in base
- **Review state** — comments, approvals, requested changes, CI status

The PR is "open" while under review, and "merged" or "closed" once resolved.

## Implementation Notes

**Open a PR:**
1. Push a feature branch to the remote (`git push -u origin feature/x`)
2. Go to the repo on GitHub/GitLab — a banner typically appears to open a PR
3. Set base branch, write a description, assign reviewers and labels

**PR description best practices:**
- Explain *why* the change is needed, not just *what* it does
- Reference related issues: `Closes #42` auto-closes the issue on merge
- Include screenshots or test steps for UI changes

**Merge strategies available on most platforms:**

| Strategy | Result | When to use |
|----------|--------|-------------|
| Merge commit | Preserves full branch history, adds merge commit | Maintain history, team preference |
| Squash and merge | All commits squashed into one on `main` | Noisy feature commits, clean main |
| Rebase and merge | Feature commits replayed linearly onto main | Linear history, atomic commits |

**After merging:**
- Delete the remote feature branch (most platforms offer a one-click delete)
- Locally: `git switch main && git pull && git branch -d feature/x`

## Trade-offs

- PRs slow down solo work but are essential for team quality control.
- Squash merge loses commit granularity — useful if feature commits were messy, but you lose the story of how the feature evolved.
- Very large PRs are hard to review thoroughly; keep PRs small and focused (<400 lines is a common guideline).
- Draft PRs allow early feedback without being ready to merge — use them for WIP that needs visibility.

## References

- [GitHub Docs: Pull Requests](https://docs.github.com/en/pull-requests)
- [GitLab Docs: Merge Requests](https://docs.gitlab.com/ee/user/project/merge_requests/)

## Links
- [[git_branching|Git Branching]]
- [[git_merge|Git Merge]]
- [[git_rebase|Git Rebase]]
- [[git_workflow|Git Workflow]]
- [[git_remotes|Git Remotes]]
