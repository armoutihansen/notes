---
layer: 09_projects
type: workflow
status: growing
owner: jesper
created: 2026-03-09
tags: [mlops, ml-engineering, fastapi, mlflow, airflow, pydantic]
---

# CitiBike MLOps — Step-by-Step Implementation Guide

Companion to [[09_projects/_active/citibike-mlops/overview|Overview]].

Each phase builds on the previous. Phases 0–2 produce the data and feature layer; Phases 3–5 produce trained, registered models; Phases 6–9 wrap them in a production-style service. Phase 10 describes natural extensions.

**Data structure note:** CitiBike is **panel time series** — multiple stations each with a daily time series of net flow and crash events. This structure has three important consequences: (1) lag and rolling features must be computed per station, never across stations; (2) cross-validation must respect temporal order — random splits leak future values into training; (3) models must handle cross-sectional dependence (nearby stations share traffic patterns).

---

## Phase 0 — Problem Framing & Data Audit

**Purpose:** Define what success looks like *before* touching model code, and audit the time series structure of the data to choose appropriate model families. Skipping this phase leads to misspecified models (e.g., Poisson on overdispersed data), leaking cross-validation splits, and post-hoc metric selection. This phase produces a `DECISIONS.md` file that records all modelling assumptions.

**Tasks:**
- [ ] Write success metrics and SLA thresholds in `DECISIONS.md`
- [ ] Create repo, `uv` environment, and Docker Compose stack
- [ ] Plot ACF/PACF of net flow for a sample of 10 stations — record which lags are significant
- [ ] Run ADF stationarity test on net flow per station
- [ ] Decompose net flow with STL to identify seasonal components
- [ ] Check zero-inflation in crash counts — what fraction of station-days have zero crashes?
- [ ] Check panel balance — are all stations observed daily, or are there gaps?

### 0.1 Clarify success metrics

Record the following in `DECISIONS.md` before writing any model code. Having explicit thresholds prevents post-hoc metric selection.

- **Primary metric (risk model):** Poisson deviance on held-out stations (spatial holdout)
- **Primary metric (net flow model):** MAE on a temporally held-out period (last 90 days); MASE relative to seasonal naive baseline
- **Serving SLA:** p95 latency < 200ms, error rate < 0.1%
- **Drift threshold:** alert when PSI > 0.2 on any input feature or prediction distribution
- **Promotion threshold:** new model must improve primary metric by > 1% over production to be auto-promoted

### 0.2 Time series data audit

Before choosing a model, characterise the temporal structure of the target variables. This determines whether lag features need differencing, whether seasonal patterns are additive or multiplicative, and whether a Poisson or Negative Binomial family is appropriate for crash counts.

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from statsmodels.tsa.stattools import adfuller
from statsmodels.tsa.seasonal import STL
from statsmodels.graphics.tsaplots import plot_acf, plot_pacf

# Load gold-layer station-day data
df = pd.read_parquet("data/gold/station_day.parquet")

# --- Net flow: ACF / PACF for a sample of stations ---
sample_stations = df["station_id"].value_counts().head(10).index
fig, axes = plt.subplots(10, 2, figsize=(14, 30))
for i, sid in enumerate(sample_stations):
    s = df.query(f"station_id == '{sid}'").set_index("date")["net_flow"].dropna()
    plot_acf(s, ax=axes[i, 0], lags=28, title=f"ACF: {sid}")
    plot_pacf(s, ax=axes[i, 1], lags=28, title=f"PACF: {sid}")
plt.tight_layout()
plt.savefig("eda/acf_pacf_netflow.png")
# Record: which lags are significant? Typically 1, 7 for daily CitiBike data.

# --- Stationarity: ADF test per station ---
adf_results = []
for sid, grp in df.groupby("station_id"):
    series = grp.set_index("date")["net_flow"].dropna()
    if len(series) < 30:
        continue
    stat, pval, *_ = adfuller(series)
    adf_results.append({"station_id": sid, "adf_stat": stat, "p_value": pval, "stationary": pval < 0.05})
adf_df = pd.DataFrame(adf_results)
print(f"Fraction of stationary stations: {adf_df['stationary'].mean():.2%}")
# If most stations are non-stationary → consider first-differencing before lag features

# --- Seasonality: STL decomposition on the aggregate (all stations pooled) ---
agg = df.groupby("date")["net_flow"].mean().sort_index()
stl = STL(agg, period=7)   # weekly seasonality
result = stl.fit()
fig = result.plot()
plt.savefig("eda/stl_decomposition.png")
# Look at: trend (is there a secular drift?), seasonal (is the weekly pattern stable?), residual (is noise IID?)

# --- Zero-inflation check for crash rate ---
crash_zero_frac = (df["crash_count"] == 0).mean()
print(f"Fraction of station-days with zero crashes: {crash_zero_frac:.2%}")
# > 90% zeros → consider zero-inflated Poisson (ZIP) or Negative Binomial
# Check overdispersion: variance >> mean → Negative Binomial preferred over Poisson
mean_c = df["crash_count"].mean()
var_c  = df["crash_count"].var()
print(f"Crash count: mean={mean_c:.4f}, var={var_c:.4f}, ratio={var_c/mean_c:.1f}")
# Ratio >> 1 → overdispersed → Negative Binomial

# --- Panel balance: are all stations observed every day? ---
n_stations = df["station_id"].nunique()
n_days     = df["date"].nunique()
n_obs      = len(df)
print(f"Stations: {n_stations}, Days: {n_days}, Expected full panel: {n_stations*n_days}, Actual: {n_obs}")
# Missing station-days → forward-fill or exclude from lag computation
```

**What to record in `DECISIONS.md`:**
- Which lags are significant in the PACF (typically 1 and 7 for daily CitiBike data)
- Fraction of non-stationary stations (if high → use differenced net flow as target)
- Whether crash counts are zero-inflated and overdispersed (if ratio > 3 → use Negative Binomial)
- Whether seasonal pattern is additive or multiplicative (usually additive for net flow)

### 0.3 Repository setup

```bash
mkdir citibike-mlops && cd citibike-mlops
git init
uv init
uv add fastapi uvicorn "pydantic>=2" mlflow apache-airflow \
       lightgbm xgboost catboost scikit-learn statsmodels pmdarima prophet \
       duckdb pyarrow evidently python-dotenv pytest pytest-asyncio httpx
```

Recommended project structure:

```
citibike-mlops/
├── dags/
│   ├── ingest_dag.py
│   └── retrain_dag.py
├── src/
│   ├── features/
│   │   ├── schemas.py        # Pydantic feature schemas
│   │   └── engineer.py       # Feature engineering functions
│   ├── models/
│   │   ├── risk.py           # Risk model (NegBin GLM / XGBoost Poisson)
│   │   └── netflow.py        # Net flow regression model
│   ├── train.py              # MLflow training entry point
│   └── evaluate.py           # Offline evaluation metrics
├── api/
│   ├── main.py               # FastAPI app
│   ├── schemas.py            # Request/response Pydantic models
│   └── model_loader.py       # MLflow model loading
├── monitoring/
│   └── drift.py              # Evidently drift reports
├── cache/
│   └── station_features.parquet  # Precomputed lag features for inference
├── eda/                      # EDA scripts and plots (not committed)
├── tests/
│   ├── test_features.py
│   ├── test_models.py
│   └── test_api.py
├── data/
│   ├── bronze/               # Raw Parquet (no transformation)
│   ├── silver/               # Cleaned, validated
│   └── gold/                 # Feature-engineered, model-ready
├── DECISIONS.md              # Modelling assumptions and threshold choices
├── docker-compose.yml
└── pyproject.toml
```

### 0.4 Docker Compose stack

The local stack has three services. Run `docker compose up` before starting any other phase.

```yaml
services:
  mlflow:
    image: ghcr.io/mlflow/mlflow:latest
    ports: ["5000:5000"]
    command: mlflow server --host 0.0.0.0 --backend-store-uri sqlite:///mlflow.db --artifact-root /mlruns
    volumes: ["./mlruns:/mlruns"]

  airflow:
    image: apache/airflow:2.9.1
    environment:
      AIRFLOW__CORE__EXECUTOR: LocalExecutor
      AIRFLOW__DATABASE__SQL_ALCHEMY_CONN: sqlite:////opt/airflow/airflow.db
    volumes: ["./dags:/opt/airflow/dags", "./data:/data"]
    ports: ["8080:8080"]
    command: standalone

  api:
    build: .
    ports: ["8000:8000"]
    environment:
      MLFLOW_TRACKING_URI: http://mlflow:5000
    command: uvicorn api.main:app --host 0.0.0.0 --port 8000 --reload
    depends_on: [mlflow]
```

Verify the stack: `curl http://localhost:5000/health` (MLflow), open `http://localhost:8080` (Airflow UI), `curl http://localhost:8000/health` (FastAPI).

---

## Phase 1 — Data Pipeline (Airflow)

**Purpose:** Automate the full data ingestion and cleaning pipeline as a reproducible, scheduled DAG. Without this, the data pipeline is a collection of manual scripts run in the right order — non-reproducible and brittle. The output (bronze → silver → gold Parquet tables) is the stable input to all downstream phases. Changes to cleaning logic are captured in version-controlled DAG code, not in an analyst's notebook history. The gold layer also writes a daily **feature cache** (`cache/station_features.parquet`) that the API needs at inference time.

**Tasks:**
- [ ] Write `ingest_dag.py` with download, Parquet conversion, and silver cleaning tasks
- [ ] Implement DuckDB silver cleaning: standardise columns, filter invalid rows, merge collision data
- [ ] Write station-day aggregate to `data/gold/station_day.parquet`
- [ ] Write the daily feature cache to `cache/station_features.parquet` (lag features needed by API)
- [ ] Trigger the DAG manually in the Airflow UI and verify all three layers are populated
- [ ] Inspect row counts at each layer to confirm no data loss

### 1.1 Bronze layer: raw ingestion DAG

`dags/ingest_dag.py` — runs weekly. The DAG has three tasks: download, convert to Parquet, clean to silver.

```python
from airflow.decorators import dag, task
from datetime import datetime

@dag(schedule="@weekly", start_date=datetime(2024, 1, 1), catchup=False,
     tags=["ingest"])
def ingest_dag():

    @task
    def download_citibike(month: str) -> str:
        """Download one month of CitiBike CSVs from the public S3 bucket.
        Returns the local path of the downloaded zip file."""
        import urllib.request, os
        url = f"https://s3.amazonaws.com/tripdata/{month}-citibike-tripdata.csv.zip"
        out = f"/data/raw/citibike/{month}.csv.zip"
        os.makedirs("/data/raw/citibike", exist_ok=True)
        urllib.request.urlretrieve(url, out)
        return out

    @task
    def convert_to_parquet(zip_path: str) -> str:
        """Unzip and convert raw CSV to Parquet — bronze layer.
        No transformations: schema must exactly match the source."""
        import duckdb, os
        out = zip_path.replace("/raw/citibike/", "/bronze/citibike/").replace(".csv.zip", ".parquet")
        os.makedirs(os.path.dirname(out), exist_ok=True)
        duckdb.sql(
            f"COPY (SELECT * FROM read_csv_auto('{zip_path}')) TO '{out}' (FORMAT PARQUET)"
        )
        return out

    @task
    def clean_to_silver(parquet_path: str) -> str:
        """Validate schema, standardise column names, filter invalid rows → silver layer.
        Silver = bronze + cleaning; no feature engineering yet."""
        import duckdb, os
        out = parquet_path.replace("/bronze/", "/silver/")
        os.makedirs(os.path.dirname(out), exist_ok=True)
        duckdb.sql(f"""
            COPY (
                SELECT
                    ride_id,
                    rideable_type,
                    strptime(started_at, '%Y-%m-%d %H:%M:%S') AS started_at,
                    strptime(ended_at,   '%Y-%m-%d %H:%M:%S') AS ended_at,
                    start_station_id,
                    end_station_id,
                    start_lat, start_lng,
                    end_lat,   end_lng,
                    member_casual
                FROM read_parquet('{parquet_path}')
                WHERE start_station_id IS NOT NULL
                  AND end_station_id   IS NOT NULL
                  AND started_at < ended_at
                  AND date_diff('minute', started_at, ended_at) < 1440
            ) TO '{out}' (FORMAT PARQUET)
        """)
        return out

    zip_path = download_citibike("{{ ds[:7] }}")
    parquet  = convert_to_parquet(zip_path)
    clean_to_silver(parquet)

ingest_dag()
```

### 1.2 Gold layer: station-day aggregates (DuckDB)

DuckDB reads Parquet directly and avoids pandas memory ceiling — a lesson from the original DSC analysis on 80M+ trip rows. The gold layer contains the model-ready aggregated tables.

```python
import duckdb, os

conn = duckdb.connect()
os.makedirs("data/gold", exist_ok=True)

# Station-day aggregate: used for net flow model and daily risk model
conn.execute("""
    COPY (
        SELECT
            start_station_id                                          AS station_id,
            date_trunc('day', started_at)::DATE                      AS date,
            COUNT(*)                                                  AS total_departures,
            SUM(rideable_type = 'electric_bike')                      AS electric_departures,
            -- net_flow computed separately by joining arrivals; shown here for illustration
            0::INTEGER                                                AS net_flow_placeholder
        FROM read_parquet('data/silver/citibike/*.parquet')
        GROUP BY 1, 2
    ) TO 'data/gold/station_day.parquet' (FORMAT PARQUET)
""")

# Merge NYPD collision counts: join on station nearest to collision, day
conn.execute("""
    COPY (
        SELECT
            s.*,
            COALESCE(c.crash_count, 0)  AS crash_count,
            COALESCE(c.severity_score, 0) AS crash_severity
        FROM read_parquet('data/gold/station_day.parquet') s
        LEFT JOIN read_parquet('data/silver/collisions_station_day.parquet') c
            ON s.station_id = c.station_id AND s.date = c.date
    ) TO 'data/gold/station_day_risk.parquet' (FORMAT PARQUET)
""")
```

### 1.3 Feature cache for inference

The API needs lag features at inference time (lag-1, lag-7, rolling-7). These cannot be computed on-the-fly from a single request. Write a daily **feature cache** at the end of `ingest_dag` so the API can look them up:

```python
@task
def update_feature_cache():
    """Compute lag features for all stations for the most recent date.
    Written to cache/station_features.parquet for use by the FastAPI service."""
    import duckdb, pandas as pd, numpy as np

    df = duckdb.sql("""
        SELECT station_id, date, net_flow, crash_count
        FROM read_parquet('data/gold/station_day_risk.parquet')
        ORDER BY station_id, date
    """).df()

    df = df.sort_values(["station_id", "date"])
    for col in ["net_flow", "crash_count"]:
        df[f"{col}_lag1"]  = df.groupby("station_id")[col].shift(1)
        df[f"{col}_lag7"]  = df.groupby("station_id")[col].shift(7)
        df[f"{col}_roll7"] = df.groupby("station_id")[col].transform(
            lambda x: x.shift(1).rolling(7, min_periods=1).mean()
        )

    # Keep only the most recent date per station for the cache
    latest = df.groupby("station_id")["date"].max().reset_index()
    cache  = df.merge(latest, on=["station_id", "date"])
    cache.to_parquet("cache/station_features.parquet", index=False)
```

---

## Phase 2 — Feature Engineering & Schema Validation (Pydantic)

**Purpose:** Transform raw aggregates into model-ready features that capture the temporal structure of the data, and validate them with typed schemas at every system boundary. Without this phase: (1) time-related features encoded as raw integers leak temporal order to tree models in unexpected ways; (2) lag features computed without a per-station sort quietly introduce cross-station leakage; (3) features passed between pipeline stages have no enforced schema, making silent dtype mismatches common. Pydantic schemas serve double duty: pipeline validation during training and API request validation at inference.

**Tasks:**
- [ ] Implement `cyclical_encode` for hour, day-of-week, month, and day-of-year
- [ ] Implement `fourier_features` for weekly and annual seasonality (Fourier terms)
- [ ] Implement `add_lag_features` with per-station sort and `shift(1)` leakage guard
- [ ] Implement spatial lag: mean net flow of the k nearest stations at lag-1
- [ ] Add `is_holiday` flag using the `holidays` package
- [ ] Run PACF on the gold-layer net flow series and confirm which lags to include
- [ ] Write `RiskFeatures` and `NetFlowFeatures` Pydantic v2 models with field validators

### 2.1 Why cyclical encoding

Tree-based models (XGBoost, LightGBM, CatBoost) split on raw numeric values. If hour-of-day is encoded as an integer 0–23, the model cannot infer that hour 23 and hour 0 are adjacent — it sees them as maximally distant. Cyclical encoding maps each period onto a unit circle using `sin` and `cos`, so all models (tree-based or linear) see temporal proximity correctly.

```python
import numpy as np, pandas as pd

def cyclical_encode(series: pd.Series, period: float) -> tuple[pd.Series, pd.Series]:
    """Encode a periodic integer variable onto (sin, cos) on the unit circle.
    period: the number of unique values before the cycle repeats (e.g., 24 for hours).
    Returns (sin_component, cos_component) with values in [-1, 1]."""
    angle = 2 * np.pi * series / period
    return np.sin(angle), np.cos(angle)

def add_time_features(df: pd.DataFrame) -> pd.DataFrame:
    """Add all cyclical time encodings to a station-day or station-hour DataFrame.
    Requires columns: 'date' (pd.Timestamp)."""
    df = df.copy()
    df["dow"]  = df["date"].dt.dayofweek   # 0=Monday
    df["month"] = df["date"].dt.month
    df["doy"]   = df["date"].dt.dayofyear
    df["hour"]  = df["date"].dt.hour if "hour" in df.columns else 0

    df["hour_sin"],  df["hour_cos"]  = cyclical_encode(df["hour"], 24)
    df["dow_sin"],   df["dow_cos"]   = cyclical_encode(df["dow"],  7)
    df["month_sin"], df["month_cos"] = cyclical_encode(df["month"], 12)
    df["doy_sin"],   df["doy_cos"]   = cyclical_encode(df["doy"],  365.25)
    return df
```

### 2.2 Fourier features for seasonality (better than single sin/cos)

A single sin/cos pair captures only a pure sinusoidal seasonality. Real CitiBike weekly patterns have sharp Monday/Friday transitions and flat weekends — this requires multiple harmonics. Use Fourier features with `n_terms` pairs per seasonal period.

```python
def fourier_features(
    dates: pd.Series,
    period: float,
    n_terms: int,
    prefix: str,
) -> pd.DataFrame:
    """Generate K pairs of (sin, cos) Fourier terms for a given seasonal period.
    period: float — length of the cycle in days (7.0 for weekly, 365.25 for annual).
    n_terms: int  — number of harmonics (start with 3 for weekly, 4 for annual).
    prefix: str   — column name prefix, e.g. 'week' or 'year'.
    """
    t = np.arange(len(dates), dtype=float)
    cols = {}
    for k in range(1, n_terms + 1):
        cols[f"{prefix}_sin_{k}"] = np.sin(2 * np.pi * k * t / period)
        cols[f"{prefix}_cos_{k}"] = np.cos(2 * np.pi * k * t / period)
    return pd.DataFrame(cols, index=dates.index)

# Usage — call after sorting by date:
weekly_fourier = fourier_features(df["date"], period=7.0,    n_terms=3, prefix="week")
annual_fourier = fourier_features(df["date"], period=365.25, n_terms=4, prefix="year")
df = pd.concat([df, weekly_fourier, annual_fourier], axis=1)
```

### 2.3 PACF-guided lag selection

Before hardcoding lag-1 and lag-7, verify which lags matter for net flow by inspecting the PACF from Phase 0. Partial autocorrelation at lag $k$ measures the correlation between $y_t$ and $y_{t-k}$ after removing the effect of all intermediate lags.

```python
from statsmodels.graphics.tsaplots import plot_pacf
import matplotlib.pyplot as plt

# Use a representative high-volume station
station = df.query("station_id == '6926.10'").set_index("date")["net_flow"]
fig, ax = plt.subplots(figsize=(12, 4))
plot_pacf(station.dropna(), lags=28, ax=ax)
ax.set_title("PACF of net_flow — station 6926.10")
plt.savefig("eda/pacf_netflow.png")

# Decision rule: include lag k if the PACF bar exceeds ±1.96/√n (the 95% CI band)
# Typical finding for daily CitiBike: significant at lags 1 and 7
```

### 2.4 Lag features with leakage guard

Every lag and rolling feature must be computed with a per-station sort and a `shift(1)` before any rolling window. Without the shift, the rolling mean at time $t$ includes $y_t$ itself, leaking the target into a feature.

```python
def add_lag_features(df: pd.DataFrame, target_col: str, lags: list[int] = [1, 7]) -> pd.DataFrame:
    """Add lag and rolling features per station. ALWAYS sort by (station_id, date) first.
    shift(1) ensures no leakage: the value at row t sees only rows t-1, t-2, ..."""
    df = df.sort_values(["station_id", "date"]).copy()
    g = df.groupby("station_id", sort=False)[target_col]

    for lag in lags:
        df[f"{target_col}_lag{lag}"] = g.shift(lag)

    # Rolling statistics: shift(1) before rolling to end the window at t-1
    df[f"{target_col}_roll7_mean"] = g.transform(lambda x: x.shift(1).rolling(7,  min_periods=1).mean())
    df[f"{target_col}_roll7_std"]  = g.transform(lambda x: x.shift(1).rolling(7,  min_periods=1).std())
    df[f"{target_col}_roll28_mean"]= g.transform(lambda x: x.shift(1).rolling(28, min_periods=1).mean())
    return df
```

### 2.5 Spatial lag (cross-station dependence)

Neighbouring stations share rebalancing pressure: if station A is over-supplied, bikes are redistributed to nearby station B. Encoding the mean lag-1 net flow of the k nearest neighbours captures this signal.

```python
from sklearn.neighbors import BallTree

def add_spatial_lag(df: pd.DataFrame, k: int = 5) -> pd.DataFrame:
    """Add the mean lag-1 net flow of the k nearest stations as a feature.
    Requires 'lat' and 'lng' columns on the station metadata joined into df."""
    stations = df[["station_id", "lat", "lng"]].drop_duplicates()
    coords   = np.radians(stations[["lat", "lng"]].values)
    tree     = BallTree(coords, metric="haversine")

    # For each station, find k nearest neighbours (excluding itself)
    distances, indices = tree.query(coords, k=k + 1)
    neighbour_map = {
        row["station_id"]: stations.iloc[idx[1:]]["station_id"].tolist()
        for _, row, idx in zip(range(len(stations)), stations.itertuples(), indices)
    }

    # Compute lag-1 net flow per station, then average over neighbours
    lag1 = df.sort_values(["station_id", "date"]).copy()
    lag1["net_flow_lag1"] = lag1.groupby("station_id")["net_flow"].shift(1)
    lag1_lookup = lag1.set_index(["station_id", "date"])["net_flow_lag1"]

    def spatial_mean(row):
        neighbours = neighbour_map.get(row["station_id"], [])
        vals = [lag1_lookup.get((n, row["date"]), np.nan) for n in neighbours]
        return np.nanmean(vals) if vals else np.nan

    df["spatial_lag_net_flow"] = df.apply(spatial_mean, axis=1)
    return df
```

### 2.6 Holiday flag

```python
import holidays

def add_holiday_flag(df: pd.DataFrame) -> pd.DataFrame:
    us_holidays = holidays.US(years=range(2023, 2026))
    df["is_holiday"] = df["date"].dt.date.map(lambda d: d in us_holidays).astype(int)
    return df
```

### 2.7 Pydantic schemas for feature validation

`src/features/schemas.py` — validate every feature row before it enters the model. Pydantic v2 coerces types at the boundary (JSON → Python), generates a JSON Schema (consumed by FastAPI for OpenAPI docs), and provides field-level validators with descriptive error messages.

```python
from pydantic import BaseModel, Field, field_validator
from enum import Enum

class BikeType(str, Enum):
    electric = "electric_bike"
    classic  = "classic_bike"

class RiskFeatures(BaseModel):
    """Features for the crash-rate prediction model."""
    station_id:       str
    hour_sin:         float = Field(..., ge=-1, le=1)
    hour_cos:         float = Field(..., ge=-1, le=1)
    dow_sin:          float = Field(..., ge=-1, le=1)
    dow_cos:          float = Field(..., ge=-1, le=1)
    month_sin:        float = Field(..., ge=-1, le=1)
    month_cos:        float = Field(..., ge=-1, le=1)
    bike_type:        BikeType
    log_exposure:     float = Field(..., ge=0, description="log(total_trips) — offset for Poisson model")
    crash_rate_roll30: float = Field(default=0.0, ge=0, description="30-day rolling crash rate (causal)")

    @field_validator("log_exposure")
    @classmethod
    def exposure_nonneg(cls, v: float) -> float:
        if v < 0:
            raise ValueError("log_exposure must be non-negative")
        return v

class NetFlowFeatures(BaseModel):
    """Features for the net flow regression model."""
    station_id:           str
    net_flow_lag1:        float
    net_flow_lag7:        float
    net_flow_roll7_mean:  float
    net_flow_roll28_mean: float
    spatial_lag_net_flow: float
    week_sin_1:           float = Field(..., ge=-1, le=1)
    week_cos_1:           float = Field(..., ge=-1, le=1)
    week_sin_2:           float = Field(..., ge=-1, le=1)
    week_cos_2:           float = Field(..., ge=-1, le=1)
    week_sin_3:           float = Field(..., ge=-1, le=1)
    week_cos_3:           float = Field(..., ge=-1, le=1)
    year_sin_1:           float = Field(..., ge=-1, le=1)
    year_cos_1:           float = Field(..., ge=-1, le=1)
    station_cluster:      int   = Field(..., ge=0, le=9)
    is_holiday:           int   = Field(..., ge=0, le=1)
```

---

## Phase 3 — Experimentation (MLflow)

**Purpose:** Train and compare multiple models in a rigorous, reproducible way. MLflow records every run — parameters, metrics, code version, and the trained model artifact — so that you can answer "why did we choose this model?" six months later. Without experiment tracking you rely on scattered notebooks and memory, making the choice of production model untraceable and hard to reproduce. The discipline of logging baselines first forces an honest baseline comparison before investing in tuning.

**Tasks:**
- [ ] Start the MLflow tracking server (see Phase 0.4 Docker stack)
- [ ] Create experiments `citibike-risk` and `citibike-netflow` in the MLflow UI
- [ ] Log all baselines first (EB formula, persistence, seasonal naive, Prophet)
- [ ] Log Negative Binomial GLM baseline for risk model
- [ ] Log XGBoost / LightGBM / CatBoost runs with consistent parameter and metric keys
- [ ] Run a grid search over key hyperparameters, one MLflow run per configuration
- [ ] Compare runs in the MLflow UI — verify that at least one model beats every baseline

### 3.1 MLflow run structure

Every model variant is a **named MLflow run** inside a named experiment. Use consistent parameter and metric keys so runs are comparable across experiments.

```python
import mlflow, mlflow.sklearn
import xgboost as xgb
from sklearn.metrics import mean_squared_error, mean_absolute_error
import numpy as np

def poisson_deviance(y_true: np.ndarray, y_pred: np.ndarray) -> float:
    """Poisson deviance: 2 * sum(y*log(y/yhat) - (y - yhat)).
    Lower is better. Zero if predictions are perfect."""
    y_pred = np.clip(y_pred, 1e-8, None)
    return float(2 * np.sum(y_true * np.log(np.maximum(y_true, 1e-8) / y_pred) - (y_true - y_pred)))

mlflow.set_tracking_uri("http://localhost:5000")
mlflow.set_experiment("citibike-risk")

with mlflow.start_run(run_name="xgboost-poisson"):
    mlflow.log_params({
        "model_type":    "xgboost",
        "objective":     "count:poisson",
        "n_estimators":  500,
        "max_depth":     6,
        "learning_rate": 0.05,
        "feature_set":   "v2",     # version tag for the feature schema
    })

    model = xgb.XGBRegressor(objective="count:poisson", n_estimators=500, max_depth=6, learning_rate=0.05)
    model.fit(X_train, y_train, eval_set=[(X_val, y_val)], verbose=False)

    y_pred = model.predict(X_val)
    mlflow.log_metrics({
        "val_poisson_deviance": poisson_deviance(y_val, y_pred),
        "val_rmse":  mean_squared_error(y_val, y_pred, squared=False),
        "val_mae":   mean_absolute_error(y_val, y_pred),
    })

    mlflow.sklearn.log_model(model, artifact_path="model")
    fig = plot_feature_importance(model)
    mlflow.log_figure(fig, "feature_importance.png")
```

### 3.2 Baseline runs — log these before any model

Baselines must be logged as MLflow runs so they appear in the UI alongside model runs. Never tune a model without knowing what you are beating.

**Risk model baselines:**

```python
# Baseline 1: global mean crash rate
with mlflow.start_run(run_name="baseline-global-mean"):
    mlflow.log_param("model_type", "global_mean")
    y_pred_baseline = np.full_like(y_val, y_train.mean(), dtype=float)
    mlflow.log_metric("val_poisson_deviance", poisson_deviance(y_val, y_pred_baseline))
    mlflow.log_metric("val_mae",  mean_absolute_error(y_val, y_pred_baseline))

# Baseline 2: Empirical Bayes formula (replicate the original DSC risk score)
# λ = crashes/trips; smoothed with EB credibility weight
with mlflow.start_run(run_name="baseline-eb-formula"):
    mlflow.log_param("model_type", "empirical_bayes")
    global_rate = y_train.sum() / exposure_train.sum()
    eb_weight   = exposure_val / (exposure_val + 50)   # EB credibility, C=50
    y_pred_eb   = eb_weight * (crashes_val / exposure_val) + (1 - eb_weight) * global_rate
    mlflow.log_metric("val_poisson_deviance", poisson_deviance(y_val, y_pred_eb))

# Baseline 3: Negative Binomial GLM (correct for overdispersion)
import statsmodels.api as sm
with mlflow.start_run(run_name="baseline-negbin-glm"):
    mlflow.log_param("model_type", "negative_binomial_glm")
    glm = sm.GLM(y_train, X_train_glm, family=sm.families.NegativeBinomial()).fit()
    y_pred_nb = glm.predict(X_val_glm)
    mlflow.log_metric("val_poisson_deviance", poisson_deviance(y_val, y_pred_nb))
    mlflow.log_metric("val_mae",  mean_absolute_error(y_val, y_pred_nb))
    mlflow.sklearn.log_model(glm, artifact_path="model")
```

**Net flow baselines:**

```python
# Baseline 1: persistence (yesterday's net flow)
with mlflow.start_run(run_name="baseline-persistence"):
    mlflow.log_param("model_type", "persistence")
    y_pred_persist = X_val["net_flow_lag1"].values
    mlflow.log_metric("val_mae",  mean_absolute_error(y_val, y_pred_persist))
    mlflow.log_metric("val_rmse", mean_squared_error(y_val, y_pred_persist, squared=False))
    mlflow.log_metric("val_mase", mase(y_val, y_pred_persist, y_val))   # MASE=1 by definition

# Baseline 2: seasonal naive (same day last week)
with mlflow.start_run(run_name="baseline-seasonal-naive"):
    mlflow.log_param("model_type", "seasonal_naive")
    y_pred_snv = X_val["net_flow_lag7"].values
    mlflow.log_metric("val_mae",  mean_absolute_error(y_val, y_pred_snv))
    mlflow.log_metric("val_mase", mase(y_val, y_pred_snv, y_pred_persist))

# Baseline 3: Prophet (per station, then aggregate metrics)
from prophet import Prophet

def fit_prophet(station_df: pd.DataFrame) -> tuple[Prophet, pd.DataFrame]:
    """Fit Prophet to a single station's daily net flow time series."""
    df_prophet = station_df.rename(columns={"date": "ds", "net_flow": "y"})
    model = Prophet(
        weekly_seasonality=True,
        yearly_seasonality=True,
        seasonality_mode="additive",     # net flow can be negative
        changepoint_prior_scale=0.05,    # conservative, reduce overfitting
    )
    model.add_country_holidays(country_name="US")
    model.fit(df_prophet[df_prophet["ds"] < val_start])
    future  = model.make_future_dataframe(periods=len(df_prophet.query("ds >= @val_start")))
    forecast = model.predict(future)
    return model, forecast

with mlflow.start_run(run_name="baseline-prophet"):
    mlflow.log_param("model_type", "prophet")
    mae_list = []
    for sid, grp in df_val.groupby("station_id"):
        _, forecast = fit_prophet(df_full.query(f"station_id == '{sid}'"))
        val_pred = forecast.set_index("ds").loc[grp["date"]]["yhat"].values
        mae_list.append(mean_absolute_error(grp["net_flow"], val_pred))
    mlflow.log_metric("val_mae_mean_across_stations", np.mean(mae_list))
```

### 3.3 Overdispersion check before choosing Poisson vs Negative Binomial

Before fitting any Poisson model to crash counts, check whether the data is overdispersed ($\text{Var}(y) \gg \mathbb{E}[y]$). Overdispersion violates the Poisson assumption and inflates standard errors.

```python
mean_c = y_train.mean()
var_c  = y_train.var()
ratio  = var_c / mean_c
print(f"Dispersion ratio: {ratio:.2f}")
# ratio ≈ 1 → Poisson is appropriate
# ratio > 3 → use Negative Binomial (statsmodels NegativeBinomial family)
# ratio > 10 and many zeros → consider Zero-Inflated NegBin (pscl or statsmodels)
mlflow.log_metric("dispersion_ratio", ratio)
```

### 3.4 Hyperparameter search

One MLflow run per configuration. Keep the grid small (2–3 values per parameter) for this learning project.

```python
from itertools import product

param_grid = {
    "n_estimators":  [200, 500],
    "max_depth":     [4, 6, 8],
    "learning_rate": [0.05, 0.1],
}

for combo in [dict(zip(param_grid, v)) for v in product(*param_grid.values())]:
    run_name = f"xgb-n{combo['n_estimators']}-d{combo['max_depth']}-lr{combo['learning_rate']}"
    with mlflow.start_run(run_name=run_name):
        mlflow.log_params({**combo, "model_type": "xgboost", "feature_set": "v2"})
        model = xgb.XGBRegressor(objective="count:poisson", **combo)
        model.fit(X_train, y_train)
        y_pred = model.predict(X_val)
        mlflow.log_metrics({
            "val_poisson_deviance": poisson_deviance(y_val, y_pred),
            "val_mae": mean_absolute_error(y_val, y_pred),
        })
        mlflow.sklearn.log_model(model, artifact_path="model")
```

---

## Phase 4 — Offline Evaluation & Model Selection

**Purpose:** Rigorously evaluate candidate models on held-out data before committing to production. For time series data, evaluation methodology matters as much as model choice: random splits leak future information, naive metrics without a scale reference make RMSE uninterpretable across stations, and uncalibrated risk scores are dangerous for insurance pricing. This phase produces the evaluation results that justify the model selected in Phase 5.

**Tasks:**
- [ ] Implement walk-forward `TimeSeriesSplit` CV with a 7-day gap between folds
- [ ] Add a `leakage_guard` assertion to verify val period is always after train period
- [ ] Implement MASE to compare against the seasonal naive baseline
- [ ] Implement directional accuracy for net flow (does the sign of the prediction match?)
- [ ] Implement a decile calibration check for the risk model
- [ ] Run Durbin-Watson test on residuals to check for unexplained temporal autocorrelation
- [ ] Run spatial holdout: train on 80% of stations, evaluate on held-out 20%
- [ ] Log all evaluation metrics and plots as MLflow artifacts

### 4.1 Walk-forward cross-validation with leakage guard

```python
from sklearn.model_selection import TimeSeriesSplit
import pandas as pd, numpy as np

# Sort the panel by date — NOT by station. TimeSeriesSplit splits on row indices.
df_sorted = df.sort_values("date").reset_index(drop=True)
X = df_sorted[FEATURE_COLS]
y = df_sorted[TARGET_COL]
dates = df_sorted["date"]

tscv = TimeSeriesSplit(n_splits=5, gap=7)   # 7-day gap: prevents lag features from leaking across the boundary
cv_scores = []

for fold, (train_idx, val_idx) in enumerate(tscv.split(X)):
    # Leakage guard: val must start after train ends (plus gap)
    train_end = dates.iloc[train_idx].max()
    val_start = dates.iloc[val_idx].min()
    assert val_start > train_end + pd.Timedelta(days=7), \
        f"Fold {fold}: val_start={val_start} not after train_end={train_end} + 7d gap!"

    X_train, y_train = X.iloc[train_idx], y.iloc[train_idx]
    X_val,   y_val   = X.iloc[val_idx],   y.iloc[val_idx]

    model.fit(X_train, y_train)
    y_pred = model.predict(X_val)

    fold_metrics = {
        "fold": fold,
        "train_size": len(train_idx),
        "val_size":   len(val_idx),
        "val_mae":    mean_absolute_error(y_val, y_pred),
        "val_rmse":   mean_squared_error(y_val, y_pred, squared=False),
        "val_mase":   mase(y_val.values, y_pred, X_val["net_flow_lag7"].values),
    }
    cv_scores.append(fold_metrics)
    mlflow.log_metrics({k: v for k, v in fold_metrics.items() if k != "fold"}, step=fold)

summary = pd.DataFrame(cv_scores)
print(summary.describe())
mlflow.log_metric("cv_mae_mean",  summary["val_mae"].mean())
mlflow.log_metric("cv_mase_mean", summary["val_mase"].mean())
```

### 4.2 MASE — scale-free forecast accuracy

Mean Absolute Error is scale-dependent: an MAE of 5 trips means nothing without knowing the typical fluctuation size. MASE normalises against the seasonal naive forecast (same day last week), making results comparable across stations and time periods.

$$
\text{MASE} = \frac{\text{MAE}(\hat{y}, y)}{\text{MAE}(y_{t-7}, y)}
$$

- MASE < 1: model beats seasonal naive
- MASE = 1: model is equivalent to seasonal naive (no value added)
- MASE > 1: model is worse than seasonal naive — investigate before deploying

```python
def mase(y_true: np.ndarray, y_pred: np.ndarray, y_naive: np.ndarray) -> float:
    """MASE relative to a naive forecast. Lower is better; < 1 beats the naive."""
    mae_model = np.mean(np.abs(y_true - y_pred))
    mae_naive = np.mean(np.abs(y_true - y_naive))
    return mae_model / (mae_naive + 1e-8)
```

### 4.3 Directional accuracy for net flow

Operational interventions (dispatching rebalancing trucks) depend on the sign of net flow: is a station going to be over- or under-supplied? A model with poor RMSE but high directional accuracy may still be operationally useful.

```python
def directional_accuracy(y_true: np.ndarray, y_pred: np.ndarray) -> float:
    """Fraction of predictions where sign(pred) == sign(true).
    Threshold 0: balanced stations are excluded from the calculation."""
    mask = np.abs(y_true) > 0
    return float(np.mean(np.sign(y_true[mask]) == np.sign(y_pred[mask])))

mlflow.log_metric("val_directional_accuracy", directional_accuracy(y_val.values, y_pred))
```

### 4.4 Calibration check for the risk model

A risk model used for insurance pricing must be calibrated: predicted rates must match observed rates in aggregate across the score distribution. Miscalibration means the insurer is systematically under- or over-pricing risk.

```python
import matplotlib.pyplot as plt

def plot_calibration(y_true: np.ndarray, y_pred: np.ndarray, n_bins: int = 10) -> plt.Figure:
    """Decile calibration plot: mean predicted vs mean observed per score bucket.
    Points on the diagonal = perfect calibration."""
    df_cal = pd.DataFrame({"pred": y_pred, "obs": y_true})
    df_cal["bucket"] = pd.qcut(df_cal["pred"], q=n_bins, duplicates="drop")
    cal_df = df_cal.groupby("bucket").mean()

    fig, ax = plt.subplots(figsize=(6, 6))
    ax.plot(cal_df["pred"], cal_df["obs"], "o-", label="Model")
    lim = max(cal_df["pred"].max(), cal_df["obs"].max()) * 1.05
    ax.plot([0, lim], [0, lim], "--", color="gray", label="Perfect calibration")
    ax.set_xlabel("Mean predicted crash rate")
    ax.set_ylabel("Mean observed crash rate")
    ax.legend()
    ax.set_title("Calibration plot (deciles)")
    return fig

fig = plot_calibration(y_val.values, y_pred)
mlflow.log_figure(fig, "calibration.png")
```

### 4.5 Durbin-Watson residual autocorrelation test

If the model has captured all temporal structure, residuals should be approximately IID (no serial autocorrelation). The Durbin-Watson statistic tests for first-order autocorrelation in residuals: a value near 2.0 means no autocorrelation.

```python
from statsmodels.stats.stattools import durbin_watson

residuals = y_val.values - y_pred
dw = durbin_watson(residuals)
mlflow.log_metric("durbin_watson", dw)
print(f"Durbin-Watson: {dw:.3f}")
# dw ≈ 2.0 → no autocorrelation → model has captured temporal structure
# dw < 1.5 → positive autocorrelation → model is missing temporal patterns (add more lags)
# dw > 2.5 → negative autocorrelation → possible over-differencing
```

### 4.6 Spatial holdout

Walk-forward CV tests temporal generalisation. Spatial holdout tests whether the model generalises to stations it has never seen — which matters when new stations are added to the CitiBike network.

```python
from sklearn.model_selection import GroupShuffleSplit

# Hold out 20% of stations completely from training
splitter = GroupShuffleSplit(n_splits=1, test_size=0.2, random_state=42)
train_stations, val_stations = next(splitter.split(df, groups=df["station_id"]))

X_train_s = df.iloc[train_stations][FEATURE_COLS]
y_train_s = df.iloc[train_stations][TARGET_COL]
X_val_s   = df.iloc[val_stations][FEATURE_COLS]
y_val_s   = df.iloc[val_stations][TARGET_COL]

model.fit(X_train_s, y_train_s)
y_pred_s = model.predict(X_val_s)
mlflow.log_metric("spatial_holdout_mae",  mean_absolute_error(y_val_s, y_pred_s))
mlflow.log_metric("spatial_holdout_mase", mase(y_val_s.values, y_pred_s, X_val_s["net_flow_lag7"].values))
# Spatial MAE >> temporal MAE → model is over-fitted to seen stations → needs station cluster features
```

### 4.7 Selecting the production model

Choose the model with the best **mean CV MASE** (net flow) or **mean CV Poisson deviance** (risk) across all folds. Record the decision in MLflow:

```python
mlflow.set_tag("selection_reason",
    "Best mean CV MASE=0.82 (beats seasonal naive); spatial holdout MAE within 10% of temporal CV; well-calibrated (max decile deviation < 0.002)")
```

---

## Phase 5 — Model Packaging & Registry

**Purpose:** Register the selected model in the MLflow Model Registry so that it can be loaded by name and alias, not by a brittle run ID. The registry provides a single, auditable source of truth for which model is in production, when it was promoted, and what metrics justified the promotion. MLflow 2.x uses **aliases** (not stage labels) to tag model versions — always use `@production` / `@staging`.

**Tasks:**
- [ ] Register the winning run's model artifact in the registry under `citibike-risk-model` and `citibike-netflow-model`
- [ ] Set the `@staging` alias on the newly registered version
- [ ] Run the full evaluation suite against the staging version
- [ ] If it beats the current `@production` version, promote to `@production`
- [ ] Log a model card as a text artifact with training data range, key metrics, known limitations
- [ ] Verify the production model loads correctly via its alias URI

### 5.1 Register and promote

```python
from mlflow.tracking import MlflowClient
import mlflow

client = MlflowClient()

# Step 1: register the winning run (use the run_id from MLflow UI or from the best-CV run)
run_id = "abc123..."   # replace with actual run ID
mv = mlflow.register_model(
    model_uri=f"runs:/{run_id}/model",
    name="citibike-risk-model"
)
new_version = mv.version

# Step 2: tag as staging while evaluating
client.set_registered_model_alias(
    name="citibike-risk-model",
    alias="staging",
    version=new_version
)

# Step 3: compare against current production
def get_production_metric(model_name: str, metric: str) -> float | None:
    """Return the logged metric for the current @production version, or None if no production exists."""
    try:
        prod_version = client.get_model_version_by_alias(model_name, "production")
        run = client.get_run(prod_version.run_id)
        return run.data.metrics.get(metric)
    except Exception:
        return None

def promote_if_better(
    model_name: str,
    candidate_run_id: str,
    metric: str,
    lower_is_better: bool = True,
    min_improvement: float = 0.01,
) -> bool:
    """Promote candidate to @production only if it improves on the current production model.

    Args:
        model_name:       registered model name
        candidate_run_id: MLflow run ID of the candidate
        metric:           metric key to compare (e.g. 'cv_mae_mean')
        lower_is_better:  True for MAE/deviance; False for accuracy/F1
        min_improvement:  minimum relative improvement required to promote (default 1%)

    Returns:
        True if promoted, False otherwise.
    """
    prod_metric = get_production_metric(model_name, metric)
    candidate_run = client.get_run(candidate_run_id)
    cand_metric = candidate_run.data.metrics.get(metric)

    if cand_metric is None:
        raise ValueError(f"Metric '{metric}' not found in candidate run {candidate_run_id}")

    if prod_metric is None:
        # No production model yet — always promote
        print(f"No production model found. Promoting candidate (metric={cand_metric:.4f})")
    else:
        improvement = (prod_metric - cand_metric) / abs(prod_metric) if lower_is_better \
                 else (cand_metric - prod_metric) / abs(prod_metric)
        if improvement < min_improvement:
            print(f"Candidate did not improve sufficiently: prod={prod_metric:.4f}, cand={cand_metric:.4f}, rel_improvement={improvement:.3f}")
            return False
        print(f"Promoting: prod={prod_metric:.4f} → cand={cand_metric:.4f} (rel_improvement={improvement:.3%})")

    # Set @production alias on the newly registered version
    cand_version = client.get_model_version_by_alias(model_name, "staging").version
    client.set_registered_model_alias(name=model_name, alias="production", version=cand_version)
    client.set_model_version_tag(model_name, cand_version, "promoted_by", "promote_if_better")
    return True

promoted = promote_if_better("citibike-risk-model", run_id, metric="cv_mae_mean")
```

### 5.2 Model card

Log a structured model card as a text artifact alongside the model. This is the human-readable audit trail for the production deployment.

```python
model_card = f"""
Model card: citibike-risk-model v{new_version}
=============================================
Algorithm:       XGBoost, objective=count:poisson
Feature set:     v2 (Fourier + PACF lags + spatial lag + holiday)
Training data:   CitiBike + NYPD 2023-01-01 – 2024-12-31
Held-out period: 2025-01-01 – 2025-03-01
Dispersion ratio at training time: {dispersion_ratio:.2f}

Metrics (walk-forward CV, 5 folds, 7-day gap):
  mean CV Poisson deviance: {cv_deviance:.4f}  (baseline EB formula: {baseline_deviance:.4f})
  mean CV MAE:              {cv_mae:.4f}
  mean CV MASE:             {cv_mase:.4f}        (< 1 = beats seasonal naive)
  mean Durbin-Watson:       {cv_dw:.3f}          (should be ≈ 2.0)

Spatial holdout (20% of stations):
  holdout MAE:  {spatial_mae:.4f}
  holdout MASE: {spatial_mase:.4f}

Known limitations:
  - Stations with < 50 trips/day have high uncertainty (no EB smoothing)
  - Cold-start stations inherit cluster-average features from Phase 2 spatial fallback
  - Temporal scope: 2023-2024 only — post-2025 infrastructure changes not captured
"""

with open("/tmp/model_card.txt", "w") as f:
    f.write(model_card)

with mlflow.start_run(run_id=run_id):
    mlflow.log_artifact("/tmp/model_card.txt", artifact_path="model_card")
```

### 5.3 Load the production model

```python
import mlflow.pyfunc

# Load by alias (not by version number — aliases are stable pointers)
model = mlflow.pyfunc.load_model("models:/citibike-risk-model@production")
y_pred = model.predict(X_test)
```

---

## Phase 6 — Serving API (FastAPI + Pydantic)

**Purpose:** Expose both models as a production-grade REST API so that downstream systems (mobile app, rebalancing dashboard, insurance pricing engine) can request predictions in real time. FastAPI + Pydantic enforces strict request/response schemas, provides automatic OpenAPI documentation, and validates input before it ever reaches the model — preventing runtime errors and making the API self-documenting. The key engineering challenge for a time-series model is that lag features require historical context, which a stateless API request does not carry. The solution is a **feature cache** written daily by the Airflow ingest DAG and queried at inference time.

**Tasks:**
- [ ] Implement `build_risk_features_at_inference` that reads from the feature cache, not from raw data
- [ ] Implement a cold-start fallback for stations not yet in the cache
- [ ] Implement `lifespan` model loading so models load once at startup, not per request
- [ ] Add `/predict/risk`, `/predict/netflow`, `/model/info`, `/health` endpoints
- [ ] Run `uvicorn api.main:app --reload` and verify with `curl` or the Swagger UI at `/docs`

### 6.1 The inference-time feature problem

Lag features (e.g., `net_flow_lag1`, `net_flow_lag7`) require the previous 7+ days of data for a given station. A single API request (`station_id`, `datetime`) doesn't carry this context. The solution is a **daily feature cache** written by the Airflow ingest DAG (`update_feature_cache` task from Phase 1):

```
cache/station_features.parquet
  station_id | date | net_flow_lag1 | net_flow_lag7 | rolling_mean7 | rolling_std7 | ...
  6926.10    | 2025-06-15 | 3.2 | -1.1 | 2.4 | 1.8 | ...
```

At inference time, the API reads the most recent row for the requested station from the cache. This means predictions are always made with features computed from yesterday's data — appropriate for a daily operational model.

```python
# src/features/inference_cache.py
import pandas as pd
from functools import lru_cache
from pathlib import Path

CACHE_PATH = Path("cache/station_features.parquet")

@lru_cache(maxsize=1)
def _load_cache() -> pd.DataFrame:
    """Load the feature cache once and cache in memory. Reload on API restart."""
    return pd.read_parquet(CACHE_PATH).set_index("station_id")

def get_station_features(station_id: str) -> dict:
    """Look up precomputed lag features for a station from the daily cache.

    Raises:
        KeyError: if station_id is not found in the cache (triggers 404 in the API)
    """
    cache = _load_cache()
    if station_id not in cache.index:
        raise KeyError(f"Station '{station_id}' not in feature cache. Cold-start fallback applies.")
    return cache.loc[station_id].to_dict()

def get_cluster_fallback_features(station_id: str) -> dict:
    """Cold-start fallback: return the mean features of the nearest cluster.
    New stations have no history — inherit the cluster average as a prior."""
    cache = _load_cache()
    # In production: look up station_id → cluster mapping from a pre-built cluster table
    # For now, return the global mean as a conservative fallback
    return cache.mean(numeric_only=True).to_dict()
```

### 6.2 Build features at inference time

```python
# src/features/engineer.py (inference path)
from datetime import datetime
from .inference_cache import get_station_features, get_cluster_fallback_features
from .encode import cyclical_encode, fourier_features, add_holiday_flag
import numpy as np, pandas as pd

def build_risk_features_at_inference(
    station_id: str,
    dt: datetime,
    bike_type: str,
) -> pd.DataFrame:
    """Assemble a single-row feature DataFrame for real-time inference.

    Temporal features are computed from the request datetime.
    Lag features are looked up from the daily feature cache.
    Cold-start stations fall back to cluster-average lag features.
    """
    # Temporal features — from the request itself
    row = {
        "hour":              dt.hour,
        "hour_sin":          np.sin(2 * np.pi * dt.hour / 24),
        "hour_cos":          np.cos(2 * np.pi * dt.hour / 24),
        "dow":               dt.weekday(),
        "dow_sin":           np.sin(2 * np.pi * dt.weekday() / 7),
        "dow_cos":           np.cos(2 * np.pi * dt.weekday() / 7),
        "month":             dt.month,
        "is_weekend":        int(dt.weekday() >= 5),
        "is_holiday":        int(add_holiday_flag(pd.Timestamp(dt))),
        "bike_type_electric": int(bike_type == "electric_bike"),
    }

    # Fourier features for annual seasonality (day-of-year)
    doy = dt.timetuple().tm_yday
    for k in range(1, 4):
        row[f"doy_sin_{k}"] = np.sin(2 * np.pi * k * doy / 365.25)
        row[f"doy_cos_{k}"] = np.cos(2 * np.pi * k * doy / 365.25)

    # Lag features — from the daily feature cache
    try:
        lag_feats = get_station_features(station_id)
    except KeyError:
        lag_feats = get_cluster_fallback_features(station_id)

    row.update({k: v for k, v in lag_feats.items() if k not in row})

    return pd.DataFrame([row])
```

### 6.3 Request/response schemas

`api/schemas.py`:

```python
from pydantic import BaseModel, Field
from datetime import datetime
from enum import Enum

class BikeType(str, Enum):
    electric = "electric_bike"
    classic  = "classic_bike"

class RiskTier(str, Enum):
    low       = "Low"
    medium    = "Medium"
    high      = "High"
    very_high = "Very High"

class RiskRequest(BaseModel):
    station_id: str   = Field(..., examples=["6926.10"])
    datetime:   datetime = Field(..., examples=["2025-06-15T17:30:00"])
    bike_type:  BikeType = Field(..., examples=["electric_bike"])

class RiskResponse(BaseModel):
    station_id:    str
    risk_score:    float  = Field(..., ge=0, description="Predicted crash rate per 1 000 trips")
    risk_tier:     RiskTier
    model_version: str
    cold_start:    bool   = Field(False, description="True if lag features came from cluster fallback")

class NetFlowRequest(BaseModel):
    station_id: str
    date: str = Field(..., examples=["2025-06-15"])

class NetFlowResponse(BaseModel):
    station_id:      str
    net_flow:        float
    imbalance_class: str   = Field(..., description="importer | balanced | exporter")
    model_version:   str
    cold_start:      bool  = False
```

### 6.4 FastAPI app

`api/main.py`:

```python
from fastapi import FastAPI, HTTPException
from contextlib import asynccontextmanager
import mlflow.pyfunc
from .schemas import RiskRequest, RiskResponse, NetFlowRequest, NetFlowResponse
from src.features.engineer import build_risk_features_at_inference

RISK_ALIAS    = "models:/citibike-risk-model@production"
NETFLOW_ALIAS = "models:/citibike-netflow-model@production"
models: dict = {}

@asynccontextmanager
async def lifespan(app: FastAPI):
    # Load models once at startup — NOT per request
    models["risk"]    = mlflow.pyfunc.load_model(RISK_ALIAS)
    models["netflow"] = mlflow.pyfunc.load_model(NETFLOW_ALIAS)
    yield
    models.clear()

app = FastAPI(title="CitiBike MLOps API", version="1.0", lifespan=lifespan)

@app.get("/health")
def health():
    return {"status": "ok", "models_loaded": len(models) == 2}

@app.get("/model/info")
def model_info():
    return {"risk_model": RISK_ALIAS, "netflow_model": NETFLOW_ALIAS}

@app.post("/predict/risk", response_model=RiskResponse)
def predict_risk(req: RiskRequest):
    cold_start = False
    try:
        feats = build_risk_features_at_inference(req.station_id, req.datetime, req.bike_type)
    except KeyError:
        feats = build_risk_features_at_inference(req.station_id, req.datetime, req.bike_type)
        cold_start = True

    score = float(models["risk"].predict(feats)[0])
    return RiskResponse(
        station_id=req.station_id,
        risk_score=round(score, 4),
        risk_tier=_risk_tier(score),
        model_version=RISK_ALIAS,
        cold_start=cold_start,
    )

def _risk_tier(score: float) -> str:
    if score < 0.5:  return "Low"
    if score < 1.5:  return "Medium"
    if score < 3.0:  return "High"
    return "Very High"
```

### 6.5 Run and smoke-test

```bash
# Start the API (assumes MLflow tracking server and feature cache already present)
uvicorn api.main:app --reload --port 8000

# Smoke test
curl -X POST http://localhost:8000/predict/risk \
  -H "Content-Type: application/json" \
  -d '{"station_id": "6926.10", "datetime": "2025-06-15T17:30:00", "bike_type": "electric_bike"}'

# View Swagger docs
open http://localhost:8000/docs
```

---

## Phase 7 — Monitoring & Drift Detection (Evidently)

**Purpose:** A deployed ML model degrades silently unless you monitor it. For a CitiBike model, drift can occur seasonally (summer vs winter ridership patterns), after infrastructure events (new stations, network expansion), or due to data pipeline failures (missing upstream data producing silent NaN inputs). This phase distinguishes **data drift** (input features shift) from **target drift** (the relationship between features and the target changes), and adds a temporal-aware comparison strategy to avoid false alarms from known seasonal patterns.

**Tasks:**
- [ ] Log every prediction to `data/monitoring/predictions.parquet` with inputs, output, and timestamp
- [ ] Build a reference dataset from the training period (features + known targets)
- [ ] Run a weekly Evidently drift report comparing last 30 days against reference
- [ ] Add temporal drift awareness: compare against the same period from the prior year, not just a rolling baseline
- [ ] Set up alerting when drift is detected (Airflow will pick this up in Phase 8)

### 7.1 Log predictions

Every prediction is appended to a Parquet log. In production, use an append-optimised store (Delta Lake, Iceberg, or a simple Parquet directory with `append` mode).

```python
import pandas as pd
from pathlib import Path

PREDICTION_LOG = Path("data/monitoring/predictions.parquet")

def log_prediction(request: dict, response: dict, cold_start: bool = False):
    """Append one prediction to the monitoring log."""
    row = {
        **{f"req_{k}": v for k, v in request.items()},
        **{f"resp_{k}": v for k, v in response.items()},
        "cold_start": cold_start,
        "logged_at":  pd.Timestamp.now(tz="UTC"),
    }
    existing = pd.read_parquet(PREDICTION_LOG) if PREDICTION_LOG.exists() else pd.DataFrame()
    pd.concat([existing, pd.DataFrame([row])]).to_parquet(PREDICTION_LOG, index=False)
```

### 7.2 Evidently drift report

```python
from evidently.report import Report
from evidently.metric_preset import DataDriftPreset, TargetDriftPreset

def run_drift_report(
    reference_path: str,
    current_path: str,
    output_path: str,
) -> list[str]:
    """Run an Evidently drift report. Returns list of drifted feature names."""
    reference = pd.read_parquet(reference_path)
    current   = pd.read_parquet(current_path)

    report = Report(metrics=[DataDriftPreset(), TargetDriftPreset()])
    report.run(reference_data=reference, current_data=current)
    report.save_html(output_path)

    result = report.as_dict()
    drifted = [
        m["result"]["column_name"]
        for m in result["metrics"]
        if m.get("result", {}).get("drift_detected")
    ]
    return drifted
```

### 7.3 Temporal drift awareness

CitiBike data has strong annual seasonality — a model compared against its training data will always appear to drift in winter vs summer. To avoid false positives, compare the current 30-day window against the **same 30-day window from the prior year**, not against the full training dataset.

```python
def build_seasonal_reference(
    full_reference: pd.DataFrame,
    current_start: pd.Timestamp,
    window_days: int = 30,
) -> pd.DataFrame:
    """Slice the reference to the same seasonal window from the prior year.

    This prevents seasonal patterns from triggering false drift alerts.
    """
    prior_year_start = current_start - pd.DateOffset(years=1)
    prior_year_end   = prior_year_start + pd.Timedelta(days=window_days)
    return full_reference[
        (full_reference["date"] >= prior_year_start) &
        (full_reference["date"] < prior_year_end)
    ]
```

---

## Phase 8 — Retraining DAG (Airflow)

**Purpose:** Close the MLOps loop by automating retraining on a schedule. The DAG ensures the production model stays fresh as new CitiBike and NYPD data accumulates, and that it is only replaced when the new model is demonstrably better. Without this automation, retraining becomes a manual, error-prone process that gets deferred indefinitely. The `promote_if_better` task from Phase 5 is reused here — the DAG calls the same function, ensuring the promotion logic is DRY and tested.

**Tasks:**
- [ ] Create `dags/retrain_dag.py` with `@weekly` schedule
- [ ] Wire tasks in order: `check_drift` → `ingest` → `build_features` → `train` → `evaluate` → `promote_if_better` → `update_feature_cache`
- [ ] Pass `promote_if_better` the same metric key and threshold used in Phase 5
- [ ] Verify the DAG runs end-to-end in the Airflow UI (trigger manually first)

### 8.1 Full retraining DAG

`dags/retrain_dag.py`:

```python
from airflow.decorators import dag, task
from datetime import datetime

@dag(
    schedule="@weekly",
    start_date=datetime(2024, 1, 1),
    catchup=False,
    tags=["citibike", "retraining"],
    doc_md="""Weekly CitiBike model retraining pipeline.
    Checks for drift → ingests new data → retrains → promotes if better → refreshes feature cache.""",
)
def retrain_dag():

    @task
    def check_drift() -> bool:
        from monitoring.drift import run_drift_report, build_seasonal_reference
        import pandas as pd

        full_ref    = pd.read_parquet("data/monitoring/reference.parquet")
        current     = pd.read_parquet("data/monitoring/predictions.parquet")
        current_start = current["logged_at"].max() - pd.Timedelta(days=30)

        seasonal_ref = build_seasonal_reference(full_ref, current_start)
        drifted = run_drift_report(seasonal_ref, current, "data/monitoring/drift_report.html")
        return len(drifted) > 0

    @task
    def ingest_new_data():
        """Download and process new CitiBike + NYPD data into silver/gold tables."""
        import subprocess
        subprocess.run(["python", "-m", "src.data.ingest", "--month", "{{ ds[:7] }}"], check=True)

    @task
    def build_features():
        """Re-run feature engineering over updated silver/gold tables."""
        import subprocess
        subprocess.run(["python", "-m", "src.features.build"], check=True)

    @task
    def train_model(drift_detected: bool) -> str:
        """Run MLflow training script. Returns the new run_id."""
        import subprocess, json
        result = subprocess.run(
            ["python", "-m", "src.train", "--experiment", "citibike-risk", "--log-drift", str(drift_detected)],
            capture_output=True, text=True, check=True
        )
        # Training script prints the run_id as JSON to stdout: {"run_id": "abc123"}
        return json.loads(result.stdout.strip())["run_id"]

    @task
    def promote_model(run_id: str) -> bool:
        from src.registry import promote_if_better
        promoted = promote_if_better(
            model_name="citibike-risk-model",
            candidate_run_id=run_id,
            metric="cv_mae_mean",
            lower_is_better=True,
            min_improvement=0.01,
        )
        return promoted

    @task
    def refresh_feature_cache(promoted: bool):
        """Rebuild the inference feature cache regardless of whether we promoted.
        Even if the model did not change, the features must reflect the latest data."""
        import subprocess
        subprocess.run(["python", "-m", "src.features.build_cache"], check=True)

    # Wire tasks
    drift     = check_drift()
    _ingest   = ingest_new_data()
    _features = build_features()
    run_id    = train_model(drift)
    promoted  = promote_model(run_id)
    refresh_feature_cache(promoted)

    # Ordering: ingest → features → train (drift flag passed in); cache always refreshes last
    _ingest >> _features >> run_id

retrain_dag()
```

---

## Phase 9 — Testing

**Purpose:** Automated tests catch regressions before they reach production. For this project, two categories of bugs are especially dangerous: (1) **temporal leakage** — using future data to predict past targets, which inflates offline metrics but causes production models to degrade badly; and (2) **API contract violations** — response schemas that change silently, breaking downstream consumers. Tests for both are required before the project is considered complete.

**Tasks:**
- [ ] Write unit tests for all feature engineering functions (cyclical encoding, lag generation, Fourier features)
- [ ] Write a leakage test asserting lag features shift correctly per station
- [ ] Write a temporal ordering test for `TimeSeriesSplit`
- [ ] Write API integration tests using `httpx.AsyncClient` against the full FastAPI app
- [ ] Run `pytest tests/ -v` and confirm all pass

### 9.1 Feature engineering unit tests

`tests/test_features.py`:

```python
import pandas as pd
import numpy as np
import pytest
from src.features.engineer import cyclical_encode, add_lag_features, fourier_features

def test_cyclical_encode_range():
    """Cyclical encodings must stay in [-1, 1] for any input."""
    s = pd.Series(range(24))
    sin_enc, cos_enc = cyclical_encode(s, 24)
    assert sin_enc.between(-1, 1).all()
    assert cos_enc.between(-1, 1).all()

def test_cyclical_encode_periodicity():
    """Hour 0 and hour 24 must produce identical encodings."""
    s = pd.Series([0, 24])
    sin_enc, cos_enc = cyclical_encode(s, 24)
    assert np.isclose(sin_enc.iloc[0], sin_enc.iloc[1])

def test_lag_features_no_leakage():
    """Lag 1 at time t must equal y at time t-1 (within each station)."""
    df = pd.DataFrame({
        "station_id": ["A"] * 10,
        "date":       pd.date_range("2024-01-01", periods=10),
        "net_flow":   list(range(10)),
    })
    df = add_lag_features(df, "net_flow")
    # First row: no prior data → NaN
    assert pd.isna(df["net_flow_lag1"].iloc[0])
    # Row 1: lag1 should equal row 0's net_flow (0)
    assert df["net_flow_lag1"].iloc[1] == 0
    # Row 7: lag7 should equal row 0's net_flow (0)
    assert df["net_flow_lag7"].iloc[7] == 0

def test_lag_features_cross_station_isolation():
    """Lag features must NOT cross station boundaries.
    Station B's lag1 at its first row must be NaN, not Station A's last value."""
    df = pd.DataFrame({
        "station_id": ["A"] * 5 + ["B"] * 5,
        "date":       list(pd.date_range("2024-01-01", periods=5)) * 2,
        "net_flow":   [10, 11, 12, 13, 14, 20, 21, 22, 23, 24],
    })
    df = add_lag_features(df, "net_flow")
    # First row of Station B must have NaN lag1, not Station A's last value (14)
    b_first = df[df["station_id"] == "B"].iloc[0]
    assert pd.isna(b_first["net_flow_lag1"]), \
        f"Cross-station leakage: B's lag1={b_first['net_flow_lag1']}, expected NaN"

def test_fourier_features_shape():
    """Fourier features must produce 2*n_terms columns."""
    dates = pd.date_range("2024-01-01", periods=365)
    result = fourier_features(dates, period=365.25, n_terms=3)
    assert result.shape[1] == 6   # 3 sin + 3 cos

def test_timeseries_split_temporal_ordering():
    """Walk-forward CV: validation period must always be strictly after training period."""
    from sklearn.model_selection import TimeSeriesSplit
    dates = pd.date_range("2024-01-01", periods=100)

    tscv = TimeSeriesSplit(n_splits=5, gap=7)
    for train_idx, val_idx in tscv.split(dates):
        train_end = dates[train_idx].max()
        val_start = dates[val_idx].min()
        assert val_start > train_end, \
            f"Temporal ordering violated: val_start={val_start}, train_end={train_end}"
```

### 9.2 API integration tests

`tests/test_api.py`:

```python
import pytest
from httpx import AsyncClient, ASGITransport
from api.main import app

@pytest.mark.asyncio
async def test_health():
    async with AsyncClient(transport=ASGITransport(app=app), base_url="http://test") as client:
        r = await client.get("/health")
    assert r.status_code == 200
    assert r.json()["status"] == "ok"

@pytest.mark.asyncio
async def test_predict_risk_schema():
    """Response must match RiskResponse schema with required fields and types."""
    async with AsyncClient(transport=ASGITransport(app=app), base_url="http://test") as client:
        r = await client.post("/predict/risk", json={
            "station_id": "6926.10",
            "datetime":   "2025-06-15T17:30:00",
            "bike_type":  "electric_bike",
        })
    assert r.status_code == 200
    body = r.json()
    assert "risk_score"    in body
    assert "risk_tier"     in body
    assert "model_version" in body
    assert body["risk_score"] >= 0
    assert body["risk_tier"] in ["Low", "Medium", "High", "Very High"]

@pytest.mark.asyncio
async def test_predict_risk_invalid_station():
    """Unknown station_id should still return a 200 with cold_start=True (fallback)."""
    async with AsyncClient(transport=ASGITransport(app=app), base_url="http://test") as client:
        r = await client.post("/predict/risk", json={
            "station_id": "DOES_NOT_EXIST_9999",
            "datetime":   "2025-06-15T08:00:00",
            "bike_type":  "classic_bike",
        })
    assert r.status_code == 200
    assert r.json()["cold_start"] is True

@pytest.mark.asyncio
async def test_predict_risk_invalid_bike_type():
    """Invalid bike_type enum value must return 422 Unprocessable Entity."""
    async with AsyncClient(transport=ASGITransport(app=app), base_url="http://test") as client:
        r = await client.post("/predict/risk", json={
            "station_id": "6926.10",
            "datetime":   "2025-06-15T08:00:00",
            "bike_type":  "unicycle",   # not in BikeType enum
        })
    assert r.status_code == 422
```

### 9.3 Run the test suite

```bash
# Install test dependencies
uv add --dev pytest pytest-asyncio httpx

# Run all tests
uv run pytest tests/ -v

# Run just the feature tests
uv run pytest tests/test_features.py -v
```

---

## Phase 10 — Extensions

**Purpose:** These extensions go beyond the core ML engineering lifecycle and explore advanced forecasting and reliability techniques. They are not required for a working production system but deepen understanding of uncertainty quantification, hierarchical modelling, and multi-step forecasting — all directly relevant to the CitiBike problem.

**Tasks (pick any):**
- [ ] Quantile regression prediction intervals (LightGBM)
- [ ] Conformal prediction wrapper for distribution-free coverage guarantees
- [ ] Hierarchical forecasting (borough → neighbourhood → station)
- [ ] Multi-step forecasting: DIRECT vs recursive strategy comparison
- [ ] Weather as exogenous variable (Open-Meteo API)

### 10.1 Prediction intervals (quantile regression)

Point predictions do not convey uncertainty. A station with net_flow prediction = 0 ± 20 needs very different operational treatment than net_flow = 0 ± 2. LightGBM's `objective="quantile"` trains separate models for each quantile:

```python
import lightgbm as lgb

def train_quantile_model(
    X_train, y_train,
    alpha: float = 0.1,
) -> lgb.Booster:
    """Train a quantile regression model at level alpha.
    Call once with alpha=0.1 (lower bound) and once with alpha=0.9 (upper bound)."""
    params = {
        "objective":        "quantile",
        "alpha":            alpha,          # quantile level
        "n_estimators":     500,
        "max_depth":        6,
        "learning_rate":    0.05,
        "verbose":         -1,
    }
    dtrain = lgb.Dataset(X_train, label=y_train)
    return lgb.train(params, dtrain)

model_lo = train_quantile_model(X_train, y_train, alpha=0.1)
model_hi = train_quantile_model(X_train, y_train, alpha=0.9)

y_lo = model_lo.predict(X_val)
y_hi = model_hi.predict(X_val)

# Coverage: what fraction of true values fall within [lo, hi]?
coverage = np.mean((y_val >= y_lo) & (y_val <= y_hi))
print(f"80% PI coverage: {coverage:.3f}")   # should be ≈ 0.80 for calibrated model
```

### 10.2 Conformal prediction (distribution-free coverage)

Quantile regression relies on the model being well-calibrated. Conformal prediction provides a **coverage guarantee** without that assumption, using a held-out calibration set.

```python
import numpy as np

def conformal_prediction_interval(
    point_model,
    X_calib: np.ndarray,
    y_calib: np.ndarray,
    X_test:  np.ndarray,
    alpha:   float = 0.1,
) -> tuple[np.ndarray, np.ndarray]:
    """Split-conformal prediction intervals at (1-alpha) coverage level.

    Guarantees that P(y_test ∈ [lo, hi]) ≥ 1 - alpha for exchangeable data.
    """
    # Calibration nonconformity scores (absolute residuals)
    y_pred_calib = point_model.predict(X_calib)
    scores = np.abs(y_calib - y_pred_calib)

    # (1-alpha) quantile of the calibration scores (with finite-sample correction)
    n = len(scores)
    q = np.quantile(scores, np.ceil((n + 1) * (1 - alpha)) / n, interpolation="higher")

    # Apply to test predictions
    y_pred_test = point_model.predict(X_test)
    return y_pred_test - q, y_pred_test + q

lo, hi = conformal_prediction_interval(
    point_model=best_model,
    X_calib=X_calib, y_calib=y_calib.values,
    X_test=X_test,
    alpha=0.1,
)
coverage = np.mean((y_test.values >= lo) & (y_test.values <= hi))
print(f"90% conformal PI coverage: {coverage:.3f}")  # guaranteed ≥ 0.90
```

### 10.3 Hierarchical forecasting

Individual station-level models are noisy. Borough-level predictions are smoother and can be disaggregated to station level. This is useful for rebalancing logistics (fleet managers think at the borough level) and for regularising sparse stations.

```python
# Bottom-up reconciliation:
# 1. Fit one model per level (station, neighbourhood, borough)
# 2. Sum station-level predictions to get the bottom-up borough forecast
# 3. If station sum ≠ borough model prediction, scale station forecasts proportionally

def reconcile_bottom_up(
    station_preds: pd.DataFrame,  # columns: station_id, neighbourhood, borough, prediction
) -> pd.DataFrame:
    """Reconcile station-level predictions to be consistent with borough sums.
    Uses bottom-up method: borough = sum of stations (no scaling)."""
    station_preds["borough_sum"] = station_preds.groupby("borough")["prediction"].transform("sum")
    return station_preds

# For top-down or MinT reconciliation, use the `hierarchicalforecast` library:
# pip install hierarchicalforecast
```

### 10.4 Multi-step forecasting (DIRECT vs recursive)

For rebalancing logistics, a 7-day-ahead forecast is more useful than a 1-day-ahead forecast. Two strategies:

| Strategy | How | Pros | Cons |
|---|---|---|---|
| **Recursive** | Predict h=1, feed prediction as lag, repeat | One model | Error accumulates over horizon |
| **DIRECT** | Train separate model for each horizon h=1…7 | No error accumulation | 7× training cost; no shared structure |

```python
# DIRECT multi-step: train one model per horizon
from sklearn.multioutput import MultiOutputRegressor

# Build a target matrix: [net_flow_h1, net_flow_h2, ..., net_flow_h7]
Y_multi = pd.concat(
    [df.groupby("station_id")["net_flow"].shift(-h).rename(f"net_flow_h{h}") for h in range(1, 8)],
    axis=1
)
Y_multi = Y_multi.dropna()
X_multi = df.loc[Y_multi.index, FEATURE_COLS]

multi_model = MultiOutputRegressor(lgb.LGBMRegressor(n_estimators=200))
multi_model.fit(X_multi, Y_multi)
y_pred_multi = multi_model.predict(X_test_multi)  # shape: (n_samples, 7)
```

### 10.5 Weather as exogenous variable (Open-Meteo)

Weather explains a large fraction of daily ridership variance — rain and cold dramatically reduce bike trips. The Open-Meteo API is free and provides historical + forecast weather data.

```python
import requests
import pandas as pd

def fetch_weather(lat: float, lon: float, start: str, end: str) -> pd.DataFrame:
    """Fetch daily temperature and precipitation from Open-Meteo (free, no API key)."""
    url = "https://archive-api.open-meteo.com/v1/archive"
    params = {
        "latitude":    lat,
        "longitude":   lon,
        "start_date":  start,       # YYYY-MM-DD
        "end_date":    end,
        "daily":       "temperature_2m_mean,precipitation_sum",
        "timezone":    "America/New_York",
    }
    r = requests.get(url, params=params)
    r.raise_for_status()
    data = r.json()["daily"]
    return pd.DataFrame({
        "date":         pd.to_datetime(data["time"]),
        "temp_mean_c":  data["temperature_2m_mean"],
        "precip_mm":    data["precipitation_sum"],
    })

# Merge with the gold feature table
weather = fetch_weather(lat=40.7128, lon=-74.0060, start="2023-01-01", end="2024-12-31")
df_gold = df_gold.merge(weather, on="date", how="left")

# As a feature: rain indicator and temperature bucket
df_gold["is_rainy"]     = (df_gold["precip_mm"] > 1.0).astype(int)
df_gold["temp_bucket"]  = pd.cut(df_gold["temp_mean_c"], bins=[-20, 0, 10, 20, 40],
                                  labels=["freezing", "cold", "mild", "warm"])
```

---

## Summary Checklist

Use this as a progress tracker as you work through the project.

| Phase | Task | Done |
|---|---|---|
| 0 | Write success metrics and model selection criteria before coding | ☐ |
| 0 | Time series data audit: ACF/PACF, ADF test, STL decomposition | ☐ |
| 0 | Zero-inflation check on crash counts (zero fraction > 90%?) | ☐ |
| 0 | Create repo + uv environment | ☐ |
| 0 | Docker Compose stack running locally (Airflow, MLflow, MinIO) | ☐ |
| 1 | `ingest_dag` downloads bronze Parquet (CitiBike + NYPD) | ☐ |
| 1 | Silver cleaning pipeline (DuckDB SQL, bad trips filtered) | ☐ |
| 1 | Gold feature aggregation tables written | ☐ |
| 1 | `update_feature_cache` task writes inference-time lag features | ☐ |
| 2 | Cyclical (hour, dow) and Fourier (annual) feature functions | ☐ |
| 2 | PACF-guided lag selection (1, 7, 14, 28 days) | ☐ |
| 2 | Spatial lag feature via BallTree (neighbour-average net flow) | ☐ |
| 2 | `RiskFeatures` and `NetFlowFeatures` Pydantic v2 schemas | ☐ |
| 3 | MLflow tracking server running | ☐ |
| 3 | Overdispersion ratio logged before choosing Poisson vs NegBin | ☐ |
| 3 | All baselines logged (EB formula, NegBin GLM, persistence, seasonal naive, Prophet) | ☐ |
| 3 | All model variants logged with consistent params/metrics | ☐ |
| 3 | Grid search over key hyperparameters, one run per combination | ☐ |
| 4 | Walk-forward `TimeSeriesSplit(n_splits=5, gap=7)` implemented | ☐ |
| 4 | Leakage guard assertion in each CV fold | ☐ |
| 4 | MASE metric implemented and logged | ☐ |
| 4 | Directional accuracy logged for net flow model | ☐ |
| 4 | Durbin-Watson test on validation residuals logged | ☐ |
| 4 | Decile calibration plot for risk model logged to MLflow | ☐ |
| 4 | Spatial holdout evaluation (20% of stations held out) | ☐ |
| 5 | Best model registered in MLflow model registry | ☐ |
| 5 | `promote_if_better` function implemented with real MLflow comparison | ☐ |
| 5 | Model promoted to `@production` alias | ☐ |
| 5 | Model card artifact logged (data range, metrics, limitations) | ☐ |
| 6 | Daily feature cache built by Airflow ingest DAG | ☐ |
| 6 | `build_risk_features_at_inference` reads from cache, not raw data | ☐ |
| 6 | Cold-start fallback for new stations (cluster-average features) | ☐ |
| 6 | Pydantic v2 request/response schemas with `cold_start` flag | ☐ |
| 6 | FastAPI `lifespan` model loading (once at startup) | ☐ |
| 6 | `/predict/risk` and `/predict/netflow` endpoints working | ☐ |
| 6 | Swagger UI (`/docs`) verified | ☐ |
| 7 | Prediction logging to Parquet with inputs + outputs + timestamp | ☐ |
| 7 | Evidently drift report runs and returns drifted feature list | ☐ |
| 7 | Temporal-aware reference: compare against same period last year | ☐ |
| 8 | `retrain_dag` orchestrates full pipeline (drift → ingest → train → promote → cache) | ☐ |
| 8 | `promote_if_better` in DAG uses same function as Phase 5 (DRY) | ☐ |
| 8 | Feature cache refreshed at end of every DAG run | ☐ |
| 9 | Unit tests: cyclical encoding, lag generation, Fourier features | ☐ |
| 9 | Cross-station leakage test (station B lag1 must not be station A's last value) | ☐ |
| 9 | `TimeSeriesSplit` temporal ordering assertion test | ☐ |
| 9 | API integration tests: health, valid request, cold-start, invalid input | ☐ |
| 9 | All tests pass: `uv run pytest tests/ -v` | ☐ |
| 10 | (Optional) Quantile regression PIs: 80% coverage verified | ☐ |
| 10 | (Optional) Conformal prediction intervals: ≥90% coverage guaranteed | ☐ |
| 10 | (Optional) Hierarchical forecasting: borough → station disaggregation | ☐ |
| 10 | (Optional) DIRECT multi-step forecasting (7-day horizon) | ☐ |
| 10 | (Optional) Weather features via Open-Meteo API integrated | ☐ |

---

## References

## Links

- [[09_projects/_active/citibike-mlops/overview|CitiBike MLOps Overview]]
- [[05_ml_engineering/01_principles_and_lifecycle/ml_lifecycle|ML Lifecycle]]
- [[05_ml_engineering/02_data_engineering/data_pipeline_patterns|Data Pipeline Patterns]]
- [[05_ml_engineering/05_model_development/experiment_tracking|Experiment Tracking]]
- [[05_ml_engineering/06_deployment_and_serving/serving_patterns|Serving Patterns]]
- [[05_ml_engineering/07_monitoring_and_observability/drift_detection|Drift Detection]]
- [[05_ml_engineering/08_continual_learning/retraining_strategies|Retraining Strategies]]
- [[09_projects/_completed/citibike-nyc-axa/overview|CitiBike NYC AXA (Source Analysis)]]
