---
layer: 07_applications
type: application
status: growing
tags: [forecasting, time-series, tabular]
created: 2026-03-11
---

# Demand Forecasting

## Problem

Predict future product demand at a specified granularity (SKU × location × week) to drive replenishment, production, and logistics decisions. Errors have asymmetric costs: overstocking ties up capital and generates waste; stockouts lose revenue and customer trust. The forecasting problem has two regimes — base demand (slow-moving trend + seasonality) and uplift demand (driven by promotions, pricing changes, external events).

## Users / Stakeholders

| Role | Decision driven by forecast |
|---|---|
| Supply chain planner | When and how much to order / produce |
| Merchandiser | Which products to promote; how deep to discount |
| Logistics manager | Warehouse space allocation, distribution routing |
| Finance | Working capital and inventory provision |
| Product manager | New product launch planning (cold start) |

Forecasts are primarily consumed via a **planning system** (SAP, o9, Blue Yonder) — direct human consumption is at aggregate level (category, region). Individual SKU forecasts are machine-consumed.

## Domain Context

- **Hierarchy**: Demand is hierarchical (total → category → brand → SKU → store). Forecasts must be consistent across levels (hierarchical reconciliation: bottom-up, top-down, MinT).
- **Intermittent demand**: Low-volume SKUs have many zeros. Standard ARIMA assumptions fail. Use Croston's method or Tweedie regression.
- **Causal drivers**: Promotions (type, depth, display), pricing elasticity, competitor actions, weather, holidays, product lifecycle stage.
- **External data**: Macroeconomic indicators, weather APIs, Google Trends — high value but adds pipeline complexity.
- **Cold start**: New SKUs have no history. Transfer learning from category-level model or feature-based regression.
- **Regulatory / privacy**: Minimal — demand data is internal operational data. GDPR applies only if linked to identifiable customer purchases.
- **Seasonality patterns**: May be multiple (weekly, annual, promotional calendar). Fourier terms or STL decomposition needed.

## Inputs and Outputs

**Input features**:
```
Historical sales time series (unit sales, revenue)
Price (own + competitor)
Promotion flags (on_promotion, promo_type, discount_depth)
Calendar features (day_of_week, week_of_year, holiday_flag)
External: weather (temperature, precipitation), events
Lag features (lag_1w, lag_4w, lag_52w, rolling_mean_4w)
Product attributes (category, brand, weight, shelf_life)
Store attributes (size, location_type, demographic_index)
```

**Output**:
```
Point forecast:  ŷ_{t+h}  for horizons h = 1..12 weeks
Prediction intervals: P10, P50, P90 quantiles
Bias metric: MASE, wMAPE per SKU
Uncertainty flag: high-variance SKUs for human review
```

**Data volumes**: A mid-size retailer has 50K–500K SKU×store combinations. Full refresh weekly; rolling 2-year history. Batch inference on order of 10M rows per cycle.

## Decision or Workflow Role

```
Forecast generation (weekly batch)
  ↓
Planners review exceptions (auto-approve if within tolerance)
  ↓
Replenishment orders generated in ERP/WMS
  ↓
Delivery + actual sales → feedback into training data
  ↓
Model performance monitoring → retraining trigger
```

The forecast is an **input to a constrained optimisation** (replenishment / allocation). The optimisation translates forecast + cost parameters into an order quantity. ML model quality directly impacts order quality and working capital.

## Modeling / System Options

| Approach | Strength | Weakness | When to use |
|---|---|---|---|
| Statistical (ARIMA, ETS) | Interpretable, fast, well-understood | No cross-series learning, limited causal features | Mature SKUs, stable demand, small portfolio |
| LightGBM + lag features | Handles many causal features, cross-series learning, fast | No explicit temporal structure | Large portfolio, rich causal data |
| Prophet | Automatic seasonality + holidays, interpretable | Slow at scale, weak on promotions | Medium portfolio, business stakeholder reports |
| DeepAR (LSTM) | Captures complex patterns, probabilistic | Data-hungry, slow training, hard to debug | Very large portfolio, sufficient history |
| N-BEATS / N-HiTS | State-of-the-art point accuracy, no feature engineering | Black box, complex to tune | Accuracy-critical portfolios |
| Hierarchical ensemble | Best aggregate accuracy | Complex reconciliation logic | When consistency across levels is critical |

**Recommended starting point**: LightGBM with lag features + calendar + promotion dummies. Hierarchically reconcile using MinT (shrinkage). Add DeepAR for high-velocity SKUs if compute budget allows.

See [[03_modeling/05_time_series/01_classical_forecasting/time_series_implementation|Time Series Implementation]] and [[08_implementations/02_end_to_end_examples/demand_forecasting_pipeline|Demand Forecasting Pipeline]] for code.

## Deployment Constraints

- **Latency**: Batch — weekly refresh. Not latency-sensitive. Full inference cycle budget: 2–4 hours for 10M rows.
- **Throughput**: Parallel inference by product category. LightGBM handles millions of rows in minutes.
- **Interpretability**: Planners need to understand *why* the forecast changed. SHAP waterfall charts per SKU at exception review. Feature importance must be stable across runs.
- **Stale forecast handling**: If pipeline fails, fallback to last valid forecast. Staleness SLA: ≤14 days before mandatory escalation.
- **Retraining cadence**: Weekly retrain on new actuals. Full retrain monthly with all history. Trigger alert if MASE > 1.5 on any category.

## Risks and Failure Modes

| Risk | Description | Mitigation |
|---|---|---|
| **Data leakage** | Using future promotion flags at train time as if known | Strict point-in-time feature construction; cutoff validation |
| **Distribution shift** | Demand patterns change at new stores, post-COVID, price changes | Segment-specific models; rolling window training; drift monitor |
| **Intermittent demand** | MAPE is undefined when actual = 0; mean over-weights outliers | Use MASE; evaluate separately for intermittent SKUs |
| **Promotion blindness** | Model not exposed to future promotions at inference | Promotional calendar must be loaded into feature pipeline |
| **Concept drift** | Category trends change (e.g., energy drinks growing 30% YoY) | Trend features; short training windows for fast-moving categories |
| **Cold start** | New SKUs have zero history | Attribute-based regression; seed from similar existing SKU |
| **Outlier actuals** | One-off events (store fire, supply disruption) inflate error | Outlier detection + annotation pipeline; conditional training |

## Success Metrics

| Metric | Target | Notes |
|---|---|---|
| wMAPE | < 25% at SKU×week | Weighted by volume; less sensitive to low-volume SKUs |
| MASE | < 1.0 (better than seasonal naïve) | Model-agnostic, scale-invariant |
| Bias | |MPE| < 5% at category level | Systematic bias is more damaging than random noise |
| Fill rate | > 97.5% | Downstream business metric; driven by order quantity not just forecast |
| Inventory turnover | Increase YoY | True business outcome; harder to attribute to model alone |
| Planner override rate | < 15% | High override rate signals model distrust |

## References

- Makridakis, S. et al. (2020). *M5 Accuracy competition: Results, findings and conclusions.*
- Hyndman, R. & Athanasopoulos, G. (2021). *Forecasting: Principles and Practice* (3rd ed.)
- Salinas, D. et al. (2019). *DeepAR: Probabilistic Forecasting with Autoregressive Recurrent Networks.*

## Links

**Modeling**
- [[03_modeling/05_time_series/index|Time Series Models]] — ARIMA, state space, seasonality decomposition
- [[03_modeling/01_supervised_learning/02_tree_based_models/gradient_boosting|Gradient Boosting]] — LightGBM for tabular demand forecasting

**ML Engineering**
- [[05_ml_engineering/06_deployment_and_serving/index|Deployment and Serving]] — batch inference at scale
- [[05_ml_engineering/07_monitoring_and_observability/index|Monitoring and Observability]] — forecast accuracy tracking

**Reference Implementations**
- [[03_modeling/05_time_series/01_classical_forecasting/time_series_implementation|Time Series Implementation]]
- [[08_implementations/02_end_to_end_examples/demand_forecasting_pipeline|Demand Forecasting Pipeline]]

**Adjacent Applications**
- [[churn_prediction|Churn Prediction]]
- [[07_applications/08_domain_verticals/04_ecommerce/inventory_optimization|Inventory Optimization]]
