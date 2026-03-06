---
layer: 07_applications
type: application
status: growing
tags: [workflow, forecasting, tabular]
created: 2026-03-11
---

# Route Optimization

## Problem

Find the lowest-cost sequence of stops for a fleet of vehicles to fulfil a set of deliveries or pickups, subject to constraints: vehicle capacity, time windows, driver hours, toll road preferences, and real-time traffic. The Capacitated Vehicle Routing Problem (CVRP) is NP-hard — exact solutions are infeasible for large instances. ML enhances classical operations research (OR) solvers with learned cost predictions, traffic forecasting, and demand forecasting.

## Users / Stakeholders

| Role | Decision |
|---|---|
| Route planner | Review and approve generated routes |
| Driver | Execute route; handle exceptions |
| Fleet manager | Vehicle utilisation; fuel costs |
| Customer | Delivery time window accuracy |
| Operations manager | Cost per delivery; on-time performance |

## Domain Context

- **OR + ML hybrid**: Route optimisation is fundamentally an OR problem (VRP solver). ML contributes: travel time prediction, demand forecasting (how many deliveries tomorrow?), and learning-based improvement heuristics.
- **Real-time traffic**: Static route plans are suboptimal in congested networks. Real-time traffic data (HERE, Google Maps API, proprietary sensors) enables dynamic re-routing.
- **Last-mile economics**: Last-mile delivery is 50–60% of total logistics cost. Dense urban delivery (grocery, parcel) and sparse rural delivery have very different optimal strategies.
- **EV fleet constraints**: Battery range, charging station availability add complex constraints to route planning.
- **Regulatory**: Driving hours (EU Regulation 561/2006 tachograph rules), weight restrictions, low-emission zones (ULEZ). Hard constraints that must not be violated.

## Inputs and Outputs

**Inputs**:
```
Orders: delivery_id, location (lat/lng), time_window_start, time_window_end, volume_kg
Vehicles: vehicle_id, capacity_kg, start_depot, end_depot, driver_id
Traffic: real-time travel time matrix (OSRM / HERE API)
Constraints: max_drive_hours, toll_avoidance, vehicle_restrictions_by_zone
```

**Output**:
```
routes: [
  {vehicle_id, ordered_stop_sequence, estimated_arrival_times, total_distance_km}
]
ETA_per_delivery: customer-visible expected arrival time
utilisation_metrics: vehicle_load_factor, km_per_delivery, cost_per_km
```

## Decision or Workflow Role

```
Day-before planning run (22:00–00:00)
  ↓
Demand forecast: predict unconfirmed orders for next day
  ↓
VRP solver (Google OR-Tools / VROOM / commercial: Routific)
  → Objective: minimise total distance or cost
  → Constraints: capacity, time windows, driver hours
  ↓
Routes exported to driver apps
  ↓
Morning: confirmed orders loaded; re-solve if material changes
  ↓
Real-time: traffic re-routing at junction-level
  ↓
Outcome: actual times logged vs predicted → model retraining
```

## Modeling / System Options

| Component | Approach | Notes |
|---|---|---|
| VRP solver | Google OR-Tools (open-source), VROOM, commercial | Core solver; heuristic approach |
| Travel time | OSRM (open), HERE API, learned model on historical GPS | Historical GPS beats map API for repeated routes |
| Demand forecast | LightGBM (daily orders by zone) | Input to vehicle requirement forecasting |
| Learning-based heuristics | Attention/pointer networks | Research; not yet production standard for most |

## Deployment Constraints

- **Planning horizon**: Day-before routes generated overnight. Near-real-time re-routing for exceptions.
- **Driver app integration**: Routes must be compatible with navigation apps (Waze, Google Maps, custom).
- **Manual override**: Drivers know local conditions (access issues, parking) that the model doesn't. Override mechanism required with outcome logging.

## Risks and Failure Modes

| Risk | Description | Mitigation |
|---|---|---|
| **Traffic prediction error** | Major congestion → ETAs wrong → failed deliveries | Real-time traffic update; conservative time window buffers |
| **Demand over-forecast** | Too many vehicles dispatched | Demand model accuracy monitoring |
| **Regulatory violation** | Route exceeds driving hours → fine/risk | Hard constraint in solver; tachograph monitoring |
| **Last-minute order changes** | Orders added/cancelled after route creation | Re-solve trigger threshold; soft insertion |

## Success Metrics

| Metric | Target | Notes |
|---|---|---|
| On-time delivery rate | > 95% | Customer SLA |
| Cost per delivery | -5–15% vs manual routing | Cost efficiency |
| Vehicle utilisation | > 85% capacity | Asset efficiency |
| km per delivery | Reduction vs baseline | Environmental metric |
| Failed delivery rate | < 2% | Last attempt failures |

## Links

**Modeling**
- [[03_modeling/01_supervised_learning/02_tree_based_models/gradient_boosting|Gradient Boosting]] — demand forecasting for vehicle capacity planning
- [[03_modeling/05_time_series/index|Time Series Models]] — demand forecasting input

**Adjacent Applications**
- [[predictive_maintenance|Predictive Maintenance]]
- [[07_applications/01_prediction_and_forecasting/demand_forecasting|Demand Forecasting]]
- [[07_applications/08_domain_verticals/06_operations/capacity_planning|Capacity Planning]]
