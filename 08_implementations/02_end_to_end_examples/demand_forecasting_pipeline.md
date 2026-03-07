---
layer: 08_implementations
type: application
domain: forecasting
stakeholders: [data-scientist, ml-engineer]
regulatory: []
status: growing
tags: [workflow, time-series, forecasting]
created: 2026-03-06
---

# Demand Forecasting Pipeline

## Purpose

A complete demand forecasting system for business time series (e.g., insurance renewals, product sales, energy demand) combining statistical and ML approaches. Integrates EDA, preprocessing, modelling, walk-forward evaluation, and batch inference.

### Examples

- Monthly sales forecast with SARIMA + LightGBM ensemble
- Walk-forward evaluation with MAE/MAPE comparison
- Feature-engineered lag-based approach for scalable multi-series forecasting

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                Raw Data Store                       │
│  (time-indexed tabular: sales, dates, covariates)   │
└───────────────────┬─────────────────────────────────┘
                    │
           ┌────────▼────────┐
           │  Data Pipeline  │  EDA + stationarity checks
           │  & Preprocessing│  Differencing, lag features
           └────────┬────────┘
                    │
         ┌──────────┴──────────┐
         │                     │
┌────────▼────────┐   ┌────────▼────────┐
│  Statistical    │   │  ML Approach    │
│  SARIMA / ETS   │   │  LightGBM +     │
│                 │   │  Lag features   │
└────────┬────────┘   └────────┬────────┘
         │                     │
         └──────────┬──────────┘
                    │
           ┌────────▼────────┐
           │ Ensemble / Best │  Select by walk-forward CV
           │    Model        │
           └────────┬────────┘
                    │
           ┌────────▼────────┐
           │ Batch Inference │  Monthly forecast generation
           │ + CI generation │
           └─────────────────┘
```

## Step-by-Step Implementation

### 1. Load and Explore Data

```python
import pandas as pd
import matplotlib.pyplot as plt
from statsmodels.graphics.tsaplots import plot_acf, plot_pacf

df = pd.read_csv("sales.csv", parse_dates=["date"], index_col="date")
series = df["sales"].resample("MS").sum()   # monthly start frequency

# Basic EDA
print(series.describe())
series.plot(title="Monthly Sales")
plt.show()

# Decomposition
from statsmodels.tsa.seasonal import seasonal_decompose
decomp = seasonal_decompose(series, model="additive", period=12)
decomp.plot()
```

### 2. Stationarity and Differencing

```python
from statsmodels.tsa.stattools import adfuller

def check_and_difference(s, max_diffs=2):
    d = 0
    while d <= max_diffs:
        result = adfuller(s.dropna())
        print(f"d={d}: ADF p={result[1]:.4f}")
        if result[1] < 0.05:
            return s, d
        s = s.diff()
        d += 1
    return s, d

series_stat, d_order = check_and_difference(series)
```

### 3. SARIMA Forecasting

```python
import pmdarima as pm

# Auto-select (p, d, q)(P, D, Q)[12]
sarima = pm.auto_arima(
    train, seasonal=True, m=12, stepwise=True,
    information_criterion="bic", suppress_warnings=True
)
forecast_sarima, ci_sarima = sarima.predict(n_periods=12, return_conf_int=True)
```

### 4. LightGBM with Lag Features

```python
import numpy as np
import lightgbm as lgb
from sklearn.metrics import mean_absolute_error

def make_features(series: pd.Series, lags=(1, 2, 3, 6, 12)):
    df = pd.DataFrame({"y": series})
    for lag in lags:
        df[f"lag_{lag}"] = df["y"].shift(lag)
    df["roll_mean_3"] = df["y"].shift(1).rolling(3).mean()
    df["roll_std_3"]  = df["y"].shift(1).rolling(3).std()
    df["month"]       = df.index.month
    df["year"]        = df.index.year
    return df.dropna()

feat_df = make_features(series)
X = feat_df.drop("y", axis=1)
y = feat_df["y"]

# Time-based split (no shuffling)
cutoff = int(len(X) * 0.8)
X_tr, X_te = X.iloc[:cutoff], X.iloc[cutoff:]
y_tr, y_te = y.iloc[:cutoff], y.iloc[cutoff:]

lgbm = lgb.LGBMRegressor(n_estimators=500, learning_rate=0.05, num_leaves=31)
lgbm.fit(X_tr, y_tr, eval_set=[(X_te, y_te)],
         callbacks=[lgb.early_stopping(30), lgb.log_evaluation(0)])
forecast_lgbm = lgbm.predict(X_te)
print(f"LGBM MAE: {mean_absolute_error(y_te, forecast_lgbm):.2f}")
```

### 5. Walk-Forward Cross-Validation

```python
def walk_forward_evaluate(series, model_fn, n_test=12, n_folds=5):
    maes, mapes = [], []
    arr = np.array(series)
    for i in range(n_folds):
        te_end   = len(arr) - i * n_test
        te_start = te_end - n_test
        tr_data  = arr[:te_start]
        te_data  = arr[te_start:te_end]
        if len(tr_data) < 36:
            continue
        preds = model_fn(tr_data, n_test)
        maes.append(mean_absolute_error(te_data, preds))
        mapes.append(np.mean(np.abs((te_data - preds) / te_data)) * 100)
    return {"MAE": np.mean(maes), "MAPE": np.mean(mapes)}

# Compare models
sarima_fn = lambda tr, n: pm.auto_arima(tr, m=12, stepwise=True,
                                         suppress_warnings=True).predict(n)
print("SARIMA:", walk_forward_evaluate(series, sarima_fn))
```

### 6. Ensemble and Deploy

```python
# Simple average ensemble
forecast_ensemble = 0.5 * forecast_sarima + 0.5 * forecast_lgbm

# Persist model
import joblib
joblib.dump(lgbm, "lgbm_sales_forecast.pkl")
sarima.update(new_train_obs)   # pmdarima supports incremental updates
```

## Trade-offs

- SARIMA: interpretable, solid for seasonal series with ~100+ obs; cannot incorporate external regressors without SARIMAX extension.
- LGBM features: scales to many series, supports covariates; requires adequate history for lag features.
- Ensemble: combines strengths; evaluate if the average outperforms individual models on your walk-forward CV before deploying.

## Links

- [[03_modeling/05_time_series/01_classical_forecasting/time_series_models|Time Series Models (theory)]]
- [[02_data_science/03_exploratory_data_analysis/exploratory_data_analysis|EDA]]
- [[02_data_science/05_experimentation_and_validation/data_preprocessing|Data Preprocessing]]
- [[03_modeling/07_evaluation_and_model_selection/evaluation_and_validation|Walk-forward evaluation]]
- [[03_modeling/05_time_series/01_classical_forecasting/time_series_implementation|Time Series Implementation]]
- [[03_modeling/01_supervised_learning/02_tree_based_models/tree_ensembles_implementation|Tree Ensembles (LightGBM)]]
