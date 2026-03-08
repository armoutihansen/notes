---
layer: 03_modeling
type: concept
status: seed
tags: [algorithm]
created: 2026-03-02
---

# Multi-Layer Perceptron (MLP)

## Definition

A fully-connected feedforward neural network consisting of an input layer, one or more hidden layers, and an output layer; each neuron in a layer is connected to every neuron in the adjacent layers via learned weights, and non-linear activation functions are applied at each hidden layer.

## Intuition

A single linear layer can only represent linear decision boundaries. Stacking layers with non-linear activations (ReLU, sigmoid, tanh) allows the network to compose simple transformations into arbitrarily complex functions. Each hidden layer learns a new representation of the data; deeper layers capture increasingly abstract features. Given enough neurons, even a single hidden layer can approximate any continuous function — but depth makes this far more parameter-efficient.

## Formal Description

**Architecture:** $L$-layer network with weight matrices $W^{[l]} \in \mathbb{R}^{n_l \times n_{l-1}}$ and bias vectors $b^{[l]} \in \mathbb{R}^{n_l}$ for $l = 1, \ldots, L$.

**Forward pass:**

$$
z^{[l]} = W^{[l]} a^{[l-1]} + b^{[l]}, \qquad a^{[l]} = g^{[l]}(z^{[l]})
$$

where $a^{[0]} = x$ (input), $g^{[l]}$ is the activation function at layer $l$.

**Common activation functions:**
- **ReLU:** $g(z) = \max(0, z)$ — most widely used in hidden layers; avoids vanishing gradients for positive inputs
- **Sigmoid:** $g(z) = 1/(1+e^{-z})$ — used for binary output; saturates at extremes
- **Tanh:** $g(z) = (e^z - e^{-z})/(e^z + e^{-z})$ — zero-centered; preferred over sigmoid for hidden layers historically
- **Softmax:** $g(z_j) = e^{z_j}/\sum_k e^{z_k}$ — used for multi-class output

**Backpropagation:** apply the chain rule recursively from output to input to compute $\partial \mathcal{L}/\partial W^{[l]}$ and $\partial \mathcal{L}/\partial b^{[l]}$:

$$
\delta^{[L]} = \nabla_{a^{[L]}} \mathcal{L} \odot g'^{[L]}(z^{[L]})
$$

$$
\delta^{[l]} = (W^{[l+1]})^\top \delta^{[l+1]} \odot g'^{[l]}(z^{[l]})
$$

$$
\frac{\partial \mathcal{L}}{\partial W^{[l]}} = \delta^{[l]} (a^{[l-1]})^\top, \qquad \frac{\partial \mathcal{L}}{\partial b^{[l]}} = \delta^{[l]}
$$

Weights are updated via gradient descent (or Adam): $W^{[l]} \leftarrow W^{[l]} - \eta \partial \mathcal{L}/\partial W^{[l]}$.

**Universal Approximation Theorem:** a single hidden layer MLP with a non-polynomial activation and sufficiently many neurons can approximate any continuous function on a compact domain to arbitrary precision. The theorem guarantees expressibility, not trainability or generalization — depth is needed for practical efficiency.

**Parameter count:** for a network with layer sizes $[n_0, n_1, \ldots, n_L]$:

$$
\text{params} = \sum_{l=1}^{L} (n_{l-1} \cdot n_l + n_l) = \sum_{l=1}^{L} n_l(n_{l-1} + 1)
$$

## Applications

Tabular data classification and regression (often outperformed by gradient boosted trees), final classification head in CNNs and Transformers, function approximation, reinforcement learning value/policy networks, autoencoders (encoder/decoder components).

## Trade-offs

- No inductive bias for spatial structure (CNNs) or sequential structure (RNNs/Transformers) — requires more data when structure is present
- Fully connected layers have $O(n_{l-1} \cdot n_l)$ parameters — expensive for high-dimensional inputs (e.g., raw images)
- Susceptible to vanishing gradients without careful initialization (Xavier/He) and normalization (BatchNorm)
- Depth helps expressiveness but complicates optimization; residual connections (ResNet) extend the MLP idea to very deep networks
- Overfitting risk grows with model size; mitigated by dropout, L2 regularization, early stopping

## Links

- [[cnn_architecture]]
- [[recurrent_networks]]
- [[03_modeling/04_deep_learning/index|Deep Learning]]
- [[01_foundations/06_deep_learning_theory/backpropagation|Backpropagation — chain rule, gradient computation]]
- [[01_foundations/06_deep_learning_theory/activation_functions|Activation Functions — ReLU, sigmoid, GELU]]
- [[01_foundations/06_deep_learning_theory/weight_initialization|Weight Initialization — Xavier, Kaiming]]
