---
layer: 01_foundations
type: concept
status: growing
tags: [algorithm, training]
created: 2026-03-02
---

# Weight Initialization

## Definition

The strategy for setting initial parameter values before training begins. Critical for two reasons: breaking symmetry (so neurons learn different features) and controlling gradient magnitudes (so gradients neither vanish nor explode during backprop).

## Intuition

All-zero initialization causes symmetry — every neuron in a layer receives identical gradients and learns the same thing forever. Too-large initialization causes exploding gradients as the signal amplifies through layers. Too-small initialization causes vanishing gradients as the signal attenuates. Good initialization keeps activations and gradients at a reasonable scale throughout the entire network depth.

## Formal Description

**Symmetry breaking:** If $W^{[l]}$ is all-zeros, $dW^{[l]}$ is identical for all neurons → they never differentiate. Must use random initialization.

**Vanishing/exploding gradients:** Gradients scale as $\prod_l W^{[l]}$; if weights have $|w| < 1$, gradients vanish exponentially with depth; if $|w| > 1$, they explode exponentially.

**Xavier/Glorot init:** $W \sim \mathcal{N}(0,\, 1/n^{[l-1]})$ — designed for linear/tanh activations; keeps variance of activations constant across layers.

**He init:** $W \sim \mathcal{N}(0,\, 2/n^{[l-1]})$ — designed for ReLU activations; the factor of 2 accounts for the dead half of ReLU (which zeroes out half the activations in expectation).

**Biases:** Initialized to zero (no symmetry issue for biases since weights already break symmetry).

## Applications

Initialization is applied once before training. He init is the standard choice for ReLU networks. Xavier/Glorot is preferred for tanh/sigmoid networks. Residual networks are less sensitive to initialization due to skip connections providing identity paths for gradients.

## Trade-offs

Poor initialization can make training practically impossible or extremely slow to recover from. Good initialization does not replace batch normalization for very deep networks — both are typically needed. Orthogonal initialization can help RNNs maintain gradient flow over long sequences.

## Links

- [[01_foundations/06_deep_learning_theory/backpropagation]]
- [[01_foundations/06_deep_learning_theory/batch_normalization]]
- [[01_foundations/06_deep_learning_theory/adaptive_optimizers]]
