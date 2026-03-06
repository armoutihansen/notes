---
layer: 07_applications
type: application
status: growing
tags: [forecasting, time-series, anomaly-detection]
created: 2026-03-11
---

# Predictive Maintenance

## Problem

Predict when a vehicle, machine, or piece of equipment is likely to fail so that maintenance can be scheduled before the failure occurs — avoiding unplanned downtime, safety incidents, and emergency repair costs. Predictive maintenance (PdM) sits between reactive maintenance (fix after failure) and preventive maintenance (replace on fixed schedule), using sensor data and ML to identify degradation signatures before failure.

## Users / Stakeholders

| Role | Decision |
|---|---|
| Fleet manager | Schedule maintenance; vehicle availability planning |
| Maintenance engineer | Plan parts and labour for upcoming interventions |
| Operations director | Total cost of ownership; fleet reliability KPIs |
| Safety officer | Prevent safety-critical failures |
| Customer (vehicle owner) | Warranty claims; service scheduling |

## Domain Context

- **Failure rarity**: Equipment failures are rare events. Data is highly imbalanced and often there are very few historical failure examples for rare failure modes.
- **Sensor data streams**: OBD-II (automotive), CAN bus, IoT sensors provide continuous telemetry. Volume is high; relevant signal extraction is critical.
- **Right-censoring**: Vehicles in service have not yet failed — their time to failure is unknown. Survival analysis approaches handle censoring properly.
- **Maintenance records**: Planned maintenance changes the degradation trajectory. Must condition on maintenance history to avoid confounding.
- **Physical domain knowledge**: Vibration frequency analysis (FFT) for bearing wear, temperature trends for electrical components, oil viscosity for engine health — physical models inform feature engineering.
- **Edge deployment**: Vehicles are mobile. Inference may need to run on-vehicle (embedded system, ECU) or at fleet depot.

## Inputs and Outputs

**Telemetry features**:
```
Engine: oil_temperature, coolant_temperature, RPM, torque, fuel_consumption
Transmission: gear_shifts_per_km, clutch_slip_events, gear_oil_temperature
Brakes: brake_pad_wear_sensor, brake_temperature, ABS_events_per_km
Electrical: battery_voltage, alternator_current, fault_codes_count
Usage: mileage, hours_operated, load_factor, terrain_type
Maintenance: days_since_last_service, km_since_oil_change
```

**Output**:
```
failure_probability_30d:  P(failure within 30 days) ∈ [0, 1]
rul_estimate:             Remaining Useful Life in days or km
failure_mode:             Component most likely to fail
maintenance_urgency:      SCHEDULE_NEXT_WINDOW / SCHEDULE_WITHIN_7D / URGENT / CRITICAL
recommended_action:       Oil change, brake pad replacement, battery replacement, etc.
```

## Decision or Workflow Role

```
Telemetry ingested (real-time OBD-II stream or daily sync)
  ↓
Feature engineering: rolling statistics, FFT features, deviation from fleet average
  ↓
Failure probability model: P(failure within 30/60/90 days)
  ↓
CRITICAL / URGENT → fleet manager alert → immediate scheduling
SCHEDULE_7D → added to maintenance planner queue
NORMAL → next scheduled service
  ↓
Maintenance performed → maintenance record logged → model update
  ↓
Failure events (if they occur) → confirmed label → retraining
```

## Modeling / System Options

| Approach | Strength | Weakness | When to use |
|---|---|---|---|
| Threshold rules on sensor values | Simple; no training; immediate | Fragile; misses gradual degradation | Baseline; safety-critical hard limits |
| LightGBM on rolling features | Handles tabular sensor features; interpretable | Time-unaware; point-in-time snapshot | Most maintenance prediction use cases |
| LSTM Autoencoder anomaly detection | Detects novel degradation patterns | Threshold calibration; limited labels | Limited failure labels; novel failure mode detection |
| Survival analysis (Cox PH, AFT) | Handles censoring; estimates time-to-failure | Proportional hazards assumption | Time-to-failure estimation |
| Physics-informed ML | Incorporates domain knowledge; extrapolates beyond data | Domain expertise required | Safety-critical with physics models available |

**Recommended**: LightGBM for failure probability classification. Cox PH for remaining useful life estimation. Threshold rules as safety net.

## Deployment Constraints

- **Edge vs cloud**: Fleet vehicles may be offline. Critical alerts must be triggered locally. Batch sync when connected.
- **Latency**: Non-critical predictions: daily batch. Safety-critical: real-time (engine temperature overheat → immediate).
- **Sensor calibration**: Sensor drift over time causes model performance degradation. Periodic sensor calibration checks needed.
- **False alarm cost**: Unnecessary maintenance = wasted cost + vehicle downtime. Balance with failure cost.

## Risks and Failure Modes

| Risk | Description | Mitigation |
|---|---|---|
| **Rare failure events** | Few historical failures → poor model training | Transfer learning; fleet pooling; synthetic augmentation |
| **Sensor failure** | Sensor malfunctions → missing data → incorrect prediction | Sensor health monitoring; missing value imputation |
| **Failure mode shift** | New vehicle generation has different failure patterns | Model per vehicle generation; retraining on new fleet |
| **Planned maintenance confounding** | Maintenance resets degradation clock; model unaware | Include maintenance event features |

## Success Metrics

| Metric | Target | Notes |
|---|---|---|
| Unplanned downtime reduction | > 20% | Primary business metric |
| Failure detection recall | > 85% | Fraction of failures predicted before occurrence |
| False alarm rate | < 15% | Unnecessary maintenance triggers |
| Maintenance cost reduction | > 10% | Labour + parts savings |
| Vehicle availability | > 95% | Fleet utilisation |

## References

- Saxena, A. et al. (2008). *Damage Propagation Modeling for Aircraft Engine Run-to-Failure Simulation.* ICSN.
- Carvalho, T. et al. (2019). *A systematic literature review of machine learning methods applied to predictive maintenance.* Computers & Industrial Engineering.

## Links

**Modeling**
- [[03_modeling/05_time_series/index|Time Series Models]] — temporal degradation patterns
- [[03_modeling/02_unsupervised_learning/index|Unsupervised Learning]] — anomaly detection

**Reference Implementations**
- [[08_reference_implementations/01_model_implementations/time_series_implementation|Time Series Implementation]]
- [[08_reference_implementations/01_model_implementations/unsupervised_learning_implementation|Unsupervised Learning Implementation]]
- [[08_reference_implementations/03_end_to_end_examples/anomaly_detection_pipeline|Anomaly Detection Pipeline]]

**Adjacent Applications**
- [[route_optimization|Route Optimization]]
- [[07_applications/03_detection_and_monitoring/anomaly_detection_operations|Anomaly Detection — Operations]]
- [[07_applications/08_domain_verticals/06_operations/quality_control_vision|Quality Control Vision]]
