---
layer: 03_modeling
type: concept
status: seed
tags: [algorithm, forecasting, tabular]
created: 2026-05-31
---

# ML and DL Forecasting

## Core Idea

Machine learning methods treat time series forecasting as supervised learning: construct features from historical observations (lags, rolling statistics, calendar features) and train a standard model — gradient boosting, random forest, or a neural network — to predict the next value(s). Deep learning approaches learn the temporal structure directly from raw sequences.

This contrasts with classical statistical methods (ARIMA, ETS) which model the time series process directly.

## Mathematical Formulation

### Tabular Forecasting with Gradient Boosting

Transform a univariate time series $y_1, y_2, \ldots, y_T$ into a supervised dataset by windowing:

$$(\mathbf{x}_t, y_t), \quad \mathbf{x}_t = [y_{t-1}, y_{t-2}, \ldots, y_{t-p}, \text{calendar features}, \text{static covariates}]$$

This is the **lag feature** approach. Gradient boosting (XGBoost, LightGBM) excels here due to its robustness to scale, handling of missing lags, and built-in non-linearity.

```python
import pandas as pd
import numpy as np
from lightgbm import LGBMRegressor

def make_lag_features(y: pd.Series, lags: list[int], 
                      rolling_windows: list[int]) -> pd.DataFrame:
    df = pd.DataFrame({'y': y})
    for lag in lags:
        df[f'lag_{lag}'] = y.shift(lag)
    for w in rolling_windows:
        df[f'roll_mean_{w}'] = y.shift(1).rolling(w).mean()
        df[f'roll_std_{w}']  = y.shift(1).rolling(w).std()
    df['month']      = y.index.month
    df['day_of_week'] = y.index.dayofweek
    return df.dropna()

df_feat = make_lag_features(train_series, lags=[1,2,3,7,14], rolling_windows=[7, 28])
X_train, y_train = df_feat.drop('y', axis=1), df_feat['y']

model = LGBMRegressor(n_estimators=500, learning_rate=0.05, num_leaves=31)
model.fit(X_train, y_train)
```

**Multi-step forecasting strategies:**
- **Recursive**: predict $\hat{y}_{t+1}$, append to history, repeat.
- **Direct**: train a separate model for each horizon $h \in \{1, \ldots, H\}$.
- **MIMO** (multi-output): a single model predicts all $H$ horizons simultaneously.

### LSTM-Based Forecasting

LSTMs process the entire history as a sequence, learning what to remember vs forget via gating:

```python
import torch
import torch.nn as nn

class LSTMForecaster(nn.Module):
    def __init__(self, input_size, hidden_size, num_layers, forecast_horizon):
        super().__init__()
        self.lstm = nn.LSTM(input_size, hidden_size, num_layers, batch_first=True)
        self.linear = nn.Linear(hidden_size, forecast_horizon)

    def forward(self, x):
        # x: (batch, seq_len, input_size)
        out, _ = self.lstm(x)
        return self.linear(out[:, -1, :])  # last hidden state → forecast
```

### N-BEATS (Neural Basis Expansion Analysis)

N-BEATS (Oreshkin et al., 2020) is a pure DL architecture without LSTM or attention, based on residual stacks of fully connected layers with basis function outputs:

- Each block produces a **backcast** (fitted to history) and **forecast** (output).
- Residuals from the backcast are passed to the next block.
- Interpretable variant uses Fourier and polynomial basis functions.

N-BEATS consistently outperforms traditional methods on the M3/M4 benchmarks.

### Temporal Fusion Transformer (TFT)

TFT (Lim et al., 2021) is a multi-horizon attention-based architecture:
- Handles static covariates, known future covariates, and observed past covariates.
- Uses a **Variable Selection Network** to identify informative inputs.
- Interpretable: attention weights show which past time steps are most relevant per forecast step.

## Inductive Bias

- **Lag-feature + GBM**: assumes the relationship between past observations and future values is well-captured by a finite set of handcrafted lags; can model complex non-linear patterns.
- **LSTM**: assumes temporal dependencies are captured by a compact hidden state; good at long-range dependencies but requires large data.
- **N-BEATS/TFT**: architectures designed explicitly for time series; less inductive bias than classical models.

## Training Objective

| Approach | Typical loss | Notes |
|---|---|---|
| Point forecasting | MAE, RMSE, MAPE | Choose based on domain (asymmetric errors → MAPE) |
| Quantile forecasting | Pinball/quantile loss | For prediction intervals |
| Probabilistic | NLL (Gaussian, NegBin) | Calibrated distributions |

## Strengths

- GBM lag approach is fast, interpretable, and robust; works well for tabular/panel data.
- Deep learning models capture complex interactions and scale to many series.
- Can incorporate exogenous covariates naturally.

## Weaknesses

- Lag features require careful engineering; wrong lags → poor performance.
- LSTMs are hard to train for very long sequences; prone to vanishing gradients.
- All ML approaches require careful temporal cross-validation to avoid data leakage (future test points must not appear in training features).

## Variants

- **Prophet** (Meta): additive model with piecewise linear trend, Fourier seasonality, and holiday effects. Good for business forecasting with strong seasonality.
- **Temporal Convolutional Network (TCN)**: dilated causal convolutions for long-range dependencies without recurrence.
- **DeepAR** (Amazon): autoregressive RNN that outputs full probability distributions.
- **PatchTST**: treats time series as patches fed to a standard Transformer.

## References

- Oreshkin, B. et al. (2020). "N-BEATS: Neural basis expansion analysis for interpretable time series forecasting." ICLR.
- Lim, B. et al. (2021). "Temporal Fusion Transformers for interpretable multi-horizon time series forecasting." *Int. J. Forecasting*.
- Godahewa, R. et al. (2021). "Monash Time Series Forecasting Archive." NeurIPS.

## Links

- [[03_modeling/05_time_series/01_classical_forecasting/time_series_models|Time Series Models — ARIMA and statistical baseline]]
- [[03_modeling/01_supervised_learning/02_tree_based_models/tree_ensembles|Tree Ensembles — gradient boosting backbone]]
- [[03_modeling/04_deep_learning/03_sequence_models/recurrent_networks|Recurrent Networks — LSTM architecture]]
- [[02_data_science/04_feature_engineering/feature_engineering|Feature Engineering — lag and rolling window features]]
- [[01_foundations/06_deep_learning_theory/backpropagation_through_time|Backpropagation Through Time]] — gradient flow in RNNs and LSTMs used for sequence forecasting
