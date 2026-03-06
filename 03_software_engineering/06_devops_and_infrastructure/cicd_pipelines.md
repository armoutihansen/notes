---
layer: 03_software_engineering
type: engineering
tool: GitHub Actions
status: growing
tags: [ci-cd, github-actions, devops, automation, ml-ops]
created: 2026-03-05
---

# CI/CD Pipelines

## Purpose

CI/CD automates the path from a developer's commit to a running service. Continuous Integration catches regressions early by running tests on every push. Continuous Delivery ensures every passing commit is releasable. Continuous Deployment pushes passing commits all the way to production automatically. In ML pipelines CI/CD also gates on data quality and model performance, not just code correctness.

## Architecture

```
  Developer push / PR
         │
         ▼
  ┌─────────────┐
  │  CI: Test   │  lint, unit tests, integration tests, type checks
  └──────┬──────┘
         │ pass
         ▼
  ┌──────────────────┐
  │  CI: Build Image │  docker build + push to registry
  └──────┬───────────┘
         │ on merge to main
         ▼
  ┌─────────────────────┐
  │  CD: Deploy Staging │  apply K8s manifests / Helm upgrade
  └──────┬──────────────┘
         │ smoke tests pass
         ▼
  ┌──────────────────────────┐
  │  CD: Deploy Production   │  manual gate or automatic
  └──────────────────────────┘
```

## Implementation Notes

### GitHub Actions Workflow Structure

```yaml
name: CI

# Trigger conditions
on:
  push:
    branches: [main, "release/**"]
  pull_request:
    branches: [main]
  workflow_dispatch:            # manual trigger

# Top-level env vars
env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  test:
    name: Test
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["3.11", "3.12"]
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}

      - name: Cache pip
        uses: actions/cache@v4
        with:
          path: ~/.cache/pip
          key: ${{ runner.os }}-pip-${{ hashFiles('**/requirements*.txt') }}
          restore-keys: |
            ${{ runner.os }}-pip-

      - run: pip install -r requirements-dev.txt
      - run: ruff check .
      - run: mypy src/
      - run: pytest --cov=src --cov-report=xml -q

      - uses: codecov/codecov-action@v4
        with:
          token: ${{ secrets.CODECOV_TOKEN }}

  build:
    name: Build & Push Image
    needs: test
    runs-on: ubuntu-latest
    # Only run on merge to main, not on PRs
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    permissions:
      contents: read
      packages: write

    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Log in to GHCR
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=sha,prefix=git-
            type=ref,event=branch
            type=semver,pattern={{version}}

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          build-args: |
            GIT_SHA=${{ github.sha }}

  deploy-staging:
    name: Deploy to Staging
    needs: build
    runs-on: ubuntu-latest
    environment: staging         # GitHub environment with required reviewers / branch protection

    steps:
      - uses: actions/checkout@v4

      - name: Install Helm
        uses: azure/setup-helm@v4

      - name: Configure kubectl
        uses: azure/k8s-set-context@v4
        with:
          kubeconfig: ${{ secrets.KUBECONFIG_STAGING }}

      - name: Helm upgrade
        run: |
          helm upgrade --install my-app ./chart \
            --namespace staging \
            --set image.tag=git-${{ github.sha }} \
            --set replicaCount=1 \
            --atomic \
            --timeout 5m \
            -f helm/values.staging.yaml
```

### Key Concepts

**`needs`** — declares job dependencies, enabling a DAG of jobs:
```yaml
jobs:
  lint:   ...
  test:   needs: lint
  build:  needs: test
  deploy: needs: build
```

**`matrix`** — runs a job across a combination of values:
```yaml
strategy:
  matrix:
    os: [ubuntu-latest, macos-latest]
    python: ["3.11", "3.12"]
  fail-fast: false               # don't cancel other matrix jobs on first failure
```

**`if` conditionals**:
```yaml
# Only on push to main
if: github.ref == 'refs/heads/main' && github.event_name == 'push'

# Skip on [skip ci] in commit message
if: "!contains(github.event.head_commit.message, '[skip ci]')"

# Only on manual trigger or schedule
if: github.event_name == 'workflow_dispatch' || github.event_name == 'schedule'
```

**Reusable workflows** — DRY across repositories:
```yaml
# .github/workflows/reusable-test.yml
on:
  workflow_call:
    inputs:
      python-version:
        type: string
        default: "3.12"
    secrets:
      codecov-token:
        required: true
```

```yaml
# caller workflow
jobs:
  test:
    uses: org/shared-workflows/.github/workflows/reusable-test.yml@main
    with:
      python-version: "3.12"
    secrets:
      codecov-token: ${{ secrets.CODECOV_TOKEN }}
```

### Caching

`actions/cache` is the primary caching primitive. Key design: make the cache key specific enough to avoid stale hits, broad enough to get cache hits across minor changes.

```yaml
# Python dependencies
- uses: actions/cache@v4
  with:
    path: |
      ~/.cache/pip
      .venv
    key: ${{ runner.os }}-py${{ matrix.python }}-${{ hashFiles('**/requirements*.txt', '**/pyproject.toml') }}
    restore-keys: |
      ${{ runner.os }}-py${{ matrix.python }}-
      ${{ runner.os }}-py-

# Node modules
- uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}

# Docker layer cache via GitHub Actions Cache backend
- uses: docker/build-push-action@v5
  with:
    cache-from: type=gha
    cache-to: type=gha,mode=max
```

### Secrets Management

Secrets are stored in GitHub repository/organisation/environment settings. In workflows they are accessed via `${{ secrets.MY_SECRET }}`. Rules:

- Secrets are **never** echoed in logs (GitHub masks them automatically).
- Use **environment secrets** to scope sensitive credentials to specific deployment environments.
- For dynamic secrets (short-lived tokens), use OIDC to federate with cloud providers without storing long-lived credentials.

```yaml
# OIDC with AWS — no stored AWS keys
- name: Configure AWS credentials
  uses: aws-actions/configure-aws-credentials@v4
  with:
    role-to-assume: arn:aws:iam::123456789012:role/github-actions
    aws-region: us-east-1
```

### ML-Specific CI/CD Patterns

**Data Validation Step**

Before training, validate that input data schema and statistics are within expected bounds:

```yaml
  validate-data:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run data validation
        run: |
          python scripts/validate_data.py \
            --schema schemas/training_data.json \
            --data gs://my-bucket/data/latest/
        env:
          GOOGLE_APPLICATION_CREDENTIALS_JSON: ${{ secrets.GCP_SA_KEY }}
```

**Model Evaluation Gate**

After training or when updating model serving code, run offline evaluation and block merge if performance regresses:

```yaml
  evaluate-model:
    runs-on: [self-hosted, gpu]
    steps:
      - name: Run evaluation
        run: python evaluate.py --model-path ${{ steps.train.outputs.model-path }} --threshold 0.92
      - name: Post metrics as PR comment
        uses: actions/github-script@v7
        with:
          script: |
            const metrics = require('./eval_results.json');
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `## Model Evaluation\n- Accuracy: ${metrics.accuracy}\n- F1: ${metrics.f1}`
            });
```

**Automated Retraining Trigger**

```yaml
name: Scheduled Retraining

on:
  schedule:
    - cron: "0 2 * * 1"        # every Monday at 02:00 UTC
  workflow_dispatch:
    inputs:
      force:
        description: "Force retrain even if data drift threshold not met"
        type: boolean
        default: false

jobs:
  check-drift:
    runs-on: ubuntu-latest
    outputs:
      should-retrain: ${{ steps.drift.outputs.should-retrain }}
    steps:
      - name: Check data drift
        id: drift
        run: |
          DRIFT=$(python scripts/check_drift.py)
          echo "should-retrain=$DRIFT" >> $GITHUB_OUTPUT

  retrain:
    needs: check-drift
    if: needs.check-drift.outputs.should-retrain == 'true' || inputs.force == 'true'
    uses: ./.github/workflows/train.yml
    secrets: inherit
```

**Self-Hosted Runners for GPU Jobs**

GPU jobs can't run on GitHub-hosted runners. Use self-hosted runners registered on your GPU machines or K8s pods (via [actions-runner-controller](https://github.com/actions/actions-runner-controller)):

```yaml
  train:
    runs-on: [self-hosted, gpu, linux]
```

```yaml
  train:
    runs-on: [self-hosted, a100]
```

### Workflow Patterns Reference

| Pattern | Trigger | Use Case |
|---------|---------|----------|
| Test on PR | `pull_request` | Catch regressions before merge |
| Build + push image | `push` to main | Produce deployable artefact |
| Deploy staging | After build job | Automated staging deploy |
| Deploy production | GitHub environment approval | Human gate before prod |
| Scheduled retraining | `schedule` cron | Data drift / freshness |
| Release drafter | `push` to main | Auto-generate changelog |
| Dependency updates | `schedule` | Keep deps fresh (Dependabot) |

## Trade-offs

| Approach | Pro | Con |
|----------|-----|-----|
| GitHub-hosted runners | Zero infra, always fresh | No GPU, limited RAM, slow for large deps |
| Self-hosted runners | GPU, custom env, fast | Infra overhead, security responsibility |
| Matrix builds | Broad compatibility coverage | Long wall-clock time, high minutes usage |
| OIDC for cloud auth | No long-lived secrets | Cloud-specific setup per provider |
| Monorepo path filters | Only test affected code | Complex filter rules |

## References

- [GitHub Actions documentation](https://docs.github.com/en/actions)
- [actions/cache](https://github.com/actions/cache)
- [docker/build-push-action](https://github.com/docker/build-push-action)
- [actions-runner-controller](https://github.com/actions/actions-runner-controller)
- [OpenID Connect in GitHub Actions](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect)

## Links
- [[docker_patterns|Docker]]
- [[kubernetes_basics|Kubernetes]]
- [[github_actions|GitHub Actions]]
- [[04_ml_engineering/05_deployment_and_serving/rollout_strategies|Rollout Strategies]]
