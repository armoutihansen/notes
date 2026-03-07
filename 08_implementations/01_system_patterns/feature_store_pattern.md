---
layer: 08_implementations
type: application
status: growing
tags: [pattern, feature-engineering]
created: 2026-05-10
---

# Feature Store Pattern (Feast)

## Purpose

A feature store centralises the definition, computation, storage, and serving of ML features. It eliminates training-serving skew (the model sees different feature values during training and inference) and prevents teams from independently re-implementing the same features with subtle differences. The pattern is most valuable when ≥3 models share features, when real-time features are required, or when point-in-time correctness is critical.

### Examples

**Fraud detection**: Compute velocity features (transaction count in past 1h, 24h, 7d per card) once; serve them both to the training pipeline and the real-time scoring API.

**Recommendation**: Pre-materialise item embedding and user history features offline; serve at <10 ms via Redis online store during inference.

---

## Architecture

```
Data sources (Kafka, DWH, batch jobs)
        ↓
   Feature transformations (Python, Spark)
        ↓
   ┌────────────────┬──────────────────┐
   │  Offline store │  Online store    │
   │  (S3/Parquet)  │  (Redis/DynamoDB)│
   │  historical    │  latest values   │
   │  point-in-time │  low latency     │
   └────────────────┴──────────────────┘
        ↓                    ↓
   Training dataset     Serving endpoint
```

---

## Setup

```bash
pip install feast
feast init my_feature_repo
cd my_feature_repo
# Edit feature_store.yaml for offline/online store type
```

**feature_store.yaml:**
```yaml
project: my_project
registry: data/registry.db
provider: local          # or gcp, aws
offline_store:
  type: file             # or bigquery, snowflake
online_store:
  type: sqlite           # or redis, dynamodb
```

---

## Feature Definition

```python
# features.py
from datetime import timedelta
from feast import Entity, FeatureView, Field, FileSource, ValueType
from feast.types import Float32, Int64

# --- Entity: the key used to look up features ---
customer = Entity(name="customer_id", value_type=ValueType.INT64)

# --- Data source: parquet file or BigQuery table ---
source = FileSource(
    path="data/customer_features.parquet",
    timestamp_field="event_timestamp",
    created_timestamp_column="created",
)

# --- Feature view: a group of related features with TTL ---
customer_fv = FeatureView(
    name="customer_features",
    entities=[customer],
    ttl=timedelta(days=30),
    schema=[
        Field(name="txn_count_1h",  dtype=Float32),
        Field(name="txn_count_24h", dtype=Float32),
        Field(name="avg_txn_amount_7d", dtype=Float32),
        Field(name="days_since_last_login", dtype=Int64),
    ],
    source=source,
)
```

---

## Materialise and Serve

```python
import subprocess
from datetime import datetime, timezone

# Apply feature definitions to registry
subprocess.run(["feast", "apply"], cwd="my_feature_repo")

# Materialise offline → online (batch job; run on schedule)
subprocess.run([
    "feast", "materialize",
    "2024-01-01T00:00:00",
    datetime.now(timezone.utc).isoformat(),
], cwd="my_feature_repo")
```

---

## Point-in-Time Correct Training Dataset

```python
from feast import FeatureStore
import pandas as pd

store = FeatureStore(repo_path="my_feature_repo")

# Entity DataFrame: each row = (entity_id, event_timestamp, label)
# Feast fetches the most recent feature values BEFORE event_timestamp for each row
entity_df = pd.DataFrame({
    "customer_id":      [1001, 1002, 1001, 1003],
    "event_timestamp":  pd.to_datetime(["2024-02-01", "2024-02-05", "2024-02-10", "2024-03-01"], utc=True),
    "label_fraud":      [0, 1, 0, 1],
})

training_df = store.get_historical_features(
    entity_df=entity_df,
    features=[
        "customer_features:txn_count_1h",
        "customer_features:txn_count_24h",
        "customer_features:avg_txn_amount_7d",
    ],
).to_df()

print(training_df.head())
```

---

## Online Feature Retrieval (Serving)

```python
# Low-latency lookup at inference time (after materialize)
online_features = store.get_online_features(
    features=[
        "customer_features:txn_count_1h",
        "customer_features:txn_count_24h",
        "customer_features:avg_txn_amount_7d",
    ],
    entity_rows=[{"customer_id": 1001}, {"customer_id": 1002}],
).to_df()

# Returns the latest feature values for those entity IDs
print(online_features)
```

---

## When to Use a Feature Store

| Scenario | Use feature store |
|---|---|
| 1 model, 1 team, batch only | No — simple pipeline is fine |
| Multiple models sharing features | Yes |
| Real-time serving with historical aggregates | Yes |
| Feature audit / lineage requirements | Yes |
| Multiple teams, preventing duplication | Yes |

---

## References

## Links

**ML Engineering**
- [[05_ml_engineering/02_data_engineering/feature_store|Feature Store]] — training-serving skew, offline/online stores, point-in-time joins theory
- [[05_ml_engineering/02_data_engineering/data_pipeline_patterns|Data Pipeline Patterns]] — upstream data ingestion feeding the feature store

**Applications**
- [[training_pipeline_pattern|Training Pipeline Pattern]] — feature store is the first stage of the training pipeline
- [[tabular_classification_pipeline|Tabular Classification Pipeline]] — end-to-end example using feature engineering
