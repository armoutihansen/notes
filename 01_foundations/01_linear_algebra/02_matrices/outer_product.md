---
layer: 01_foundations
type: concept
status: seed
tags: [theory]
created: 2026-03-01
---

# Outer Product

## Definition

The **outer product** of two column vectors produces a matrix; it is the reverse of the inner product in the sense that it expands rather than contracts dimensionality.

## Intuition

Where the inner product asks "how aligned are these two vectors?" and returns a number, the outer product asks "what matrix captures every pairwise interaction between the components of these two vectors?" The result is always a rank-1 matrix — a single "direction" stretched across both vector spaces simultaneously. Sums of outer products build up higher-rank matrices and underlie SVD and low-rank approximations.

## Formal Description

For $3\times 1$ column vectors $\boldsymbol{u},\boldsymbol{v}$:
$$
\boldsymbol{u}\boldsymbol{v}^{\top}=\begin{pmatrix}u_1\\ u_2\\ u_3\end{pmatrix}\begin{pmatrix}v_1 & v_2 & v_3\end{pmatrix}=\begin{pmatrix}u_1v_1 & u_1v_2 & u_1v_3\\ u_2v_1 & u_2v_2 & u_2v_3\\ u_3v_1 & u_3v_2 & u_3v_3\end{pmatrix}
$$

- Every column of $\boldsymbol{u}\boldsymbol{v}^{\top}$ is a scalar multiple of $\boldsymbol{u}$; every row is a scalar multiple of $\boldsymbol{v}^{\top}$.
- The resulting matrix has rank at most 1.
- For vectors $\boldsymbol{u}\in\mathbb{R}^m$ and $\boldsymbol{v}\in\mathbb{R}^n$, the outer product $\boldsymbol{u}\boldsymbol{v}^{\top}\in\mathbb{R}^{m\times n}$.

## Applications

- SVD expresses any matrix as a weighted sum of rank-1 outer products: $\boldsymbol{A}=\sum_i \sigma_i \boldsymbol{u}_i\boldsymbol{v}_i^{\top}$.
- Low-rank approximations truncate this sum to compress matrices (image compression, collaborative filtering).
- Covariance matrices are sums of outer products of mean-centred data vectors: $\boldsymbol{\Sigma}=\frac{1}{n}\sum_i \boldsymbol{x}_i\boldsymbol{x}_i^{\top}$.

## Trade-offs

- The outer product always yields a **rank-1** matrix, so it cannot represent general linear maps on its own; a full-rank matrix requires at least $\min(m,n)$ outer products summed together.
- Unlike the inner product, the outer product is **not commutative**: $\boldsymbol{u}\boldsymbol{v}^{\top}\neq\boldsymbol{v}\boldsymbol{u}^{\top}$ in general (the two results have transposed shapes unless $m=n$).
- Memory cost is $O(mn)$ — for large vectors this is expensive compared to storing the two vectors separately ($O(m+n)$).

## Links

- [[inner_product|Inner Product]]
- [[matrix_operations|Matrix Operations]]
