---
layer: 02_data_science
type: concept
status: growing
tags: [theory, evaluation]
created: 2026-03-07
---

# Causal Inference

## Problem Context

Observational data encodes correlation, not causation. A model predicting churn from support ticket volume cannot tell us whether calling support *causes* churn or whether both are caused by a bad product experience. Causal inference provides formal tools for answering *what would happen if we intervene?* — the question that drives decisions.

When to use causal methods vs standard predictive ML:
- **Predictive ML**: best when all you need is the most accurate forecast of Y given X (no intervention contemplated)
- **Causal inference**: required when you want to estimate the effect of an action (treatment), run policy simulations, or avoid acting on spurious correlations

## Core Concepts

**Potential outcomes (Rubin framework)**
For each unit i, define Y_i(1) = outcome if treated, Y_i(0) = outcome if untreated. The **Individual Treatment Effect (ITE)** is Y_i(1) - Y_i(0). We can only observe one potential outcome per unit — the **fundamental problem of causal inference**.

**Average Treatment Effect (ATE)**: E[Y(1) - Y(0)]
**Average Treatment Effect on the Treated (ATT)**: E[Y(1) - Y(0) | T=1]

**Confounders**: variables that affect both treatment assignment and outcome. Failing to control for confounders yields biased effect estimates.

**DAGs (Pearl framework)**: Directed Acyclic Graphs encoding causal assumptions. Identification analysis (d-separation, backdoor criterion) determines whether a causal effect is estimable from observational data and which variables to condition on.

## Identification Strategies

| Strategy | Assumption | Use when |
|---|---|---|
| **Randomized experiment (RCT)** | Randomisation eliminates confounding | You can run a controlled experiment |
| **Propensity score matching / IPW** | Unconfoundedness given observables | Treatment assignment depends only on observed covariates |
| **Instrumental variables (IV)** | Valid instrument Z: affects T but only affects Y through T | You have a natural instrument (e.g., lottery, policy cutoff) |
| **Difference-in-differences (DiD)** | Parallel trends in absence of treatment | Panel data with staggered rollout or pre/post comparison |
| **Regression discontinuity (RDD)** | Units just above/below cutoff are comparable | Continuous treatment assignment rule with a threshold |
| **Synthetic control** | Donor pool can construct a valid counterfactual | One treated unit, multiple controls, aggregate data |

## Practical Estimation

**Propensity score methods**:
```python
from sklearn.linear_model import LogisticRegression
import numpy as np

# Estimate propensity scores
ps_model = LogisticRegression()
ps_model.fit(X, T)
ps = ps_model.predict_proba(X)[:, 1]

# Inverse Probability Weighting (IPW) ATE estimator
ipw_ate = np.mean(T * Y / ps - (1 - T) * Y / (1 - ps))
```

**DoWhy + EconML** (recommended for production causal analysis):
```python
import dowhy
from dowhy import CausalModel

model = CausalModel(data=df, treatment="T", outcome="Y",
                    common_causes=["X1", "X2"])
identified = model.identify_effect()
estimate = model.estimate_effect(identified,
    method_name="backdoor.propensity_score_weighting")
```

**Heterogeneous treatment effects (HTE)** with causal forests (EconML):
```python
from econml.dml import CausalForestDML
cf = CausalForestDML(n_estimators=200)
cf.fit(Y, T, X=X, W=W)   # W = controls, X = effect modifiers
te = cf.effect(X)          # ITE estimates per unit
```

## Causal vs Predictive Models

A standard XGBoost classifier trained on observational data will learn the correlation P(Y=1 | X, T), not the causal effect E[Y(1) - Y(0) | X]. Naively reading feature importances as causal drivers is a common mistake. Use causal tools when:
- You want to estimate ROI of an intervention
- You need to compare two policies
- Treatment assignment in training data is non-random

## References
- Pearl, J. (2009). *Causality: Models, Reasoning, and Inference*. Cambridge University Press.
- Imbens, G. & Rubin, D. (2015). *Causal Inference for Statistics, Social, and Biomedical Sciences*. Cambridge University Press.
- Athey, S. & Imbens, G. (2019). Machine Learning Methods for Estimating Heterogeneous Causal Effects. https://arxiv.org/abs/1504.01132
- DoWhy documentation: https://py-why.github.io/dowhy/
- EconML documentation: https://econml.azurewebsites.net/

## Links
- [[02_data_science/05_experimentation_and_validation/ab_testing_and_experiment_design|A/B Testing and Experiment Design]]
- [[02_data_science/07_decision_analysis_and_business_metrics/index|07 — Decision Analysis]]
- [[01_foundations/03_probability_and_statistics/index|Probability and Statistics]]
- [[03_modeling/07_evaluation_and_model_selection/index|Evaluation and Model Selection]]
