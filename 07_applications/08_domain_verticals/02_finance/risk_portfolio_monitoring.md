---
layer: 07_applications
type: application
status: growing
tags: [monitoring, time-series, anomaly-detection]
created: 2026-03-11
---

# Portfolio Risk Monitoring

## Problem

Monitor the risk profile of an investment portfolio in real time — tracking exposure to market factors, detecting concentration breaches, flagging anomalous price movements, and generating risk reports. Portfolio risk management translates abstract model outputs (VaR, factor loadings, stress tests) into actionable alerts and reports for risk managers, portfolio managers, and regulators.

## Users / Stakeholders

| Role | Decision |
|---|---|
| Portfolio manager | Adjust positions to manage factor exposure |
| Risk manager | Trigger risk limits; escalate to investment committee |
| CRO | Board-level risk reporting; regulatory capital |
| Compliance | UCITS/AIFMD investment restriction monitoring |
| Operations | Margin calls, collateral management |

## Domain Context

- **Multi-asset**: Equities, fixed income, derivatives, FX — each has different risk measurement approaches (duration for bonds, delta/gamma for options).
- **Factor exposure**: Modern portfolio theory: risk is factor exposure (beta, duration, credit spread, FX). Factor-based risk decomposition is standard.
- **Real-time requirement**: Intraday P&L and risk monitoring. Position data updates tick-by-tick or at end of each trade.
- **Regulatory**: UCITS (diversification limits), AIFMD (leverage limits), Basel III (VaR and stressed VaR). Internal model approval by regulator.
- **Model risk**: VaR models require backtesting (Kupiec test, christoffersen test). Stressed scenarios must include 2008, COVID-2020, etc.

## Inputs and Outputs

**Inputs**:
```
Portfolio: position data (asset, quantity, entry price, currency)
Market data: real-time prices, yield curves, FX rates, volatility surfaces
Risk factors: equity factor loadings (Barra, Axioma), duration, credit spreads
Scenarios: historical stress scenarios, hypothetical shocks
Constraints: investment mandate limits, regulatory limits
```

**Outputs**:
```
VaR_1d_99%:     Daily 99% Value at Risk
CVaR:           Conditional VaR (expected shortfall)
factor_exposures: Beta, duration, credit_spread_dv01, FX_delta
concentration: Top-10 holdings; sector/country exposure
stress_P&L:    P&L under 2008 crisis / COVID / rate shock scenarios
alerts:        Limit breach flags with severity
```

## Decision or Workflow Role

```
Trade data + market data → position valuation → real-time P&L
  ↓
Risk analytics engine (Python / QuantLib / commercial: Axioma, Riskmetrics)
  ↓
Factor model: returns decomposition → factor exposures
  ↓
VaR calculation: parametric + historical simulation + Monte Carlo
  ↓
Limit monitoring: compare to mandate limits → flag breaches
  ↓
Anomaly detection: flag unusual factor exposures vs historical range
  ↓
Risk dashboard update (intraday) + risk report (end of day)
  ↓
Breach alert → PM/risk team → position adjustment or limit override request
```

## Modeling / System Options

| Component | Approach | Notes |
|---|---|---|
| VaR | Historical simulation (10y lookback) | Regulatory standard |
| Factor model | PCA on returns or commercial (Barra MSCI) | Decompose systematic vs idiosyncratic |
| Stress testing | Historical scenarios + reverse stress | Complementary to VaR |
| Anomaly detection | Mahalanobis distance on factor exposures | Flags unusual positioning |
| Concentration | Rules-based + UCITS limits | Deterministic |

## Deployment Constraints

- **Latency**: Intraday risk update: 5-minute granularity. EOD full risk report: <30 minutes.
- **Regulatory reporting**: UCITS daily compliance report. Solvency II SCR for insurance portfolios.
- **Model governance**: VaR backtesting required quarterly. Model approval process for regulatory internal models.

## Risks and Failure Modes

| Risk | Description | Mitigation |
|---|---|---|
| **VaR underestimation** | VaR model assumes normal returns; fat tails in crisis | CVaR; stress testing; historical simulation |
| **Data latency** | Stale price data → wrong risk measures | Data freshness monitoring; stale price alerts |
| **Factor model instability** | Correlations change in crisis | Stressed correlation matrix; regime-conditional model |
| **Regulatory breach** | Investment limit breach not caught → fine/fund wind-up | Real-time limit monitoring; hard limits with pre-trade checks |

## Success Metrics

| Metric | Target | Notes |
|---|---|---|
| VaR breach rate | ~1% of days (99% VaR by definition) | Backtesting KPI; Kupiec test |
| Risk report delivery | < 30min post-close | Operational SLA |
| Limit breach detection latency | < 5 minutes | Real-time monitoring |
| Stress test coverage | 100% of regulatory scenarios | Completeness |

## References

- Jorion, P. (2006). *Value at Risk: The New Benchmark for Managing Financial Risk.* McGraw-Hill.
- RiskMetrics Group (1994). *RiskMetrics Technical Document.*

## Links

**Modeling**
- [[03_modeling/05_time_series/index|Time Series Models]] — volatility modelling
- [[03_modeling/02_unsupervised_learning/index|Unsupervised Learning]] — anomaly detection, PCA factor decomposition

**Reference Implementations**
- [[08_reference_implementations/01_model_implementations/unsupervised_learning_implementation|Unsupervised Learning Implementation]]
- [[08_reference_implementations/02_system_patterns/drift_monitoring_with_evidently|Drift Monitoring with Evidently]]

**Adjacent Applications**
- [[algorithmic_trading_signals|Algorithmic Trading Signals]]
- [[07_applications/03_detection_and_monitoring/anomaly_detection_operations|Anomaly Detection — Operations]]
