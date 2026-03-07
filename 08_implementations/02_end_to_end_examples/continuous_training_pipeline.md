---
layer: 08_implementations
type: application
status: growing
tags: [workflow, training, monitoring, mlops]
created: 2026-03-10
---

# Continuous Training Pipeline

## Purpose

A complete reference architecture for a continuous training (CT) system that automatically detects drift, triggers retraining, validates the new model, and deploys it via a canary rollout. This closes the MLOps loop: the model adapts to distribution shift without manual intervention while safety gates prevent silent regressions.

### Examples

**Fraud detection CT**: When PSI on transaction features exceeds 0.2 or prediction score distribution shifts, trigger a retraining job; if the new model passes AUC threshold and shadow evaluation, promote it as the new champion.

**Recommendation freshness**: Retrain on a rolling 30-day window of interaction data; use Thompson Sampling to decide whether to roll out the challenger or keep the champion.

## Architecture

### System Overview

```
Production Traffic
       │
       ▼
┌──────────────────┐     weekly     ┌─────────────────┐
│ Serving Layer    │────features───▶│ Drift Monitor   │
│ (FastAPI)        │                │ (Evidently)     │
│                  │◀───new model───│                 │
└──────────────────┘                └────────┬────────┘
                                             │ drift alert
                                             ▼
                                    ┌─────────────────┐
                                    │ Retraining Job  │
                                    │ (Accelerate /   │
                                    │  scikit-learn)  │
                                    └────────┬────────┘
                                             │ new model version
                                             ▼
                                    ┌─────────────────┐
                                    │ Evaluation Gate │
                                    │ AUC ≥ threshold?│
                                    │ > champion?     │
                                    └────────┬────────┘
                                             │ pass
                                             ▼
                                    ┌─────────────────┐
                                    │ Canary Deploy   │
                                    │ 10% → 50% →100% │
                                    └─────────────────┘
```

### Component 1 — Feature Logging

```python
# Serving layer: log every prediction request's features for drift monitoring
# app/main.py (excerpt)
import json
from datetime import datetime
import boto3

s3 = boto3.client("s3")
log_buffer = []

@app.post("/predict")
async def predict(request: PredictRequest):
    features = request.model_dump()
    prediction = model.predict(pd.DataFrame([features]))[0]

    # Async feature log (don't block inference)
    log_entry = {**features, "prediction": float(prediction), "ts": datetime.utcnow().isoformat()}
    log_buffer.append(log_entry)

    # Flush buffer every 100 requests
    if len(log_buffer) >= 100:
        batch = log_buffer.copy()
        log_buffer.clear()
        s3.put_object(
            Bucket="prediction-logs",
            Key=f"features/{datetime.utcnow().strftime('%Y/%m/%d/%H')}/{datetime.utcnow().timestamp()}.jsonl",
            Body="\n".join(json.dumps(r) for r in batch),
        )
    return {"prediction": float(prediction)}
```

### Component 2 — Drift Trigger

```python
# scripts/drift_trigger.py — runs on a schedule (hourly / daily)
import pandas as pd
from evidently.test_suite import TestSuite
from evidently.tests import TestShareOfDriftedColumns, TestColumnDrift

DRIFT_THRESHOLDS = {
    "transaction_amount": ("psi", 0.20),
    "merchant_category": ("chi_square", 0.05),
    "hour_of_day": ("ks", 0.01),
}

def check_and_trigger(reference_path: str, current_path: str) -> bool:
    reference = pd.read_parquet(reference_path)
    current = pd.read_parquet(current_path)

    tests = [TestShareOfDriftedColumns(lt=0.30)]
    for col, (stattest, threshold) in DRIFT_THRESHOLDS.items():
        tests.append(TestColumnDrift(col, stattest=stattest, stattest_threshold=threshold))

    suite = TestSuite(tests=tests)
    suite.run(reference_data=reference, current_data=current)
    passed = suite.as_dict()["summary"]["all_passed"]

    if not passed:
        print("Drift detected — triggering retraining")
        trigger_retraining()
        return True
    return False

def trigger_retraining():
    """Trigger GitHub Actions workflow or Airflow DAG."""
    import subprocess
    subprocess.run([
        "gh", "workflow", "run", "train_validate.yml",
        "--ref", "main",
        "--field", "trigger=drift",
    ], check=True)
```

### Component 3 — Retraining Job

```python
# scripts/retrain.py
import mlflow, mlflow.sklearn
import pandas as pd
from sklearn.ensemble import GradientBoostingClassifier
from sklearn.metrics import roc_auc_score

def retrain(trigger: str = "scheduled"):
    """Retrain on the most recent N days of labeled data."""
    # Rolling window: most recent 90 days of labels
    data = pd.read_parquet("s3://data/labeled/rolling_90d.parquet")
    X = data.drop(columns=["label", "id", "event_date"])
    y = data["label"]

    mlflow.set_experiment("fraud-ct")

    with mlflow.start_run(run_name=f"retrain-trigger={trigger}") as run:
        mlflow.log_param("trigger", trigger)
        mlflow.log_param("training_rows", len(data))
        mlflow.log_param("training_date_range", f"{data['event_date'].min()} – {data['event_date'].max()}")

        model = GradientBoostingClassifier(n_estimators=400, learning_rate=0.05, max_depth=4, random_state=42)
        model.fit(X, y)

        val = pd.read_parquet("s3://data/labeled/holdout.parquet")
        val_auc = roc_auc_score(val["label"], model.predict_proba(val.drop(columns=["label", "id", "event_date"]))[:, 1])
        mlflow.log_metric("val_auc", val_auc)

        mlflow.sklearn.log_model(
            model,
            artifact_path="model",
            registered_model_name="fraud-detector",
        )

        return run.info.run_id, val_auc
```

### Component 4 — Automated Promotion Gate

```python
# scripts/promote_or_reject.py
from mlflow.tracking import MlflowClient
import mlflow.pyfunc
import pandas as pd
from sklearn.metrics import roc_auc_score

client = MlflowClient()

def promote_or_reject(model_name: str, new_version: str, threshold: float = 0.91):
    # Load models
    new_model = mlflow.pyfunc.load_model(f"models:/{model_name}/{new_version}")
    try:
        champion = mlflow.pyfunc.load_model(f"models:/{model_name}@champion")
        has_champion = True
    except Exception:
        has_champion = False

    # Evaluate on holdout
    holdout = pd.read_parquet("s3://data/labeled/holdout.parquet")
    X = holdout.drop(columns=["label", "id", "event_date"])
    y = holdout["label"]

    new_auc = roc_auc_score(y, new_model.predict(X))
    print(f"New model AUC: {new_auc:.4f}")

    if has_champion:
        champ_auc = roc_auc_score(y, champion.predict(X))
        print(f"Champion AUC: {champ_auc:.4f}")
        should_promote = new_auc >= threshold and new_auc > champ_auc
    else:
        should_promote = new_auc >= threshold

    if should_promote:
        # Assign challenger alias first for canary
        client.set_registered_model_alias(model_name, "challenger", new_version)
        print(f"✓ Assigned version {new_version} as challenger → start canary")
    else:
        client.set_model_version_tag(model_name, new_version, "rejected", "true")
        print(f"✗ Rejected: AUC={new_auc:.4f} < threshold or champion")

    return should_promote
```

### Component 5 — Canary Rollout and Final Promotion

```python
# scripts/canary_rollout.py
import time, subprocess
from mlflow.tracking import MlflowClient
import prometheus_client as prom   # read from Prometheus

client = MlflowClient()

def canary_rollout(model_name: str, challenger_version: str, canary_weights=(10, 50, 100)):
    for weight in canary_weights:
        # Update traffic weight in load balancer config
        subprocess.run([
            "kubectl", "patch", "virtualservice", "fraud-api",
            "--type=merge",
            f'--patch={{"spec":{{"http":[{{"route":[{{"destination":{{"host":"fraud-api","subset":"champion"}},"weight":{100 - weight}}},{{"destination":{{"host":"fraud-api","subset":"challenger"}},"weight":{weight}}}]}}]}}}}'
        ], check=True)

        print(f"Canary at {weight}% — monitoring for 10 minutes...")
        time.sleep(600)

        # Check error rate from Prometheus
        error_rate = query_prometheus("rate(http_requests_total{status=~'5..'}[5m]) / rate(http_requests_total[5m])")
        if error_rate > 0.01:
            print(f"Error rate {error_rate:.1%} > 1% — rolling back")
            rollback(model_name)
            return False

    # All stages passed — promote challenger to champion
    client.set_registered_model_alias(model_name, "champion", challenger_version)
    print(f"✓ Promoted {challenger_version} to champion")
    return True
```

### Airflow DAG — Full CT Loop

```python
from airflow.decorators import dag, task
from datetime import datetime

@dag(schedule="0 6 * * *", start_date=datetime(2026, 1, 1), catchup=False)
def continuous_training():

    @task
    def detect_drift() -> bool:
        from scripts.drift_trigger import check_and_trigger
        return check_and_trigger(
            reference_path="s3://data/reference/train_sample.parquet",
            current_path="s3://data/production/yesterday_features.parquet",
        )

    @task
    def retrain_if_drift(drift_detected: bool) -> str:
        if not drift_detected:
            return "no_retrain"
        from scripts.retrain import retrain
        run_id, val_auc = retrain(trigger="drift")
        return run_id

    @task
    def promote_if_better(run_id: str) -> bool:
        if run_id == "no_retrain":
            return False
        from scripts.promote_or_reject import promote_or_reject
        versions = MlflowClient().search_model_versions(f"name='fraud-detector'")
        latest_version = max(versions, key=lambda v: int(v.version)).version
        return promote_or_reject("fraud-detector", latest_version)

    @task
    def canary_if_promoted(promoted: bool):
        if not promoted:
            return
        from scripts.canary_rollout import canary_rollout
        versions = MlflowClient().search_model_versions("name='fraud-detector'")
        challenger_version = [v for v in versions if "challenger" in v.aliases][0].version
        canary_rollout("fraud-detector", challenger_version)

    drift = detect_drift()
    run_id = retrain_if_drift(drift)
    promoted = promote_if_better(run_id)
    canary_if_promoted(promoted)

continuous_training()
```

## References

- [MLflow Documentation](https://mlflow.org/docs/latest/)
- [DVC Documentation](https://dvc.org/doc)

## Links
- [[05_ml_engineering/08_continual_learning/retraining_strategies|Retraining Strategies]]
- [[05_ml_engineering/08_continual_learning/testing_in_production|Testing in Production]]
- [[05_ml_engineering/07_monitoring_and_observability/drift_detection|Drift Detection]]
- [[05_ml_engineering/07_monitoring_and_observability/ml_observability|ML Observability]]
- [[05_ml_engineering/06_deployment_and_serving/rollout_strategies|Model Rollout Strategies]]
- [[05_ml_engineering/05_model_development/experiment_tracking|Experiment Tracking]]
- [[05_ml_engineering/09_infrastructure_and_platform/ml_platform_architecture|ML Platform Architecture]]
- [[mlflow_experiment_tracking|MLflow Experiment Tracking Pattern]]
- [[drift_monitoring_with_evidently|Drift Monitoring with Evidently]]
- [[model_serving_with_fastapi|Model Serving with FastAPI]]
- [[cicd_for_ml|CI/CD for ML Pipelines]]
- [[batch_ml_prediction_pipeline|Batch ML Prediction Pipeline]]
