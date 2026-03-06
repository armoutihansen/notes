---
layer: 06_applications
type: application
status: growing
tags: [pattern, monitoring, mlops]
created: 2026-05-10
---

# Model Monitoring System

## Purpose

A model monitoring system continuously tracks data drift, prediction drift, and data quality in production, then triggers alerts and automated retraining when thresholds are exceeded. Without monitoring, silent model degradation is invisible until business metrics suffer. This pattern closes the MLOps loop: deployed models are watched, and stale models are automatically replaced.

### Examples

**Fraud model monitoring**: Compare incoming transaction features to training distribution weekly using PSI; alert when PSI > 0.2 on any top-5 feature; trigger DVC retraining pipeline automatically.

**LLM output monitoring**: Track output length distribution and refusal rate hourly; alert on sudden changes that may indicate prompt injection or model regression.

---

## Architecture

```
Production traffic (features + predictions)
        ↓
   Log to time-series store (Prometheus, S3, DWH)
        ↓
   Drift detection (Evidently) — batch or streaming window
        ├── Data drift report (covariate: P(X))
        └── Prediction drift test (concept proxy: P(Y_hat))
        ↓
   Prometheus metrics export → Grafana dashboard
        ↓
   Alert rule (PSI > threshold) → PagerDuty / Slack
        ↓
   Retraining trigger → Airflow DAG / DVC repro
```

---

## Setup

```bash
pip install evidently prometheus-client
```

---

## Data Drift Report (Batch)

```python
import pandas as pd
from evidently import ColumnMapping
from evidently.report import Report
from evidently.metric_preset import DataDriftPreset, DataQualityPreset

# Reference = training data; current = last 7 days of production data
reference_data = pd.read_parquet("data/training_features.parquet")
current_data   = pd.read_parquet("data/production_features_last_week.parquet")

column_mapping = ColumnMapping(
    target="churn_label",
    prediction="churn_score",
    numerical_features=["txn_count_1h", "txn_count_24h", "avg_txn_amount_7d"],
    categorical_features=["customer_segment"],
)

report = Report(metrics=[DataDriftPreset(), DataQualityPreset()])
report.run(reference_data=reference_data, current_data=current_data, column_mapping=column_mapping)
report.save_html("reports/drift_report.html")

# Programmatic check
result = report.as_dict()
n_drifted = result["metrics"][0]["result"]["number_of_drifted_columns"]
print(f"Drifted columns: {n_drifted}")
```

---

## Prediction Drift Test Suite

```python
from evidently.test_suite import TestSuite
from evidently.tests import TestShareOfDriftedColumns, TestColumnDrift

test_suite = TestSuite(tests=[
    TestShareOfDriftedColumns(lt=0.3),              # fail if >30% of features drift
    TestColumnDrift(column_name="churn_score"),     # fail if prediction drift detected
])
test_suite.run(reference_data=reference_data, current_data=current_data, column_mapping=column_mapping)
test_suite.save_html("reports/test_suite.html")

if not test_suite.as_dict()["summary"]["all_passed"]:
    print("ALERT: Drift tests failed — triggering retraining")
    # Fire Airflow DAG or DVC repro here
```

---

## Prometheus Metric Export

```python
from prometheus_client import Gauge, start_http_server
import time

# Define Prometheus gauges for key drift metrics
psi_gauge        = Gauge("ml_feature_psi",         "PSI score for features",     ["feature_name"])
drift_flag_gauge = Gauge("ml_drift_detected",      "1 if drift detected, else 0")
pred_drift_gauge = Gauge("ml_prediction_drift_psi","PSI on prediction distribution")

# Compute PSI manually or from Evidently result dict
def compute_psi(reference: pd.Series, current: pd.Series, bins: int = 10) -> float:
    """Compute Population Stability Index between two distributions."""
    breakpoints = pd.qcut(reference, q=bins, duplicates="drop", retbins=True)[1]
    ref_counts  = pd.cut(reference, bins=breakpoints).value_counts(normalize=True).sort_index()
    cur_counts  = pd.cut(current,   bins=breakpoints).value_counts(normalize=True).sort_index()
    ref_counts  = ref_counts.clip(lower=1e-4)   # avoid log(0)
    cur_counts  = cur_counts.clip(lower=1e-4)
    psi = float(((cur_counts - ref_counts) * (cur_counts / ref_counts).apply(pd.np.log)).sum())
    return psi

start_http_server(8000)   # Prometheus scrapes :8000/metrics

while True:
    for feat in ["txn_count_1h", "txn_count_24h", "avg_txn_amount_7d"]:
        psi = compute_psi(reference_data[feat], current_data[feat])
        psi_gauge.labels(feature_name=feat).set(psi)
        if psi > 0.2:
            drift_flag_gauge.set(1)
    time.sleep(3600)   # update hourly
```

---

## Alerting Rule (Prometheus / Alertmanager)

```yaml
# prometheus_rules.yaml
groups:
  - name: ml_drift_alerts
    rules:
      - alert: FeatureDriftHigh
        expr: ml_feature_psi > 0.2
        for: 1h
        labels:
          severity: warning
        annotations:
          summary: "Feature drift detected: {{ $labels.feature_name }}"
          description: "PSI = {{ $value | printf \"%.3f\" }} — model retraining recommended"

      - alert: DriftRetrainingTrigger
        expr: ml_drift_detected == 1
        for: 2h
        labels:
          severity: critical
        annotations:
          summary: "Drift threshold exceeded — auto-retraining triggered"
```

---

## Automated Retraining Trigger

```python
# Called from alert receiver (webhook) or Airflow sensor
import subprocess

def trigger_retraining():
    """Trigger DVC pipeline retraining and evaluation."""
    result = subprocess.run(
        ["dvc", "repro", "--force"],
        cwd="/pipeline",
        capture_output=True, text=True
    )
    if result.returncode != 0:
        raise RuntimeError(f"DVC repro failed:\n{result.stderr}")
    # evaluate.py includes the evaluation gate — only promotes if challenger wins
    subprocess.run(["python", "src/evaluate.py"], cwd="/pipeline", check=True)
    print("Retraining complete.")
```

---

## Monitoring Decision Matrix

| Drift type | Method | Threshold | Action |
|---|---|---|---|
| Covariate (features) | PSI per feature | > 0.2 (moderate) | Alert + investigate |
| Covariate (severe) | PSI | > 0.25 | Trigger retraining |
| Prediction distribution | KS test / PSI | p < 0.05 | Immediate investigation |
| Data quality | Missing rate | > 5% | Alert data engineering |
| Performance | AUC vs. champion | < -0.02 | Immediate retraining |

---

## Links

**ML Engineering**
- [[04_ml_engineering/06_monitoring_and_observability/drift_detection|Drift Detection]] — PSI, KS test, covariate vs. concept drift theory
- [[04_ml_engineering/06_monitoring_and_observability/ml_observability|ML Observability]] — dashboards, logging patterns, alerting setup

**System Patterns**
- [[drift_monitoring_with_evidently|Drift Monitoring with Evidently]] — Evidently-specific implementation reference
- [[training_pipeline_pattern|Training Pipeline Pattern]] — the retraining pipeline triggered by this monitor
- [[model_serving_with_fastapi|Model Serving with FastAPI]] — the deployment being monitored

**End-to-End Examples**
- [[tabular_classification_pipeline|Tabular Classification Pipeline]] — full system integrating monitoring
- [[continuous_training_pipeline|Continuous Training Pipeline]] — monitors feeding automated retraining
