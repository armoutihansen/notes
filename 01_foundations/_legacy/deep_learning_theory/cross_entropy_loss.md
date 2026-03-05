---
layer: 01_foundations
type: concept
status: seed
tags: [cross-entropy, loss-function, softmax, KL-divergence, maximum-likelihood]
created: 2026-03-02
---

# Cross-Entropy Loss

## Definition

The negative log-likelihood under a categorical distribution; measures the expected surprise of predictions relative to true labels.

## Intuition

The log penalizes confident wrong predictions heavily — if the model assigns near-zero probability to the true class, the loss explodes. Minimizing cross-entropy is equivalent to maximum likelihood estimation for categorical models.

## Formal Description

**Binary CE** (paired with sigmoid output):

$$\ell(\hat y, y) = -\bigl[y\log\hat y + (1-y)\log(1-\hat y)\bigr]$$

**Multi-class CE** (paired with softmax):

$$\ell(\hat y, y) = -\sum_{k=1}^K y_k\log\hat y_k, \qquad \hat y = \text{softmax}(z), \quad \text{softmax}(z)_k = \frac{e^{z_k}}{\sum_j e^{z_j}}$$

**Combined gradient (softmax + CE):** unusually clean — the upstream gradient is simply the residual:

$$\frac{\partial\ell}{\partial z_k} = \hat y_k - y_k$$

**Dataset loss:**

$$J = \frac{1}{m}\sum_{i=1}^m \ell(\hat y^{(i)}, y^{(i)})$$

**Connection to KL divergence:**

$$H(p, q) = H(p) + D_{\mathrm{KL}}(p \| q)$$

Since $H(p)$ is fixed given the data, minimizing cross-entropy $H(p,q)$ is equivalent to minimizing $D_{\mathrm{KL}}(p \| q)$ — i.e., making the model distribution close to the true distribution.

## Applications

- Binary and multi-class classification
- Language modeling (next-token prediction)
- Any task with categorical outputs

## Trade-offs

- Sensitive to class imbalance — minority-class errors contribute little to the average; consider focal loss or per-class reweighting
- Numerically unstable if $\hat y \to 0$; use the log-sum-exp trick when computing softmax + CE together
- The clean softmax-CE gradient makes implementation straightforward but obscures what happens numerically

## Links

- [[01_foundations/_legacy/deep_learning_theory/logistic_regression]]
- [[01_foundations/_legacy/deep_learning_theory/supervised_learning]]
- [[01_foundations/_legacy/deep_learning_theory/backpropagation]]
