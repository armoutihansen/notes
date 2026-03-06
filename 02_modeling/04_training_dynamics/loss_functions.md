---
layer: 02_modeling
type: concept
status: seed
tags: [algorithm, training, evaluation]
created: 2026-03-02
---

# Loss Functions

## Definition

A loss function (objective function) measures the discrepancy between model predictions and ground truth labels for a single example. The training objective is to minimize the average loss over the dataset, possibly with regularization terms.

## Intuition

The choice of loss function encodes your assumptions about the error structure and what kinds of mistakes are costly. Squared error penalizes large residuals heavily; absolute error is robust to outliers; cross-entropy penalizes confident wrong predictions more than timid ones; hinge loss only cares whether the correct class wins by a margin.

## Formal Description

### Regression Losses

**Mean Squared Error (MSE / L2 loss)**

$$\ell(\hat y, y) = (\hat y - y)^2, \qquad J = \frac{1}{m}\sum_i (\hat y^{(i)} - y^{(i)})^2$$

Differentiable everywhere; penalizes outliers heavily; equivalent to MLE under Gaussian noise assumption.

**Mean Absolute Error (MAE / L1 loss)**

$$\ell(\hat y, y) = |\hat y - y|$$

Robust to outliers; non-differentiable at zero (subgradient used in practice); equivalent to MLE under Laplace noise assumption.

**Huber loss** interpolates: uses L2 for small residuals ($|r| \leq \delta$) and L1 for large ones, combining differentiability with outlier robustness.

---

### Classification Losses

**Binary Cross-Entropy (log loss)**

$$\ell(\hat y, y) = -\bigl[y\log\hat y + (1-y)\log(1-\hat y)\bigr]$$

Paired with sigmoid output. Minimizing is equivalent to maximum likelihood under Bernoulli distribution.

**Multi-class Cross-Entropy**

$$\ell(\hat y, y) = -\sum_{k=1}^K y_k\log\hat y_k, \qquad \hat y = \text{softmax}(z)$$

The softmax + CE gradient has a clean form: $\partial \ell / \partial z_k = \hat y_k - y_k$. Cross-entropy is equal to KL divergence from the empirical distribution plus an entropy constant, so minimizing CE is equivalent to minimizing $D_{\mathrm{KL}}(p \| q)$.

**Hinge loss (SVM)**

$$\ell(\hat y, y) = \max(0,\, 1 - y\hat y), \quad y \in \{-1, +1\}$$

Only penalizes predictions within margin $= 1$ of the decision boundary. Correct predictions beyond the margin incur zero loss. Multiclass extension (Weston-Watkins): $\sum_{j \neq y_i} \max(0, \hat y_j - \hat y_{y_i} + 1)$.

**Focal loss**

$$\ell(\hat y, y) = -\alpha_t (1 - \hat y_t)^\gamma \log(\hat y_t)$$

Extension of binary CE that down-weights easy, well-classified examples (small loss) and focuses learning on hard examples. Parameters: $\gamma > 0$ (focusing), $\alpha_t$ (class balancing). Originally proposed for object detection with severe class imbalance (RetinaNet). At $\gamma = 0$ it reduces to weighted CE.

---

### Practical Notes

- **Class imbalance**: use focal loss, per-class weighting, or oversampling with CE
- **Numerical stability**: compute softmax + CE together via log-sum-exp trick; never compute $\log(\text{softmax}(z))$ naively
- **Ordinal targets**: consider ranked loss or distance-aware objectives rather than plain CE

## Applications

| Loss | Typical use |
|---|---|
| MSE | Regression, autoencoders (output layer) |
| MAE | Robust regression, median estimation |
| Binary CE | Binary classification, sigmoid outputs |
| Multi-class CE | Multi-class classification, language modeling |
| Hinge | SVMs, margin-based classifiers |
| Focal | Object detection, severe class imbalance |

## Trade-offs

- MSE is differentiable and analytically convenient but can dominate training when large outliers exist
- MAE/Huber improve robustness but require careful $\delta$ tuning for Huber
- CE assumes well-calibrated probabilities; can overfit to label noise
- Focal loss adds two hyperparameters ($\alpha, \gamma$) that require tuning

## Links

- [[optimization_algorithms|Optimization Algorithms]]
- [[regularization|Regularization]]
- [[02_modeling/index|Modeling]]
- [[01_foundations/06_deep_learning_theory/cross_entropy_loss|Cross-Entropy Loss — categorical and binary derivation]]
- [[01_foundations/06_deep_learning_theory/triplet_loss|Triplet Loss — metric learning losses]]
- [[01_foundations/03_probability_and_statistics/probability_theory|Probability Theory — MLE derivation of loss functions]]
