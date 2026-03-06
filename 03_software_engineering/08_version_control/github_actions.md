---
layer: 03_software_engineering
type: engineering
tool: GitHub Actions
status: growing
tags: [github-actions, ci-cd, automation, workflows, runners, secrets]
created: 2026-03-02
---

# GitHub Actions

## Purpose

GitHub Actions is GitHub's native CI/CD and automation platform. Workflows are YAML files in `.github/workflows/` that run on events (push, PR, schedule, manual dispatch). Each workflow is composed of jobs, which run on a runner and execute a sequence of steps. Actions from the marketplace can be composed like functions. See [[cicd_pipelines|CI/CD Pipelines]] for GitHub Actions used as a full ML CI/CD pipeline.

## Architecture

```
Event (push / pull_request / schedule / workflow_dispatch)
    │
    ▼
Workflow (.github/workflows/*.yml)
    ├── Job A  (runs-on: ubuntu-latest)
    │   ├── Step 1: actions/checkout@v4
    │   ├── Step 2: actions/setup-python@v5
    │   ├── Step 3: pip install ...
    │   └── Step 4: pytest
    └── Job B  (needs: [A])        ← sequential dependency
        └── Step 1: deploy
```

Jobs run in parallel by default; `needs:` creates sequential dependencies. Each job runs in a fresh VM or container.

## Implementation Notes

### Anatomy of a Workflow

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["3.11", "3.12"]

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}

      - name: Install dependencies
        run: pip install -e ".[test]"

      - name: Run tests
        run: pytest --cov=src --cov-report=xml

      - uses: codecov/codecov-action@v4
        with:
          token: ${{ secrets.CODECOV_TOKEN }}
```

### Caching Dependencies

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.cache/pip
    key: ${{ runner.os }}-pip-${{ hashFiles('**/requirements*.txt') }}
    restore-keys: |
      ${{ runner.os }}-pip-
```

### Secrets and Environment Variables

```yaml
- name: Deploy
  env:
    API_KEY: ${{ secrets.MY_API_KEY }}     # encrypted secret from repo settings
    DEBUG: "false"                         # plain env var
  run: ./deploy.sh
```

Secrets are set in *Repo Settings → Secrets and variables → Actions*. They are never exposed in logs (GitHub replaces them with `***`).

### Conditional Steps and Jobs

```yaml
- name: Deploy to production
  if: github.ref == 'refs/heads/main' && github.event_name == 'push'
  run: ./deploy.sh

# Skip job if no relevant files changed
jobs:
  test:
    if: github.event.pull_request.draft == false
```

### Reusable Workflows

```yaml
# .github/workflows/reusable-test.yml
on:
  workflow_call:
    inputs:
      python-version:
        required: true
        type: string

# Caller
jobs:
  call:
    uses: ./.github/workflows/reusable-test.yml
    with:
      python-version: "3.12"
```

### Docker Build and Push

```yaml
- uses: docker/setup-buildx-action@v3
- uses: docker/login-action@v3
  with:
    registry: ghcr.io
    username: ${{ github.actor }}
    password: ${{ secrets.GITHUB_TOKEN }}

- uses: docker/build-push-action@v5
  with:
    context: .
    push: ${{ github.ref == 'refs/heads/main' }}
    tags: ghcr.io/${{ github.repository }}:latest
    cache-from: type=gha
    cache-to: type=gha,mode=max
```

### Self-Hosted Runners

Register a runner:
```bash
# On the runner machine
./config.sh --url https://github.com/ORG/REPO --token TOKEN
./run.sh    # or install as a service
```

Use in workflow:
```yaml
runs-on: self-hosted  # or label like: [self-hosted, gpu]
```

Self-hosted runners are required for: GPU jobs, large RAM, private network access, or custom software.

### OIDC for Cloud Authentication (No Long-Lived Secrets)

```yaml
permissions:
  id-token: write
  contents: read

- uses: aws-actions/configure-aws-credentials@v4
  with:
    role-to-assume: arn:aws:iam::123456789:role/github-actions
    aws-region: eu-west-1
```

Configure the IAM role to trust GitHub's OIDC provider for the specific repo and branch.

## Trade-offs

| Choice | Pro | Con |
|--------|-----|-----|
| GitHub-hosted runners | Zero infra, auto-updated | No GPU, 7 GB RAM, slow for large deps |
| Self-hosted runners | GPU, custom env, fast | Infra overhead, security responsibility |
| `actions/cache` | Speeds up repeated installs | Cache invalidation on key change |
| Matrix builds | Wide compatibility coverage | Long wall-clock time, high minutes usage |
| OIDC cloud auth | No long-lived secrets | Per-cloud setup required |
| Reusable workflows | DRY, versioned | Harder to debug cross-repo |

## References

- [GitHub Actions documentation](https://docs.github.com/en/actions)
- [actions/checkout](https://github.com/actions/checkout)
- [actions/cache](https://github.com/actions/cache)
- [docker/build-push-action](https://github.com/docker/build-push-action)
- [OIDC in GitHub Actions](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect)

## Links
- [[cicd_pipelines|CI/CD Pipelines]]
- [[github_workflows|GitHub Workflows]]
- [[github_permissions|GitHub Permissions]]
- [[docker_patterns|Docker Patterns]]
- [[04_ml_engineering/08_infrastructure_and_platform/ml_environment_management|ML Environment Management]]
