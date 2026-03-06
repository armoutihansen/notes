---
layer: 04_ml_engineering
type: engineering
tool: general
status: growing
tags: [observability, monitoring, metrics, logging, alerting, mlops, grafana]
created: 2026-03-05
---

# ML Observability

## Purpose

A deployed model is a live system. Without observability, it is a black box that may degrade silently — whether due to infrastructure issues, upstream data changes, or model quality degradation. ML observability extends classical software observability (metrics, logs, traces) with ML-specific signals: prediction distributions, feature health, and label feedback. The goal is to detect problems fast, diagnose their root cause, and support incident response.

## Architecture

### Operational Metrics

These mirror standard software SRE metrics and are prerequisites before adding any ML-specific instrumentation:

| Metric | Description | Typical SLO |
|---|---|---|
| **Latency p50/p95/p99** | Inference time at percentiles | p99 < 200 ms (user-facing) |
| **Throughput (RPS)** | Requests per second | Capacity planning target |
| **Error rate** | 5xx / total requests | < 0.1% |
| **Uptime / Availability** | Successful responses / total | ≥ 99.9% |
| **Queue depth** | Pending requests in async systems | < threshold |

Instrument via Prometheus client libraries (`prometheus_client` for Python). Expose a `/metrics` endpoint scraped by Prometheus; visualise in Grafana.

```python
from prometheus_client import Histogram, Counter
REQUEST_LATENCY = Histogram('inference_latency_seconds', 'Inference latency', buckets=[.01, .05, .1, .25, .5, 1])
REQUEST_COUNT = Counter('inference_requests_total', 'Total inference requests', ['status'])
```

### ML-Specific Metrics

**Prediction Distribution**
Track the histogram of model output scores or predicted class frequencies over time. A shift in the prediction distribution is often the earliest observable signal of concept drift or data quality issues — visible before labels arrive.

- For classifiers: mean prediction probability, fraction of predictions in each class bucket (e.g., 10 equal-width bins).
- For regressors: mean, variance, min, max, percentiles of predicted values.

**Feature Value Distributions**
For top-N features by importance, track summary statistics in each serving window:
- Continuous: mean, std, p5, p95, null rate, out-of-range rate.
- Categorical: frequency per category, unseen category rate.

Feature health checks catch upstream pipeline regressions before they affect predictions.

**Null / Missing Rate**
The fraction of requests where a feature arrives as null or outside expected range. Sudden spikes indicate upstream schema changes or ETL failures.

**Label Distribution (Online Feedback)**
When labels are available (even with delay) — e.g., whether a recommended item was clicked, whether a flagged transaction was confirmed as fraud — track the observed label distribution and compare it to training-time priors. Rapid divergence indicates concept or label drift.

## Implementation Notes

### Logging Strategy

**What to log** (per request):
- Request ID, timestamp, model version, serving environment.
- Input feature values (or a sample thereof).
- Model output (score / class / regression value).
- Latency breakdown (pre-processing, inference, post-processing).
- (When available, asynchronously) ground-truth label.

**Sampling**
Logging every request at high throughput is costly. Common strategies:
- **Random sampling**: Log 1–10% of requests uniformly. Inexpensive; captures distributional statistics but misses rare events.
- **Stratified sampling**: Oversample tail predictions (high-confidence anomalies, near-threshold outputs).
- **Full logging for flagged events**: Always log when an alert condition is triggered or when the model's output exceeds a business threshold.

**Storage costs**: Structured logs → Kafka → object store (S3/GCS) or columnar store (BigQuery, Snowflake) for downstream analysis. Budget ~100–500 bytes per logged request depending on feature count.

### Dashboards

**Grafana** is the standard for operational metrics. Key panels:
- Latency heatmap (p50/p99 over time).
- Prediction score distribution histogram (updated hourly or daily).
- Feature drift PSI per top-10 feature (bar chart, coloured by threshold).
- Error rate and null rate time series.

**Custom Streamlit dashboards**: Useful for exploratory monitoring — interactive slicing by segment, cohort, or time window. Not appropriate for production alerting (no SLO-backed uptime).

**ML Platform Dashboards**: MLflow, Vertex AI Model Monitoring, AWS SageMaker Model Monitor, Evidently AI provide pre-built monitoring UIs integrated with training/serving pipelines.

### Alerting Design

**Threshold-based alerts**
Simple, interpretable, low false-negative rate when thresholds are well-chosen. Example rules:
- `error_rate_5m > 1%` → page on-call immediately.
- `prediction_score_mean > 0.9 (sustained > 30 min)` → warn (possible data leak or model overfitting to recent distribution).
- `feature_null_rate{feature="income"} > 5%` → warn (upstream pipeline issue).

**Anomaly detection alerts**
Statistical baselines (mean ± k·std over rolling window) or ML-based anomaly detectors (Isolation Forest, EWMA control charts) reduce alert noise for slowly drifting metrics. Higher implementation cost; risk of false negatives during genuine drift events.

**Alert Fatigue**
The main failure mode of monitoring systems. Mitigations:
- Set alert thresholds at 3–5× normal variance, not at the first deviation.
- Require a sustained condition (e.g., 15-minute window) before paging.
- Group related alerts (feature drift + prediction drift → single "distribution shift" alert).
- Review and tune thresholds quarterly; retire alerts that consistently fire without producing incidents.

### Model Performance Degradation Signals

In the absence of ground-truth labels (the common case — labels are delayed), use **proxy metrics**:
- **Prediction confidence decay**: If the model's mean confidence (e.g., max softmax probability) decreases over time, it is encountering more ambiguous inputs.
- **Business outcome proxies**: Click-through rate, conversion rate, fraud callback rate — lagged but ground-truth signals. Track with a sliding 7-day window.
- **Reference model comparison**: Run the previous champion model on sampled production data; if its outputs diverge significantly from the new model, investigate.

### Incident Response Playbook for ML Systems

1. **Detection**: Alert fires (operational or ML metric threshold).
2. **Triage**: Is this a software/infrastructure issue (error spikes, latency regression) or an ML quality issue (drift, degraded predictions)?
3. **Scope**: Which features / segments / time ranges are affected? Query the feature log store.
4. **Immediate mitigation**: If software issue — rollback or scale up. If ML quality issue — assess severity; if critical, rollback to champion model.
5. **Root cause analysis**: Trace back through the data pipeline. Identify the change event (upstream schema change, world event, A/B test interaction).
6. **Resolution**: Fix data pipeline OR retrain model with updated data.
7. **Post-mortem**: Document the timeline, detection gap, and monitoring improvements to close the gap.

## Trade-offs

- **More metrics** → better coverage, higher storage and compute cost; prioritise ruthlessly.
- **Alerting sensitivity** trades off false positives (alert fatigue) against false negatives (missed degradations); tune conservatively at first.
- **Logging all features** enables richer debugging but increases storage cost linearly with feature count; use importance-based prioritisation.
- **Real-time dashboards** are expensive to maintain; balance with batch-computed overnight reports for non-critical signals.

## References

- Kleppmann, *Designing Data-Intensive Applications*, Ch. 10–11.
- Google SRE Book, Ch. 6 (Monitoring Distributed Systems).
- Sculley et al., "Hidden Technical Debt in ML Systems", *NeurIPS* 2015.
- Evidently AI, *ML Monitoring Guide*, evidentlyai.com.
- Prometheus & Grafana documentation.

## Links
- [[drift_detection|Drift Detection]]
- [[rollout_strategies|Model Rollout Strategies]]
- [[serving_patterns|Model Serving Patterns]]
