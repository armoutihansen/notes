---
layer: 01_foundations
type: concept
status: growing
tags: [algorithm, training]
created: 2026-03-02
---

# Batch Normalization

## Definition

Normalizes pre-activations within a mini-batch using batch statistics, then rescales with learned parameters $(\gamma, \beta)$. Subsumes input normalization as a special case applied at the data layer.

## Intuition

**Input normalization:** equalizing feature scales removes elongated, axis-misaligned loss contours that force small learning rates and cause slow, zigzagging descent. The same principle applied deep inside a network is batch normalization. As activations shift during training (internal covariate shift), each layer must constantly readapt to a moving distribution. Batch norm pins each layer's input distribution near zero mean and unit variance, smoothing the optimization landscape and permitting higher learning rates.

## Formal Description

**Input normalization** (applied to raw features before training):
$$
x_j \leftarrow \frac{x_j - \mu_j}{\sigma_j}
$$
where $\mu_j$ and $\sigma_j$ are the mean and standard deviation of feature $j$ over the training set. Ensures all input features have comparable scale, improving gradient descent convergence.

**Batch normalization** (applied to pre-activations $z^{[l]}$ at layer $l$):

*Forward pass (training):*
$$
\mu_\mathcal{B} = \frac{1}{B}\sum_{i=1}^{B} z_i, \qquad
\sigma_\mathcal{B}^2 = \frac{1}{B}\sum_{i=1}^{B}(z_i - \mu_\mathcal{B})^2
$$

$$
\hat{z}_i = \frac{z_i - \mu_\mathcal{B}}{\sqrt{\sigma_\mathcal{B}^2 + \epsilon}}, \qquad
y_i = \gamma\,\hat{z}_i + \beta
$$
$\gamma$ and $\beta$ are learned per-feature parameters; $\epsilon \approx 10^{-8}$ prevents division by zero.

*Test time:* replace $\mu_\mathcal{B}, \sigma_\mathcal{B}^2$ with exponential moving averages accumulated during training:
$$
\mu_{\text{EMA}} \leftarrow \rho\,\mu_{\text{EMA}} + (1-\rho)\,\mu_\mathcal{B}
$$
This makes inference deterministic and independent of batch composition.

*Placement:* typically inserted before the activation function (e.g., Linear → BN → ReLU), though post-activation is also used.

## Applications

- Virtually all modern deep networks (ResNets, Transformers use it or a variant)
- Enables significantly deeper networks by stabilizing gradient flow
- Acts as a mild regularizer, sometimes reducing the need for dropout

## Trade-offs

- **Noise as regularization:** batch statistics introduce per-batch noise, which has a regularizing effect but also means training and test behavior differ
- **Train vs. test mismatch:** moving average estimates must be tracked carefully; bugs here are a common source of subtle errors
- **Small batch sizes:** batch statistics become unreliable; prefer **Layer Normalization** (normalizes over features instead of batch) for small batches, RNNs, and Transformers
- **Overhead:** adds parameters $(\gamma, \beta)$ and a forward-pass computation per layer, though cost is generally negligible
- [[01_foundations/06_deep_learning_theory/weight_initialization]] becomes less critical when batch norm is used, since activations are re-centered at each layer

## Links

- [[01_foundations/06_deep_learning_theory/gradient_descent]]
- [[01_foundations/06_deep_learning_theory/weight_initialization]]
- [[01_foundations/06_deep_learning_theory/adaptive_optimizers]]
