---
layer: 01_foundations
type: concept
status: growing
tags: [algorithm]
created: 2026-03-06
---

# Activation Functions

## Definition

An **activation function** $\phi: \mathbb{R} \to \mathbb{R}$ applied element-wise after a linear transformation introduces **non-linearity** into a neural network. Without activation functions, a deep network collapses to a single linear map regardless of depth.

## Intuition

A linear layer computes a linear combination of inputs. Stacking many linear layers does nothing that one linear layer couldn't do. Activation functions break linearity, enabling the network to approximate arbitrary functions (universal approximation theorem). The choice of activation affects gradient flow, training speed, and expressiveness.

## Formal Description

**Sigmoid:**

$$
\sigma(z) = \frac{1}{1 + e^{-z}} \in (0,1)
$$

Output bounded in $(0,1)$ — historically used for binary classification output and LSTMs. Saturates for $|z| \gg 0$, causing vanishing gradients in deep networks.

**Tanh:**

$$
\tanh(z) = \frac{e^z - e^{-z}}{e^z + e^{-z}} \in (-1,1)
$$

Zero-centred (better than sigmoid for hidden units); still saturates. $\tanh(z) = 2\sigma(2z) - 1$.

**ReLU (Rectified Linear Unit):**

$$
\text{ReLU}(z) = \max(0, z)
$$

Currently the most widely used. Advantages: sparse activations, no saturation for $z > 0$, cheap to compute. Problem: **dying ReLU** — if $z$ is always negative (e.g., large negative bias), the gradient is always zero and the neuron never updates. Can be mitigated by careful initialization.

**Leaky ReLU:**

$$
\text{LeakyReLU}(z) = \max(\alpha z, z), \quad \alpha \approx 0.01
$$

Allows a small gradient for $z < 0$, addressing dying ReLU. Parametric ReLU (PReLU) learns $\alpha$.

**ELU (Exponential Linear Unit):**

$$
\text{ELU}(z) = \begin{cases} z & z \geq 0 \\ \alpha(e^z - 1) & z < 0 \end{cases}
$$

Smooth, negative-side saturation pushes mean activations towards zero, speeding up learning.

**GELU (Gaussian Error Linear Unit):**

$$
\text{GELU}(z) = z \cdot \Phi(z) \approx 0.5z\!\left(1 + \tanh\!\left[\sqrt{2/\pi}(z + 0.044715z^3)\right]\right)
$$

where $\Phi$ is the Gaussian CDF. Smooth, non-monotone; used in BERT, GPT, and most modern transformers. Outperforms ReLU on many benchmarks.

**Swish / SiLU:**

$$
\text{Swish}(z) = z \cdot \sigma(z)
$$

Self-gated, smooth, unbounded above; used in EfficientNet, some LLMs. Very close to GELU empirically.

**Universal Approximation Theorem:** a feed-forward network with one hidden layer of sufficient width and a non-polynomial continuous activation function can approximate any continuous function on a compact domain to arbitrary accuracy. Depth analogues (depth-separation theorems) show exponential advantages for deep networks over shallow ones.

**Softmax** (output layer, multi-class):

$$
\text{softmax}(\mathbf{z})_i = \frac{e^{z_i}}{\sum_j e^{z_j}}
$$

Converts logits to a probability simplex. For numerical stability, compute $\text{softmax}(\mathbf{z} - \max(\mathbf{z}))$.

## Applications

| Use case | Recommended activation |
|---|---|
| Hidden layers (general) | ReLU or GELU |
| Transformers (FFN layers) | GELU, SwiGLU |
| Recurrent networks (LSTM gates) | Sigmoid, tanh |
| Binary classification output | Sigmoid |
| Multi-class output | Softmax |
| Regression output | Linear (none) |

## Trade-offs

| Activation | Pros | Cons |
|---|---|---|
| Sigmoid | Smooth, bounded | Saturates, not zero-centred |
| ReLU | Fast, sparse | Dying ReLU, not smooth |
| GELU | Smooth, strong empirical performance | Slower to compute than ReLU |
| Softmax | Valid probability output | Can saturate (peaked distributions) |

## Links

- [[weight_initialization|Weight Initialization]] — initialization must account for activation gain (Kaiming for ReLU, Xavier for sigmoid/tanh)
- [[backpropagation|Backpropagation]] — gradient of activation must be non-zero for signal to propagate
- [[batch_normalization|Batch Normalization]] — often placed before or after activation; order matters
- [[01_foundations/06_deep_learning_theory/gradient_descent|Gradient Descent]] — activation choice directly affects gradient flow
