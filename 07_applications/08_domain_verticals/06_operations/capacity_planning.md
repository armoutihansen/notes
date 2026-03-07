---
layer: 07_applications
type: application
status: growing
tags: [forecasting, workflow, tabular]
created: 2026-03-11
---

# Capacity Planning

## Problem

Forecast future demand for resources — staff headcount, server compute, warehouse space, call centre seats — and plan capacity accordingly. Capacity planning operates on longer time horizons (weeks to months) than real-time demand forecasting, and the cost of under-capacity (service degradation, SLA breaches) and over-capacity (idle costs) are both significant.

## Users / Stakeholders

| Role | Decision |
|---|---|
| Operations manager | Staffing levels; shift planning |
| Finance | Cost forecasts; budget allocation |
| Technology leader | Infrastructure provisioning; cloud spend |
| HR/Workforce | Hiring plans; contractor engagement |
| Customer | Service quality; SLA compliance |

## Domain Context

- **Long lead times**: Hiring, training, and infrastructure provisioning have lead times of weeks to months. The forecast must be accurate far enough ahead to enable action.
- **Hierarchical**: Total demand breaks down by region, product, channel, time-of-day. Each level has different planning implications.
- **Demand drivers**: Business volume (policy renewals, order volumes, support tickets) drives resource demand. External events (weather, regulation changes, product launches) create step changes.
- **Workforce constraints**: Minimum staffing levels, shift patterns, skills-based routing — operational constraints that pure forecasting models don't capture.
- **Cloud auto-scaling**: For compute capacity, auto-scaling handles short-term (minutes to hours) adjustments. Capacity planning focuses on baseline provisioning and cost optimisation.

## Inputs and Outputs

**Features**:
```
Historical demand: volume_last_52w, volume_same_period_prior_year
Calendar: day_of_week, holiday_flags, product_season, renewal_cycle
Business drivers: new_business_rate, retention_rate, product_mix
External: economic_index, weather (call centre: bad weather = more calls)
Pipeline: sales_pipeline, marketing_spend, campaign_schedule
Operational: handling_time_per_unit, automation_rate
```

**Output**:
```
demand_forecast:     Weekly volume by resource type for next 26 weeks
capacity_requirement: FTE required, seats required, compute_units required
hiring_recommendation: Headcount delta with lead time factored in
budget_forecast:     Cost at recommended capacity
risk_scenario:       P90 demand scenario → capacity buffer recommendation
```

## Decision or Workflow Role

```
Quarterly capacity review (strategic) + monthly rolling update
  ↓
Demand forecast: 26-week forward view by resource type
  ↓
Capacity gap analysis: forecast vs current approved headcount
  ↓
Scenario analysis: base / upside / downside scenarios
  ↓
Decisions: hire / train / redeploy / contract / defer
  ↓
Operational: weekly workforce scheduling (WFM system)
  ↓
Actuals vs forecast → model retraining
```

## Modeling / System Options

| Approach | Strength | Weakness | When to use |
|---|---|---|---|
| ARIMA + external regressors | Handles seasonality; interpretable | Limited feature set | Simple, stable demand |
| LightGBM with calendar + business features | Captures complex relationships | Requires feature engineering | Rich business driver data available |
| Prophet (Facebook) | Automatic seasonality + holidays; interpretable | Slow at scale | Quarterly planning; business stakeholder reporting |
| Erlang C (call centre) | Industry standard for staffing; analytical | Assumes stationary Poisson process | Call centre / service desk capacity |

**Recommended**: LightGBM for forecast. Erlang C for translating call volume to agent headcount. Prophet for stakeholder-facing reports.

## Deployment Constraints

- **Planning horizon**: 26-week forward view. Weekly refresh. Scenario outputs needed.
- **Stakeholder reporting**: Finance and HR teams are consumers — not data scientists. Visual dashboards with uncertainty ranges, not model outputs.
- **Decision latency**: Hiring decisions need 8–12 weeks lead time. Forecast must be actionable within that window.

## Risks and Failure Modes

| Risk | Description | Mitigation |
|---|---|---|
| **Demand step change** | New product launch or regulation → non-historical demand | Scenario planning; manual uplift |
| **Forecast optimism** | Systematic under-forecast → chronic understaffing | Bias monitoring; post-season review |
| **Long lead time risk** | Hiring too late → SLA breach | Lead-time-adjusted planning; contractor buffer |
| **Automation rate change** | Automation improves; model uses old handling time | Handling time as input parameter; quarterly review |

## Success Metrics

| Metric | Target | Notes |
|---|---|---|
| Forecast accuracy (MAPE) | < 10% at 4-week horizon | Planning accuracy |
| SLA compliance | > 99% | Downstream service quality |
| Cost per unit | Stable or decreasing | Operational efficiency |
| Capacity utilisation | 85–90% | Asset efficiency without overloading |
| Hiring lead time met | > 95% | Operational outcome |

## Links

**Modeling**
- [[03_modeling/05_time_series/index|Time Series Models]] — long-horizon time series forecasting
- [[03_modeling/01_supervised_learning/02_tree_based_models/gradient_boosting|Gradient Boosting]] — demand forecasting with business features

**Reference Implementations**
- [[03_modeling/05_time_series/01_classical_forecasting/time_series_implementation|Time Series Implementation]]
- [[08_implementations/02_end_to_end_examples/demand_forecasting_pipeline|Demand Forecasting Pipeline]]

**Adjacent Applications**
- [[quality_control_vision|Quality Control Vision]]
- [[07_applications/01_prediction_and_forecasting/demand_forecasting|Demand Forecasting]]
- [[07_applications/08_domain_verticals/05_mobility/route_optimization|Route Optimization]]
