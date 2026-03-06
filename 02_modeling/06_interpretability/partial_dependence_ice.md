---
layer: 02_modeling
type: concept
status: seed
tags: [interpretability, algorithm]
created: 2026-03-02
---

# Partial Dependence Plots, ICE, and ALE

## Definition

Methods for visualizing the marginal effect of one or two features on a model's predicted outcome, averaged (PDP) or shown per instance (ICE), or estimated via local contrasts to avoid extrapolation artifacts (ALE).

## Intuition

A partial dependence plot asks: "If I fix all other features and vary only feature $X_j$, how does the prediction change on average?" It marginalizes over the distribution of all other features. ICE shows the same relationship for each individual training point, revealing heterogeneity that PDP hides by averaging. ALE avoids the extrapolation problem of PDP by conditioning on small intervals rather than marginalizing globally.

## Formal Description

### Partial Dependence Plot (PDP)

For feature(s) $X_S$ and model $\hat f$:

$$\hat f_S(x_S) = E_{X_C}[\hat f(x_S, X_C)] \approx \frac{1}{n}\sum_{i=1}^n \hat f(x_S, x_C^{(i)})$$

where $X_C$ are the complement features. Implementation: grid over $x_S$ values; for each grid value, replace $X_S$ for all training instances, compute predictions, average. Two-way PDPs show interaction effects between two features.

**Limitations:**
- Assumes feature independence; when features are correlated, marginalizing over $X_C$ creates unrealistic input combinations (e.g., forcing height=200cm with weight=40kg simultaneously)
- Shows the average effect; heterogeneous subgroup effects are hidden
- Computationally $O(n \cdot G)$ where $G$ is the grid size

---

### Individual Conditional Expectation (ICE)

Plot one line per training instance: how does the prediction for instance $i$ change as $X_j$ varies? The PDP is the mean of all ICE curves.

$$\hat f_j^{(i)}(x_j) = \hat f(x_j, x_{-j}^{(i)})$$

**Centered ICE (c-ICE)**: subtract each curve's value at a reference point $x_j^0$ to highlight interaction effects: $\tilde f_j^{(i)}(x_j) = \hat f_j^{(i)}(x_j) - \hat f_j^{(i)}(x_j^0)$.

ICE reveals when the PDP average masks heterogeneous effects (some instances have positive, others negative marginal effects).

---

### Accumulated Local Effects (ALE)

ALE avoids the unrealistic combinations problem by estimating feature effects using local differences within narrow data-conditional intervals:

$$\hat{\text{ALE}}_j(x) = \sum_{k=1}^{k_x} \frac{1}{|N_j(k)|}\sum_{i \in N_j(k)} \left[\hat f(z_{k,j}, x_{-j}^{(i)}) - \hat f(z_{k-1,j}, x_{-j}^{(i)})\right]$$

where $z_{k-1,j}$ and $z_{k,j}$ are the interval boundaries and $N_j(k)$ is the set of training instances falling in interval $k$. ALE accumulates these local differences (hence the name) and centers the result to have zero mean.

**ALE advantages over PDP:**
- Unbiased with correlated features
- Faster to compute (no full grid substitution)
- Interpretable as the local effect of changing a feature within its observed range

## Applications

- Understanding non-linear feature effects in tree models and neural networks
- Regulatory explanations ("how does income affect predicted probability of default across its range?")
- Model debugging (detecting unexpected non-monotonicities or discontinuities)
- Two-way PDPs for identifying feature interactions

## Trade-offs

| Method | Handles correlation | Shows heterogeneity | Speed | Extrapolation |
|---|---|---|---|---|
| PDP | No | No (averaged) | Medium | Yes (problem) |
| ICE | No | Yes | Medium | Yes (problem) |
| ALE | Yes | No | Fast | No |

- PDPs are widely understood and easy to explain; ALE is preferred when features are correlated
- ICE plots can be cluttered for large datasets; subsample for visualization
- All methods show marginal effects, not causal effects — correlational feature structure affects interpretation

## Links

- [[interpretability_overview|Interpretability Overview]]
- [[shap_and_feature_attribution|SHAP and Feature Attribution]]
- [[02_modeling/index|Modeling]]
- [[01_foundations/03_probability_and_statistics/probability_theory|Probability Theory — marginalisation over feature distributions]]
- [[01_foundations/02_calculus_and_analysis/01_differentiation/partial_derivatives|Partial Derivatives — measuring isolated feature effect]]
