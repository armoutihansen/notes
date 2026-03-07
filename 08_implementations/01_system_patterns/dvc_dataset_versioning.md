---
layer: 08_implementations
type: application
status: growing
tags: [pattern, data, mlops]
created: 2026-03-10
---

# DVC Dataset Versioning Pattern

## Purpose

Implements dataset versioning, data pipeline reproducibility, and artifact lineage tracking using DVC (Data Version Control). DVC extends Git to track large data files and ML artifacts without storing them in the repository, using remote storage backends for the actual bytes.

### Examples

**Training data versioning**: Track evolving training sets across preprocessing iterations; reproduce any experiment's exact dataset with `git checkout` + `dvc pull`.

**Data pipeline automation**: Define multi-stage preprocessing pipelines in `dvc.yaml`; `dvc repro` executes only stages whose inputs changed — identical to `make` for ML data.

## Architecture

### Installation and Remote Setup

```bash
pip install dvc dvc-s3    # or dvc-gs, dvc-azure, dvc-ssh

cd my-ml-project
git init
dvc init                  # creates .dvc/ directory

# Configure S3 remote
dvc remote add -d myremote s3://my-bucket/dvc-store
dvc remote modify myremote region eu-west-1

# Commit DVC config to git
git add .dvc/config
git commit -m "Add DVC remote"
```

### Tracking Data Files

```bash
# Add a dataset to DVC tracking
dvc add data/raw/train.parquet

# DVC creates data/raw/train.parquet.dvc:
# outs:
# - md5: a3f8b2c1d9e4...
#   size: 1073741824
#   path: train.parquet

git add data/raw/train.parquet.dvc data/raw/.gitignore
git commit -m "Add training dataset v1"

# Push data to remote
dvc push
```

### Reproducing a Past Dataset Version

```bash
# Restore code to an earlier commit
git checkout <earlier-commit>

# Pull the corresponding dataset version
dvc pull

# data/raw/train.parquet is now the version from that commit
```

### Defining a Reproducible Data Pipeline

```yaml
# dvc.yaml
stages:
  preprocess:
    cmd: python scripts/preprocess.py
    deps:
      - scripts/preprocess.py
      - data/raw/train.parquet
    outs:
      - data/processed/train_clean.parquet
    params:
      - params.yaml:
          - preprocess.drop_nulls_threshold
          - preprocess.date_cutoff

  featurise:
    cmd: python scripts/featurise.py
    deps:
      - scripts/featurise.py
      - data/processed/train_clean.parquet
    outs:
      - data/features/train_features.parquet

  split:
    cmd: python scripts/split.py
    deps:
      - scripts/split.py
      - data/features/train_features.parquet
    outs:
      - data/splits/train.parquet
      - data/splits/val.parquet
    params:
      - params.yaml:
          - split.val_fraction
          - split.random_seed
```

```bash
# Run only stages with changed inputs
dvc repro

# Force rerun all stages
dvc repro --force
```

### Experiment Tracking with DVC

```bash
# Run an experiment (captures params, metrics, and artifacts)
dvc exp run --set-param model.learning_rate=0.01

# Show all experiments
dvc exp show

# Compare two experiments
dvc exp diff exp-abc123 exp-def456

# Promote best experiment to workspace
dvc exp apply exp-abc123
```

### Linking Dataset Version to MLflow Run

```python
import mlflow
import subprocess

# Get the current DVC-tracked dataset hash
result = subprocess.run(
    ["dvc", "params", "diff", "--show-md5"],
    capture_output=True, text=True
)

with mlflow.start_run():
    mlflow.log_param("data_version", open("data/raw/train.parquet.dvc").read().split("md5: ")[1].split("\n")[0])
    mlflow.log_param("git_commit", subprocess.check_output(["git", "rev-parse", "HEAD"]).decode().strip())
    # ... rest of training
```

### CI Integration

```yaml
# .github/workflows/data-pipeline.yml
name: Validate Data Pipeline
on: [push]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup DVC
        run: pip install dvc dvc-s3
      - name: Pull data
        run: dvc pull
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      - name: Reproduce pipeline
        run: dvc repro --dry-run   # check if any stage is out of date
```

## References

- [DVC Documentation](https://dvc.org/doc)

## Links
- [[05_ml_engineering/03_training_data/dataset_versioning|Dataset Versioning and Lineage]]
- [[05_ml_engineering/02_data_engineering/data_pipeline_patterns|Data Pipeline Patterns]]
- [[mlflow_experiment_tracking|MLflow Experiment Tracking Pattern]]
- [[cicd_for_ml|CI/CD for ML Pipelines]]
- [[batch_ml_prediction_pipeline|Batch ML Prediction Pipeline]]
