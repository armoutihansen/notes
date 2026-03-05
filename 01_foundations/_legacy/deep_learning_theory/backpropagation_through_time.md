---
layer: 01_foundations
type: concept
status: seed
tags: [BPTT, RNN, recurrent-networks, vanishing-gradient, exploding-gradient]
created: 2026-03-02
---

# Backpropagation Through Time

## Definition

Backpropagation applied to unrolled recurrent networks; gradients flow backward through both layers and time steps.

## Intuition

An RNN unrolled over $T$ steps is a deep network of depth $T$, where all time steps share the same weights. Gradients must traverse the entire time dimension, passing through the same weight matrix repeatedly — this is what causes both vanishing and exploding gradients.

## Formal Description

**Unrolled RNN:** at each step $t$:

$$a^{\langle t \rangle} = g\!\left(W_{aa}\,a^{\langle t-1 \rangle} + W_{ax}\,x^{\langle t \rangle} + b_a\right), \qquad J = \sum_t \ell^{\langle t \rangle}$$

**Gradient of $W_{aa}$** involves products of Jacobians across all time steps:

$$\frac{\partial a^{\langle T \rangle}}{\partial a^{\langle 0 \rangle}} = \prod_{t=1}^T W_{aa}^\top \operatorname{diag}\!\left(g'\!\left(z^{\langle t \rangle}\right)\right)$$

**Vanishing gradient:** when $\|W_{aa}\| < 1$ repeatedly, the product shrinks exponentially → early time steps receive near-zero gradient → model cannot learn long-range dependencies.

**Exploding gradient:** when $\|W_{aa}\| > 1$ repeatedly, the product grows exponentially → numerical instability → remedy: gradient clipping by norm.

**Truncated BPTT:** backpropagate only $k$ steps back in time to reduce memory usage and gradient path length. Introduces bias in gradient estimates but is often sufficient in practice.

## Applications

- Training vanilla RNNs, LSTMs, and GRUs on sequence tasks (language modeling, time-series, speech)
- Any model where dependencies span multiple discrete time steps

## Trade-offs

- Gradient vanishing/explosion makes training long sequences with vanilla RNNs difficult
- LSTMs and GRUs mitigate vanishing gradients via gating mechanisms (additive updates through the cell state)
- Transformers avoid BPTT entirely via self-attention, enabling direct gradient flow between any two positions
- Truncated BPTT reduces memory and stabilizes training but introduces gradient bias and may prevent learning very long-range dependencies

## Links

- [[01_foundations/_legacy/deep_learning_theory/backpropagation]]
- [[01_foundations/_legacy/deep_learning_theory/weight_initialization]]
- [[recurrent_networks]] (in 02_modeling/sequence_models)
