---
layer: 03_modeling
type: concept
status: seed
tags: [algorithm, training]
created: 2026-03-02
---

# Regularization in Deep Networks

## Definition

Techniques that reduce overfitting by constraining model complexity, adding noise, or increasing effective training data size, without necessarily reducing model capacity.

## Intuition

Overfitting happens when the model memorizes training examples rather than learning generalizable structure. Regularization introduces constraints or noise that force the model to find more robust solutions — smaller weights, sparser activations, earlier stopping, or exposure to more varied inputs.

## Formal Description

**L2 regularization (weight decay)**

Adds a penalty term to the loss:

$$
J_{\text{reg}} = J + \frac{\lambda}{2m}\sum_l \|W^{[l]}\|_F^2
$$

Backprop becomes:

$$
dW^{[l]} \leftarrow dW^{[l]} + \frac{\lambda}{m}W^{[l]}
$$

Equivalent to multiplying weights by $(1 - \alpha\lambda/m)$ each step (hence "weight decay"). Encourages small weights → smoother, less complex function. $\lambda$ controls regularization strength.

---

**Dropout**

At each training forward pass, independently zero each activation with probability $(1-p)$ (keep prob $p$). Inverted dropout scales kept activations by $1/p$ during training so the expected activation equals the original:

```python
mask = (np.random.rand(*A.shape) < p) / p
A *= mask
```

At test time, use the full network without dropout. A different mask is sampled every forward pass, creating an implicit ensemble of $2^n$ sub-networks.

---

**Data augmentation**

Generate new training examples by applying label-preserving transformations:

- **Vision**: random horizontal flips, random crops, color jitter (brightness, contrast, saturation, hue), rotation, mixup
- **NLP**: back-translation, synonym replacement, random deletion

Effectively increases dataset size at no additional labeling cost.

## Applications

- **L2** is the default baseline for most architectures.
- **Dropout** is standard for fully-connected layers and transformers; less common in CNNs after batch normalization became prevalent.
- **Data augmentation** is essential for vision models with limited labeled data.

## Trade-offs

| Technique | Limitation |
|---|---|
| L2 | Penalizes all weights equally regardless of their importance |
| Dropout | Slows convergence (needs more epochs); interacts poorly with batch norm |
| Data augmentation | Domain-specific; over-application can produce unrealistic or label-inconsistent examples |

## Links

- [[early_stopping|Early Stopping]]
- [[03_modeling/index|Modeling]]
- [[01_foundations/06_deep_learning_theory/dropout|Dropout — inverted dropout, regularization theory]]
- [[01_foundations/06_deep_learning_theory/batch_normalization|Batch Normalization — normalization as regularizer]]
- [[01_foundations/05_statistical_learning_theory/bias_variance_analysis|Bias–Variance Analysis — underfitting vs overfitting]]
