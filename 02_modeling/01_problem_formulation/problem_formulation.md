---
layer: 02_modeling
type: concept
status: growing
tags: [workflow, classification, regression]
created: 2026-03-06
---

# Problem Formulation

## Definition

The process of translating a real-world task into a well-specified ML problem: choosing the input space, output space, loss function, and success criteria before selecting any model.

## Intuition

Jumping directly to model selection is a common mistake. The right problem formulation determines everything downstream: what data you need, which algorithms are candidates, and how you know you've succeeded. Garbage in, garbage out — and a poorly framed problem can make even perfect modeling useless.

## Formal Description

### Supervised Learning Setup

Given a dataset $\mathcal{D} = \{(\mathbf{x}_i, y_i)\}_{i=1}^n$ where $\mathbf{x}_i \in \mathcal{X}$ and $y_i \in \mathcal{Y}$, find a function $f: \mathcal{X} \to \mathcal{Y}$ that minimizes expected risk:

$$R(f) = \mathbb{E}_{(\mathbf{x},y) \sim P}[\ell(f(\mathbf{x}), y)]$$

**Problem types by output space:**

| Problem type | $\mathcal{Y}$ | Default loss | Example |
|---|---|---|---|
| Binary classification | $\{0,1\}$ | Log-loss | Fraud detection |
| Multi-class classification | $\{1,\ldots,K\}$ | Categorical cross-entropy | Image classification |
| Regression | $\mathbb{R}$ | MSE / MAE | House price prediction |
| Multi-label classification | $\{0,1\}^K$ | Binary cross-entropy per label | Tag prediction |
| Ordinal regression | $\{1,\ldots,K\}$ (ordered) | Ordinal loss | Credit rating |
| Ranking | Partial order over items | Pairwise / listwise loss | Search ranking |
| Sequence-to-sequence | $\mathcal{Y}^*$ | Token-level cross-entropy | Machine translation |

### Choosing a Loss Function

The loss must reflect what you actually care about:

- **MSE** is sensitive to outliers (penalises large errors quadratically); prefer **MAE** or **Huber** for heavy-tailed targets.
- **Log-loss** is proper (incentivises calibrated probabilities); use it for probabilistic classifiers.
- **Class imbalance**: consider focal loss, weighted cross-entropy, or optimise directly for the business metric (e.g., PR-AUC instead of accuracy).
- Never optimise accuracy for imbalanced binary problems — it can be misleadingly high.

### Baseline Strategies

Always establish baselines before building complex models:

1. **Naive baseline**: always predict the majority class or overall mean. Defines the floor.
2. **Human-level performance (HLP)**: upper-bound proxy for irreducible error; skip when unavailable.
3. **Simple model baseline**: linear model, decision stump, or nearest-neighbour. Often surprisingly competitive.
4. **Previous best known result**: published benchmarks or prior production system.

The gap between naive and HLP tells you how much improvement is theoretically possible.

### Success Criteria

Define before any modeling:

- **Primary metric**: the single metric that drives decisions (e.g., F1 @ threshold 0.5, RMSE in dollars).
- **Constraints**: latency (< 100 ms), memory (< 2 GB), interpretability (must explain each decision), fairness (max 5 pp demographic parity gap).
- **Business acceptance threshold**: the minimum performance required for deployment.
- **Offline → online gap**: clarify whether offline metrics proxy for actual business value; validate with an A/B test.

### Framing Checklist

1. What is the prediction target? (exact label definition)
2. What are the natural input features? (avoid leakage)
3. What is the deployment context? (batch vs online, cold-start vs warm-start)
4. Is this a distribution shift risk? (train/test from same distribution?)
5. What are the cost asymmetries? (FP vs FN costs)
6. What human fallback exists?

## Applications

- Reframing fraud detection as a ranking problem (flag top $k$ suspicious cases) rather than binary classification when label noise is high.
- Choosing MAE over MSE for insurance claim amount prediction where large outliers are legitimate.
- Identifying data leakage: a feature recorded after the prediction time must be excluded.

## Trade-offs

- A tightly specified metric may not align with the true business objective (Goodhart's law).
- Multi-objective problems (e.g., accuracy + fairness + cost) require explicit weighting or constraint satisfaction — there is no free lunch.

## Links

- [[01_foundations/05_statistical_learning_theory/evaluation_metrics|Evaluation Metrics]]
- [[01_foundations/05_statistical_learning_theory/bias_variance_analysis|Bias–Variance Analysis]]
- [[02_modeling/02_data_science/exploratory_data_analysis|Exploratory Data Analysis]]
- [[02_modeling/04_training_dynamics/loss_functions|Loss Functions]]
- [[02_modeling/05_evaluation_and_validation/index|Evaluation and Validation]]
