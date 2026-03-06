---
layer: 02_modeling
type: concept
status: growing
tags: [cross-validation, metrics, calibration, model-selection, roc, auc, f1, precision, recall]
created: 2026-03-06
---

# Evaluation and Validation

## Definition

The set of techniques for measuring model performance on held-out data, selecting between competing models, and assessing calibration, using principled data splitting strategies to obtain unbiased estimates.

## Intuition

A model that memorises training data scores perfectly in-sample but fails in production. Evaluation is about measuring how well the model will generalise to new, unseen data. The choice of splitting strategy and metric must match the deployment context — what matters in production determines what you should optimise and report.

## Formal Description

### Data Splitting Strategies

**Simple hold-out:** 80/20 or 70/30 split. Fast but high variance for small datasets.

**K-Fold Cross-Validation:**

```
Split data into K equal folds.
For each fold k:
    Train on all folds except k
    Evaluate on fold k
Report mean and std of K scores
```

$k=5$ and $k=10$ are standard. Provides $K \times$ more evaluation data than hold-out; more stable estimates.

**Stratified K-Fold:** preserves class proportions in each fold; essential for imbalanced classification.

**Temporal / Walk-forward CV:** for time series, folds must respect time order — no future data in training:

```
Fold 1: Train [t=0..t100],  Test [t=101..t110]
Fold 2: Train [t=0..t110],  Test [t=111..t120]
...
```

**Nested CV:** outer loop for model selection, inner loop for hyperparameter tuning — avoids optimistic bias from tuning on the test set.

### Classification Metrics

**Confusion matrix:** TP, FP, TN, FN.

$$\text{Precision} = \frac{TP}{TP+FP}, \quad \text{Recall} = \frac{TP}{TP+FN}$$

$$\text{F1} = \frac{2 \cdot \text{Precision} \cdot \text{Recall}}{\text{Precision} + \text{Recall}}, \quad F_\beta = \frac{(1+\beta^2)\text{PR}}{{\beta^2 P + R}}$$

Use $F_\beta > 1$ to prioritise recall (e.g., disease detection); $F_\beta < 1$ to prioritise precision (e.g., spam filtering).

**ROC-AUC:** area under the TPR vs FPR curve; threshold-invariant. Equal to the probability that a randomly chosen positive ranks higher than a random negative.

**PR-AUC (Average Precision):** area under the precision–recall curve. More informative than ROC-AUC for imbalanced datasets (many negatives inflate AUC).

**Cohen's Kappa:** agreement beyond chance; useful when class distributions vary across datasets.

### Regression Metrics

| Metric           | Formula                                   | When to use                          |     |                                         |     |                                        |
| ---------------- | ----------------------------------------- | ------------------------------------ | --- | --------------------------------------- | --- | -------------------------------------- |
| MAE              | $\frac{1}{n}\sum                          | y_i - \hat{y}_i                      | $   | Robust to outliers, interpretable units |     |                                        |
| RMSE             | $\sqrt{\frac{1}{n}\sum(y_i-\hat{y}_i)^2}$ | Penalises large errors, same units   |     |                                         |     |                                        |
| $R^2$            | $1 - \text{SS}_{res}/\text{SS}_{tot}$     | Proportion of variance explained     |     |                                         |     |                                        |
| MAPE             | $\frac{100}{n}\sum                        | y_i-\hat{y}_i                        | /   | y_i                                     | $   | Scale-free; avoid when $y_i \approx 0$ |
| Tweedie deviance | Distribution-specific                     | Compound Poisson targets (insurance) |     |                                         |     |                                        |

### Calibration

A model is **calibrated** if its predicted probabilities match empirical frequencies: among all instances where $\hat{p} = 0.7$, 70% should be positive.

**Reliability diagram (calibration curve):**

```python
from sklearn.calibration import calibration_curve, CalibrationDisplay
CalibrationDisplay.from_estimator(model, X_test, y_test, n_bins=10)
```

**Expected Calibration Error (ECE):** weighted mean absolute difference between predicted and empirical probability across bins.

**Recalibration:**
- **Platt scaling:** fit logistic regression on $\hat{f}(x_i)$ vs $y_i$.
- **Isotonic regression:** monotonic non-parametric fit; can overfit with small validation sets.

### Model Selection

**AIC** (Akaike): $-2\ln\hat{L} + 2k$ — rewards goodness of fit, penalizes parameters.

**BIC** (Bayesian): $-2\ln\hat{L} + k\ln n$ — stronger penalty; consistent for model selection.

**Bias-variance trade-off view:** simpler models have higher bias, lower variance; favor simpler models when data is scarce.

**Statistical comparison:** use paired t-test or Wilcoxon signed-rank test on cross-validation fold scores; never compare single-split scores without uncertainty quantification.

## Applications

- Selecting between a logistic regression and a gradient boosting model using 5-fold CV
- Recalibrating an XGBoost classifier for a use case requiring well-calibrated probabilities (e.g., pricing)
- Walk-forward evaluation for insurance renewal propensity model

## Trade-offs

- **Leaky evaluation:** including future data or target information during preprocessing inside CV folds inflates performance estimates; always build preprocessing + model into a single Pipeline fitted inside the fold.
- **Single metric pitfalls:** optimizing for AUC can give a model with bad precision at the operating threshold; always inspect the full confusion matrix at the deployment threshold.
- **Nested CV overhead:** $k_{outer} \times k_{inner}$ model fits; often only needed when the dataset is small.

## Links

- [[01_foundations/05_statistical_learning_theory/bias_variance_analysis|Bias–Variance Analysis]]
- [[01_foundations/05_statistical_learning_theory/evaluation_metrics|Evaluation Metrics (theory)]]
- [[01_foundations/03_probability_and_statistics/hypothesis_testing|Hypothesis Testing (model comparison)]]
- [[02_modeling/04_training_dynamics/hyperparameter_tuning|Hyperparameter Tuning]]
- [[02_modeling/06_interpretability/interpretability_overview|Interpretability Overview]]
