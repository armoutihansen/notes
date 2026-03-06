---
layer: 01_foundations
type: concept
status: growing
tags: [optimization, adaptive_lr, momentum, adam, rmsprop]
created: 2026-03-02
---

# Adaptive Optimizers

## Definition

Optimizers that adapt per-parameter learning rates using estimates of gradient moments. Rather than using a single global learning rate, they maintain running statistics of past gradients and scale each parameter update individually.

## Intuition

**Momentum** smooths out oscillations by accumulating a velocity vector: frequent gradient directions build up speed, noisy orthogonal directions cancel out. This helps navigate ravines where curvature differs sharply across dimensions.

**RMSProp** scales updates by the recent magnitude of gradients per parameter: parameters with large recent gradients get smaller steps, parameters with small gradients get larger steps. This equalizes update sizes across parameters with very different gradient scales.

**Adam** combines both: it uses a momentum term (1st moment) to smooth direction and an RMSProp-like term (2nd moment) to normalize scale — giving it the benefits of both. Bias correction compensates for the fact that the moment estimates are initialized at zero and are therefore too small early in training.

## Formal Description

Let $g_t = \nabla_\theta J_t(\theta)$ be the gradient at step $t$.

---

**Momentum:**
$$
v_t \leftarrow \beta_1 v_{t-1} + (1-\beta_1)\, g_t
$$

$$
\theta \leftarrow \theta - \alpha\, v_t
$$
Typical $\beta_1 = 0.9$. Initialise $v_0 = 0$.

---

**RMSProp:**
$$
s_t \leftarrow \beta_2 s_{t-1} + (1-\beta_2)\, g_t^2
$$

$$
\theta \leftarrow \theta - \alpha\, \frac{g_t}{\sqrt{s_t + \epsilon}}
$$
Typical $\beta_2 = 0.999$, $\epsilon = 10^{-8}$. Initialise $s_0 = 0$.

---

**Adam** (Adaptive Moment Estimation):

Maintain both 1st and 2nd moment estimates:
$$
v_t \leftarrow \beta_1 v_{t-1} + (1-\beta_1)\, g_t \qquad \text{(1st moment / momentum)}
$$

$$
s_t \leftarrow \beta_2 s_{t-1} + (1-\beta_2)\, g_t^2 \qquad \text{(2nd moment / RMSProp)}
$$

Bias correction (compensates for zero initialisation):
$$
\hat{v}_t = \frac{v_t}{1 - \beta_1^t}, \qquad \hat{s}_t = \frac{s_t}{1 - \beta_2^t}
$$

Parameter update:
$$
\theta \leftarrow \theta - \alpha\, \frac{\hat{v}_t}{\sqrt{\hat{s}_t} + \epsilon}
$$

**Default hyperparameters:** $\beta_1 = 0.9$, $\beta_2 = 0.999$, $\epsilon = 10^{-8}$, $\alpha = 10^{-3}$.

## Applications

- Training deep networks across virtually all domains (vision, NLP, RL)
- Adam is the default choice in most modern research and production work
- Momentum (with SGD) is still widely used in computer vision where careful tuning yields strong generalization
- RMSProp was historically popular for RNNs before Adam became dominant

## Trade-offs

- **Generalization:** Adam can generalize slightly worse than SGD+momentum on some tasks (e.g., image classification benchmarks); SGD+momentum with a tuned schedule sometimes reaches lower test error
- **Epsilon sensitivity:** the $\epsilon$ stabilizer affects update magnitude when $\hat{s}_t$ is very small; too small $\epsilon$ can cause instability, too large dampens adaptation
- **Momentum hyperparameter:** $\beta_1$ governs how much past gradients matter; high values (0.95+) can cause the optimizer to overshoot in quickly changing loss landscapes
- **Memory overhead:** Adam stores two extra gradient moment vectors per parameter, doubling memory vs. plain SGD (significant for very large models)
- **Learning rate schedules** still matter with Adam; cosine annealing or linear warmup + decay are common choices — see [[01_foundations/06_deep_learning_theory/gradient_descent]]

## Links

- [[01_foundations/06_deep_learning_theory/gradient_descent]]
- [[01_foundations/06_deep_learning_theory/backpropagation]]
