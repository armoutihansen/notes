---
layer: 07_applications
type: application
status: growing
tags: [forecasting, tabular, workflow]
created: 2026-03-11
---

# Inventory Optimization

## Problem

Determine optimal stock levels, reorder points, and order quantities across a SKU-location hierarchy to minimise total cost: holding cost + stockout cost + ordering cost + waste (perishables). Inventory optimisation translates demand forecasts into replenishment decisions, accounting for lead times, demand uncertainty, supplier constraints, and service level objectives.

This is the **decision layer** downstream of demand forecasting — it translates a probabilistic forecast into an order quantity using inventory policy theory (EOQ, newsvendor, safety stock calculations).

## Users / Stakeholders

| Role | Decision |
|---|---|
| Supply chain planner | Approve/adjust system-generated order recommendations |
| Category buyer | Negotiate MOQ (minimum order quantities) and lead times |
| Warehouse manager | Capacity planning; slotting optimisation |
| Finance | Working capital target; inventory provision |
| Store manager | Maintain in-store availability; manage waste |

## Domain Context

- **Lead time uncertainty**: Supplier lead times vary. Safety stock must buffer demand variation AND lead time variation.
- **Perishability**: Fresh food, pharmaceuticals have shelf-life constraints. Waste cost can equal or exceed stockout cost. FEFO (First Expired, First Out) rotation is operational constraint.
- **MOQ constraints**: Suppliers have minimum order quantities that may exceed optimal reorder quantities. Integer programming required.
- **Multi-echelon**: Warehouse → DC → store. Stock allocation across echelons affects service levels. Multi-echelon optimisation is complex but high-value.
- **Seasonal demand**: Safety stock must increase ahead of seasonal peaks. Dynamic safety stock calculation required.
- **Data freshness**: Inventory positions must be current. Point-of-sale data and warehouse management system data latency directly impacts decision quality.

## Inputs and Outputs

**Inputs**:
```
Demand forecast: point forecast + prediction intervals (P10/P50/P90) per SKU per location
Current inventory: on_hand, on_order, in_transit, allocated
Supplier data: lead_time_mean, lead_time_std, MOQ, case_pack_size, cost
Policy parameters: service_level_target (e.g., 97.5%), holding_cost_rate, ordering_cost
Product attributes: shelf_life, units_per_case, weight, volume, supplier_id
```

**Output**:
```
reorder_point:    Inventory level that triggers a new order
order_quantity:   Recommended order quantity (EOQ-based or min-cost)
safety_stock:     Buffer stock recommendation vs current safety stock
expected_service_level: Predicted fill rate under recommended policy
days_of_cover:    Expected inventory coverage in days
action:           ORDER / HOLD / CANCEL_PENDING / EMERGENCY_ORDER
```

## Decision or Workflow Role

```
Daily batch run (after demand forecast refresh)
  ↓
Safety stock calculation: σ_demand × σ_lead_time × z(service_level)
  ↓
Reorder point: demand_during_lead_time + safety_stock
  ↓
For SKUs below reorder point:
  → Calculate EOQ: √(2 × annual_demand × ordering_cost / holding_cost)
  → Apply MOQ constraints
  → Generate purchase order recommendation
  ↓
Planner review: approve / adjust / reject (exception-based)
  ↓
Orders placed via ERP → supplier fulfils → inventory received
  ↓
Actual demand + actual replenishment → forecast accuracy feedback
```

## Modeling / System Options

| Component | Approach | Notes |
|---|---|---|
| Demand forecast | LightGBM / ARIMA (see [[demand_forecasting|Demand Forecasting]]) | Feeds into inventory model |
| Safety stock | Analytical (cycle service level formula) | z × σ_dL formula |
| Order quantity | EOQ + integer constraints | Classic operations research |
| Lead time model | Empirical distribution from supplier history | Non-parametric sufficient |
| Multi-echelon | Stochastic dynamic programming / simulation | High value; high complexity |

**Note**: Inventory optimisation is primarily an operations research problem once the demand forecast is available. ML is in the demand forecast; classical OR is in the replenishment decision.

## Deployment Constraints

- **Integration**: Must connect to ERP (SAP, Oracle) for inventory positions and order placement. Data latency from ERP affects decision quality.
- **Planner override**: System generates recommendations; planners override for supplier relationship reasons, promotion knowledge, etc. Override rate should be monitored.
- **Seasonal pre-build**: Safety stock must increase ahead of seasonal peaks. Requires forward-looking demand signal.

## Risks and Failure Modes

| Risk | Description | Mitigation |
|---|---|---|
| **Forecast error propagation** | Poor forecast → wrong reorder point → stockout/overstock | Forecast MASE monitoring; robust safety stock formula |
| **Lead time surprise** | Supplier fails to deliver → stockout | Emergency order trigger; lead time distribution update |
| **MOQ trap** | MOQ forces overorder of slow-moving SKU | MOQ negotiation; consignment stock agreements |
| **Seasonal under-stock** | Safety stock not increased for peak | Pre-peak safety stock review; seasonal uplift logic |

## Success Metrics

| Metric | Target | Notes |
|---|---|---|
| In-stock rate (fill rate) | > 97.5% | Service level KPI |
| Inventory turnover | Improvement vs prior year | Capital efficiency |
| Waste / expired stock | < 2% of sales (perishables) | Waste KPI |
| Working capital | -5–15% reduction | Finance KPI |
| Planner intervention rate | < 20% of orders | Automation quality |

## References

- Silver, E. et al. (2017). *Inventory and Production Management in Supply Chains.* CRC Press.
- Zipkin, P. (2000). *Foundations of Inventory Management.* McGraw-Hill.

## Links

**Modeling**
- [[03_modeling/05_time_series/index|Time Series Models]] — demand forecasting input
- [[03_modeling/01_supervised_learning/02_tree_based_models/gradient_boosting|Gradient Boosting]] — LightGBM demand forecast

**Reference Implementations**
- [[03_modeling/05_time_series/01_classical_forecasting/time_series_implementation|Time Series Implementation]]
- [[08_implementations/02_end_to_end_examples/demand_forecasting_pipeline|Demand Forecasting Pipeline]]

**Adjacent Applications**
- [[personalized_pricing|Personalized Pricing]]
- [[07_applications/01_prediction_and_forecasting/demand_forecasting|Demand Forecasting]]
