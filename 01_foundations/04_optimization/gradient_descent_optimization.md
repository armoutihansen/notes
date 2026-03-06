---
layer: 01_foundations
type: concept
status: growing
tags: [algorithm, training]
created: 2026-03-06
---

# Gradient Descent and Variants

## Definition

Gradient descent is a first-order iterative optimization algorithm that minimizes an objective function by repeatedly moving in the direction of steepest descent (negative gradient).

## Intuition

Imagine standing on a hilly terrain in fog. You can only feel the slope under your feet. Gradient descent says: always step in the direction that goes most steeply downhill. The step size (learning rate) controls how far you stride each step — too large and you overshoot; too small and you take forever to converge.

## Formal Description

### Batch Gradient Descent

Given $f(\boldsymbol{\theta}) = \frac{1}{n}\sum_{i=1}^n \ell(\boldsymbol{\theta}; x_i)$, update:

$$\boldsymbol{\theta}_{t+1} = \boldsymbol{\theta}_t - \eta \nabla_{\boldsymbol{\theta}} f(\boldsymbol{\theta}_t)$$

$\eta > 0$ is the **learning rate**. Computes gradient over the full dataset — expensive for large $n$.

### Stochastic Gradient Descent (SGD)

Uses one randomly sampled example per update:

$$\boldsymbol{\theta}_{t+1} = \boldsymbol{\theta}_t - \eta \nabla \ell(\boldsymbol{\theta}_t; x_i)$$

Noisy estimate of the true gradient; the noise can help escape shallow local minima. Complexity per step: $O(1)$ vs $O(n)$ for batch.

**Mini-batch SGD** (standard in deep learning): average gradient over a batch of $B$ examples:

$$\boldsymbol{\theta}_{t+1} = \boldsymbol{\theta}_t - \frac{\eta}{B}\sum_{i \in \mathcal{B}} \nabla \ell(\boldsymbol{\theta}_t; x_i)$$

Typical batch size: 32–512. GPU parallelism makes mini-batch efficient.

### Convergence Rate

For $L$-smooth, $m$-strongly convex $f$ with fixed $\eta \leq 1/L$:

$$f(\boldsymbol{\theta}_T) - f(\boldsymbol{\theta}^*) \leq \left(1 - \frac{m}{L}\right)^T (f(\boldsymbol{\theta}_0) - f(\boldsymbol{\theta}^*))$$

Linear convergence — the condition number $\kappa = L/m$ governs speed.

For convex (not strongly convex): $O(1/T)$ convergence.

### Momentum

Accumulates an exponential moving average of gradients to dampen oscillations:

$$\mathbf{v}_{t+1} = \beta\mathbf{v}_t + (1-\beta)\nabla f(\boldsymbol{\theta}_t)$$

$$\boldsymbol{\theta}_{t+1} = \boldsymbol{\theta}_t - \eta\mathbf{v}_{t+1}$$

Typical $\beta = 0.9$. Nesterov momentum evaluates the gradient at the "lookahead" point $\boldsymbol{\theta}_t - \beta\mathbf{v}_t$, improving convergence rate to $O(1/T^2)$ for convex problems.

### RMSProp

Divides the learning rate by a running average of squared gradients, adapting per-parameter:

$$\mathbf{s}_{t+1} = \rho\mathbf{s}_t + (1-\rho)\nabla f \odot \nabla f$$

$$\boldsymbol{\theta}_{t+1} = \boldsymbol{\theta}_t - \frac{\eta}{\sqrt{\mathbf{s}_{t+1}} + \epsilon}\nabla f$$

Effective in non-stationary settings; reduces the learning rate for frequently updated parameters.

### Adam (Adaptive Moment Estimation)

Combines momentum (first moment) and RMSProp (second moment) with bias correction:

$$\mathbf{m}_{t+1} = \beta_1 \mathbf{m}_t + (1-\beta_1)\nabla f$$

$$\mathbf{v}_{t+1} = \beta_2 \mathbf{v}_t + (1-\beta_2)\nabla f \odot \nabla f$$

$$\hat{\mathbf{m}} = \mathbf{m}_{t+1}/(1-\beta_1^{t+1}), \quad \hat{\mathbf{v}} = \mathbf{v}_{t+1}/(1-\beta_2^{t+1})$$

$$\boldsymbol{\theta}_{t+1} = \boldsymbol{\theta}_t - \frac{\eta}{\sqrt{\hat{\mathbf{v}}} + \epsilon}\hat{\mathbf{m}}$$

Defaults: $\beta_1 = 0.9$, $\beta_2 = 0.999$, $\epsilon = 10^{-8}$, $\eta = 10^{-3}$.

**AdamW**: decouples weight decay from the gradient update (correct form):
$$\boldsymbol{\theta}_{t+1} = \boldsymbol{\theta}_t - \frac{\eta}{\sqrt{\hat{\mathbf{v}}}+\epsilon}\hat{\mathbf{m}} - \eta\lambda\boldsymbol{\theta}_t$$

### Learning Rate Schedules

| Schedule | Description |
|---|---|
| Step decay | Multiply by $\gamma < 1$ every $k$ epochs |
| Cosine annealing | $\eta_t = \eta_{\min} + \frac{1}{2}(\eta_{\max}-\eta_{\min})(1 + \cos(\pi t/T))$ |
| Warmup | Linear ramp from 0 to $\eta_{\max}$ over first $T_w$ steps |
| Cyclic LR (CLR) | Oscillate between $\eta_{\min}$ and $\eta_{\max}$ |

Warmup + cosine decay is standard for transformer training.

### Challenges in Non-Convex Optimization

- **Saddle points**: gradient is zero but not a minimum; second-order methods or noise escape these.
- **Vanishing/exploding gradients**: deep networks can have exponentially small/large gradients; mitigated by normalisation, residual connections, gradient clipping.
- **Learning rate sensitivity**: too large diverges, too small converges slowly; use learning rate finders.

## Applications

- Training any differentiable model: linear regression, logistic regression, neural networks
- Specific optimizer choice matters: Adam is default for deep learning; SGD with momentum can generalise better for image models (via implicit regularisation from noise)

## Trade-offs

| Optimizer | Pros | Cons |
|---|---|---|
| SGD (+ momentum) | Good generalisation, well-studied theory | Sensitive to LR, slow on ill-conditioned problems |
| Adam | Fast convergence, robust to LR | Can generalise worse than SGD for image models; weight decay must use AdamW |
| RMSProp | Good for RNNs | No bias correction |

## Links

- [[convex_optimization|Convex Optimization]]
- [[lagrangian_and_constrained_optimization|Lagrangian and Constrained Optimization]]
- [[01_foundations/02_calculus_and_analysis/01_differentiation/partial_derivatives|Partial Derivatives]]
- [[01_foundations/06_deep_learning_theory/gradient_descent|Gradient Descent (DL Theory)]]
- [[02_modeling/04_training_dynamics/optimization_algorithms|Optimization Algorithms (Modeling)]]
