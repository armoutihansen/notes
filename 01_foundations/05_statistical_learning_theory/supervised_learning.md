---
layer: 01_foundations
type: concept
status: seed
tags: [theory, classification, regression]
created: 2026-03-02
---

# Supervised Learning

## Definition

Learning a parameterized mapping $f_\theta: \mathcal{X} \to \mathcal{Y}$ from labeled examples by minimizing empirical risk.

## Intuition

Find parameters that best explain the training labels; generalization is the key challenge. The model must learn structure that transfers beyond the training set — not just memorize labels.

## Formal Description

**Setup:** dataset $\{(x^{(i)}, y^{(i)})\}_{i=1}^m$, hypothesis class $\{f_\theta\}$.

**Empirical risk minimization:**

$$\hat\theta = \arg\min_\theta \frac{1}{m}\sum_{i=1}^m \ell(f_\theta(x^{(i)}), y^{(i)}) + R(\theta)$$

where $\ell$ is the task loss and $R(\theta)$ is an optional regularizer.

**Task types:**

| Task | Output | Loss |
|---|---|---|
| Binary classification | sigmoid | BCE |
| Multi-class classification | softmax | CE |
| Regression | linear | MSE / MAE |
| Structured output | sequence, bounding box | task-specific |

**Bias-variance decomposition:**

$$\mathbb{E}[\text{error}] = \text{bias}^2 + \text{variance} + \text{irreducible noise}$$

High bias → underfitting (model too simple); high variance → overfitting (model too complex). Regularization, more data, and architectural choices all shift this tradeoff.

## Applications

- Image classification (ImageNet, medical imaging)
- Speech recognition
- Machine translation
- Fraud detection
- Medical diagnosis

## Trade-offs

- Requires labeled data — expensive to collect and annotate
- Assumes train and test distributions match; distribution shift degrades performance
- Capacity vs. generalization tradeoff: larger models can overfit with little data
- More data generally helps but with diminishing returns; data quality often matters more than quantity

## Links

- [[01_foundations/06_deep_learning_theory/logistic_regression]]
- [[01_foundations/06_deep_learning_theory/cross_entropy_loss]]
- [[data_splits_and_distribution]] (in 01_foundations/statistical_learning_theory)
