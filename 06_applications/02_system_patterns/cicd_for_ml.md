---
layer: 06_applications
type: application
status: growing
tags: [pattern, mlops, deployment]
created: 2026-03-10
---

# CI/CD for ML Pipelines

## Purpose

Implements continuous integration and continuous delivery for machine learning systems using GitHub Actions. Extends standard software CI/CD with ML-specific gates: data pipeline validation, model evaluation against performance thresholds, and automated model registration. Every pull request that touches training code, features, or model config is validated before merge.

### Examples

**Training pipeline CI**: On every PR touching `src/` or `configs/`, run the full training pipeline on a sample of data, check that validation AUC exceeds a threshold, and report metrics as a PR comment.

**Model CD**: On merge to `main`, retrain on full data, register the new model version in MLflow, and deploy to the staging environment as a canary if metrics exceed the champion's baseline.

## Architecture

### Repository Layout

```
.
├── .github/workflows/
│   ├── ci.yml              # lint, unit tests, data pipeline check
│   ├── train_validate.yml  # model training + evaluation gate
│   └── deploy.yml          # model registry update + deployment
├── src/
│   ├── train.py
│   ├── evaluate.py
│   └── serve.py
├── tests/
│   ├── test_features.py    # unit tests for feature engineering
│   └── test_pipeline.py    # integration tests for pipeline stages
├── dvc.yaml                # data pipeline definition
└── params.yaml             # hyperparameters
```

### CI Workflow — Lint, Test, and Data Validation

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  PYTHON_VERSION: "3.11"

jobs:
  lint-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ env.PYTHON_VERSION }}
          cache: pip

      - name: Install dependencies
        run: pip install -r requirements-dev.txt

      - name: Lint (ruff)
        run: ruff check src/ tests/

      - name: Type check (mypy)
        run: mypy src/

      - name: Unit tests
        run: pytest tests/test_features.py tests/test_pipeline.py -v --cov=src --cov-report=xml

      - name: Upload coverage
        uses: codecov/codecov-action@v4

  data-pipeline-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up DVC
        run: pip install dvc dvc-s3

      - name: Pull reference data sample
        run: dvc pull data/sample/
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

      - name: Check pipeline is reproducible
        run: dvc repro --dry-run
```

### Training and Evaluation Gate

```yaml
# .github/workflows/train_validate.yml
name: Train and Validate

on:
  push:
    branches: [main]
    paths: ["src/**", "configs/**", "params.yaml"]

jobs:
  train:
    runs-on: self-hosted   # GPU runner or cloud runner with GPU
    steps:
      - uses: actions/checkout@v4

      - name: Pull training data
        run: dvc pull data/splits/
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

      - name: Train model
        run: python src/train.py --experiment-name ci-${{ github.sha }} --run-name pr-${{ github.run_number }}
        env:
          MLFLOW_TRACKING_URI: ${{ secrets.MLFLOW_TRACKING_URI }}

      - name: Evaluate and gate
        id: evaluate
        run: |
          python src/evaluate.py \
            --run-name pr-${{ github.run_number }} \
            --threshold-auc 0.92 \
            --output metrics.json
          echo "auc=$(jq .val_auc metrics.json)" >> $GITHUB_OUTPUT
        env:
          MLFLOW_TRACKING_URI: ${{ secrets.MLFLOW_TRACKING_URI }}

      - name: Comment metrics on PR
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `## Model Evaluation Results\n**Val AUC**: ${{ steps.evaluate.outputs.auc }}\n✅ Passed threshold (0.92)`
            })
```

### Model Registration and Canary Deployment

```yaml
# .github/workflows/deploy.yml
name: Deploy Model

on:
  workflow_run:
    workflows: ["Train and Validate"]
    types: [completed]
    branches: [main]

jobs:
  register-and-deploy:
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Register model in MLflow
        run: python scripts/register_model.py --alias challenger
        env:
          MLFLOW_TRACKING_URI: ${{ secrets.MLFLOW_TRACKING_URI }}

      - name: Deploy canary (10% traffic)
        run: |
          kubectl set image deployment/fraud-api \
            api=ghcr.io/${{ github.repository }}:${{ github.sha }}
          kubectl annotate deployment/fraud-api \
            "deploy.kubernetes.io/canary-weight=10"
        env:
          KUBECONFIG: ${{ secrets.KUBECONFIG }}

      - name: Monitor canary (5 min)
        run: python scripts/monitor_canary.py --duration 300 --max-error-rate 0.02

      - name: Promote champion alias
        run: python scripts/promote_model.py --version latest --alias champion
        env:
          MLFLOW_TRACKING_URI: ${{ secrets.MLFLOW_TRACKING_URI }}
```

### evaluate.py — Performance Gate Script

```python
# src/evaluate.py
import argparse, json, sys
import mlflow
from mlflow.tracking import MlflowClient

def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("--run-name", required=True)
    parser.add_argument("--threshold-auc", type=float, default=0.90)
    parser.add_argument("--output", default="metrics.json")
    args = parser.parse_args()

    client = MlflowClient()
    runs = client.search_runs(
        experiment_ids=["1"],
        filter_string=f"tags.mlflow.runName = '{args.run_name}'",
        order_by=["start_time DESC"],
        max_results=1,
    )

    if not runs:
        print(f"No run found for {args.run_name}", file=sys.stderr)
        sys.exit(1)

    metrics = runs[0].data.metrics
    auc = metrics.get("val_auc", 0.0)

    with open(args.output, "w") as f:
        json.dump(metrics, f)

    if auc < args.threshold_auc:
        print(f"FAIL: val_auc={auc:.4f} < threshold={args.threshold_auc}", file=sys.stderr)
        sys.exit(1)

    print(f"PASS: val_auc={auc:.4f}")

if __name__ == "__main__":
    main()
```

## Links
- [[03_software_engineering/06_devops_and_infrastructure/cicd_pipelines|CI/CD Pipelines]]
- [[03_software_engineering/08_version_control/github_workflows|GitHub Workflows]]
- [[04_ml_engineering/07_continual_learning/retraining_strategies|Retraining Strategies]]
- [[04_ml_engineering/05_deployment_and_serving/rollout_strategies|Model Rollout Strategies]]
- [[mlflow_experiment_tracking|MLflow Experiment Tracking Pattern]]
- [[dvc_dataset_versioning|DVC Dataset Versioning Pattern]]
- [[docker_ml_pipeline|Docker for ML Pipelines]]
