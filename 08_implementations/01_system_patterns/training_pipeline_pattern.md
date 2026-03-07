---
layer: 08_implementations
type: application
status: growing
tags: [pattern, training, mlops]
created: 2026-05-10
---

# Training Pipeline Pattern

## Purpose

An end-to-end ML training pipeline automates the full path from raw data to a registered, evaluated model: data ingestion → validation → feature engineering → model training → evaluation gate → model registry. Codifying this pipeline as a DAG makes experiments reproducible, enables continuous training on fresh data, and creates an auditable trail of every trained model.

### Examples

**Weekly churn model**: Run every Sunday; pull 6 months of transaction data, validate schema, engineer features, train XGBoost, evaluate AUC vs. champion, push to MLflow registry if challenger wins.

**Nightly NLP retraining**: Ingest new labelled tickets from Jira, validate class balance, fine-tune classifier, evaluate on held-out set, deploy automatically if F1 improves.

---

## Architecture

```
Data source (S3, DWH, Kafka)
        ↓
  [1] Data ingestion + schema validation (Great Expectations)
        ↓
  [2] Feature engineering (Spark / pandas)
        ↓
  [3] Train / validation split
        ↓
  [4] Model training (sklearn, XGBoost, PyTorch)
        ↓
  [5] Evaluation gate (AUC / F1 ≥ threshold vs. champion)
        ↓
  [6] Model registry (MLflow) → promotion to staging / production
```

---

## DVC Pipeline (dvc.yaml)

```yaml
# dvc.yaml — define pipeline as a DAG of stages
stages:
  ingest:
    cmd: python src/ingest.py --output data/raw/dataset.parquet
    deps: [src/ingest.py]
    outs: [data/raw/dataset.parquet]

  validate:
    cmd: python src/validate.py --input data/raw/dataset.parquet
    deps: [src/validate.py, data/raw/dataset.parquet]
    outs: [data/validated/dataset.parquet]
    metrics: [reports/validation_metrics.json]

  featurize:
    cmd: python src/featurize.py --input data/validated/dataset.parquet --output data/features/
    deps: [src/featurize.py, data/validated/dataset.parquet]
    outs: [data/features/]

  train:
    cmd: python src/train.py --features data/features/ --model models/challenger.pkl
    deps: [src/train.py, data/features/, params.yaml]
    outs: [models/challenger.pkl]
    metrics: [reports/train_metrics.json:
                 cache: false]

  evaluate:
    cmd: python src/evaluate.py --model models/challenger.pkl --data data/features/
    deps: [src/evaluate.py, models/challenger.pkl]
    metrics: [reports/eval_metrics.json:
                 cache: false]
```

```bash
# Run the full pipeline (only re-runs changed stages)
dvc repro

# Show pipeline DAG
dvc dag

# Compare metrics across runs
dvc metrics diff
```

---

## MLflow Experiment Tracking + Evaluation Gate

```python
# src/train.py
import mlflow, mlflow.sklearn
from sklearn.ensemble import GradientBoostingClassifier
from sklearn.metrics import roc_auc_score
import pandas as pd, joblib

mlflow.set_tracking_uri("http://mlflow-server:5000")
mlflow.set_experiment("churn-prediction")

with mlflow.start_run(run_name="challenger_v2"):
    # Load features
    X_train = pd.read_parquet("data/features/train.parquet")
    y_train = X_train.pop("label")
    X_val   = pd.read_parquet("data/features/val.parquet")
    y_val   = X_val.pop("label")

    # Train
    params = {"n_estimators": 300, "max_depth": 5, "learning_rate": 0.05}
    model = GradientBoostingClassifier(**params)
    model.fit(X_train, y_train)

    # Evaluate
    val_auc = roc_auc_score(y_val, model.predict_proba(X_val)[:, 1])

    mlflow.log_params(params)
    mlflow.log_metric("val_auc", val_auc)
    mlflow.sklearn.log_model(model, "model",
                              registered_model_name="churn_predictor")
    joblib.dump(model, "models/challenger.pkl")
    print(f"Challenger AUC: {val_auc:.4f}")
```

```python
# src/evaluate.py — evaluation gate: promote if challenger beats champion
import mlflow

client = mlflow.tracking.MlflowClient()

# Get champion metric
champion_versions = client.get_latest_versions("churn_predictor", stages=["Production"])
if champion_versions:
    champion_run = client.get_run(champion_versions[0].run_id)
    champion_auc = champion_run.data.metrics["val_auc"]
else:
    champion_auc = 0.0

# Get challenger metric (latest run)
challenger_run = client.search_runs("churn-prediction", order_by=["start_time DESC"], max_results=1)[0]
challenger_auc = challenger_run.data.metrics["val_auc"]

print(f"Champion AUC: {champion_auc:.4f} | Challenger AUC: {challenger_auc:.4f}")

if challenger_auc > champion_auc + 0.002:   # promote only on meaningful improvement
    client.transition_model_version_stage(
        name="churn_predictor",
        version=challenger_run.data.tags.get("mlflow.source.git.commit", "latest"),
        stage="Production",
    )
    print("Challenger promoted to Production.")
else:
    print("Champion retained.")
```

---

## Airflow DAG (Orchestration)

```python
from airflow import DAG
from airflow.operators.bash import BashOperator
from datetime import datetime, timedelta

with DAG("weekly_churn_pipeline",
         schedule_interval="@weekly",
         start_date=datetime(2024, 1, 1),
         catchup=False,
         default_args={"retries": 1, "retry_delay": timedelta(minutes=5)}) as dag:

    ingest    = BashOperator(task_id="ingest",    bash_command="cd /pipeline && dvc repro ingest")
    validate  = BashOperator(task_id="validate",  bash_command="cd /pipeline && dvc repro validate")
    featurize = BashOperator(task_id="featurize", bash_command="cd /pipeline && dvc repro featurize")
    train     = BashOperator(task_id="train",     bash_command="cd /pipeline && dvc repro train")
    evaluate  = BashOperator(task_id="evaluate",  bash_command="cd /pipeline && python src/evaluate.py")

    ingest >> validate >> featurize >> train >> evaluate
```

---

## References

## Links

**ML Engineering**
- [[05_ml_engineering/05_model_development/experiment_tracking|Experiment Tracking]] — MLflow runs, params, metrics, model registry
- [[05_ml_engineering/02_data_engineering/data_pipeline_patterns|Data Pipeline Patterns]] — ingestion patterns (batch vs. stream)
- [[05_ml_engineering/03_training_data/dataset_versioning|Dataset Versioning]] — DVC data versioning alongside model artifacts

**System Patterns**
- [[feature_store_pattern|Feature Store Pattern]] — upstream feature serving for the training pipeline
- [[model_monitoring_system|Model Monitoring System]] — downstream monitoring after the training pipeline deploys
- [[mlflow_experiment_tracking|MLflow Experiment Tracking]] — MLflow setup and artifact management
- [[dvc_dataset_versioning|DVC Dataset Versioning]] — DVC pipeline and data versioning

**End-to-End Examples**
- [[tabular_classification_pipeline|Tabular Classification Pipeline]] — full system including this pattern
