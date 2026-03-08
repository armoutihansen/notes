---
layer: 03_modeling
type: concept
status: growing
tags: [algorithm, tabular, classification, regression]
created: 2026-03-06
---

# Tree Ensembles

## Definition

Tree ensembles combine many decision trees to produce predictions that are more accurate and robust than any individual tree. The two dominant approaches are **bagging** (random forests) and **boosting** (gradient boosting machines).

## Intuition

A single decision tree is unstable: small data changes lead to very different trees. Ensembling averages out this variance. Bagging does this in parallel by training many trees on bootstrap samples; boosting does it sequentially by having each tree correct the errors of its predecessors.

## Formal Description

### Decision Tree

Greedily partitions feature space by choosing splits that maximise a purity criterion:

**Gini impurity** (for classification):

$$
G = 1 - \sum_k p_k^2
$$

**Entropy:**

$$
H = -\sum_k p_k \log_2 p_k
$$

**MSE** (for regression): choose split minimising weighted average of child variances.

The algorithm recurses until a stopping criterion (max depth, min samples per leaf, min impurity decrease).

**Key hyperparameters:** `max_depth`, `min_samples_leaf`, `max_features`.

**Inductive bias:** axis-aligned splits — struggles with linear decision boundaries in general position.

### Random Forest (Bagging)

For $T$ trees:
1. Sample $n$ rows with replacement (bootstrap sample).
2. At each split, consider only $m \approx \sqrt{d}$ random features.
3. Grow tree to maximum depth (no pruning).
4. Aggregate: majority vote (classification) or mean (regression).

**Variance reduction:** averaging $T$ i.i.d. trees with variance $\sigma^2$ gives variance $\sigma^2/T$ if trees are uncorrelated. Feature subsampling decorrelates trees.

**Out-of-bag (OOB) error:** each sample is out-of-bag for ~37% of trees; provides free internal validation.

**Feature importance:** mean decrease in impurity (MDI) or mean decrease in accuracy via permutation. MDI is fast but biased toward high-cardinality features; permutation importance is more reliable.

### Gradient Boosting

Sequentially builds $T$ trees where tree $t$ fits the **pseudo-residuals** of the current ensemble:

$$
F_t(\mathbf{x}) = F_{t-1}(\mathbf{x}) + \nu \cdot h_t(\mathbf{x})
$$

where $h_t$ is fit to $-\partial L / \partial F_{t-1}(\mathbf{x}_i)$ (the negative gradient of the loss).

For MSE loss: pseudo-residuals = $y_i - F_{t-1}(\mathbf{x}_i)$ (true residuals).
For log-loss: pseudo-residuals = $y_i - \sigma(F_{t-1}(\mathbf{x}_i))$ (error in probability space).

**Shrinkage (learning rate) $\nu$:** smaller $\nu$ requires more trees but generalises better.

### XGBoost

XGBoost adds a **second-order (Newton) approximation** and explicit regularisation:

$$
\mathcal{L}^{(t)} \approx \sum_i [g_i f_t(\mathbf{x}_i) + \frac{1}{2}h_i f_t^2(\mathbf{x}_i)] + \Omega(f_t)
$$

where $g_i = \partial_{\hat{y}}\ell(y_i, \hat{y})$, $h_i = \partial^2_{\hat{y}}\ell$, and $\Omega(f) = \gamma T + \frac{1}{2}\lambda\|\mathbf{w}\|^2$ penalises tree complexity.

Optimal leaf weight: $w_j^* = -G_j / (H_j + \lambda)$; optimal split gain:

$$
\text{Gain} = \frac{1}{2}\left[\frac{G_L^2}{H_L+\lambda} + \frac{G_R^2}{H_R+\lambda} - \frac{(G_L+G_R)^2}{H_L+H_R+\lambda}\right] - \gamma
$$

### LightGBM

LightGBM uses **GOSS** (Gradient-based One-Side Sampling) and **EFB** (Exclusive Feature Bundling) for speed, plus **leaf-wise** (best-first) tree growth instead of level-wise:

- Leaf-wise growth finds deeper, more complex splits that can overfit on small datasets; mitigate with `min_child_samples` and `num_leaves`.
- LightGBM is typically 10–20× faster than vanilla GBDT for large datasets.

### Hyperparameter Tuning Guidance

| Hyperparameter | Effect | Range |
|---|---|---|
| `n_estimators` | More trees = better up to diminishing returns | 100–5000 |
| `max_depth` (XGB/RF) | Depth of individual trees | 3–8 |
| `num_leaves` (LGB) | Max leaves (leaf-wise) | 31–255 |
| `learning_rate` (GBM) | Shrinkage; lower = better generalisation | 0.01–0.3 |
| `subsample` | Row subsampling per tree (stochastic GBM) | 0.5–1.0 |
| `colsample_bytree` | Column subsampling per tree | 0.5–1.0 |
| `min_child_weight` | Min sum of Hessians in leaf (regularisation) | 1–300 |
| `reg_alpha/lambda` | L1/L2 regularisation on leaf weights | 0–10 |

## Applications

- Tabular ML competitions: GBMs dominate (XGBoost, LightGBM, CatBoost)
- Insurance: claim frequency/severity prediction; risk segmentation
- Fraud detection: high recall at low FPR
- Feature importance for regulatory model documentation

## Trade-offs

| Model | Pros | Cons |
|---|---|---|
| Decision tree | Interpretable, fast | High variance, low accuracy |
| Random forest | Low variance, robust, OOB estimate | Slower inference, less accurate than GBM |
| GBM (XGB/LGB) | State-of-art tabular accuracy | Slow training, many hyperparameters, overfits with few data |

## Links

- [[01_foundations/03_probability_and_statistics/probability_theory|Probability Theory (Bootstrap)]]
- [[01_foundations/04_optimization/gradient_descent_optimization|Gradient Descent (Boosting derivation)]]
- [[03_modeling/06_training_and_regularization/regularization|Regularization]]
- [[03_modeling/06_training_and_regularization/hyperparameter_tuning|Hyperparameter Tuning]]
- [[03_modeling/07_evaluation_and_model_selection/shap_and_feature_attribution|SHAP and Feature Attribution]]
