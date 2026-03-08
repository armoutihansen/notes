---
layer: 01_foundations
type: concept
status: seed
tags: [theory]
created: 2026-03-02
---

# Neural Network Notation

## Definition

The standard indexing and shape conventions used throughout deep learning formulations, covering layer indices, example indices, time steps, and matrix dimensions for both parameters and activations.

## Intuition

Consistent notation prevents shape bugs and makes it straightforward to verify implementations — if you know the convention, you can derive the shape of any tensor at any point in a network from first principles.

## Formal Description

**Index superscripts:**
- $(i)$ = i-th training example
- $[l]$ = l-th layer (1-indexed from input layer)
- $\langle t \rangle$ = t-th time step (sequence models)

**Activations and pre-activations:**

$$
z^{[l]} = W^{[l]}a^{[l-1]} + b^{[l]}, \qquad a^{[l]} = g^{[l]}(z^{[l]})
$$

with $a^{[0]} = x$ (the input).

**Layer sizes:** $n^{[l]}$ = number of units in layer $l$; $n^{[0]} = n_x$ = input dimension; $L$ = total number of layers.

**Matrix shapes (vectorized over $m$ examples):**

| Tensor | Shape | Description |
|--------|-------|-------------|
| $X$ | $\mathbb{R}^{n_x \times m}$ | features × examples |
| $W^{[l]}$ | $\mathbb{R}^{n^{[l]} \times n^{[l-1]}}$ | output units × input units |
| $b^{[l]}$ | $\mathbb{R}^{n^{[l]} \times 1}$ | broadcast over examples |
| $Z^{[l]}, A^{[l]}$ | $\mathbb{R}^{n^{[l]} \times m}$ | activations over batch |

**Gradient shapes:** $dW^{[l]}$ has the same shape as $W^{[l]}$; $db^{[l]}$ has the same shape as $b^{[l]}$.

## Applications

Use as a reference when implementing forward and backward passes. Verify correctness by checking that shapes are consistent at each layer: $W^{[l]} A^{[l-1]}$ requires the inner dimensions to match ($n^{[l-1]}$).

## Trade-offs

Conventions differ between frameworks — PyTorch's `nn.Linear` stores weight matrices transposed relative to this convention (shape $(n^{[l-1]}, n^{[l]})$). Be consistent within a codebase and document which convention is in use.

## Links

- [[01_foundations/06_deep_learning_theory/backpropagation]]
- [[01_foundations/06_deep_learning_theory/vectorization]]
