---
layer: 07_applications
type: application
status: growing
tags: [forecasting, time-series, tabular]
created: 2026-03-11
---

# Algorithmic Trading Signals

## Problem

Generate predictive signals for systematic trading strategies — predicting short-term directional price movements, volatility regimes, or cross-asset relative value — using ML models trained on market data, alternative data, and fundamental indicators. The signals are inputs to a portfolio construction and execution engine; the ML component is responsible for alpha generation, not order execution.

**Distinct from HFT**: This covers medium-frequency systematic (holding periods hours to weeks), not microsecond market-making.

## Users / Stakeholders

| Role | Decision |
|---|---|
| Quantitative researcher | Design and validate signals; backtest strategies |
| Portfolio manager | Allocate capital to strategies; adjust risk limits |
| Risk manager | Monitor live strategy risk; enforce drawdown limits |
| Compliance officer | Trade surveillance; MAR (Market Abuse Regulation) compliance |
| Technology team | Production signal generation infrastructure |

## Domain Context

- **Non-stationarity**: Financial time series are non-stationary. Relationships that held historically may break. Models trained on 2015–2019 data may fail in 2020–2022. Regime detection is critical.
- **Low signal-to-noise ratio**: Markets are semi-efficient. Genuine alpha is small and erodes as it is discovered and traded. Typical Sharpe ratios of 0.5–1.5 are considered good.
- **Lookahead bias**: The primary risk in backtesting. Any use of data not available at prediction time produces falsely optimistic backtests. Strict point-in-time feature computation.
- **Market impact**: A signal may be profitable in backtest but generate costs that exceed the alpha in live trading (market impact, slippage, transaction costs). Signal capacity matters.
- **Regulatory**: MiFID II requires systematic internaliser reporting, best execution documentation, and algorithm notification. MAR prohibits market manipulation — ML strategies must be reviewed for potential manipulation.
- **Data quality and survivorship bias**: Historical price data has survivorship bias (delisted stocks absent). Fundamental data has point-in-time problems (earnings restatements). Data vendors must provide point-in-time clean data.

## Inputs and Outputs

**Feature categories**:
```
Price/volume: returns_1d/5d/21d, volume_ratio, price_momentum, RSI, MACD
Microstructure: bid-ask spread, order flow imbalance, market depth
Alternative data: earnings call sentiment NLP, satellite imagery, credit card spend
Fundamental: P/E ratio, earnings revision, debt/equity, ROE, analyst estimates
Macro: yield curve shape, credit spreads, VIX regime, sector rotation
Cross-asset: equity-bond correlation, currency factor, commodity signal
```

**Output**:
```
signal_score:     Directional score ∈ [-1, 1] per asset
signal_confidence: Model confidence (used for position sizing)
regime_flag:      TRENDING / MEAN_REVERTING / HIGH_VOLATILITY
holding_period:   Expected holding period (days)
```

## Decision or Workflow Role

```
Data ingestion: market data + alternative data (daily or intraday)
  ↓
Feature construction: with strict point-in-time guarantees
  ↓
Signal model inference: score per asset
  ↓
Portfolio construction: signal → weight (mean-variance optimisation or risk parity)
  ↓
Risk overlay: position limits, factor exposure limits, drawdown controls
  ↓
Execution: orders routed via TCA-optimised execution algorithm
  ↓
P&L attribution: signal contribution vs execution slippage vs market impact
  ↓
Model performance monitoring → retrain trigger → research cycle
```

## Modeling / System Options

| Approach | Strength | Weakness | When to use |
|---|---|---|---|
| LightGBM / XGBoost | Robust to non-linear relationships; feature importance | Overfits to regime; slow to adapt | Cross-sectional equity signals |
| LSTM / Transformer | Captures temporal patterns | Data-hungry; overfits easily in finance | High-frequency intraday signals with many assets |
| Linear factor model | Interpretable; regime-stable; low capacity consumption | Misses non-linearity | Classic quant approach; first factor model |
| Ensemble of diverse models | Reduces regime sensitivity | Complex to manage | Production with multiple signal sources |
| Reinforcement learning | Adaptive; optimises portfolio directly | Extremely hard to train; high variance | Research exploration; not yet production standard |

**Recommended**: Ensemble of LightGBM cross-sectional + linear factor model for equity signals. Strict information coefficient (IC) validation before production.

## Deployment Constraints

- **Latency**: Daily signals: end-of-day processing. Intraday: sub-minute for medium-frequency.
- **Reproducibility**: Every production signal must be exactly reproducible from historical inputs. Deterministic random seeds; versioned feature code.
- **Backtesting infrastructure**: Walk-forward backtesting with transaction costs, market impact, and capacity constraints. Overfit-aware evaluation (deflated Sharpe ratio).
- **Regulatory reporting**: MAR algorithm notification; audit trail of all signals and orders.

## Risks and Failure Modes

| Risk | Description | Mitigation |
|---|---|---|
| **Lookahead bias** | Future data in features → false alpha | Strict point-in-time feature store; backtesting audit |
| **Overfitting** | Backtest looks good; live performance fails | Out-of-sample validation; paper trading period |
| **Regime change** | Model trained on bull market fails in crisis | Regime-conditional model; equity volatility filter |
| **Capacity constraint** | Signal degrades when position size scaled up | Capacity analysis in backtest; market impact model |
| **Data vendor error** | Bad input data → incorrect signals | Data quality monitoring; cross-vendor reconciliation |

## Success Metrics

| Metric | Target | Notes |
|---|---|---|
| Information coefficient (IC) | > 0.03 (daily) | Correlation of signal with next-period returns |
| Sharpe ratio (live) | > 0.8 annualised | After transaction costs |
| Max drawdown | < 15% | Risk management constraint |
| Hit rate | > 52% | Fraction of profitable trades |
| IC decay | Stable over 6m | Signal persistence |

## References

- Chincarini, L. & Kim, D. (2006). *Quantitative Equity Portfolio Management.* McGraw-Hill.
- López de Prado, M. (2018). *Advances in Financial Machine Learning.* Wiley.

## Links

**Modeling**
- [[03_modeling/05_time_series/index|Time Series Models]] — time series forecasting
- [[03_modeling/01_supervised_learning/02_tree_based_models/gradient_boosting|Gradient Boosting]] — cross-sectional signal generation

**Reference Implementations**
- [[08_reference_implementations/01_model_implementations/time_series_implementation|Time Series Implementation]]
- [[08_reference_implementations/01_model_implementations/tree_ensembles_implementation|Tree Ensembles Implementation]]

**Adjacent Applications**
- [[risk_portfolio_monitoring|Portfolio Risk Monitoring]]
- [[07_applications/03_detection_and_monitoring/fraud_detection|Fraud Detection]]
