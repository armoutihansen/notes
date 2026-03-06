---
layer: 03_software_engineering
type: engineering
tool: GitHub
status: growing
tags: [github, git-workflow, pull-requests, trunk-based, branching-strategy, code-review]
created: 2026-03-02
---

# GitHub Workflows

## Purpose

GitHub Workflows describes the *social and process* layer on top of Git: how teams branch, review, merge, and release code using GitHub features. The dominant patterns are **GitHub Flow** (simple, continuous delivery) and **trunk-based development** (advanced, high-frequency integration). Distinct from [[github_actions|GitHub Actions]] which covers the automation/CI platform.

## Architecture

### GitHub Flow (default for most projects)

```
main  ────●─────────────────────────●──────────────────────►
           \                       /
  feature   ●──●──●  (open PR)  ●─●  (merge after review)
```

1. Branch from `main` — `git checkout -b feat/my-feature`
2. Commit small, focused changes
3. Open PR → CI runs → reviewers approve
4. Merge to `main` (squash, merge commit, or rebase)
5. Delete branch; deploy from `main`

### Trunk-Based Development

```
main  ────●──●──●──●──●──●──●──►    (commits every day, feature flags hide WIP)
```

- Everyone integrates to `main` (or `trunk`) at least once per day
- Feature flags control exposure, not long-lived branches
- Requires strong CI and good test coverage
- Scales well for large teams with high velocity

### Release Branching (for versioned software)

```
main  ──●──●──●──●──●──●──────►
         \
 release  ●──● (cherry-pick hotfixes) ──► tag v1.0
```

For libraries, CLI tools, or products requiring versioned releases. Hotfixes are cherry-picked to both the release branch and `main`.

## Implementation Notes

### Pull Request Best Practices

**Author checklist:**
- ≤ 400 lines changed per PR (smaller is more reviewable)
- PR description states *what* changed and *why* (not *how*)
- Link to issue or ticket
- Include screenshots for UI changes
- Self-review the diff before requesting review

**Reviewer checklist:**
- Read the PR description first
- Pull and run locally for non-trivial changes
- Approve only when you could defend the code in production
- Request changes rather than approving-with-comments if blocking issues exist

### Branch Protection Rules

Configure under *Repo Settings → Branches → Add Rule*:
- ✅ **Require a pull request before merging** — prevents direct pushes to `main`
- ✅ **Require status checks to pass** — CI must be green
- ✅ **Require approvals** — ≥1 or ≥2 reviewers
- ✅ **Dismiss stale reviews when new commits are pushed**
- ✅ **Require linear history** — enforces squash or rebase merges

### Merge Strategies

| Strategy | Effect | When to use |
|----------|--------|-------------|
| **Merge commit** | Preserves all commits + merge node | Audit trail important |
| **Squash merge** | All commits collapsed to one | Clean `main` history, feature branches |
| **Rebase merge** | Replays commits on top of main | Linear history, no merge commits |

Most teams use **squash merge** for feature branches; **merge commit** for release branches.

### GitHub CLI

```bash
# Create a PR
gh pr create --title "Add feature X" --body "Closes #42" --reviewer @alice

# Check out a PR locally
gh pr checkout 42

# List open PRs
gh pr list --assignee @me

# View CI status
gh pr checks

# Merge a PR
gh pr merge 42 --squash --delete-branch
```

### Code Owners

`.github/CODEOWNERS` — auto-requests review from specific owners when matching files change:
```
# Syntax: pattern   owners
*                   @global-owner
/src/auth/          @security-team
*.tf                @infra-team @devops-lead
docs/               @docs-team
```

### Dependabot for Automated Dependency Updates

`.github/dependabot.yml`:
```yaml
version: 2
updates:
  - package-ecosystem: "pip"
    directory: "/"
    schedule:
      interval: "weekly"
    reviewers: ["alice"]
    labels: ["dependencies"]
```

Dependabot opens PRs to bump outdated dependencies. Merge them to keep your supply chain fresh.

## Trade-offs

| Pattern | Pro | Con |
|---------|-----|-----|
| GitHub Flow | Simple, low overhead | Hard for teams needing stable release branches |
| Trunk-based | Fast integration, small conflicts | Requires feature flags and disciplined CI |
| Release branching | Stable release management | Long-lived branches, frequent merge conflicts |
| Squash merge | Clean history | Loses individual commit context |
| Rebase merge | Linear history | Rewrites commit SHAs (confusing in large teams) |

## References

- [GitHub Flow](https://docs.github.com/en/get-started/using-github/github-flow)
- [Trunk Based Development](https://trunkbaseddevelopment.com/)
- [About pull requests](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests)
- [GitHub CLI](https://cli.github.com/)
- [CODEOWNERS syntax](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners)

## Links
- [[github_actions|GitHub Actions]]
- [[github_permissions|GitHub Permissions]]
- [[github_repository_management|GitHub Repository Management]]
- [[00_git/git_workflow|Git Workflow]]
- [[00_git/git_pull_requests|Git Pull Requests]]
- [[cicd_pipelines|CI/CD Pipelines]]
