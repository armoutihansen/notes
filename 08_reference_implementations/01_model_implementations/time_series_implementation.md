---
layer: 08_reference_implementations
type: application
domain: forecasting
stakeholders: []
regulatory: []
status: growing
tags: [algorithm, time-series, forecasting]
created: 2026-03-06
---

# Time Series Models — Implementation

## Goal

Implement ARIMA, ETS/Holt-Winters, and neural time series models with walk-forward cross-validation.

## Conceptual Counterpart

- [[03_modeling/05_time_series/index|Time Series Models]] — stationarity, ARIMA identification, seasonal decomposition, state space models
- [[07_applications/01_prediction_and_forecasting/demand_forecasting|Demand Forecasting]] — production use case combining SARIMAX and LightGBM
- [[07_applications/08_domain_verticals/04_ecommerce/inventory_optimization|Inventory Optimization]] — downstream decision system consuming forecasts

## Purpose

Practical implementation of ARIMA, ETS, and neural time series models.

### Examples

- Stationarity testing and differencing
- Auto-ARIMA with pmdarima
- Holt-Winters ETS with statsmodels
- Walk-forward cross-validation

## Architecture

```
Raw time series → Stationarity check → Differencing if needed
                → Model fitting (ARIMA / ETS / LGBM-based)
                → Walk-forward evaluation
                → Rolling forecast
```

## Implementation

### Setup

```bash
pip install statsmodels pmdarima scikit-learn
```

### Stationarity Testing

```python
from statsmodels.tsa.stattools import adfuller
import pandas as pd

def test_stationarity(series: pd.Series) -> bool:
    result = adfuller(series.dropna())
    p_value = result[1]
    print(f"ADF Statistic: {result[0]:.4f}, p-value: {p_value:.4f}")
    return p_value < 0.05   # True → stationary

# Differencing if non-stationary
diff1 = series.diff().dropna()
diff2 = diff1.diff().dropna()
```

### Auto-ARIMA (pmdarima)

```python
import pmdarima as pm
from statsmodels.tsa.statespace.sarimax import SARIMAX

# Auto ARIMA with stepwise search
auto_model = pm.auto_arima(
    train,
    seasonal=True,
    m=12,               # seasonal period (12 = monthly)
    stepwise=True,
    information_criterion="aic",
    suppress_warnings=True
)
print(auto_model.summary())

# Forecast
n_forecast = 12
forecast, conf_int = auto_model.predict(n_periods=n_forecast, return_conf_int=True)
```

### SARIMAX (statsmodels)

```python
from statsmodels.tsa.statespace.sarimax import SARIMAX

model = SARIMAX(
    endog=train,
    order=(1, 1, 1),          # (p, d, q)
    seasonal_order=(1, 1, 1, 12),  # (P, D, Q, s)
    trend="c"
)
res = model.fit(disp=False)
print(res.summary())

# Forecast
pred = res.get_forecast(steps=12)
forecast = pred.predicted_mean
ci      = pred.conf_int()
```

### Holt-Winters (ETS)

```python
from statsmodels.tsa.holtwinters import ExponentialSmoothing

ets = ExponentialSmoothing(
    train,
    trend="add",             # "add", "mul", None
    seasonal="add",          # "add", "mul", None
    seasonal_periods=12,
    damped_trend=True
)
ets_fit = ets.fit(optimized=True)
forecast = ets_fit.forecast(12)
```

### Walk-Forward Validation

```python
import numpy as np
from sklearn.metrics import mean_absolute_error

def walk_forward_cv(series, n_test=12, n_folds=5):
    """Walk-forward cross-validation for time series."""
    maes = []
    series_array = np.array(series)
    fold_size = n_test
    for i in range(n_folds):
        test_end   = len(series_array) - i * fold_size
        test_start = test_end - fold_size
        train_data = series_array[:test_start]
        test_data  = series_array[test_start:test_end]
        if len(train_data) < 24:
            continue
        try:
            m = pm.auto_arima(train_data, seasonal=True, m=12, stepwise=True,
                              suppress_warnings=True)
            preds, _ = m.predict(n_periods=n_test, return_conf_int=True)
            maes.append(mean_absolute_error(test_data, preds))
        except Exception as e:
            print(f"Fold {i} failed: {e}")
    return np.mean(maes), np.std(maes)

mae_mean, mae_std = walk_forward_cv(series)
print(f"Walk-forward MAE: {mae_mean:.3f} ± {mae_std:.3f}")
```

### LGBM as a Time Series Regressor (feature-based)

```python
import lightgbm as lgb
import pandas as pd

def create_lag_features(series: pd.Series, lags: list) -> pd.DataFrame:
    df = pd.DataFrame({"y": series})
    for lag in lags:
        df[f"lag_{lag}"] = df["y"].shift(lag)
    df["rolling_mean_4"] = df["y"].shift(1).rolling(4).mean()
    df["rolling_std_4"]  = df["y"].shift(1).rolling(4).std()
    return df.dropna()

df = create_lag_features(series, lags=[1, 2, 3, 6, 12])
X = df.drop("y", axis=1)
y = df["y"]
# Standard train/test split by time index
```

## Trade-offs

- ARIMA: statistically rigorous, interpretable parameters; works well for short, stationary series with up to ~100 obs.
- ETS: better for strongly seasonal series; handles multiplicative seasonality.
- LGBM / feature engineering: highest accuracy for large datasets; no uncertainty quantification without extra effort.
- Always evaluate with walk-forward (not random) CV to avoid lookahead bias.

## Links

- [[03_modeling/05_time_series/01_classical_forecasting/time_series_models|Time Series Models (theory)]]
- [[01_foundations/03_probability_and_statistics/hypothesis_testing|ADF test / hypothesis testing]]
- [[03_modeling/07_evaluation_and_model_selection/evaluation_and_validation|Walk-forward evaluation]]
- [[08_reference_implementations/01_model_implementations/tree_ensembles_implementation|Tree Ensembles (LightGBM)]]
