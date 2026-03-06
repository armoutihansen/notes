---
layer: 03_software_engineering
type: engineering
tool: GitHub
status: growing
tags: [github, repository, settings, releases, packages, topics, templates]
created: 2026-03-02
---

# GitHub Repository Management

## Purpose

GitHub repository management covers the settings, features, and practices for keeping a repo well-maintained: configuration, issue/PR templates, releases, GitHub Packages, and repository hygiene at scale. Complements [[github_workflows|GitHub Workflows]] (process) and [[github_permissions|GitHub Permissions]] (access control).

## Architecture

A GitHub repository is more than a Git remote — it includes:
- **Issues + Projects** — work tracking
- **Branches + Protection Rules** — code governance
- **Actions** — automation (see [[github_actions|GitHub Actions]])
- **Environments** — deployment gates
- **Packages** — container and package registry
- **Releases** — versioned artefacts with changelogs
- **Pages** — static site hosting (docs, reports)
- **Security features** — Dependabot, code scanning, secret scanning

## Implementation Notes

### Repository Settings Checklist

*Settings → General:*
- Default branch: `main`
- Merge buttons: enable **Squash merging** and **Rebase merging**; disable merge commits (or enable all three and set policy in branch protection)
- Auto-delete head branches: ✅ (keeps repo clean)
- Allow forking: ✅ or ❌ depending on open-source vs private

*Settings → Code security and analysis:*
- Dependabot alerts: ✅ always
- Dependabot security updates: ✅ always
- Secret scanning: ✅ always (prevents committed credentials)

### Issue and PR Templates

Create templates to standardise reports and reviews:

```
.github/
├── ISSUE_TEMPLATE/
│   ├── bug_report.yml
│   └── feature_request.yml
└── PULL_REQUEST_TEMPLATE.md
```

**`.github/ISSUE_TEMPLATE/bug_report.yml`:**
```yaml
name: Bug Report
description: File a bug report
labels: [bug, triage]
body:
  - type: textarea
    id: description
    attributes:
      label: Describe the bug
      placeholder: What went wrong?
    validations:
      required: true
  - type: textarea
    id: reproduce
    attributes:
      label: Steps to reproduce
```

**`.github/PULL_REQUEST_TEMPLATE.md`:**
```markdown
## Summary
<!-- What does this PR do? -->

## Related Issue
Closes #

## Checklist
- [ ] Tests added/updated
- [ ] Documentation updated
- [ ] Changelog entry added
```

### Releases and Tags

```bash
# Create a release tag
git tag -a v1.2.0 -m "Release 1.2.0"
git push origin v1.2.0

# GitHub CLI
gh release create v1.2.0 --title "v1.2.0" --notes "## Changes\n- Feature A\n- Fix B" dist/*.whl
```

**GitHub Release Actions:**
```yaml
- uses: softprops/action-gh-release@v2
  if: startsWith(github.ref, 'refs/tags/')
  with:
    files: dist/*.whl
    generate_release_notes: true  # auto-generate from merged PRs
```

### GitHub Packages (Container Registry)

```bash
# Push container image
docker tag myapp ghcr.io/myorg/myapp:latest
docker push ghcr.io/myorg/myapp:latest
```

Access control: packages inherit org permissions by default; can be configured independently under *Package Settings → Manage Access*.

### Repository Templates

Mark a repo as a template (*Settings → General → Template repository*) to let others create new repos with your directory structure, workflows, and initial files:

```
gh repo create myorg/new-project --template myorg/project-template --private
```

### Archive and Deprecation

When a repo is no longer maintained:
- *Settings → Danger Zone → Archive this repository* — makes it read-only but keeps all history
- Add a `NOTICE.md` or update `README.md` pointing to the successor
- Close open issues/PRs with an explanation

### Repository Insights

Useful for measuring team health:
- *Insights → Traffic* — clone counts, top referrers
- *Insights → Dependency graph* — upstream vulnerability exposure
- *Insights → Contributors* — commit activity by author

## Trade-offs

| Pattern | Pro | Con |
|---------|-----|-----|
| Squash merges | Clean history | Individual commits lost |
| Auto-delete branches | Clean remote | Accidental deletion if not fetched locally first |
| Branch protection on `main` | Safety net | Blocks hotfixes without overrides |
| Monorepo | Single source of truth | CI must use path-based filtering to avoid re-running all jobs |
| Template repos | Standardised setup | Templates drift from reality over time |

## References

- [GitHub repository settings](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features)
- [Issue templates](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests)
- [GitHub Packages](https://docs.github.com/en/packages)
- [Managing releases](https://docs.github.com/en/repositories/releasing-projects-on-github/managing-releases-in-a-repository)

## Links
- [[github_actions|GitHub Actions]]
- [[github_workflows|GitHub Workflows]]
- [[github_permissions|GitHub Permissions]]
- [[cicd_pipelines|CI/CD Pipelines]]
- [[docker_patterns|Docker Patterns]]
