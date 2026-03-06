---
layer: 06_applications
type: application
status: growing
tags: [pattern, training, mlops]
created: 2026-03-10
---

# MLflow Experiment Tracking Pattern

## Purpose

Implements reproducible experiment tracking, artifact storage, and model registry lifecycle using MLflow. This pattern covers the complete workflow from logging training runs to promoting model versions to production using named aliases (MLflow 3.x).

### Examples

**Tabular ML pipeline**: Log hyperparameter grid search runs, compare metrics across experiments, register the best model, and promote via `champion` alias.

**Deep learning fine-tuning**: Log per-epoch loss/accuracy curves, checkpoint artifacts, and system metrics; retrieve the best run programmatically and register for serving.

## Architecture

### Project Structure

```
project/
├── train.py             # training script with mlflow.start_run()
├── evaluate.py          # evaluation + model registration
├── serve.py             # loads model via alias URI
├── mlflow.db            # local tracking store (dev only)
└── mlruns/              # artifact store (dev only; use S3 in prod)
```

### Installation and Server Setup

```bash
pip install mlflow>=2.10

# Development (local SQLite tracking + local artifact store)
mlflow server \
  --backend-store-uri sqlite:///mlflow.db \
  --default-artifact-root ./mlruns \
  --host 0.0.0.0 --port 5000

# Production (PostgreSQL + S3)
mlflow server \
  --backend-store-uri postgresql://user:pass@host:5432/mlflow \
  --default-artifact-root s3://my-bucket/mlflow-artifacts \
  --host 0.0.0.0 --port 5000
```

Set the tracking server URL in all clients:

```bash
export MLFLOW_TRACKING_URI=http://mlflow-server:5000
```

### Logging a Training Run

```python
import mlflow
import mlflow.sklearn
from sklearn.ensemble import GradientBoostingClassifier
from sklearn.metrics import roc_auc_score

mlflow.set_experiment("fraud-detection")

with mlflow.start_run(run_name="gbm-v3") as run:
    # Log hyperparameters
    params = {"n_estimators": 300, "learning_rate": 0.05, "max_depth": 5}
    mlflow.log_params(params)

    # Train
    model = GradientBoostingClassifier(**params)
    model.fit(X_train, y_train)

    # Log metrics
    auc = roc_auc_score(y_val, model.predict_proba(X_val)[:, 1])
    mlflow.log_metric("val_auc", auc)

    # Log model artifact
    mlflow.sklearn.log_model(
        model,
        artifact_path="model",
        registered_model_name="fraud-detector",   # auto-registers
        input_example=X_train[:5],
        signature=mlflow.models.infer_signature(X_train, model.predict(X_train)),
    )

    run_id = run.info.run_id
```

### Comparing Runs and Selecting Best

```python
from mlflow.tracking import MlflowClient

client = MlflowClient()

# Search runs in experiment
runs = client.search_runs(
    experiment_ids=["1"],
    filter_string="metrics.val_auc > 0.90",
    order_by=["metrics.val_auc DESC"],
    max_results=5,
)

best_run = runs[0]
print(f"Best run: {best_run.info.run_id}, AUC={best_run.data.metrics['val_auc']:.4f}")
```

### Model Registry with Named Aliases (MLflow 3.x)

```python
from mlflow.tracking import MlflowClient

client = MlflowClient()

# List all versions of the registered model
versions = client.search_model_versions("name='fraud-detector'")

# Assign alias to promote a version
client.set_registered_model_alias(
    name="fraud-detector",
    alias="champion",
    version=7,
)

# Optional: tag for lineage
client.set_model_version_tag(
    name="fraud-detector",
    version="7",
    key="approved_by",
    value="data-science-team",
)
```

> **Note**: MLflow 2.x used fixed stage transitions (`Staging → Production → Archived`). MLflow 3.x replaces these with free-form named aliases, allowing multiple concurrent production variants (e.g., `champion`, `challenger`, `shadow`).

### Loading a Model in Production

```python
import mlflow.pyfunc

# Load via alias (recommended — decoupled from version number)
model = mlflow.pyfunc.load_model("models:/fraud-detector@champion")

# Load specific version (for reproducibility audits)
model = mlflow.pyfunc.load_model("models:/fraud-detector/7")

# Serve via CLI
# mlflow models serve -m "models:/fraud-detector@champion" -p 8080
```

### Autologging for Supported Frameworks

```python
import mlflow

mlflow.autolog()          # scikit-learn, XGBoost, LightGBM, PyTorch, Keras

# All params, metrics, and model artifacts logged automatically
model.fit(X_train, y_train)
```

## Links
- [[04_ml_engineering/04_model_development/experiment_tracking|Experiment Tracking]]
- [[04_ml_engineering/08_infrastructure_and_platform/ml_platform_architecture|ML Platform Architecture]]
- [[04_ml_engineering/02_training_data/dataset_versioning|Dataset Versioning and Lineage]]
- [[04_ml_engineering/04_model_development/offline_evaluation|Offline Evaluation]]
- [[model_serving_with_fastapi|Model Serving with FastAPI]]
- [[cicd_for_ml|CI/CD for ML Pipelines]]
