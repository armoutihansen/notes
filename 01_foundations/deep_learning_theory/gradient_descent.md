---
layer: 01_foundations
type: concept
status: seed
tags: [optimization, gradient_descent, training, schedules]
created: 2026-03-02
---

# Gradient Descent

## Definition

Iterative optimization algorithm that updates parameters by following the negative gradient of a loss function. Variants differ in how many training examples are used per gradient estimate: full-batch, mini-batch, or stochastic (single example).

## Intuition

Imagine standing on a hilly loss surface. At each step you look around, find the direction of steepest descent, and take a step in that direction. The learning rate controls step size. Too large and you overshoot valleys; too small and convergence is slow. Using mini-batches is like estimating the slope from a random sample of the terrain—noisier but much faster per update.

## Formal Description

**Cost function.** Given $m$ training examples with inputs $x^{(i)}$ and labels $y^{(i)}$:
$$
J(\theta) = \frac{1}{m} \sum_{i=1}^{m} \mathcal{L}(\hat y^{(i)}, y^{(i)}) + \frac{\lambda}{2m}\|\theta\|^2
$$

**General update rule:**
$$
\theta \leftarrow \theta - \alpha \nabla_\theta J(\theta)
$$

**Batch variants:**

| Variant | Batch size $B$ | Gradient estimate | Notes |
|---|---|---|---|
| Full-batch GD | $B = m$ | Exact over dataset | Smooth, slow per update |
| Stochastic GD (SGD) | $B = 1$ | Single example | Very noisy, many updates |
| Mini-batch GD | $1 < B < m$ | Over $B$ examples | Standard in practice |

Mini-batch update using batch $\mathcal{B}_t$:
$$
\theta \leftarrow \theta - \frac{\alpha}{B} \sum_{i \in \mathcal{B}_t} \nabla_\theta \mathcal{L}(\hat y^{(i)}, y^{(i)})
$$

**Mini-batch size trade-offs:**
- Typical $B \in \{32, 64, 128, 256, 512\}$ (powers of 2 for GPU efficiency)
- Smaller $B$: more gradient noise, acts as regularizer, more updates per epoch, slower per step
- Larger $B$: smoother gradients, faster per step, less regularizing noise, higher memory use
- $B = 1$ (SGD): maximum noise, no GPU parallelism benefit
- $B = m$ (full-batch): exact gradient, no noise, impractical for large datasets

**Learning rate schedules.** Decaying $\alpha$ over training improves final convergence:

*Step decay* — reduce by factor $\gamma$ every $k$ epochs:
$$
\alpha_t = \alpha_0 \cdot \gamma^{\lfloor t/k \rfloor}
$$

*Exponential decay:*
$$
\alpha_t = \alpha_0 \cdot e^{-\lambda t}
$$

*1/t decay:*
$$
\alpha_t = \frac{\alpha_0}{1 + k \cdot t}
$$

*Cosine annealing:*
$$
\alpha_t = \alpha_{\min} + \frac{1}{2}(\alpha_{\max} - \alpha_{\min})\left(1 + \cos\frac{\pi t}{T}\right)
$$

## Applications

- Training all neural network models (DNNs, CNNs, RNNs, Transformers)
- Any differentiable objective minimization
- Learning rate schedules are standard practice in competitive deep learning

## Trade-offs

- **Learning rate sensitivity:** too high → divergence or oscillation; too low → slow convergence or local minima
- **Batch size vs. noise vs. compute:** small batches add beneficial noise but reduce GPU utilization; large batches converge to sharper minima (potentially worse generalization)
- **Schedule tuning:** requires additional hyperparameter choices (decay rate, warmup steps, minimum $\alpha$); cosine annealing with warm restarts is often a strong default
- Plain GD (no momentum) is rarely used; [[adaptive_optimizers]] typically preferred

## Links

- [[adaptive_optimizers]]
- [[backpropagation]]
- [[batch_normalization]]
