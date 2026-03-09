---
layer: 09_projects
type: project
status: completed
owner: jesper
created: 2026-03-09
tags: [data-science, insurance, tabular, classification]
---

# CitiBike NYC — Demand, Risk & Net Flow Analysis

A data science study combining **CitiBike trip data (2023–2025)** with **NYPD collision data** to produce demand analysis, station-level risk scoring, and a net-flow imbalance predictor. The outputs support **insurance pricing**, **user safety warnings**, and **operational interventions** at the station level.

**Published report:** https://armoutihansen.xyz/DSC/  
**Repository:** https://github.com/armoutihansen/DSC

## Goal

Derive actionable, data-driven signals from CitiBike trip and collision data to help an insurer (AXA) price micro-mobility risk and flag high-risk contexts to users in real time.

Three concrete deliverables:
1. Demand characterisation — seasonal, weekly, and hourly patterns; member vs casual split.
2. Risk measure — transparent, interpretable risk-per-trip by station, time of day, and their interaction.
3. Net-flow prediction — identify tomorrow's over-supplied (importer) and under-supplied (exporter) stations.

## Scope (In / Out)

**In:**
- CitiBike trip data (2023–2025): ~80 M trips, station coordinates, bike type, membership status, duration, distance
- NYPD Motor Vehicle Collision data filtered to cyclist involvement
- Station-level and time-of-day demand EDA
- Risk score construction: crashes-per-trip, severity weighting, station × time interaction
- Three-class net-flow imbalance classification (importer / balanced / exporter)
- Interactive HTML report published at armoutihansen.xyz/DSC

**Out:**
- Real-time pipeline / API
- Individual trip-level accident probability model
- Weather-conditioned demand forecasting (exploratory only via meteostat)
- External validation with insurance claim data

## Deliverables

| Artefact | Description |
|---|---|
| `notebooks/EDA_citibike.ipynb` | Demand, usage patterns, net flow exploratory analysis |
| `notebooks/clean_citibike.ipynb` | Cleaning workflow for raw CitiBike CSVs |
| `notebooks/clean_collision_data.ipynb` | Collision parsing — cyclist-involvement flag, severity |
| `notebooks/risk_analysis.ipynb` | Station-level, time-of-day, and interaction risk scores |
| `notebooks/net_flow_analysis.ipynb` | Baseline → logistic regression → CatBoost net-flow classifier |
| `src/download_citibike.py` | Helper to bulk-download trip data from S3 |
| `src/clean_citibike_csv.py` | Batch cleaning + Parquet export |
| `index.html` | Self-contained report with all figures (Plotly, folium maps) |

## Data

| Dataset | Source | Size |
|---|---|---|
| CitiBike trip data | [S3 bucket](https://s3.amazonaws.com/tripdata/index.html) | ~80 M rows, 2023–2025 |
| NYPD Motor Vehicle Collisions | [NYC Open Data](https://data.cityofnewyork.us/Public-Safety/Motor-Vehicle-Collisions-Crashes/h9gi-nx95/about_data) | ~2 M rows (filtered to cyclist) |

Key preprocessing steps:
- Parquet conversion via PyArrow for efficient columnar reads
- DuckDB for in-process SQL over Parquet files (avoids pandas memory ceiling)
- Cyclist-involvement flag parsed from free-text contributing factor columns
- Station-day aggregation: total trips, net flow (arrivals − departures), crash count, crash rate

## Modeling

### Risk Score

A transparent **risk-per-trip** measure computed as:

$$
\text{risk}(s, t) = \frac{\text{crashes involving cyclists at station } s \text{ during period } t}{\text{trips departing station } s \text{ during period } t}
$$

Three granularities computed: station-level, time-of-day, and station × time-of-day interaction. Severity weighting applied (fatal > injury > property damage).

### Net Flow Imbalance Classifier

**Task:** predict tomorrow's net-flow class for each station — *importer* (arrivals > departures, class +1), *balanced* (class 0), *exporter* (departures > arrivals, class −1).

**Class distribution:** ~80% balanced, ~10% each importer/exporter — severe imbalance.

| Model | Macro-F1 | Exporter Recall |
|---|---|---|
| Trivial baseline (predict majority) | 0.17 | 0.00 |
| Persistence baseline (yesterday = today) | 0.28 | 0.19 |
| Logistic Regression | 0.40 | 0.50 |
| **CatBoost** | **0.51** | **0.60** |

Features: lag-1 net flow, rolling mean (7-day), day of week, month, station capacity, historical mean net flow per station.

CatBoost was selected for its native handling of categorical features (station ID) and superior recall on the minority classes.

## Engineering

| Concern | Choice |
|---|---|
| Language | Python 3.11 |
| Data query | DuckDB (in-process SQL over Parquet) |
| Data frames | pandas 2 + PyArrow backend |
| Modelling | scikit-learn, CatBoost 1.2 |
| Visualisation | matplotlib, seaborn, Plotly, folium |
| Weather features | meteostat |
| Env management | conda (`environment.yml`) |
| Report | Static HTML (Plotly + folium inline) |

Raw data not included in repo — must be downloaded via the provided helper scripts (~100 GB uncompressed).

## Timeline

Completed (AXA data science challenge project).

## Links

- [[09_projects/_completed/citibike-nyc-axa/lessons_learned|Lessons Learned]]
- [[02_data_science/03_exploratory_data_analysis/exploratory_data_analysis|Exploratory Data Analysis]]
- [[02_data_science/04_feature_engineering/feature_engineering|Feature Engineering]]
- [[03_modeling/01_supervised_learning/02_tree_based_models/tree_ensembles|Tree Ensembles (Gradient Boosting)]]
- [[03_modeling/07_evaluation_and_model_selection/evaluation_and_validation|Evaluation and Validation]]
- [[02_data_science/01_problem_framing/problem_framing|Problem Framing]]
