---
layer: 08_implementations
type: application
status: growing
tags: [workflow, tabular, mlops]
created: 2026-03-10
---

# Batch ML Prediction Pipeline

## Purpose

A complete reference architecture for a batch ML prediction system: from raw data through feature engineering, model training, offline evaluation, batch scoring, and drift monitoring. This end-to-end example demonstrates how the individual engineering patterns combine into a production-grade system for tabular classification or regression.

### Examples

**Churn prediction**: Score all active customers nightly, write propensity scores to a data warehouse, trigger retention campaigns for high-risk segments.

**Credit risk scoring**: Retrain a gradient boosting model weekly on the latest transaction history; score the entire loan portfolio in bulk; monitor score distributions for drift.

## Architecture

### System Overview

```
Raw Data (S3/Warehouse)
         │
         ▼
┌──────────────────┐
│ Data Pipeline    │  DVC + dbt / Spark
│ (batch, daily)   │  schema validation, feature transforms
└────────┬─────────┘
         │  versioned feature parquet
         ▼
┌──────────────────┐
│ Training Job     │  Python + scikit-learn / XGBoost
│ (weekly / event) │  MLflow tracking, DVC experiment
└────────┬─────────┘
         │  registered model
         ▼
┌──────────────────┐
│ Offline Eval     │  train/val/test splits + holdout
│ & Promotion      │  AUC, calibration, fairness checks
└────────┬─────────┘
         │  champion alias set
         ▼
┌──────────────────┐
│ Batch Scorer     │  scheduled container job (Airflow DAG)
│ (nightly)        │  loads model via MLflow alias, scores data
└────────┬─────────┘
         │  predictions parquet → DWH
         ▼
┌──────────────────┐
│ Drift Monitor    │  Evidently weekly report
│ (weekly)         │  PSI check on top features + predictions
└──────────────────┘
```

### Component 1 — Data Pipeline

```python
# scripts/build_features.py
import pandas as pd
from pathlib import Path

def build_features(raw_path: str, output_path: str) -> None:
    df = pd.read_parquet(raw_path)

    # Feature engineering
    df["tenure_days"] = (pd.Timestamp.now() - pd.to_datetime(df["signup_date"])).dt.days
    df["avg_monthly_spend"] = df["total_spend"] / df["months_active"].clip(lower=1)
    df["support_contacts_30d"] = df["support_contacts"].clip(upper=10)

    # Data quality assertions
    assert df["customer_id"].nunique() == len(df), "Duplicate customer_ids"
    assert df[["tenure_days", "avg_monthly_spend"]].notna().all().all(), "Nulls in features"

    # Write versioned output
    Path(output_path).parent.mkdir(parents=True, exist_ok=True)
    df.to_parquet(output_path, index=False)
    print(f"Wrote {len(df):,} rows to {output_path}")
```

### Component 2 — Model Training

```python
# scripts/train.py
import mlflow
import mlflow.sklearn
import pandas as pd
from sklearn.ensemble import GradientBoostingClassifier
from sklearn.model_selection import StratifiedKFold, cross_val_score
from sklearn.metrics import roc_auc_score

mlflow.set_experiment("churn-prediction")

with mlflow.start_run(run_name="weekly-retrain"):
    df = pd.read_parquet("data/splits/train.parquet")
    X, y = df.drop(columns=["churned", "customer_id"]), df["churned"]

    params = {
        "n_estimators": 400,
        "learning_rate": 0.05,
        "max_depth": 4,
        "subsample": 0.8,
        "min_samples_leaf": 50,
    }
    mlflow.log_params(params)

    model = GradientBoostingClassifier(**params, random_state=42)

    # Cross-validated AUC on training set
    cv_aucs = cross_val_score(model, X, y, cv=StratifiedKFold(5), scoring="roc_auc")
    mlflow.log_metric("cv_auc_mean", cv_aucs.mean())
    mlflow.log_metric("cv_auc_std", cv_aucs.std())

    # Fit on full training data
    model.fit(X, y)

    # Evaluate on holdout
    val = pd.read_parquet("data/splits/val.parquet")
    val_auc = roc_auc_score(val["churned"], model.predict_proba(val.drop(columns=["churned", "customer_id"]))[:, 1])
    mlflow.log_metric("val_auc", val_auc)

    # Register model
    mlflow.sklearn.log_model(
        model,
        artifact_path="model",
        registered_model_name="churn-predictor",
        input_example=X.head(5),
    )
    print(f"val_auc={val_auc:.4f}")
```

### Component 3 — Offline Evaluation and Promotion

```python
# scripts/evaluate_and_promote.py
from mlflow.tracking import MlflowClient
import pandas as pd
from sklearn.metrics import roc_auc_score, average_precision_score
from sklearn.calibration import calibration_curve
import mlflow.pyfunc

client = MlflowClient()

def evaluate_and_promote(model_name: str, new_version: str, threshold: float = 0.92):
    # Load new model
    new_model = mlflow.pyfunc.load_model(f"models:/{model_name}/{new_version}")

    # Load holdout test set (never seen during training or validation)
    test = pd.read_parquet("data/splits/test.parquet")
    X_test = test.drop(columns=["churned", "customer_id"])
    y_test = test["churned"]

    probs = new_model.predict(X_test)
    auc = roc_auc_score(y_test, probs)
    ap = average_precision_score(y_test, probs)

    print(f"New model — AUC={auc:.4f}, AP={ap:.4f}")

    # Compare against champion
    try:
        champion = mlflow.pyfunc.load_model(f"models:/{model_name}@champion")
        champion_probs = champion.predict(X_test)
        champion_auc = roc_auc_score(y_test, champion_probs)
        print(f"Champion AUC={champion_auc:.4f}")

        if auc > champion_auc and auc >= threshold:
            client.set_registered_model_alias(model_name, "champion", new_version)
            print(f"✓ Promoted version {new_version} to champion")
        else:
            print("✗ New model did not outperform champion; keeping existing champion")
    except Exception:
        # No champion yet — promote directly if above threshold
        if auc >= threshold:
            client.set_registered_model_alias(model_name, "champion", new_version)
            print(f"✓ Set version {new_version} as first champion")
```

### Component 4 — Nightly Batch Scorer

```python
# scripts/batch_score.py (runs nightly via Airflow or cron)
import mlflow.pyfunc
import pandas as pd
from datetime import date

def score_daily(scoring_date: str = None):
    scoring_date = scoring_date or str(date.today())

    # Load champion model
    model = mlflow.pyfunc.load_model("models:/churn-predictor@champion")

    # Load all active customers for today
    df = pd.read_parquet(f"s3://data-lake/customers/date={scoring_date}/")
    features = df.drop(columns=["customer_id"])

    df["churn_score"] = model.predict(features)
    df["score_date"] = scoring_date
    df["model_version"] = "champion"

    # Write to data warehouse
    df[["customer_id", "churn_score", "score_date", "model_version"]].to_parquet(
        f"s3://predictions/churn/date={scoring_date}/scores.parquet", index=False
    )
    print(f"Scored {len(df):,} customers for {scoring_date}")
```

### Component 5 — Weekly Drift Check

```python
# scripts/weekly_drift.py
from evidently.test_suite import TestSuite
from evidently.tests import TestShareOfDriftedColumns, TestColumnDrift
import pandas as pd

reference = pd.read_parquet("data/reference/train_sample.parquet")
current = pd.read_parquet(f"s3://predictions/churn/date=latest/features.parquet")

suite = TestSuite(tests=[
    TestShareOfDriftedColumns(lt=0.3),
    TestColumnDrift("avg_monthly_spend", stattest="psi", stattest_threshold=0.2),
    TestColumnDrift("tenure_days", stattest="ks"),
])
suite.run(reference_data=reference, current_data=current)

if not suite.as_dict()["summary"]["all_passed"]:
    # Trigger retraining via CI
    import subprocess
    subprocess.run(["gh", "workflow", "run", "train_validate.yml", "--ref", "main"])
```

### Deployment (Airflow DAG)

```python
# dags/churn_pipeline_dag.py
from airflow.decorators import dag, task
from datetime import datetime

@dag(schedule="0 2 * * *", start_date=datetime(2026, 1, 1), catchup=False)
def churn_prediction_pipeline():

    @task
    def build_features():
        import subprocess
        subprocess.run(["dvc", "repro", "featurise"], check=True)

    @task
    def batch_score():
        from scripts.batch_score import score_daily
        score_daily()

    @task
    def check_drift():
        from scripts.weekly_drift import run_weekly_drift_check
        run_weekly_drift_check()

    build_features() >> batch_score() >> check_drift()

churn_prediction_pipeline()
```

## Links
- [[05_ml_engineering/01_principles_and_lifecycle/ml_lifecycle|ML Lifecycle]]
- [[05_ml_engineering/02_data_engineering/data_pipeline_patterns|Data Pipeline Patterns]]
- [[05_ml_engineering/04_feature_engineering/feature_engineering_patterns|Feature Engineering Patterns]]
- [[05_ml_engineering/05_model_development/experiment_tracking|Experiment Tracking]]
- [[05_ml_engineering/05_model_development/offline_evaluation|Offline Evaluation]]
- [[05_ml_engineering/06_deployment_and_serving/serving_patterns|Model Serving Patterns]]
- [[05_ml_engineering/07_monitoring_and_observability/drift_detection|Drift Detection]]
- [[mlflow_experiment_tracking|MLflow Experiment Tracking Pattern]]
- [[dvc_dataset_versioning|DVC Dataset Versioning Pattern]]
- [[drift_monitoring_with_evidently|Drift Monitoring with Evidently]]
- [[cicd_for_ml|CI/CD for ML Pipelines]]
