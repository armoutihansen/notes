---
layer: 02_modeling
type: concept
status: seed
tags: [interpretability, algorithm]
created: 2026-03-02
---

# SHAP and Feature Attribution

## Definition

Feature attribution methods assign a scalar importance score to each input feature for a given prediction, explaining how much each feature contributed to the output relative to a baseline. SHAP (SHapley Additive exPlanations) provides a theoretically grounded approach based on cooperative game theory.

## Intuition

Imagine a model's prediction as the payoff in a game. Each feature is a "player" that collaborates with others to produce the prediction. The Shapley value fairly distributes the prediction's deviation from the baseline (the "grand coalition" minus "no players") across all players by averaging their marginal contributions over all possible feature coalitions. SHAP makes this computationally tractable and links it to a class of additive explanation models.

## Formal Description

### Shapley Values

For a model $f$ and input $x$ with $n$ features, the Shapley value of feature $i$ is:

$$\phi_i = \sum_{S \subseteq F \setminus \{i\}} \frac{|S|!(n-|S|-1)!}{n!} \left[ f(x_{S \cup \{i\}}) - f(x_S) \right]$$

where $F$ is the full feature set and $x_S$ denotes the prediction with only features in $S$ active (others replaced by baseline/marginal values). Shapley values satisfy four axioms: **efficiency** (values sum to $f(x) - E[f]$), **symmetry**, **dummy** (zero contribution for irrelevant features), and **linearity**.

---

### SHAP Explanation Model

SHAP defines a local linear explanation $g(z') = \phi_0 + \sum_i \phi_i z'_i$ in a simplified binary feature space. It is the unique solution satisfying local accuracy, missingness, and consistency. This connects Shapley values to additive feature attribution.

---

### SHAP Variants

**KernelSHAP**: model-agnostic; estimates Shapley values via weighted linear regression on feature coalitions. Exact for any model but slow ($O(2^n)$ coalitions; approximated via sampling).

**TreeSHAP**: exact, polynomial-time algorithm for tree ensembles (decision trees, random forests, XGBoost, LightGBM). Exploits tree structure for $O(TLD^2)$ complexity (T trees, L leaves, D max depth). The default for gradient-boosted trees.

**DeepSHAP / DeepLIFT**: propagates SHAP values through neural network layers using the chain rule with backpropagation. Combines DeepLIFT's attribution rules with SHAP's game-theoretic properties. Fast but approximate.

---

### Integrated Gradients

An alternative attribution method for differentiable models:

$$\text{IG}_i(x) = (x_i - x_i') \int_0^1 \frac{\partial f(x' + \alpha(x-x'))}{\partial x_i}\, d\alpha$$

Integrates the gradient along a straight path from baseline $x'$ to input $x$. Satisfies completeness: $\sum_i \text{IG}_i(x) = f(x) - f(x')$. Standard for image and text attribution in neural networks.

---

### LIME (Local Interpretable Model-agnostic Explanations)

Perturbs the input around a point, queries the model, and fits a sparse linear model on the perturbed samples weighted by proximity. Fast and model-agnostic. Does **not** use Shapley values; explanations are less theoretically grounded and can vary with perturbation scheme and number of samples. Good for quick local explanations when SHAP is too slow.

## Applications

- Credit scoring: which features drove a high/low risk score for a specific applicant
- XGBoost models: TreeSHAP gives exact, fast attributions; SHAP summary plots for global feature importance
- Neural networks: integrated gradients or DeepSHAP for image/text input attribution
- Model debugging: negative SHAP values reveal counterintuitive feature influences

## Trade-offs

- **KernelSHAP**: model-agnostic but slow; approximation quality depends on number of coalition samples
- **TreeSHAP**: exact and fast for trees; not applicable to neural networks
- **DeepSHAP**: fast but approximate; sensitive to choice of baseline
- **Integrated Gradients**: clean theory; requires differentiable model; baseline choice affects attributions
- **LIME vs SHAP**: LIME is faster and simpler; SHAP has stronger theoretical guarantees and global consistency

## Links

- [[interpretability_overview|Interpretability Overview]]
- [[partial_dependence_ice|PDP and ICE]]
- [[02_modeling/index|Modeling]]
- [[01_foundations/03_probability_and_statistics/probability_theory|Probability Theory — Shapley values, cooperative game theory]]
- [[01_foundations/04_optimization/convex_optimization|Convex Optimization — linear approximation framework (LIME)]]
