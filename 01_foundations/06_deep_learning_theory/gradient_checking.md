---
layer: 01_foundations
type: concept
status: seed
tags: [workflow, training]
created: 2026-03-02
---

# Gradient Checking

## Definition

Numerically approximates partial derivatives using the centered difference formula and compares them against analytical gradients produced by backpropagation. Used exclusively as a debugging tool to verify a backprop implementation is correct.

## Intuition

Backpropagation is easy to implement incorrectly — sign errors, missing terms, wrong tensor shapes, and off-by-one errors all produce gradients that look plausible but are subtly wrong. Gradient checking acts as a unit test: if the numerical and analytical gradients agree to high precision, the implementation is almost certainly correct.

## Formal Description

**Centered difference approximation** for parameter $\theta_i$:
$$
\frac{\partial J}{\partial \theta_i} \approx \frac{J(\theta + \epsilon\, e_i) - J(\theta - \epsilon\, e_i)}{2\epsilon}
$$
where $e_i$ is the $i$-th standard basis vector and $\epsilon \approx 10^{-7}$.

The centered difference has $O(\epsilon^2)$ error vs. $O(\epsilon)$ for the one-sided difference, making it significantly more accurate.

**Relative error metric:**
$$
\text{err} = \frac{\|\nabla_\text{approx} - \nabla_\text{backprop}\|}{\|\nabla_\text{approx}\| + \|\nabla_\text{backprop}\|}
$$

**Interpretation:**

| Relative error        | Assessment                                 |
| --------------------- | ------------------------------------------ |
| $< 10^{-7}$           | Great — implementation very likely correct |
| $10^{-7}$ – $10^{-5}$ | Acceptable, borderline                     |
| $> 10^{-5}$           | Likely a bug in backprop                   |

## Applications

- Debugging new backpropagation implementations before training
- Catching sign errors, missing regularization terms, wrong matrix transpositions
- Verifying custom layers or loss functions in deep learning frameworks

## Trade-offs

- **Computational cost:** requires $O(d)$ forward passes where $d$ is the number of parameters — completely impractical for training; use only on small models or a parameter subset
- **Incompatible with dropout:** stochastic layers produce different values on each forward pass, making the numerical approximation meaningless; disable dropout during checks
- **Must include regularization:** $J$ in the numerical approximation must match $J$ used in backprop exactly, including any $L_2$ penalty
- **Numerical precision:** very small $\epsilon$ causes floating-point cancellation; very large $\epsilon$ increases approximation error; $\epsilon = 10^{-7}$ is the standard choice

## Links

- [[01_foundations/06_deep_learning_theory/backpropagation]]
- [[01_foundations/06_deep_learning_theory/gradient_descent]]
