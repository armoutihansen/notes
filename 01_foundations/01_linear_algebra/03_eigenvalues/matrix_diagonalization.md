---
layer: 01_foundations
type: concept
status: seed
tags: [algorithm]
created: 2026-03-01
---

# Matrix Diagonalization

## Definition

A square matrix $\mathrm{A}$ is **diagonalisable** if it can be written as
$$
\mathrm{A} = \mathrm{S}\Lambda\mathrm{S}^{-1},
$$
where $\mathrm{S}$ is an invertible matrix whose columns are eigenvectors of $\mathrm{A}$, and $\Lambda$ is the diagonal matrix of corresponding eigenvalues. Equivalently, $\Lambda = \mathrm{S}^{-1}\mathrm{AS}$.

## Intuition

Diagonalisation decomposes a transformation into independent "modes": change to the eigenvector basis (via $\mathrm{S}^{-1}$), apply a pure scaling along each axis (via $\Lambda$), then return to the original basis (via $\mathrm{S}$). In the eigenvector basis the transformation is trivially simple — each coordinate is just scaled by its eigenvalue with no mixing between dimensions. This separation of independent modes makes repeated application, inversion, and analysis dramatically easier.

## Formal Description

**Derivation.** Collecting the eigenrelations $\mathrm{Ax}_i = \lambda_i\mathrm{x}_i$ into a single matrix equation gives
$$
\mathrm{AS} = \mathrm{S}\Lambda,
$$
where $\mathrm{S} = [\mathrm{x}_1 \mid \cdots \mid \mathrm{x}_n]$ and $\Lambda = \operatorname{diag}(\lambda_1, \dots, \lambda_n)$. If $\mathrm{S}$ is invertible (i.e.\ the eigenvectors are linearly independent), both sides can be multiplied by $\mathrm{S}^{-1}$ to obtain $\mathrm{A} = \mathrm{S}\Lambda\mathrm{S}^{-1}$.

**Condition for diagonalisability.** An $n\times n$ matrix is diagonalisable if and only if it has $n$ linearly independent eigenvectors. In particular, a matrix with $n$ distinct eigenvalues is always diagonalisable.

**Eigendecomposition.** More generally, the **eigendecomposition** of $\mathrm{A}$ is
$$
\mathrm{A} = \mathrm{V}\operatorname{diag}(\boldsymbol{\lambda})\mathrm{V}^{-1},
$$
where the columns of $\mathrm{V}$ are eigenvectors and $\boldsymbol{\lambda}$ is the vector of eigenvalues. Not every matrix admits such a decomposition over $\mathbb{R}$, but every **real symmetric matrix** can be decomposed as
$$
\mathrm{A} = \mathrm{Q}\Lambda\mathrm{Q}^\top,
$$
where $\mathrm{Q}$ is orthogonal ($\mathrm{Q}^{-1} = \mathrm{Q}^\top$) and its columns are eigenvectors of $\mathrm{A}$. By convention eigenvalues in $\Lambda$ are sorted in descending order; the decomposition is unique (up to sign of eigenvectors) when all eigenvalues are distinct.

**Matrix powers.** Diagonalisation makes computing powers straightforward. For $\mathrm{A} = \mathrm{S}\Lambda\mathrm{S}^{-1}$,
$$
\mathrm{A}^p = \mathrm{S}\Lambda^p\mathrm{S}^{-1},
$$
where $\Lambda^p = \operatorname{diag}(\lambda_1^p, \dots, \lambda_n^p)$. Each repeated application of $\mathrm{A}$ simply raises the diagonal entries of $\Lambda$ to the next power.

**Properties from eigenvalues.**
- $\mathrm{A}$ is singular if and only if at least one eigenvalue is zero.
- $\mathrm{A}$ is **positive definite** if all eigenvalues are positive; **positive semidefinite** if all eigenvalues are non-negative. Positive semidefinite matrices satisfy $\mathrm{x}^\top\mathrm{Ax} \geq 0$ for all $\mathrm{x}$; positive definite matrices additionally have $\mathrm{Ax} = 0 \Rightarrow \mathrm{x} = 0$.
- The eigendecomposition of a real symmetric matrix can be used to optimise quadratic forms $f(\mathrm{x}) = \mathrm{x}^\top\mathrm{Ax}$ subject to $\|\mathrm{x}\|_2 = 1$.

## Applications

- Computing matrix powers and matrix exponentials efficiently (e.g., solving linear ODEs).
- PCA and spectral methods in machine learning rely on eigendecomposition of covariance or kernel matrices.
- Decoupling coupled differential equations into independent scalar equations.
- Checking positive definiteness of matrices (e.g., for valid covariance matrices or convex quadratic forms).

## Trade-offs

- Not all matrices are diagonalisable: a **defective matrix** has a repeated eigenvalue with fewer independent eigenvectors than its algebraic multiplicity. Jordan normal form is the generalisation, but it is more complex to work with.
- Even when diagonalisable, computing the full eigendecomposition costs $O(n^3)$; for very large sparse matrices only a few dominant eigenvalues are typically computed via iterative methods (e.g., Lanczos, Arnoldi).
- Symmetric eigendecomposition ($\mathrm{A} = \mathrm{Q}\Lambda\mathrm{Q}^\top$) is numerically better conditioned and always exists over $\mathbb{R}$; the general (non-symmetric) case can produce complex eigenvalues and an ill-conditioned $\mathrm{S}$.

## Links

- [[eigenvalues_and_eigenvectors|Eigenvalues and Eigenvectors]] — eigenvectors and eigenvalues are the building blocks of diagonalisation
- [[special_matrices|Special Matrices]] — identity matrix used in the eigenvalue problem; orthogonal matrices arise in symmetric eigendecomposition
