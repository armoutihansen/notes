---
layer: 01_foundations
type: concept
status: seed
tags: [theory]
created: 2026-03-01
---

# Norms

## Definition

A **norm** is a function mapping vectors to non-negative values, measuring the "size" or distance from the origin. A function $f$ is a norm if it satisfies:

- $f(\boldsymbol{x}) = 0 \Rightarrow \boldsymbol{x} = 0$
- $f(\boldsymbol{x} + \boldsymbol{y}) \leq f(\boldsymbol{x}) + f(\boldsymbol{y})$ (triangle inequality)
- $\forall \alpha \in \mathbb{R},\ f(\alpha\boldsymbol{x}) = |\alpha| f(\boldsymbol{x})$

## Intuition

Different norms measure "size" with different geometries. The $L^2$ norm measures straight-line distance; the $L^1$ norm measures Manhattan (city-block) distance and is robust to outliers; the $L^\infty$ norm is controlled by the single largest component. Choosing the right norm shapes the geometry of optimisation — $L^1$ encourages sparsity, $L^2$ encourages smoothness.

## Formal Description

The $L^p$ norm is defined as
$$
\|\boldsymbol{x}\|_p = \left(\sum_i |x_i|^p\right)^{1/p}.
$$

**$L^2$ norm (Euclidean norm).** $\|\boldsymbol{x}\|_2 = \|\boldsymbol{x}\|$. The squared $L^2$ norm $\|\boldsymbol{x}\|_2^2$ is often preferred computationally since its derivatives depend only on individual elements.

**$L^1$ norm.** Grows at the same rate everywhere; useful when distinguishing exact zeros from small nonzero values:
$$
\|\boldsymbol{x}\|_1 = \sum_i |x_i|.
$$

**$L^\infty$ norm (max norm).** Equals the absolute value of the largest element:
$$
\|\boldsymbol{x}\|_\infty = \max_i |x_i|.
$$

**Frobenius norm** (for matrices):
$$
\|\boldsymbol{A}\|_F = \sqrt{\sum_{i,j} A_{i,j}^2}.
$$

The dot product can be expressed in terms of the $L^2$ norm and the angle $\theta$ between vectors:
$$
\boldsymbol{x}^\top \boldsymbol{y} = \|\boldsymbol{x}\|_2 \|\boldsymbol{y}\|_2 \cos\theta.
$$

The $L^0$ "norm" (count of nonzero entries) is **not** a true norm, as scaling a vector does not change its nonzero count.

## Applications

- $L^2$ regularisation (ridge / weight decay) penalises large weights and yields smooth solutions.
- $L^1$ regularisation (LASSO) induces sparsity and automatic feature selection.
- $L^\infty$ norm arises in minimax problems and robustness guarantees.

## Trade-offs

$L^2$ is differentiable everywhere and computationally convenient; $L^1$ is non-differentiable at zero, requiring subgradient or proximal methods. $L^p$ norms with $p < 1$ promote stronger sparsity but are non-convex, making optimisation much harder.

## Links

- [[vector_space|Vector Space]]
- [[orthogonal_projections|Orthogonal Projections]]
