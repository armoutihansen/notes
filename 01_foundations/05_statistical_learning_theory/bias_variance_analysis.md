---
layer: 01_foundations
type: concept
status: seed
tags: [theory, evaluation]
created: 2026-03-02
---

# Bias-Variance Analysis

## Definition

A diagnostic framework for understanding whether a model's error is primarily due to underfitting (bias) or overfitting (variance), using human-level performance as a proxy for the irreducible (Bayes) error.

## Intuition

The gap between training error and Bayes error tells you how much the model is underfitting; the gap between training error and dev error tells you how much it is overfitting; targeting the larger gap gives the highest expected improvement.

## Formal Description

**Bayes error:** the theoretically minimum achievable error for a given task; unknowable in general.

**Human-level performance (HLP):** a practical proxy for Bayes error on tasks where humans perform near-optimally (perception, language). Note: HLP ≤ Bayes error is not guaranteed — humans can sometimes achieve below-Bayes error on narrow tasks, but for practical purposes HLP is used as the Bayes proxy.

**Avoidable bias:** training error − HLP; represents the gap between current training performance and the theoretical ceiling.

**Variance:** dev error − training error; represents the generalization gap.

**Four-gap diagnostic examples:**

| HLP | Train error | Dev error | Avoidable bias | Variance | Focus |
|-----|-------------|-----------|----------------|----------|-------|
| 1%  | 8%          | 10%       | 7%             | 2%       | Bias reduction |
| 7.5%| 8%          | 10%       | 0.5%           | 2%       | Variance reduction |

**Bias reduction tactics:** larger model, more features, better architecture, longer training.

**Variance reduction tactics:** more data, regularization, dropout, data augmentation.

## Applications

Any supervised learning diagnostic; especially useful when progress has stalled to identify the bottleneck.

## Trade-offs

- HLP is not always available (e.g., predicting stock prices)
- HLP underestimates Bayes error for superhuman tasks
- The framework assumes a clean train/dev split from the same distribution (use training-dev set to disentangle mismatch)

## Links
- [[01_foundations/05_statistical_learning_theory/error_analysis]]
- [[data_splits_and_distribution]]
- [[orthogonalization]]
