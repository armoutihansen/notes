---
layer: 04_ml_engineering
type: engineering
tool: general
status: growing
tags: [evaluation, metrics, calibration, offline-evaluation, validation]
created: 2026-03-05
---

# Offline Evaluation

## Purpose

Offline evaluation estimates how well a trained model will perform in production using data that was not used for training. It is the primary gate before a model is deployed: a model must demonstrate acceptable offline performance before incurring the cost and risk of a live rollout. However, offline metrics are proxies — they are only as reliable as the degree to which historical held-out data reflects the future production distribution. Understanding the gap between offline and online performance is as important as computing the metrics themselves.

## Architecture

### Evaluation Methodology

**Train / validation / test splits**

The canonical split reserves a test set that is used exactly once to report final metrics. The validation set is used for hyperparameter tuning, early stopping, and model selection. Typical splits: 70/15/15 or 80/10/10 by row count, though the right balance depends on dataset size. Never report test-set metrics during iterative development — doing so leaks test information into model selection decisions.

**Temporal splits for time-series data**

Random splits are inappropriate for sequential data because they allow future information to inform predictions of the past. A temporal split assigns the earliest observations to training, the next to validation, and the latest to testing. Walk-forward cross-validation (expanding window or sliding window) provides more robust estimates by evaluating the model repeatedly at different points in time. See [[data_leakage|Data Leakage]] for the full treatment of temporal contamination.

**K-fold cross-validation**

For small datasets, k-fold CV (typically k=5 or k=10) provides lower-variance performance estimates than a single split by averaging across k held-out folds. Use `StratifiedKFold` for classification with imbalanced classes to ensure each fold has the same class distribution.

### Metrics

**Classification**

- **AUC-ROC** (Area Under the ROC Curve): probability that the model ranks a random positive example above a random negative example. Threshold-independent; useful when class balance is expected to change. Values range from 0.5 (random) to 1.0 (perfect).
- **AUC-PR** (Area Under the Precision-Recall Curve): more informative than ROC when the positive class is rare (imbalanced data). Plotted as precision vs. recall across thresholds.
- **F1 score**: harmonic mean of precision and recall at a chosen threshold: $F1 = 2 \cdot \frac{P \cdot R}{P + R}$. Use F1 when false positives and false negatives have similar costs.
- **Log loss (cross-entropy)**: measures the quality of probability estimates, not just ranking. Penalizes confident wrong predictions heavily. Essential for evaluating calibration.

**Regression**

- **RMSE** (Root Mean Squared Error): $\sqrt{\frac{1}{n}\sum(y_i - \hat{y}_i)^2}$ — penalizes large errors quadratically; sensitive to outliers. Report in the units of the target variable.
- **MAE** (Mean Absolute Error): $\frac{1}{n}\sum|y_i - \hat{y}_i|$ — more robust to outliers than RMSE; easier to interpret as the average absolute error.
- **MAPE** (Mean Absolute Percentage Error): $\frac{100\%}{n}\sum\left|\frac{y_i - \hat{y}_i}{y_i}\right|$ — scale-independent; undefined when $y_i = 0$.
- **R²** (coefficient of determination): fraction of variance explained by the model; 1.0 is perfect, 0.0 is a constant-prediction baseline.

### Calibration

Calibration measures whether predicted probabilities reflect true frequencies. A model that outputs 0.8 for a set of examples should be correct ~80% of the time. Poor calibration is common even for high-AUC models.

- **Reliability diagram**: bin predictions by probability; plot mean predicted probability vs. observed frequency per bin. A well-calibrated model follows the diagonal.
- **Expected Calibration Error (ECE)**: weighted average of the gap between predicted confidence and observed accuracy across bins.
- **Platt scaling**: fits a logistic regression on top of model outputs using a validation set. Simple and effective for binary classifiers.
- **Isotonic regression**: non-parametric monotone calibration; more flexible than Platt scaling but requires more data.
- **Temperature scaling**: a single scalar $T$ divides logits before softmax: $\hat{p} = \text{softmax}(z / T)$. The standard calibration method for neural networks.

## Implementation Notes

- Use `sklearn.calibration.CalibratedClassifierCV` for Platt scaling or isotonic regression on sklearn-compatible models.
- Compute calibration on a held-out calibration set (distinct from the validation set used for hyperparameter tuning) to avoid overfitting the calibration.
- Always report confidence intervals for key metrics, especially on small test sets. Bootstrap resampling provides non-parametric CIs.
- Store evaluation results as MLflow artifacts to enable retrospective comparisons. See [[experiment_tracking|Experiment Tracking]].

### Evaluation on Data Slices (Subgroup Analysis)

Aggregate metrics can obscure poor performance on specific subpopulations. **Slice-based evaluation** computes metrics separately for each meaningful subgroup (e.g., by geography, device type, user segment, demographic category). This is essential for:

- Detecting fairness issues (disparate impact across protected groups).
- Finding data quality problems in specific segments.
- Identifying regions where the model underperforms and needs more training data.

Tools: `sliceformers`, Google's What-If Tool, `fairlearn`, or manual pandas groupby operations.

### Baseline Comparison

Every model evaluation must compare against a meaningful baseline:
- **Constant predictor**: always predict the majority class or mean value.
- **Simple heuristic**: rule-based model or business logic currently in production.
- **Previous model version**: the model being replaced.

A model that does not outperform the heuristic baseline on the test set should not be deployed.

### Error Analysis Workflow

1. Identify examples with the highest loss or most confident errors.
2. Cluster errors by type (e.g., FP vs. FN; error magnitude buckets).
3. Identify patterns: specific feature combinations, data quality issues, label errors.
4. Use findings to guide data collection, feature engineering, or label correction — then retrain.

## Trade-offs

### The Offline-Online Gap

Offline evaluation is a necessary but insufficient condition for production readiness. The gap between offline and online metrics arises from:

- **Distribution shift**: the test set reflects historical data; the model is deployed into a future that may differ (concept drift, demographic shifts, adversarial inputs). See [[drift_detection|Drift Detection]].
- **Feedback loops**: the model's predictions influence future data (e.g., recommendation systems that filter what users see, changing what future training data looks like).
- **Metric misalignment**: the offline metric is a proxy for the business objective. A model with higher AUC may not produce higher revenue if the decision threshold or the cost structure is not reflected in the metric.
- **Infrastructure differences**: serving latency, caching, and A/B test traffic splitting can all affect which model version is invoked, making offline-to-online comparison non-trivial.

The practical implication: offline evaluation gates deployment readiness; online A/B testing or shadow deployment verifies actual production impact.

## References

- Guo et al., "On Calibration of Modern Neural Networks" (ICML 2017)
- Sculley et al., "Hidden Technical Debt in Machine Learning Systems" (NeurIPS 2015)
- scikit-learn: `sklearn.metrics`, `sklearn.calibration`, `sklearn.model_selection`
- Hardt et al., "Equality of Opportunity in Supervised Learning" (NeurIPS 2016)

## Links
- [[experiment_tracking|Experiment Tracking]]
- [[data_leakage|Data Leakage]]
- [[drift_detection|Drift Detection]]
- [[ml_lifecycle|ML Lifecycle]]
- [[feature_engineering_patterns|Feature Engineering Patterns]]
