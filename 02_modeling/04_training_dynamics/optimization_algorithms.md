---
layer: 02_modeling
type: concept
status: seed
tags: [optimization, gradient-descent, sgd, momentum, adam, rmsprop, learning-rate, training]
created: 2026-03-02
---

# Optimization Algorithms

## Definition

Iterative algorithms that update model parameters to minimize a training loss. They differ in how the gradient signal is used: how many examples contribute per step, whether past gradients are accumulated, and whether per-parameter learning rates are adapted.

## Intuition

Imagine navigating a hilly loss surface. At each step you estimate the local slope and take a step downhill. The key questions are: how accurate is that slope estimate (batch size), how fast do you move (learning rate), and do you build up momentum to glide through valleys (adaptive methods)?

**Momentum** smooths out oscillations by accumulating a velocity vector — frequent gradient directions build speed; noisy orthogonal fluctuations cancel out.

**RMSProp** equalizes update sizes across parameters by dividing by the recent RMS of each parameter's gradient — parameters with large recent gradients get smaller steps and vice versa.

**Adam** combines both: a momentum term for direction, an RMSProp term for scale normalization, and bias correction for the cold-start problem of zero-initialized moment estimates.

## Formal Description

### Batch Variants of Gradient Descent

Let $J(\theta)$ be the loss. The general update rule is $\theta \leftarrow \theta - \alpha \nabla_\theta J$.

| Variant | Batch size $B$ | Notes |
|---|---|---|
| Full-batch GD | $B = m$ | Exact gradient; slow, impractical for large data |
| Stochastic GD (SGD) | $B = 1$ | Very noisy; many updates per epoch; no GPU parallelism benefit |
| Mini-batch GD | $1 < B < m$ | Standard; typical $B \in \{32, 64, 128, 256\}$ (powers of 2 for GPU efficiency) |

Smaller batches act as a regularizer (noise); larger batches converge to sharper minima that may generalize worse.

---

### Learning Rate Schedules

Decaying $\alpha$ over training improves final convergence:

- **Step decay:** $\alpha_t = \alpha_0 \cdot \gamma^{\lfloor t/k \rfloor}$
- **Exponential decay:** $\alpha_t = \alpha_0 \cdot e^{-\lambda t}$
- **Cosine annealing:** $\alpha_t = \alpha_{\min} + \frac{1}{2}(\alpha_{\max} - \alpha_{\min})\left(1 + \cos\frac{\pi t}{T}\right)$

Linear warmup (ramp $\alpha$ from 0 over the first few thousand steps) is standard before large-scale training to avoid early instability.

---

### Momentum

$$v_t \leftarrow \beta_1 v_{t-1} + (1-\beta_1)\, g_t, \qquad \theta \leftarrow \theta - \alpha\, v_t$$

Typical $\beta_1 = 0.9$. Initialise $v_0 = 0$.

---

### RMSProp

$$s_t \leftarrow \beta_2 s_{t-1} + (1-\beta_2)\, g_t^2, \qquad \theta \leftarrow \theta - \alpha\, \frac{g_t}{\sqrt{s_t + \epsilon}}$$

Typical $\beta_2 = 0.999$, $\epsilon = 10^{-8}$.

---

### Adam (Adaptive Moment Estimation)

Maintain 1st (momentum) and 2nd (RMSProp) moment estimates with bias correction:

$$v_t \leftarrow \beta_1 v_{t-1} + (1-\beta_1)\, g_t$$

$$s_t \leftarrow \beta_2 s_{t-1} + (1-\beta_2)\, g_t^2$$

$$\hat v_t = \frac{v_t}{1-\beta_1^t}, \quad \hat s_t = \frac{s_t}{1-\beta_2^t}$$

$$\theta \leftarrow \theta - \alpha\, \frac{\hat v_t}{\sqrt{\hat s_t} + \epsilon}$$

**Default hyperparameters:** $\beta_1 = 0.9$, $\beta_2 = 0.999$, $\epsilon = 10^{-8}$, $\alpha = 10^{-3}$.

AdamW is a variant that correctly decouples weight decay from the adaptive update (standard in transformer training).

## Applications

- **Adam/AdamW**: default for transformers, NLP, most research
- **SGD + momentum**: still competitive in computer vision; can reach lower test error with careful LR schedule tuning
- **RMSProp**: historically popular for RNNs; largely superseded by Adam

## Trade-offs

- **Generalization gap**: Adam can generalize worse than SGD+momentum on some image benchmarks due to convergence to sharper minima
- **Memory**: Adam requires two extra moment vectors per parameter — significant for very large models
- **Epsilon sensitivity**: too small $\epsilon$ causes instability when $\hat s_t \approx 0$; too large dampens adaptation
- **LR schedules still matter** even with Adam; the adaptive rate does not eliminate the need for a good base LR

## Links

- [[hyperparameter_tuning|Hyperparameter Tuning]]
- [[regularization|Regularization]]
- [[02_modeling/index|Modeling]]
- [[01_foundations/04_optimization/gradient_descent_optimization|Gradient Descent and Variants — SGD, momentum, schedules]]
- [[01_foundations/06_deep_learning_theory/adaptive_optimizers|Adaptive Optimizers — Adam, AdamW, RMSProp]]
- [[01_foundations/04_optimization/convex_optimization|Convex Optimization — loss landscape theory]]
