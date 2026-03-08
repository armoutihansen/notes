---
layer: 03_modeling
type: concept
status: seed
tags: [algorithm, tabular, classification, regression]
created: 2026-05-31
---

# Instance-Based Methods

## Core Idea

Instance-based (or memory-based) methods make predictions by finding the training examples most similar to a query point, rather than fitting a global parametric model. The hypothesis is "local": nearby training examples are most relevant for predicting any given test point.

## Mathematical Formulation

### K-Nearest Neighbours (KNN)

Given a query point $\mathbf{x}^*$ and training set $\{(\mathbf{x}_i, y_i)\}_{i=1}^n$, identify the $k$ training points closest under distance $d$:

$$
\mathcal{N}_k(\mathbf{x}^*) = \text{top-}k \text{ points by } d(\mathbf{x}_i, \mathbf{x}^*)
$$

**Classification:** majority vote over neighbours:

$$
\hat{y} = \text{mode}\{y_i : \mathbf{x}_i \in \mathcal{N}_k(\mathbf{x}^*)\}
$$

**Regression:** mean (or distance-weighted mean) over neighbours:

$$
\hat{y} = \frac{1}{k}\sum_{\mathbf{x}_i \in \mathcal{N}_k(\mathbf{x}^*)} y_i \qquad \text{(uniform)}
$$

$$
\hat{y} = \frac{\sum_i w_i y_i}{\sum_i w_i}, \quad w_i = \frac{1}{d(\mathbf{x}^*, \mathbf{x}_i)^2} \qquad \text{(distance-weighted)}
$$

### Distance Metrics

| Metric | Formula | Use case |
|---|---|---|
| Euclidean ($L_2$) | $\sqrt{\sum_j (x_j - x'_j)^2}$ | Continuous, homogeneous features |
| Manhattan ($L_1$) | $\sum_j |x_j - x'_j|$ | Robust to outliers |
| Minkowski | $(\sum_j |x_j - x'_j|^p)^{1/p}$ | General; $p=2$ is Euclidean |
| Cosine | $1 - \frac{\mathbf{x} \cdot \mathbf{x}'}{\|\mathbf{x}\|\|\mathbf{x}'\|}$ | High-dimensional text/embeddings |
| Hamming | Proportion of differing components | Binary or categorical features |

Standardise features before computing Euclidean distance — otherwise high-variance features dominate.

## Inductive Bias

KNN assumes: nearby points (in feature space) have similar labels. This is the **smoothness assumption**, also called the **manifold hypothesis** in its stronger form. It holds well for locally smooth functions and fails when the decision boundary is globally structured or when the feature space is very high-dimensional.

## Training Objective

KNN has **no training phase** — the training set is the model. Prediction requires searching over all training points, making prediction $O(n \cdot d)$ for brute-force search. Practical implementations use:
- **KD-trees**: $O(d \log n)$ for low-dimensional data ($d \lesssim 20$).
- **Ball trees**: better than KD-trees for $d > 20$.
- **Approximate nearest neighbours** (FAISS, HNSW): sub-linear search for large-scale retrieval.

## Strengths

- **No training time**: simply stores the data.
- **Non-parametric**: adapts to arbitrary decision boundary shapes.
- **Naturally handles multi-class**: majority vote scales to any $K$.
- **Interpretable locally**: the prediction is explained by the neighbours.

## Weaknesses

- **Slow prediction**: $O(n)$ per query (brute force); mitigated by ANN indices.
- **High memory**: must store entire training set.
- **Curse of dimensionality**: in high dimensions, distance concentrates and neighbourhoods become meaningless. Performance degrades rapidly for $d \gtrsim 20$.
- **No feature learning**: cannot learn task-relevant representations — sensitive to irrelevant features.
- **Sensitive to scale**: requires feature standardisation.

## Variants

- **Radius-based neighbours**: use all points within radius $r$ instead of fixed $k$.
- **Locally weighted regression (LOESS)**: fit a local polynomial to neighbours weighted by distance; used for smooth curve estimation.
- **Condensed KNN**: removes redundant training points to reduce storage.
- **Weighted KNN**: distance-weighted voting typically outperforms uniform KNN.

## References

- Cover, T. & Hart, P. (1967). "Nearest neighbor pattern classification." *IEEE Transactions on Information Theory*.
- Hastie, T., Tibshirani, R., & Friedman, J. (2009). *The Elements of Statistical Learning*. §13.3.

## Links

- [[03_modeling/01_supervised_learning/03_kernel_methods/kernel_methods|Kernel Methods — kernel regression as smooth alternative]]
- [[03_modeling/02_unsupervised_learning/01_clustering/unsupervised_learning|Unsupervised Learning — KNN used in anomaly detection]]
- [[01_foundations/01_linear_algebra/index|Linear Algebra — distance metrics, inner products]]
- [[02_data_science/04_feature_engineering/feature_engineering|Feature Engineering — scaling required before KNN]]
