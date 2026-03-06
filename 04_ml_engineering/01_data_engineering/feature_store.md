---
layer: 04_ml_engineering
type: engineering
tool: feast
status: growing
tags: [feature-store, feast, training-serving-skew, online-offline]
created: 2026-03-05
---

# Feature Store

## Purpose

A feature store is a centralised system for defining, computing, storing, and serving ML features. It solves two endemic problems in ML engineering: **training-serving skew** (the model sees different feature values during training and inference) and **feature duplication** (multiple teams independently re-implementing the same features with subtle differences). Feature stores are most valuable at scale—when many models share features, when real-time features are needed, or when point-in-time correctness is critical.

## Architecture

### Training-Serving Skew

Training-serving skew is among the most insidious failure modes in production ML. It occurs when:

- Training uses a Python function to compute a feature; serving uses a different implementation in Java or SQL.
- Training reads from a historical snapshot; serving reads from a live database that has been mutated since training.
- Preprocessing steps (normalisation, imputation, encoding) are applied in training but omitted at serving time, or applied with parameters derived from a different dataset.

A feature store addresses this by making features **compute-once, serve-anywhere**: the same transformation logic is used to populate the historical offline store (used for training) and the real-time online store (used for serving).

### Online vs. Offline Stores

**Offline store**: A columnar data lake or warehouse (e.g., S3 + Parquet, BigQuery, Snowflake) holding historical feature values timestamped to the second or millisecond. Used to construct training datasets via point-in-time correct joins. Read access is bulk and analytical; latency is seconds to minutes.

**Online store**: A low-latency key-value store (Redis, DynamoDB, Bigtable) holding only the latest feature values per entity. Read access is row-level at serving time; latency is 1–10 ms. Updated incrementally as new data arrives (streaming ingestion) or periodically via batch materialisation jobs.

The two stores are complementary: the offline store is for training, the online store is for inference.

### Point-in-Time Correct Joins

When constructing a training dataset, naively joining feature tables to label tables on entity ID produces **label leakage**: future feature values are used to predict past labels. Point-in-time correct joins (also called "as-of joins") ensure that for each training example at timestamp *t*, only feature values available before *t* are used.

Feast performs point-in-time correct joins via:
1. A label/event table with `(entity_id, event_timestamp, label)` rows.
2. Feature tables with `(entity_id, event_timestamp, feature_value)` rows.
3. For each row in the label table, Feast looks up the most recent feature value with `event_timestamp ≤ label.event_timestamp`.

This is the correct way to avoid data leakage in time-series and event-based ML problems.

### Feast Architecture Overview

[Feast](https://feast.dev) is the leading open-source feature store.

**Core concepts:**
- **Entity**: The subject of prediction (e.g., `user_id`, `item_id`). Defines the join key.
- **Feature view**: A named group of features derived from a single data source, associated with an entity and a TTL (time-to-live for online store expiry).
- **Data source**: A pointer to offline data (Parquet, BigQuery table) or a streaming source (Kafka topic).
- **Feature service**: A named collection of feature views used by a specific model. Acts as a contract between the ML team and the data engineering team.

**Workflow:**
1. Define entities, feature views, and data sources in Python.
2. `feast apply` registers definitions to the feature registry.
3. `feast materialize` populates the online store from the offline store for a given time range.
4. Training: `store.get_historical_features(entity_df, features)` returns a point-in-time correct DataFrame.
5. Serving: `store.get_online_features(entity_rows, features)` returns the latest feature values in <10 ms.

```python
# Example: retrieve online features at serving time
feature_vector = store.get_online_features(
    features=["user_stats:purchase_count_7d", "user_stats:avg_session_length"],
    entity_rows=[{"user_id": 1234}],
).to_dict()
```

### When to Use a Feature Store vs. Simple Pipelines

| Situation | Recommendation |
|---|---|
| Single model, single team, <10 features | Simple pipeline; feature store adds overhead |
| Multiple models sharing features | Feature store prevents duplication and skew |
| Real-time features required | Feature store's online store is the right abstraction |
| Strong need for point-in-time correctness | Feature store |
| Regulated environment requiring feature auditability | Feature store with lineage tracking |

For early-stage projects, a well-structured Pandas/SQL pipeline with explicit column documentation is often sufficient. Introduce a feature store when the cost of skew bugs and duplication exceeds the setup cost.

## Implementation Notes

- Always define a TTL for feature views in the online store. Stale features (e.g., a user's features from 90 days ago) are worse than missing features in many applications.
- Keep feature transformations stateless and pure where possible. Stateful transformations (e.g., rolling windows) must be computed consistently in both offline and online paths.
- Use the feature service abstraction to create a versioned contract: `model_v2_feature_service` vs. `model_v3_feature_service`. This allows safe iteration without breaking live models.
- Monitor feature freshness and null rates in the online store as part of serving health checks.

## Trade-offs

| Concern | Trade-off |
|---|---|
| Consistency (online ↔ offline) | Higher infrastructure cost and operational complexity |
| Low-latency serving | Requires maintaining and updating a separate online store |
| Point-in-time correctness | Requires timestamped data throughout; hard to retrofit |
| Feast open-source vs. managed | Flexibility vs. operational burden |

## References

- Feast documentation: docs.feast.dev
- Huyen, C. (2022). *Designing Machine Learning Systems*. O'Reilly. Chapter 5.
- Garg, N. et al. (2022). *Feature Stores for ML*. Tecton blog.

## Links
- [[data_pipeline_patterns|Data Pipeline Patterns]]
- [[dataset_versioning|Dataset Versioning and Lineage]]
- [[ml_lifecycle|ML Lifecycle]]
- [[ml_system_design|ML System Design]]
