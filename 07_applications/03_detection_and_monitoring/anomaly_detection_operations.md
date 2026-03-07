---
layer: 07_applications
type: application
status: growing
tags: [anomaly-detection, time-series, monitoring]
created: 2026-03-11
---

# Anomaly Detection — Operations

## Problem

Detect deviations from normal operating behaviour in infrastructure, manufacturing, or industrial systems — before they cause outages, production stoppages, or safety incidents. Unlike fraud detection, the adversary is the environment or system degradation, not a human actor. The signal is typically a time series (CPU utilisation, vibration amplitude, error rate, temperature) and anomalies manifest as point anomalies, contextual anomalies, or collective anomalies.

Key application domains:
- **IT / DevOps**: Server CPU/memory/latency spikes, error rate surges, cascading service failures
- **Manufacturing / IIoT**: Machine vibration, temperature, pressure out of specification
- **Predictive maintenance**: Equipment degradation signature preceding failure

## Users / Stakeholders

| Role | Decision |
|---|---|
| SRE / On-call engineer | Triage alert; decide to escalate or investigate |
| Plant operations manager | Stop/restart production line; schedule maintenance |
| Maintenance engineer | Plan unplanned vs preventive maintenance |
| Safety officer | Safety shutdown threshold triggering |
| Data centre manager | Capacity and cooling management |

## Domain Context

- **High alert volume**: Naive threshold-based alerting generates 100s of alerts/day. ML goal is to reduce false positive rate while preserving recall on true incidents.
- **No labelled anomalies (often)**: Labels are expensive. Outages are rare. Semi-supervised and unsupervised approaches dominate.
- **Contextual seasonality**: IT metrics are time-of-day seasonal. Anomaly relative to expected — not absolute — is the right framing.
- **Sensor data quality**: IoT sensors drop readings, drift over time, produce electrical noise. Data quality pipeline is as important as the model.
- **IIoT regulatory**: ISO 13381 (condition monitoring), ISO 55001 (asset management) for industrial environments. Safety-critical applications may require SIL-rated systems.
- **Streaming vs batch**: IT anomaly detection operates on streaming metrics (Prometheus). Manufacturing may allow batch (hourly scan).

## Inputs and Outputs

**IT / infrastructure**:
```
Metrics: cpu_util, mem_util, disk_io, net_throughput, request_rate, error_rate, latency_p99
Logs: error log count, log pattern change frequency
Traces: span count, span duration distribution
Metadata: service_name, host, cluster, deployment_version, time_of_day, day_of_week
```

**Manufacturing / IIoT**:
```
Sensors: vibration_rms, temperature, pressure, current_draw, rpm
Process: production_rate, cycle_time, reject_count
Metadata: machine_id, shift, product_type, time_since_last_maintenance
```

**Output**:
```
anomaly_score:    continuous ∈ [0, 1] or standard deviations from expected
anomaly_flag:     NORMAL / WARNING / CRITICAL
contributing_sensors: which signals are most anomalous (explanation)
incident_ticket:  auto-created in PagerDuty / Jira with context
```

## Decision or Workflow Role

```
Streaming metrics ingest (Prometheus / Kafka / MQTT)
  ↓
Feature engineering: rolling stats, trend, seasonality residuals
  ↓
Anomaly model: Isolation Forest / LSTM-AE / STL + z-score
  ↓
Score thresholding → alert decision
  ↓
Alert routing: PagerDuty / OpsGenie / Slack
  ↓
On-call engineer reviews → confirm/dismiss → label fed back
  ↓
Model retraining on confirmed anomaly labels (semi-supervised)
```

## Modeling / System Options

| Approach | Strength | Weakness | When to use |
|---|---|---|---|
| Statistical: z-score / IQR | Simple; no training needed | No context; misses seasonal normal | Baseline; single stationary metrics |
| STL decomposition + residual | Handles seasonality automatically | Requires consistent season length | Seasonal IT metrics (daily/weekly) |
| Isolation Forest | Unsupervised; handles multivariate; fast | Not time-aware; no sequential structure | Multivariate snapshot anomalies |
| LSTM Autoencoder | Captures temporal patterns; semi-supervised | Training complexity; threshold tuning | Sequential data; enough history |
| Prophet (Facebook) | Automatic seasonality + holiday; human-interpretable | Slow for many metrics | Business metrics with known calendars |
| ARIMA/SARIMA + residual | Well-understood; interpretable | Univariate; brittle at sudden regime change | Single stationary time series |

**Recommended**: STL + z-score for individual IT metrics. Isolation Forest for multivariate equipment sensor data. LSTM Autoencoder when history is rich (>3 months) and false positives are costly.

## Deployment Constraints

- **Latency**: Alert should fire within 1–5 minutes of anomaly onset for IT. Manufacturing may tolerate hourly batch.
- **Alert precision**: Every spurious alert erodes on-call trust. Aim for >50% alert precision (positive predictive value). Precision is more important than recall at high alert volumes.
- **Explainability**: On-call engineer needs to know *which metric* triggered and *what is abnormal about it*. Black-box score without context is not actionable.
- **Scale**: Large-scale deployments: 100K+ metrics time series. Need efficient batch inference. Per-metric models don't scale — use shared model with metric embeddings.
- **Feedback loop**: On-call dismiss/confirm actions are high-value labels. Build a feedback capture UI into alert tooling.

## Risks and Failure Modes

| Risk | Description | Mitigation |
|---|---|---|
| **Alert fatigue** | Too many false positives → engineers stop responding | Tune for precision; alert grouping; noise suppression |
| **Missed gradual degradation** | Slow drift not caught by point anomaly detector | Trend alerting; change point detection |
| **Sensor failure confusion** | Sensor dropout flagged as anomaly | Sensor health check; separate handling |
| **Model trained on anomalous baseline** | Training data includes anomalies → normal reference contaminated | Clean training data; unsupervised validation |
| **Distribution shift** | System upgrades change normal patterns → false alarms spike | Retraining triggers on deployment events |

## Success Metrics

| Metric | Target | Notes |
|---|---|---|
| Alert precision | > 50% | Fraction of alerts that are true anomalies |
| MTTD (Mean Time to Detect) | < 5 minutes | Operational SLA for IT |
| Alert volume reduction | > 60% vs threshold-based | vs naive threshold alerting baseline |
| MTTR (Mean Time to Resolve) | Decrease by 20% | Downstream operational impact |
| False negative rate | < 5% for P1 incidents | Safety-critical threshold |
| Maintenance cost reduction | > 10% YoY | For predictive maintenance applications |

## References

- Chandola, V. et al. (2009). *Anomaly Detection: A Survey.* ACM Computing Surveys.
- Tatbul, N. et al. (2018). *Precision and Recall for Time Series.* NeurIPS.

## Links

**Modeling**
- [[03_modeling/02_unsupervised_learning/index|Unsupervised Learning]] — Isolation Forest, autoencoders, clustering
- [[03_modeling/05_time_series/index|Time Series Models]] — STL decomposition, ARIMA residuals

**ML Engineering**
- [[05_ml_engineering/07_monitoring_and_observability/index|Monitoring and Observability]] — production monitoring architecture
- [[05_ml_engineering/01_data_engineering/index|Data Engineering]] — streaming data pipelines

**Reference Implementations**
- [[03_modeling/02_unsupervised_learning/01_clustering/unsupervised_learning_implementation|Unsupervised Learning Implementation]]
- [[08_implementations/01_system_patterns/model_monitoring_system|Model Monitoring System]]
- [[08_implementations/02_end_to_end_examples/anomaly_detection_pipeline|Anomaly Detection Pipeline]]

**Adjacent Applications**
- [[fraud_detection|Fraud Detection]]
- [[07_applications/08_domain_verticals/05_mobility/predictive_maintenance|Predictive Maintenance]]
- [[07_applications/08_domain_verticals/06_operations/quality_control_vision|Quality Control Vision]]
