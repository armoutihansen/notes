---
layer: 02_modeling
type: concept
status: growing
tags: [time-series, arima, stationary, autocorrelation, forecasting, state-space, lstm]
created: 2026-03-06
---

# Time Series Models

## Definition

Models for sequential data where the order of observations matters and temporal dependencies carry predictive information. Covers classical statistical methods (ARIMA, state-space models) and modern neural approaches.

## Intuition

A time series is a sequence of observations indexed by time. The key challenge is that observations are not i.i.d.: knowing yesterday's value tells you something about today's. The goal is to model this autocorrelation structure to make forecasts.

## Formal Description

### Stationarity

A time series $\{y_t\}$ is **weakly stationary** if:
- $\mathbb{E}[y_t] = \mu$ (constant mean)
- $\mathrm{Cov}(y_t, y_{t+h}) = \gamma(h)$ depends only on lag $h$, not $t$

Most classical methods assume stationarity. **Augmented Dickey-Fuller (ADF) test** checks for unit roots (non-stationarity).

**Differencing** to achieve stationarity: $\Delta y_t = y_t - y_{t-1}$. The order of differencing $d$ is chosen such that $\Delta^d y_t$ is stationary.

### ARIMA$(p, d, q)$

**AR($p$) — Autoregressive:** $y_t = \sum_{i=1}^p \phi_i y_{t-i} + \varepsilon_t$

**MA($q$) — Moving Average:** $y_t = \mu + \sum_{j=1}^q \theta_j \varepsilon_{t-j} + \varepsilon_t$

**ARMA($p,q$):** $\sum_{i=0}^p \phi_i y_{t-i} = \sum_{j=0}^q \theta_j \varepsilon_{t-j}$

**ARIMA($p,d,q$):** Apply $d$-th differencing to $y_t$, then fit ARMA($p,q$).

**SARIMA($p,d,q$)($P,D,Q$)$_s$:** adds seasonal AR, I, MA terms at lag $s$ (e.g., $s=12$ for monthly data).

**Order selection:**
- ACF (autocorrelation function): significant at lags 1..q → MA(q)
- PACF (partial autocorrelation): significant at lags 1..p → AR(p)
- Use AIC/BIC for model selection; `auto_arima` from pmdarima automates this.

### Exponential Smoothing (ETS)

Weighted average of past observations, with exponentially decaying weights:

**Simple ES:** $\hat{y}_{t+1} = \alpha y_t + (1-\alpha)\hat{y}_t$

**Holt-Winters (Triple ES):** models level, trend, and seasonality. ETS model:
- Error type (Additive/Multiplicative)
- Trend type (None/Additive/Additive-damped)
- Seasonal type (None/Additive/Multiplicative)

Multiplicative seasonality is appropriate when seasonal fluctuations are proportional to the level.

### State-Space Models (Local Linear Trend)

$$y_t = H_t \mathbf{z}_t + \varepsilon_t \quad \text{(observation equation)}$$

$$\mathbf{z}_{t+1} = F_t \mathbf{z}_t + G_t \eta_t \quad \text{(state transition equation)}$$

The **Kalman filter** computes $P(\mathbf{z}_t | y_1,\ldots,y_t)$ analytically for linear-Gaussian systems.

### VAR (Vector Autoregression)

Multivariate extension of AR for $K$ time series:

$$\mathbf{y}_t = \sum_{i=1}^p A_i \mathbf{y}_{t-i} + \boldsymbol{\varepsilon}_t$$

Models cross-series dependencies. Used in macroeconomics; Granger causality tests check whether one series helps predict another.

### Neural Time Series Models

**LSTM/GRU:** sequence-to-sequence models that learn long-range dependencies from data (see [[recurrent_networks]]).

**Temporal Convolutional Networks (TCN):** dilated causal convolutions; can outperform LSTMs with faster training.

**Transformer-based:** Informer, Autoformer, PatchTST for long-horizon forecasting; attention captures long-range dependencies without sequential processing.

**N-BEATS, N-HiTS:** pure neural, interpretable forecasting; state of the art on M4 benchmark.

### Evaluation Metrics

| Metric | Formula                                     | Properties                |     |                                     |     |                                   |
| ------ | ------------------------------------------- | ------------------------- | --- | ----------------------------------- | --- | --------------------------------- |
| MAE    | $\frac{1}{T}\sum                            | y_t - \hat{y}_t           | $   | Scale-dependent, robust to outliers |     |                                   |
| RMSE   | $\sqrt{\frac{1}{T}\sum(y_t - \hat{y}_t)^2}$ | Penalises large errors    |     |                                     |     |                                   |
| MAPE   | $\frac{100}{T}\sum\frac{                    | y_t - \hat{y}_t           | }{  | y_t                                 | }$  | Scale-free; undefined for $y_t=0$ |
| MASE   | MAE normalised by naive in-sample MAE       | Scale-free and meaningful |     |                                     |     |                                   |

## Applications

- Insurance: claims reserving (chain-ladder, stochastic development methods)
- Finance: volatility forecasting (GARCH), algorithmic trading signals
- Demand forecasting: inventory management, supply chain
- Anomaly detection: detecting unusual spikes in time series metrics

## Trade-offs

- ARIMA: interpretable, fast, principled; limited to linear dependencies and stationary processes.
- LSTM/Transformers: capture non-linear temporal patterns; require more data and tuning.
- Choose classical methods when $n < 500$ or explainability is required; neural methods for large-scale, complex patterns.

## Links

- [[01_foundations/03_probability_and_statistics/hypothesis_testing|ADF Test (Stationarity)]]
- [[02_modeling/03_model_families/08_graphical_models/graphical_models|Graphical Models (HMM, State-Space)]]
- [[02_modeling/03_model_families/04_neural_networks/recurrent_networks|Recurrent Networks (LSTM)]]
- [[02_modeling/04_training_dynamics/loss_functions|Loss Functions]]
