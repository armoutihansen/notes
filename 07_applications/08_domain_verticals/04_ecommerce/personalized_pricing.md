---
layer: 07_applications
type: application
status: growing
tags: [regression, forecasting, tabular]
created: 2026-03-11
---

# Personalized Pricing

## Problem

Set prices dynamically — varying by product, time, competitor pricing, inventory level, and demand signals — and optionally by customer segment or individual — to maximise revenue, margin, or a composite business objective. Dynamic pricing (same price for all, varies over time) is widely accepted; personalised pricing (different price per customer) is legally and ethically contentious.

**Important distinction**:
- **Dynamic/algorithmic pricing**: Price changes with demand, inventory, time — all customers see the same price at a given moment → widely accepted
- **Price discrimination by segment**: Different prices for different customer groups (e.g., loyalty tier, geography) → accepted if disclosed
- **Individual price personalisation**: Different prices per customer based on inferred willingness to pay → legally restricted in many markets

## Users / Stakeholders

| Role | Decision |
|---|---|
| Pricing manager | Set pricing rules and guardrails |
| Category manager | Competitive positioning for key products |
| Finance | Revenue and margin planning |
| Legal / compliance | Consumer protection law compliance |
| Customer | Experience transparency and fairness |

## Domain Context

- **Competitive dynamics**: Price changes trigger competitor responses. Automated repricing without limits creates price spirals (documented on Amazon).
- **Consumer Protection**: UK: Consumer Rights Act. EU: Omnibus Directive (requires showing reference price before discount). US: state-level price gouging laws triggered in emergencies. Personalised pricing based on demographics (race, gender) is illegal.
- **Price elasticity**: Core economic concept. ML learns elasticity from historical data. Elasticity varies by product category, customer segment, and time.
- **Anchoring effects**: Showing a high "was" price alongside a lower current price drives conversion — behavioural economics in pricing.
- **Data requirements**: Need matched price-demand observations with price variation. If price never changed historically, elasticity cannot be estimated.

## Inputs and Outputs

**Dynamic pricing features**:
```
Demand signals: page_views_1h, add_to_cart_rate, conversion_rate_24h
Competition: competitor_price (scraped hourly), price_rank_vs_competitors
Inventory: stock_level, days_of_cover, replenishment_lead_time
Product: category, brand, margin, cost, minimum_price_floor
Time: day_of_week, hour, days_to_expiry (perishables), seasonal_index
```

**Output**:
```
recommended_price:    £X.XX within [floor, ceiling] range
expected_revenue:     Predicted revenue at recommended price
elasticity_estimate:  % demand change per 1% price change
price_change_action:  INCREASE / DECREASE / HOLD / PROMOTE
```

## Decision or Workflow Role

```
Pricing engine runs (hourly or event-triggered by competitor price change)
  ↓
Demand model: estimate demand at current vs alternative prices
Margin model: calculate gross margin at each candidate price
  ↓
Optimisation: maximise revenue × margin subject to constraints
  (floor: cost + minimum margin; ceiling: MAP or brand guidelines)
  ↓
Guardrails: max price change per cycle; no pricing below cost; competitor band
  ↓
Price pushed to e-commerce platform via API
  ↓
Outcome: sales, revenue, conversion logged → model training data
```

## Modeling / System Options

| Approach | Strength | Weakness | When to use |
|---|---|---|---|
| Demand curve regression (log-log) | Interpretable elasticity; economically grounded | Assumes constant elasticity; limited interactions | Baseline; category-level pricing |
| LightGBM demand model | Captures complex interactions; feature-rich | Black box; harder to audit | Large catalogue; rich demand signals |
| Contextual bandits | Adaptive; optimises online; handles exploration | Exploration cost; variance | When rapid adaptation needed |
| Bayesian structural time series | Uncertainty quantification; causal price effect | Complex; slower | Attribution and A/B analysis |

**Recommended**: LightGBM demand model + price optimiser. Contextual bandits for high-velocity SKUs.

## Deployment Constraints

- **Guardrails mandatory**: Min/max price bounds; max change per hour. Without guardrails, automated pricing can create reputational damage (e.g., £700 textbooks).
- **Price change explanation**: Must be auditable. If challenged, must demonstrate price was driven by legitimate demand/competitive factors, not customer profiling.
- **Latency**: Price update can be near-real-time (1–5 minutes). Inference latency not a constraint.

## Risks and Failure Modes

| Risk | Description | Mitigation |
|---|---|---|
| **Price spiral** | Competing algorithms drive prices to extremes | Guardrails; monitoring; manual override |
| **Discriminatory pricing** | Proxy for demographics in pricing features | Feature audit; legal review |
| **Price display errors** | Wrong price shown vs charged → consumer law violation | Pre-display validation; price comparison audit |
| **Elasticity overestimate** | Model thinks demand elastic; prices raised; demand collapses | Causal inference; A/B price experiments |

## Success Metrics

| Metric | Target | Notes |
|---|---|---|
| Revenue per unit | +3–8% vs fixed pricing | Causal A/B test |
| Gross margin | +1–3pp | Not just revenue uplift |
| Conversion rate | ≥ baseline | Pricing must not hurt conversion |
| Price audit compliance | 100% prices within approved bounds | Guardrail KPI |
| Competitor price rank | Track position changes | Competitive monitoring |

## References

- Phillips, R. (2005). *Pricing and Revenue Optimization.* Stanford Business Books.
- Ferreira, K.J. et al. (2015). *Analytics for an Online Retailer: Demand Forecasting and Price Optimization.* M&SOM.

## Links

**Modeling**
- [[03_modeling/01_supervised_learning/01_linear_and_glm/linear_and_glm|Linear and GLMs]] — demand curve estimation
- [[03_modeling/01_supervised_learning/02_tree_based_models/gradient_boosting|Gradient Boosting]] — demand modelling

**Reference Implementations**
- [[03_modeling/01_supervised_learning/02_tree_based_models/tree_ensembles_implementation|Tree Ensembles Implementation]]

**Adjacent Applications**
- [[inventory_optimization|Inventory Optimization]]
- [[07_applications/01_prediction_and_forecasting/demand_forecasting|Demand Forecasting]]
- [[07_applications/02_recommendation_and_ranking/product_recommendation|Product Recommendation]]
