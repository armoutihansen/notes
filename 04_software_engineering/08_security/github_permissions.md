---
layer: 04_software_engineering
type: engineering
tool: GitHub
status: growing
tags: [pattern]
created: 2026-03-02
---

# GitHub Permissions

## Purpose

GitHub's permission model controls access to repositories, organisations, and automation tokens. Misconfigured permissions are a leading source of supply-chain vulnerabilities and accidental secret exposure. This note covers organisation/team/repo roles, token types, `GITHUB_TOKEN` scopes, and fine-grained permission policies.

## Architecture

```
GitHub Organization
├── Teams (groups of users with inherited permissions)
│   ├── Read / Triage / Write / Maintain / Admin
│   └── Assigned to repos with specific roles
├── Repositories
│   ├── Collaborators (direct user → repo role)
│   └── Branch protection rules
└── Apps and Tokens
    ├── GITHUB_TOKEN (short-lived, workflow-scoped)
    ├── Classic PAT (long-lived, user-scoped, coarse-grained)
    └── Fine-grained PAT (repo-scoped, permission-scoped, expiry)
```

## Implementation Notes

### Repository and Organisation Roles

| Role | Can do |
|------|--------|
| **Read** | View/clone repo, create issues |
| **Triage** | + label/close issues, but not push |
| **Write** | + push to non-protected branches, manage labels |
| **Maintain** | + manage repo settings (not sensitive ones) |
| **Admin** | Full control including deleting repo, managing members |

Organisation members inherit org-level base permissions; team membership overrides base for assigned repos.

### Token Types

**Classic PAT** (legacy)
- Grants permissions across all repos the user has access to
- No expiry enforcement; stored with `repo`, `workflow`, `packages` scopes
- Avoid for new projects — overly broad

**Fine-grained PAT** (recommended)
- Scoped to specific repos only
- Per-repo permissions (contents: read, pull_requests: write, etc.)
- Mandatory expiry (max 1 year)
- Require org owner approval if org policy enforces it

```
GitHub → Settings → Developer settings → Personal access tokens → Fine-grained tokens
```

**`GITHUB_TOKEN`** (for Actions)
- Automatically minted at workflow start; expires when workflow ends
- Scoped to the repo running the workflow
- Default permissions configurable in org/repo settings

```yaml
# Workflow: restrict to minimum necessary permissions
permissions:
  contents: read
  pull-requests: write
  id-token: write    # for OIDC
```

Set org/repo default to `read-only` and only expand per-workflow or per-job.

### Branch Protection Rules

Critical for protecting `main` / `release` branches:

```
Repo Settings → Branches → Branch protection rules → Add rule
```

Recommended settings:
- ✅ Require a pull request before merging
- ✅ Require approvals: 1 (small team) or 2 (critical repos)
- ✅ Dismiss stale reviews when new commits are pushed
- ✅ Require status checks to pass before merging (link your CI jobs)
- ✅ Require linear history (enforces squash or rebase)
- ✅ Restrict who can push to matching branches (only release managers)
- ✅ Include administrators (avoid "admin bypass" loophole)

### Environments and Deployment Gates

Environments add an approval layer before sensitive deployments:

```
Repo Settings → Environments → New environment → Configure
```

```yaml
jobs:
  deploy-prod:
    environment:
      name: production
      url: https://myapp.example.com
    runs-on: ubuntu-latest
    steps:
      - run: ./deploy.sh
```

With `Required reviewers` set, the workflow pauses until an approver clicks "Approve and deploy" in the GitHub UI. Combined with OIDC, this means cloud credentials are only issued after human approval.

### Audit Log

Organisation admins can review all permission changes, token usage, and repo access via *Organisation Settings → Audit log* or the REST API:

```bash
gh api /orgs/myorg/audit-log --paginate | jq '.[] | select(.action=="org.invite_member")'
```

## Trade-offs

| Pattern | Pro | Con |
|---------|-----|-----|
| Fine-grained PAT | Minimal blast radius | Must set expiry; approval required in some orgs |
| Classic PAT | Simple setup | All repos accessible; long-lived; easy to leak |
| `GITHUB_TOKEN` with `read-all` default | No setup | Over-permissioned for most workflows |
| `GITHUB_TOKEN` with `permissions:` block | Least-privilege per workflow | Verbose YAML |
| OIDC cloud auth | No stored secrets | Per-provider IAM setup required |
| Environment approvals | Human gate before production | Adds latency to CD pipelines |

## References

- [GitHub permissions documentation](https://docs.github.com/en/organizations/managing-user-access-to-your-organizations-repositories/managing-repository-roles/repository-roles-for-an-organization)
- [Fine-grained PATs](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-fine-grained-personal-access-token)
- [GITHUB_TOKEN permissions](https://docs.github.com/en/actions/security-guides/automatic-token-authentication)
- [Deployment environments](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment)

## Links
- [[github_actions|GitHub Actions]]
- [[github_workflows|GitHub Workflows]]
- [[github_repository_management|GitHub Repository Management]]
- [[cicd_pipelines|CI/CD Pipelines]]
- [[04_software_engineering/08_security/filesystem_sandboxing|Filesystem Sandboxing]]
