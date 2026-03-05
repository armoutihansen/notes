---
layer: 01_foundations
type: concept
status: seed
tags: [numpy, linear_algebra, vectorization, implementation]
created: 2026-03-02
---

# Vectorization and Broadcasting

## Definition

**Vectorization:** rewriting computations over datasets as matrix/tensor operations to exploit BLAS/GPU parallelism instead of explicit Python loops. **Broadcasting:** extending elementwise operations to arrays of different shapes by implicitly expanding singleton dimensions, according to a fixed set of compatibility rules.

## Intuition

Python loops are 10–1000× slower than equivalent NumPy/BLAS operations due to interpreter overhead. Vectorization is not premature optimization in ML — it is necessary for feasible training times even on small datasets. Broadcasting lets you write these vectorized operations concisely without manually tiling arrays.

## Formal Description

**Vectorized logistic regression** (over $m$ examples, $W \in \mathbb{R}^{n_x}$, $X \in \mathbb{R}^{n_x \times m}$):

$$Z = W^\top X + b, \qquad A = \sigma(Z)$$

$$dW = \frac{1}{m}X(A-Y)^\top, \qquad db = \frac{1}{m}\sum(A-Y)$$

Here $b \in \mathbb{R}$ broadcasts across all $m$ examples automatically.

**Broadcasting rules:** two shapes are compatible dimension-by-dimension (from the trailing dimension) if they are equal or one of them is 1; missing leading dimensions are treated as 1. Examples:

- $(3, 4) + (1, 4)$ → broadcasts to $(3, 4)$ ✓
- $(3, 4) + (4,)$ → trailing dim matches, treated as $(1, 4)$ → $(3, 4)$ ✓
- $(3, 4) + (4, 3)$ → incompatible ✗

**Common patterns:**
- Adding a bias vector $(n, 1)$ to an activation matrix $(n, m)$: broadcasts over the $m$ columns
- Scaling each sample by a per-sample scalar $(m,)$ against a matrix $(n, m)$: requires reshaping to $(1, m)$

## Applications

Forward and backward passes over mini-batches; any batch computation. NumPy, PyTorch, and JAX all follow NumPy broadcasting semantics, so the same intuitions apply across frameworks.

## Trade-offs

Unexpected broadcasting can introduce silent bugs: shape $(n,)$ and shape $(n, 1)$ behave differently when combined with a $(n, m)$ matrix. Always check shapes explicitly (e.g., `assert W.shape == (n, 1)`). Large intermediate tensors from broadcasting can cause memory pressure on GPU.

## Links

- [[01_foundations/_legacy/deep_learning_theory/neural_network_notation]]
- [[01_foundations/_legacy/deep_learning_theory/backpropagation]]
