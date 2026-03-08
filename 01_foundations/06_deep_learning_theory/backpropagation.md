---
layer: 01_foundations
type: concept
status: growing
tags: [algorithm, training]
created: 2026-03-02
---

# Backpropagation

## Definition

Reverse-mode automatic differentiation of the loss over a computation graph; efficiently computes all parameter gradients in one backward pass.

## Intuition

The computation graph records every operation during the forward pass. Gradients then flow backward through this graph via the chain rule: each node multiplies the upstream gradient by its local derivative. Because many paths share intermediate nodes, reverse-mode AD reuses computations and computes all gradients in roughly one forward-pass worth of work.

## Formal Description

**Computation graph:** a DAG where nodes are operations or values. If $u = g(v)$, then:

$$
\frac{\partial J}{\partial v} = \frac{\partial J}{\partial u} \cdot g'(v) \quad \text{(upstream gradient × local gradient)}
$$

For nodes with multiple outputs, contributions from all downstream paths are summed.

**Forward pass for layer $l$:**

$$
z^{[l]} = W^{[l]}a^{[l-1]} + b^{[l]}, \qquad a^{[l]} = g^{[l]}(z^{[l]})
$$

**Backward pass for layer $l$:**

$$
dz^{[l]} = da^{[l]} \odot g'^{[l]}(z^{[l]})
$$

$$
dW^{[l]} = \frac{1}{m}\,dz^{[l]}\,(a^{[l-1]})^\top
$$

$$
db^{[l]} = \frac{1}{m}\sum dz^{[l]}
$$

$$
da^{[l-1]} = (W^{[l]})^\top dz^{[l]}
$$

**Computational complexity:** one backward pass costs approximately the same as one forward pass. Forward-mode AD would require one pass per input dimension — infeasible for models with millions of parameters.

**Memory cost:** all intermediate activations $a^{[l]}$ must be retained from the forward pass to compute $dW^{[l]}$ during backprop. This is the dominant memory cost during training.

## Applications

- Training all gradient-based models (neural networks, linear models, etc.)
- Automatic differentiation frameworks (PyTorch `autograd`, JAX, TensorFlow) implement this automatically by tracing the computation graph

## Trade-offs

- Requires storing all intermediate activations — memory cost scales with depth and batch size; gradient checkpointing trades recomputation for memory
- Numerical stability degrades in very deep networks due to vanishing/exploding gradients → use gradient clipping, normalization, and careful initialization
- Bugs most commonly manifest as shape mismatches or sign errors; use [[01_foundations/06_deep_learning_theory/gradient_checking]] to verify implementations

## Links

- [[01_foundations/06_deep_learning_theory/gradient_descent]]
- [[01_foundations/06_deep_learning_theory/weight_initialization]]
- [[01_foundations/06_deep_learning_theory/batch_normalization]]
- [[01_foundations/06_deep_learning_theory/gradient_checking]]
