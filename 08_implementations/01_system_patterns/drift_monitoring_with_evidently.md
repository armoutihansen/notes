---
layer: 08_implementations
type: application
status: growing
tags: [pattern, monitoring, mlops]
created: 2026-03-10
---

# Drift Monitoring with Evidently

## Purpose

Implements production ML drift monitoring using Evidently AI. Covers statistical drift detection for features and predictions, report generation, real-time monitoring dashboards, and automated alerting. Synthesizes patterns for detecting covariate drift, prediction drift, and data quality degradation post-deployment.

### Examples

**Batch drift report**: Compare the current week's production data against the training distribution; generate a visual HTML report for weekly model health review.

**Real-time monitoring**: Log predictions and features to a store, run Evidently checks on a sliding window every hour, and fire alerts when PSI exceeds threshold on top-N features.

## Architecture

### Installation

```bash
pip install evidently>=0.4
```

### Reference Dataset (Training Distribution)

```python
import pandas as pd
from evidently.report import Report
from evidently.metric_preset import DataDriftPreset, DataQualityPreset

# Training data = reference distribution
reference_data = pd.read_parquet("data/train_features.parquet")

# Production data = current week's logged features
production_data = pd.read_parquet("data/production/week_2026_w10.parquet")
```

### Data Drift Report

```python
from evidently.report import Report
from evidently.metric_preset import DataDriftPreset
from evidently.metrics import DatasetDriftMetric, ColumnDriftMetric

report = Report(metrics=[
    DataDriftPreset(),              # KS test for continuous, chi-sq for categorical
    DatasetDriftMetric(),           # Overall drift summary
    ColumnDriftMetric("amount"),    # Per-feature drift
    ColumnDriftMetric("merchant_category"),
])

report.run(reference_data=reference_data, current_data=production_data)

# Save HTML report for review
report.save_html("reports/drift_2026_w10.html")

# Extract metrics programmatically
result = report.as_dict()
drift_share = result["metrics"][2]["result"]["drift_share"]
print(f"Fraction of drifted features: {drift_share:.1%}")
```

### Test Suite for Automated Alerting

```python
from evidently.test_suite import TestSuite
from evidently.tests import (
    TestNumberOfDriftedColumns,
    TestColumnDrift,
    TestShareOfDriftedColumns,
)

suite = TestSuite(tests=[
    TestShareOfDriftedColumns(lt=0.3),       # Fail if >30% of features drift
    TestColumnDrift("amount", stattest="psi", stattest_threshold=0.2),
    TestColumnDrift("merchant_category", stattest="chi_square"),
    TestNumberOfDriftedColumns(lt=5),
])

suite.run(reference_data=reference_data, current_data=production_data)
suite.save_html("reports/test_results.html")

# Boolean pass/fail for CI or alerting
passed = suite.as_dict()["summary"]["all_passed"]
if not passed:
    # Send alert
    send_pagerduty_alert("Drift threshold exceeded")
```

### Column-Level Statistic Selection

```python
from evidently.metrics import ColumnDriftMetric
from evidently.report import Report

# Choose statistical test per feature type
report = Report(metrics=[
    # KS test (default for continuous)
    ColumnDriftMetric("transaction_amount", stattest="ks"),

    # Population Stability Index (common in financial services)
    ColumnDriftMetric("credit_score", stattest="psi"),

    # Chi-squared (categorical)
    ColumnDriftMetric("merchant_category", stattest="chi_square"),

    # Jensen-Shannon divergence (stable for sparse distributions)
    ColumnDriftMetric("hour_of_day", stattest="jensenshannon"),
])
```

### Prediction Distribution Monitoring

```python
from evidently.metrics import ColumnDriftMetric, ColumnDistributionMetric

# Add model predictions to the feature DataFrames before running
reference_data["prediction"] = model.predict_proba(reference_features)[:, 1]
production_data["prediction"] = production_predictions   # logged from serving

report = Report(metrics=[
    ColumnDriftMetric("prediction", stattest="psi"),
    ColumnDistributionMetric("prediction"),
])
report.run(reference_data=reference_data, current_data=production_data)
```

### Scheduled Monitoring in Airflow

```python
# dags/drift_monitoring_dag.py
from airflow.decorators import dag, task
from datetime import datetime, timedelta

@dag(schedule="0 8 * * 1", start_date=datetime(2026, 1, 1))   # every Monday 08:00
def weekly_drift_check():

    @task
    def load_data():
        reference = pd.read_parquet("s3://bucket/data/reference.parquet")
        production = pd.read_parquet(
            f"s3://bucket/data/production/week_{datetime.now().isocalendar().week}.parquet"
        )
        return reference, production

    @task
    def run_drift_suite(data):
        reference, production = data
        suite = TestSuite(tests=[
            TestShareOfDriftedColumns(lt=0.3),
            TestColumnDrift("amount", stattest="psi", stattest_threshold=0.2),
        ])
        suite.run(reference_data=reference, current_data=production)
        return suite.as_dict()["summary"]["all_passed"]

    @task
    def alert_if_failed(passed: bool):
        if not passed:
            send_slack_alert("#ml-monitoring", "Drift check failed — review required")

    result = run_drift_suite(load_data())
    alert_if_failed(result)

weekly_drift_check()
```

## Links
- [[05_ml_engineering/07_monitoring_and_observability/drift_detection|Drift Detection]]
- [[05_ml_engineering/07_monitoring_and_observability/ml_observability|ML Observability]]
- [[05_ml_engineering/08_continual_learning/retraining_strategies|Retraining Strategies]]
- [[continuous_training_pipeline|Continuous Training Pipeline]]
- [[cicd_for_ml|CI/CD for ML Pipelines]]
