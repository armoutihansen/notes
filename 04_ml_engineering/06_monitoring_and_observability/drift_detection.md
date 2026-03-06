---
layer: 04_ml_engineering
type: engineering
tool: general
status: growing
tags: [drift, monitoring, statistics, data-quality, retraining, mlops]
created: 2026-03-05
---

# Drift Detection

## Purpose

A model trained on historical data is only as reliable as the assumption that the future resembles the past. When that assumption breaks — due to world events, system changes, or evolving user behaviour — model performance degrades silently unless a monitoring system detects it. Drift detection is the practice of identifying when the statistical properties of production data deviate meaningfully from the training distribution, triggering investigation or automated retraining.

## Architecture

### Why Drift Occurs

1. **World changes**: Underlying real-world patterns shift (economic conditions, user preferences, seasonal effects, regulatory changes).
2. **Data collection changes**: Upstream schema changes, sensor recalibration, ETL pipeline modifications, or third-party data provider updates alter the distribution of features.
3. **Label definition changes**: Business redefinition of the target variable (e.g., what constitutes a "conversion" or a "fraud" event) invalidates the training objective.

### Three Types of Drift

**Covariate Drift** — `P(X)` changes, `P(Y|X)` is stable.
The input feature distribution shifts but the relationship between features and labels remains the same. Model accuracy degrades because it encounters out-of-distribution inputs. Most commonly monitored type since features are always observed.

**Concept Drift** — `P(Y|X)` changes, `P(X)` may be stable.
The mapping from inputs to outputs changes. The world works differently than when the model was trained. Example: a fraud detection model trained pre-pandemic may not capture post-pandemic fraud patterns. Harder to detect without fresh labels.

**Label Drift** — `P(Y)` changes.
The marginal distribution of labels shifts (class imbalance changes). Example: sudden spike in positive fraud cases due to an organised attack. Detectable from prediction distributions as a proxy when labels are delayed.

## Implementation Notes

### Statistical Detection Methods

**Kolmogorov-Smirnov (KS) Test** — continuous features
Measures the maximum absolute difference between two empirical CDFs:
```
D = sup_x |F_ref(x) - F_prod(x)|
```
p-value < 0.05 indicates significant drift. Sensitive to location and shape shifts. Available via `scipy.stats.ks_2samp`.

**Chi-Squared Test** — categorical features
Tests whether the frequency distribution of categories differs between reference and production windows:
```
χ² = Σ (O_i - E_i)² / E_i
```
where O_i are observed production counts and E_i are expected counts from the reference distribution.

**Population Stability Index (PSI)** — widely used in financial services
Quantifies the shift in a variable's distribution:
```
PSI = Σ (P_prod,i - P_ref,i) · ln(P_prod,i / P_ref,i)
```
Computed over binned distributions (typically 10–20 equal-width or equal-frequency bins).

Interpretation thresholds:
- `PSI < 0.1`: No significant shift.
- `0.1 ≤ PSI < 0.2`: Moderate shift — investigate.
- `PSI ≥ 0.2`: Significant shift — action required.

**Model-Based Detection**
Train a binary classifier to distinguish reference data (label 0) from production data (label 1). If the classifier achieves high AUC (> 0.7–0.75), drift is present. Feature importances from this classifier indicate *which* features are drifting and by how much.

Monitor model confidence distributions: if the histogram of prediction scores flattens or shifts, the model is less certain — a signal of distribution shift even before labels arrive.

### Reference Window Strategies

- **Fixed reference window**: Training dataset or a fixed historical period (e.g., first month post-deployment). Simple, reproducible. May become stale as the legitimate distribution evolves.
- **Rolling reference window**: A sliding window of recent production data (e.g., prior 30 days). Adapts to gradual shifts but can mask slow drift — the reference window "chases" the drift.
- **Seasonal reference window**: Compare against the same period from the prior year. Best for highly seasonal domains (retail, advertising).

### Alerting Thresholds

Set per-feature thresholds based on feature importance: higher-importance features warrant tighter thresholds. A common practice:
- Monitor top-N features by SHAP importance.
- Alert on PSI ≥ 0.2 or KS p-value < 0.01 for any top-10 feature.
- Alert on prediction distribution PSI ≥ 0.15.

### Distinguishing Drift from Data Quality Issues

Not all distribution shifts are drift — upstream pipeline bugs produce identical statistical signatures:
- **Sudden spikes**: More likely a pipeline failure (null injection, schema change) than organic drift.
- **Single-feature shifts**: Investigate source system changes before triggering retraining.
- **Covariate shifts with no business explanation**: Cross-reference with data engineering team.

Standard practice: maintain a data quality layer (null rates, out-of-range checks, schema validation) that fires before drift alerts to separate the two concerns.

### Automated Retraining Triggers

Drift detection is an input to, not a substitute for, retraining decisions. A retraining trigger pipeline:
1. Drift alert fires (PSI threshold exceeded).
2. Human or automated check: is this drift or a data quality issue?
3. If drift confirmed, evaluate whether model performance has degraded (requires fresh labels or proxy metrics).
4. If degradation confirmed, trigger retraining job with updated data.
5. New model goes through rollout strategy (canary or shadow before promotion).

## Trade-offs

- **Statistical tests** (KS, PSI) are interpretable and fast but require choosing reference windows and thresholds carefully. They flag symptoms, not causes.
- **Model-based drift detection** is more powerful but adds a second model to maintain.
- **Monitoring all features** creates alert fatigue; prioritise by importance.
- **Concept drift** is the hardest to detect without labels, which are often delayed by hours to weeks in production systems.

## References

- Gama et al., "A Survey on Concept Drift Adaptation", *ACM Computing Surveys* 2014.
- Kleppmann, *Designing Data-Intensive Applications*, Ch. 11.
- Evidently AI documentation: `evidentlyai.com/docs`.
- Nannyml documentation (model performance without labels).

## Links
- [[ml_observability|ML Observability]]
- [[rollout_strategies|Model Rollout Strategies]]
- [[serving_patterns|Model Serving Patterns]]
- [[retraining_strategies|Retraining Strategies]]
- [[testing_in_production|Testing in Production]]
- [[offline_evaluation|Offline Evaluation]]
