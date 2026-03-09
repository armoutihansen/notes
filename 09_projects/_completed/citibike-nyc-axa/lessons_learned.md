---
layer: 09_projects
type: project
status: evergreen
owner: jesper
created: 2026-03-09
tags: [data-science, insurance, tabular, classification, lessons]
---

# Lessons Learned — CitiBike NYC Risk & Net Flow Analysis

## Working with Large Tabular Data (>80 M rows)

- **DuckDB over pandas for initial aggregation**: loading 80 M trip rows into a DataFrame is unnecessary and slow. DuckDB reads Parquet files column-selectively with SQL, keeping memory usage low. Convert to pandas only after aggregation to the required granularity (e.g., station-day).
- **Parquet as the canonical intermediate format**: PyArrow's Parquet files compress ~5× vs CSV and support column pruning. All cleaned data should be written to Parquet immediately after preprocessing.
- **Schema enforcement early**: define column types and expected ranges before any joins to catch silent casting errors (e.g., station IDs stored as float in one dataset, string in another).

## Class Imbalance in Multi-Class Classification

The net-flow prediction target had ~80% balanced / ~10% importer / ~10% exporter — classic long-tail multi-class imbalance.

Key lessons:
- **Macro-F1, not accuracy**: accuracy is meaningless when 80% of labels are one class. Macro-F1 treats all classes equally and directly measures minority-class performance.
- **Recall is the right optimisation target for operational use**: missing an exporter day (station empties) is more costly than a false alarm. Favour recall on minority classes.
- **CatBoost outperformed logistic regression on recall** (60% vs 50% for exporters) because it learns non-linear interaction effects (e.g., station × day-of-week) without manual feature crossing.
- **Persistence baseline is a strong floor**: yesterday's class is often correct for a stationary series; any ML model must beat it meaningfully to justify deployment.

## Risk Score Construction from Sparse Count Data

Building a risk-per-trip measure at fine granularity (station × time-of-day) introduced sparsity:
- Many cells have zero crashes over the study period — not because they are safe, but because they are rarely visited.
- **Exposure-normalised rates**: always express risk as crashes **per trip** (or per 1,000 trips), not raw crash counts. High-traffic stations appear dangerous in absolute terms but may be safe relative to volume.
- **Minimum exposure threshold**: cells with fewer than a configurable minimum number of trips should be reported as "insufficient data" rather than zero risk, to avoid false confidence.
- **Smoothing/regularisation**: Bayesian shrinkage toward the overall mean risk is appropriate for low-count cells (analogous to MAP estimation). Not implemented here but flagged as future work.

## Feature Engineering for Tabular Time Series

The net-flow classifier depended entirely on engineered features from the raw time series:
- **Lag features** (lag-1 net flow) were the single strongest predictor — station-days are serially correlated.
- **Rolling statistics** (7-day mean net flow) captured weekly seasonality.
- **Calendar features** (day of week, month) captured systematic patterns without stationarity assumptions.
- **Station identity** as a categorical feature (natively handled by CatBoost) captured station-level fixed effects that would otherwise require one-hot encoding.
- **Pitfall**: all lag features must be computed *before* the train/validation/test split to avoid leaking future information. Split first on time axis, then compute lags within each split's lookback window — or compute globally with strict temporal ordering.

## Spatial Joins and Station-Level Aggregation

- CitiBike stations are identified by name (not always a stable ID across years). Station name deduplication and coordinate standardisation via median lat/lon is more robust than a direct string join.
- Collision data contains GPS coordinates but not station IDs — a spatial join (nearest station within a configurable radius) is required. Using a KD-tree (`scipy.spatial.cKDTree`) is ~100× faster than brute-force pairwise distance for this join.
- **Assignment radius matters**: a large radius inflates risk at hub stations; a small radius misses collisions on nearby roads. Sensitivity analysis across radii should be reported.

## Report Communication

- Folium choropleth maps (station-level risk, net flow imbalance by station) communicate spatial patterns far better than tables to stakeholders.
- Plotly interactive charts allow stakeholders to drill down into time-of-day patterns without needing to run notebooks.
- The static HTML report (`index.html` with embedded Plotly/folium) is portable and shareable without any server infrastructure.

## Extracted to Core Layers

- [[03_modeling/01_supervised_learning/02_tree_based_models/tree_ensembles|Tree Ensembles]] — CatBoost, gradient boosting for tabular classification
- [[02_data_science/04_feature_engineering/feature_engineering|Feature Engineering]] — lag features, rolling statistics, calendar features
- [[02_data_science/03_exploratory_data_analysis/exploratory_data_analysis|Exploratory Data Analysis]] — exposure-normalised rates, sparse count data patterns
- [[03_modeling/07_evaluation_and_model_selection/evaluation_and_validation|Evaluation and Validation]] — Macro-F1, class imbalance metrics

## Links

- [[09_projects/_completed/citibike-nyc-axa/overview|Project Overview]]
