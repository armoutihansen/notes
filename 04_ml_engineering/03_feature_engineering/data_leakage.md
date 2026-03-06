---
layer: 04_ml_engineering
type: engineering
tool: general
status: growing
tags: [data-leakage, validation, preprocessing, reliability]
created: 2026-03-05
---

# Data Leakage

## Purpose

Data leakage is the introduction of information about the target variable — or about the future — into the model's training inputs in a way that will not be available at prediction time. It produces models that appear highly accurate during development but fail silently in production because the favorable signal disappears. Preventing leakage is one of the most critical reliability concerns in ML engineering, and diagnosing it is often non-trivial.

## Architecture

### Two Primary Types

**1. Target leakage** occurs when a feature used at training time is causally downstream of the target, or is directly derived from it. Examples:

- Including `had_claim = 1` as a feature when predicting insurance claim probability, where this column is populated *after* the claim occurs.
- Using a patient's discharge diagnosis code to predict readmission risk, when the code is assigned at discharge — not at admission when the model runs.
- Using an aggregated column (e.g., "total spend this month") that was computed with the current transaction included.

**2. Train-test contamination** occurs when information from the test set influences the training process, most commonly through preprocessing steps applied before the train/test split:

- Fitting a scaler (min-max, standardization) on the full dataset, so test-set statistics inform training normalization.
- Imputing missing values using the global mean, which is computed over both train and test.
- Performing feature selection or dimensionality reduction (PCA, mutual information ranking) on the full dataset before splitting.
- Computing target encoding on the full dataset: the test-set target labels implicitly influence encoding statistics.

### Common Real-World Examples

| Scenario | Leaking Feature | Reason |
|---|---|---|
| Credit risk | Payment status at loan close | Only known after outcome |
| Fraud detection | Manual review flag | Flagging happens after suspicious activity |
| Churn prediction | Support ticket count (future) | Tickets filed after churn decision |
| Medical diagnosis | Lab result ordered due to suspected diagnosis | Result available post-suspicion |
| Sales forecasting | Final monthly total used in rolling average | Includes data from future within the window |

## Implementation Notes

### Detection Methods

- **Suspiciously high accuracy**: AUC > 0.99 on a hard real-world problem is a red flag. Cross-validate and verify that performance holds on a completely held-out temporal slice.
- **Feature importance analysis**: if a feature appears disproportionately important relative to domain expectations, investigate its construction. A feature with near-zero variance but very high importance often signals leakage.
- **Temporal validation**: for time-series data, evaluate the model on future data that was strictly unavailable during training. Performance degradation compared to shuffled CV indicates temporal leakage.
- **Correlation with target at inference time**: check whether any feature has a suspiciously high Pearson or Spearman correlation with the target label; investigate the causal direction.
- **Data provenance audit**: trace the ETL pipeline for each feature back to its source and timestamp. Confirm that all feature values are available at the time the model would run in production.

### Prevention Strategies

- **Strict temporal splits**: for any sequential data, split by time, not randomly. Training data must precede validation and test data chronologically. Use time-based cross-validation (`TimeSeriesSplit` in scikit-learn).
- **Pipeline-based preprocessing**: use `sklearn.Pipeline` to ensure that all fit-dependent transformations (scalers, encoders, imputers) are fit *only* on the training fold and applied to the test fold. Never call `.fit_transform()` on the combined dataset.
- **Holdout set quarantine**: set aside a final holdout set before any exploratory analysis begins. Touch it exactly once for the final model evaluation.
- **Feature timestamp audit**: document the generation time of every feature in a feature store or metadata catalog. Flag any feature whose generation time is after the label generation time.
- **Leave-one-out target encoding**: when using target encoding, compute the encoding inside each cross-validation fold using only the training portion of that fold.

## Trade-offs

Pipeline-based preprocessing adds boilerplate and requires careful integration with cross-validation libraries. The overhead is worth it: it is the only reliable way to prevent contamination across folds. Teams that skip pipelines and manually split data frequently introduce subtle leakage that goes undetected until production. The cost of discovering leakage post-deployment — model replacement, loss of stakeholder trust, potentially harmful decisions based on inflated performance estimates — far exceeds the upfront investment in disciplined preprocessing workflows.

## References

- Kaufman et al., "Leakage in Data Mining: Formalization, Detection, and Avoidance" (2012)
- scikit-learn: `Pipeline`, `TimeSeriesSplit`
- Kaggle: "Data Leakage" (feature engineering guide)

## Links
- [[feature_engineering_patterns|Feature Engineering Patterns]]
- [[offline_evaluation|Offline Evaluation]]
- [[experiment_tracking|Experiment Tracking]]
- [[ml_lifecycle|ML Lifecycle]]
