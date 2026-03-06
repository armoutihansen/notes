---
layer: 02_modeling
type: concept
status: growing
tags: [svm, support-vector-machine, kernel-trick, rbf-kernel, kernel-regression]
created: 2026-03-06
---

# Kernel Methods and SVMs

## Definition

Kernel methods use a **kernel function** to implicitly map inputs into a high-dimensional (possibly infinite) feature space, allowing linear algorithms to learn non-linear decision boundaries. Support Vector Machines (SVMs) are the most prominent kernel method.

## Intuition

In the original feature space, the data may not be linearly separable. Mapping to a higher-dimensional space can make it separable. The kernel trick avoids explicitly computing this mapping — it computes inner products in the high-dimensional space directly from the original inputs. SVMs then find the maximum-margin hyperplane in that space.

## Formal Description

### Hard-Margin SVM (Linearly Separable)

Find the hyperplane $\mathbf{w}^\top\mathbf{x} + b = 0$ with maximum margin $2/\|\mathbf{w}\|$:

$$\min_{\mathbf{w},b} \frac{1}{2}\|\mathbf{w}\|^2 \quad \text{s.t.} \quad y_i(\mathbf{w}^\top\mathbf{x}_i + b) \geq 1 \; \forall i$$

**Dual problem** (via Lagrangian):

$$\max_{\boldsymbol{\alpha}} \sum_i \alpha_i - \frac{1}{2}\sum_{i,j}\alpha_i\alpha_j y_i y_j \mathbf{x}_i^\top\mathbf{x}_j$$

$$\text{s.t.} \quad \sum_i \alpha_i y_i = 0, \quad \alpha_i \geq 0$$

**Support vectors:** training points with $\alpha_i > 0$ (on the margin boundaries). All other $\alpha_i = 0$.

**Prediction:** $\hat{y} = \mathrm{sign}\!\left(\sum_i \alpha_i y_i \mathbf{x}_i^\top \mathbf{x} + b\right)$

### Soft-Margin SVM (C-SVM)

Introduces slack variables $\xi_i \geq 0$ to allow margin violations:

$$\min_{\mathbf{w},b,\boldsymbol{\xi}} \frac{1}{2}\|\mathbf{w}\|^2 + C\sum_i \xi_i \quad \text{s.t.} \quad y_i(\mathbf{w}^\top\mathbf{x}_i+b) \geq 1-\xi_i,\; \xi_i \geq 0$$

$C > 0$ controls the trade-off: large $C$ → low tolerance for misclassifications → narrower margin; small $C$ → wide margin, more misclassifications.

Dual with box constraints: $0 \leq \alpha_i \leq C$.

### The Kernel Trick

Replace $\mathbf{x}_i^\top\mathbf{x}_j$ with a **kernel function** $k(\mathbf{x}_i, \mathbf{x}_j) = \phi(\mathbf{x}_i)^\top\phi(\mathbf{x}_j)$:

$$\hat{y} = \mathrm{sign}\!\left(\sum_i \alpha_i y_i k(\mathbf{x}_i, \mathbf{x}) + b\right)$$

**Common kernels:**

| Kernel | Formula | Properties |
|---|---|---|
| Linear | $\mathbf{x}_i^\top\mathbf{x}_j$ | No mapping; fast |
| Polynomial | $(\gamma\mathbf{x}_i^\top\mathbf{x}_j + r)^d$ | Degree-$d$ interactions |
| RBF (Gaussian) | $\exp(-\gamma\|\mathbf{x}_i-\mathbf{x}_j\|^2)$ | Infinite-dimensional; most popular |
| Sigmoid | $\tanh(\gamma\mathbf{x}_i^\top\mathbf{x}_j + r)$ | Similar to single-layer NN |

**Mercer's condition:** a symmetric function $k$ is a valid kernel iff its Gram matrix $K_{ij} = k(\mathbf{x}_i,\mathbf{x}_j)$ is positive semidefinite for any finite sample.

**RBF kernel:** the bandwidth $\gamma = 1/(2\sigma^2)$ controls the smoothness of the decision boundary. Large $\gamma$ → narrow Gaussians → complex boundary; small $\gamma$ → smooth boundary.

### SVR (Support Vector Regression)

$\varepsilon$-insensitive loss: ignore residuals $< \varepsilon$; penalise larger residuals linearly. Dual formulation analogous to SVM.

### Kernel Ridge Regression

Closed-form kernel regression: $\hat{f}(\mathbf{x}) = \mathbf{k}^\top(K + \lambda I)^{-1}\mathbf{y}$, where $K_{ij} = k(\mathbf{x}_i,\mathbf{x}_j)$ and $\mathbf{k}_i = k(\mathbf{x}_i, \mathbf{x})$. Equivalent to ridge regression in the RKHS.

## Applications

- SVMs were dominant in text classification, image recognition, and bioinformatics before deep learning.
- Kernel methods remain useful for small datasets or specialised kernels (e.g., string kernels for sequences, graph kernels).
- SVM classifiers are a good baseline when the dataset fits in memory and has < ~10k samples.

## Trade-offs

| Aspect | SVM | Neural Network |
|---|---|---|
| Small data | ✅ Works well | ❌ May overfit |
| Large data | ❌ $O(n^2)$ kernel matrix | ✅ Scalable with SGD |
| Interpretability | Moderate (support vectors) | Low |
| Kernel choice | Requires domain knowledge | Learned end-to-end |

## Links

- [[01_foundations/04_optimization/lagrangian_and_constrained_optimization|Lagrangian and Constrained Optimization (SVM dual)]]
- [[01_foundations/04_optimization/convex_optimization|Convex Optimization (QP)]]
- [[02_modeling/03_model_families/01_linear_and_glm/linear_and_glm|Linear Models (linear kernel = linear SVM)]]
- [[02_modeling/04_training_dynamics/regularization|Regularization (C parameter)]]
