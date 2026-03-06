---
layer: 01_foundations
type: concept
status: seed
tags: [algorithm]
created: 2026-03-01
---

# Inverse Matrix

## Definition

A square matrix $\boldsymbol{A}$ is **invertible** (non-singular) if there exists a matrix $\boldsymbol{A}^{-1}$ such that
$$
\boldsymbol{A}\boldsymbol{A}^{-1}=\boldsymbol{A}^{-1}\boldsymbol{A}=\boldsymbol{I}
$$

## Intuition

The inverse undoes the transformation encoded by $\boldsymbol{A}$. Geometrically, if $\boldsymbol{A}$ stretches and rotates space, $\boldsymbol{A}^{-1}$ applies the exact reverse — restoring the original vectors. A matrix is invertible when it neither collapses any dimension to zero (full rank) nor maps multiple inputs to the same output (injective map), i.e. $\det\boldsymbol{A}\neq 0$.

## Formal Description

For a $2\times 2$ matrix, solving $\boldsymbol{A}\boldsymbol{A}^{-1}=\boldsymbol{I}$ yields:
$$
\begin{pmatrix}a & b\\ c & d\end{pmatrix}^{-1}=\frac{1}{ad-bc}\begin{pmatrix}d & -b\\ -c & a\end{pmatrix}
$$

The scalar $ad-bc=\det\boldsymbol{A}$ is the **determinant** of $\boldsymbol{A}$: the product of the diagonals minus the product of the off-diagonals.

- $\boldsymbol{A}$ is invertible if and only if $\det\boldsymbol{A}\neq 0$.
- $(\boldsymbol{AB})^{-1}=\boldsymbol{B}^{-1}\boldsymbol{A}^{-1}$
- If $\boldsymbol{A}$ is invertible, so is $\boldsymbol{A}^{\top}$, and $(\boldsymbol{A}^{\top})^{-1}=(\boldsymbol{A}^{-1})^{\top}$.

## Applications

- Solving linear systems $\boldsymbol{Ax}=\boldsymbol{b}$: if $\boldsymbol{A}$ is invertible, then $\boldsymbol{x}=\boldsymbol{A}^{-1}\boldsymbol{b}$.
- LU decomposition factorises $\boldsymbol{A}$ to compute solutions without explicitly forming $\boldsymbol{A}^{-1}$.

## Trade-offs

- The inverse **does not exist** when $\det\boldsymbol{A}=0$ (singular matrix); the linear system $\boldsymbol{Ax}=\boldsymbol{b}$ then has either no solution or infinitely many.
- Explicitly computing $\boldsymbol{A}^{-1}$ is numerically expensive ($O(n^3)$) and unstable for ill-conditioned matrices; in practice, solve $\boldsymbol{Ax}=\boldsymbol{b}$ directly using LU or QR decomposition instead.
- For non-square matrices, only the **pseudoinverse** $\boldsymbol{A}^{+}$ exists; it minimises $\|\boldsymbol{Ax}-\boldsymbol{b}\|$ in the least-squares sense.

## Links

- [[matrix_operations|Matrix Operations]]
- [[special_matrices|Special Matrices]]
- [[lu_decomposition|LU Decomposition]]
