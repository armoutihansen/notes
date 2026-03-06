---
layer: 02_modeling
type: concept
status: seed
tags: [hyperparameter-tuning, grid-search, random-search, bayesian-optimization, learning-rate, batch-size, architecture-search]
created: 2026-03-02
---

# Hyperparameter Tuning

## Definition

The process of selecting the configuration values (hyperparameters) of a machine learning model and its training procedure that are not learned from data but must be specified before training. These include architectural choices, optimization settings, and regularization strengths.

## Intuition

Think of hyperparameters as knobs on the outside of a black box. You can't compute their optimal values analytically — you must try them and observe the validation metric. The challenge is that each evaluation (training a model to convergence) is expensive, so exploration strategies matter greatly: exhaustive search is often impractical, and smart methods try to learn which regions of hyperparameter space are promising.

## Formal Description

### Common Hyperparameters

| Category | Examples |
|---|---|
| Optimization | Learning rate $\alpha$, batch size $B$, optimizer ($\beta_1, \beta_2$), LR schedule |
| Regularization | $\lambda$ (weight decay), dropout rate $p$, early stopping patience |
| Architecture | Number of layers, hidden units per layer, activation function, skip connections |
| Data | Augmentation strength, oversampling ratio |

**Learning rate** is typically the most impactful hyperparameter. A good default range to search is $[10^{-5}, 10^{-1}]$ on a log scale.

---

### Search Strategies

**Grid search**: evaluate all combinations on a discretized grid. Exhaustive but exponential in the number of hyperparameters. Only practical for 1–2 hyperparameters.

**Random search**: sample configurations uniformly (or log-uniformly for scale parameters). Empirically more efficient than grid search when only a few dimensions truly matter — the budget is not wasted on redundant combinations. Recommended by Bergstra & Bengio (2012) as a strong baseline.

**Bayesian optimization**: build a probabilistic surrogate model (Gaussian Process or Tree Parzen Estimator) of the validation metric as a function of hyperparameters. Use an acquisition function (Expected Improvement, UCB) to select the next point to evaluate that balances exploration and exploitation. More sample-efficient than random search for expensive evaluations. Implemented in Optuna, Hyperopt, and Ax.

**Successive Halving / Hyperband**: allocate a small budget to many configurations, eliminate the worst fraction, give more budget to the survivors — repeat. Dramatically reduces total compute by killing unpromising runs early. Asha (asynchronous SHA) enables parallel execution.

---

### Learning Rate Finding

The **LR range test** (Smith 2015): train for a few hundred iterations while exponentially increasing $\alpha$ from a small value. Plot train loss vs. $\alpha$; the optimal LR is just before loss starts rising sharply. Implemented as `lr_finder` in PyTorch Lightning and fastai.

---

### Schedules as Hyperparameters

Learning rate schedules introduce their own hyperparameters (warmup steps, decay factor, minimum LR). A common default: **linear warmup** for 5–10% of total steps, then **cosine decay** to $\alpha_{\min} = \alpha_0 / 100$.

## Applications

- All machine learning model development workflows
- Neural architecture search (NAS) generalizes HP tuning to architecture parameters
- AutoML systems (Auto-sklearn, H2O AutoML) wrap HP tuning to reduce manual effort

## Trade-offs

- **Grid search**: exhaustive but does not scale; safe for $\leq 2$ parameters
- **Random search**: scales better; may miss sharp optima; simple to implement
- **Bayesian optimization**: most sample-efficient; adds framework complexity; surrogate model assumptions may not hold at high dimensionality
- **Hyperband**: excellent compute efficiency; requires that performance ranking stabilizes early in training (not always true)
- **All methods**: validation metric must generalize; tune on held-out val set, do not touch test set

## Links

- [[optimization_algorithms|Optimization Algorithms]]
- [[early_stopping|Early Stopping]]
- [[regularization|Regularization]]
- [[02_modeling/index|Modeling]]
- [[01_foundations/05_statistical_learning_theory/bias_variance_analysis|Bias–Variance Analysis — model complexity trade-offs]]
- [[01_foundations/04_optimization/gradient_descent_optimization|Gradient Descent Optimization — learning rate search]]
