---
layer: 01_foundations
type: concept
status: growing
tags: [spectral-theorem, symmetric-matrix, psd, positive-semidefinite, eigendecomposition]
created: 2026-03-06
---

# Spectral Theorem and Symmetric Matrices

## Definition

A real symmetric matrix $A = A^\top \in \mathbb{R}^{n \times n}$ is always orthogonally diagonalisable:

$$A = Q \Lambda Q^\top$$

where $Q$ is orthogonal ($Q^\top Q = I$, columns are orthonormal eigenvectors) and $\Lambda = \text{diag}(\lambda_1, \ldots, \lambda_n)$ contains real eigenvalues. This is the **spectral theorem** for real symmetric matrices.

## Intuition

Symmetric matrices arise whenever a quantity is "self-adjoint" — e.g., covariance matrices, Hessians, graph Laplacians. The spectral theorem says these matrices are always nicely diagonalisable: their eigenvectors are mutually perpendicular and span the whole space, and all eigenvalues are real. There's no rotation; the matrix simply stretches along orthogonal axes.

## Formal Description

**Spectral decomposition:**

$$A = \sum_{i=1}^n \lambda_i\, q_i q_i^\top$$

The outer products $q_i q_i^\top$ are rank-1 orthogonal projection matrices. $A$ is a weighted sum of projections onto its eigendirections, with eigenvalues as weights.

**Positive semidefinite (PSD) matrices:**

$A$ is PSD if $\mathbf{x}^\top A \mathbf{x} \geq 0$ for all $\mathbf{x} \in \mathbb{R}^n$. Equivalent conditions:
- All eigenvalues $\lambda_i \geq 0$
- $A = B^\top B$ for some matrix $B$ (Cholesky-like factorisation)
- All principal minors are non-negative

**Positive definite (PD):** $\mathbf{x}^\top A \mathbf{x} > 0$ for all $\mathbf{x} \neq 0$; all eigenvalues $> 0$; $A$ is invertible.

**Covariance matrices** $\Sigma = \frac{1}{n} X^\top X$ (centred data) are always PSD: for any $v$, $v^\top \Sigma v = \frac{1}{n}\|Xv\|^2 \geq 0$.

**Cholesky decomposition:** every PD matrix $A$ factors as $A = L L^\top$ where $L$ is lower-triangular with positive diagonal. More numerically stable than eigendecomposition for solving linear systems.

**Functions of symmetric matrices:** via spectral decomposition, matrix functions are well-defined:

$$f(A) = Q\,\text{diag}(f(\lambda_1), \ldots, f(\lambda_n))\,Q^\top$$

Examples: $A^{1/2} = Q \Lambda^{1/2} Q^\top$ (matrix square root), $e^A = Q e^\Lambda Q^\top$.

**Rayleigh quotient:**

$$R(A, \mathbf{x}) = \frac{\mathbf{x}^\top A \mathbf{x}}{\mathbf{x}^\top \mathbf{x}}$$

$\lambda_{\min} \leq R(A,\mathbf{x}) \leq \lambda_{\max}$; the maximum is attained at the leading eigenvector (used in power iteration and PCA).

## Applications

| Concept | Role of spectral theorem |
|---|---|
| PCA | Eigendecomposition of covariance matrix; eigenvectors = principal directions |
| Kernel methods | Kernel matrix $K_{ij}=k(x_i,x_j)$ is PSD; Mercer's theorem guarantees spectral expansion |
| Quadratic forms in optimization | Hessian $H = \nabla^2 f$ is symmetric; PD Hessian ↔ strictly convex, unique minimum |
| Graph Laplacian | $L = D - W$ symmetric PSD; eigenvalues encode connectivity |
| Gaussian distributions | Covariance $\Sigma$ must be PSD for $\mathcal{N}(\mu, \Sigma)$ to be valid |

## Trade-offs

- The spectral theorem only applies to symmetric (or normal) matrices; general square matrices require Jordan normal form, not a clean eigendecomposition.
- Numerical eigendecomposition of symmetric matrices is well-conditioned and reliable; use `numpy.linalg.eigh` (not `eig`) for symmetric matrices — it's faster and guarantees real eigenvalues.
- PSD testing in practice: add small $\epsilon I$ (Tikhonov regularisation) to handle numerical near-singularity.

## Links

- [[eigenvalues_and_eigenvectors|Eigenvalues and Eigenvectors]]
- [[svd|Singular Value Decomposition]] — SVD of symmetric PSD $A$ coincides with eigendecomposition
- [[01_foundations/01_linear_algebra/01_vector_spaces/orthogonal_projections|Orthogonal Projections]]
- [[01_foundations/04_optimization/convex_optimization|Convex Optimization]] — PD Hessian ↔ strictly convex function
- [[01_foundations/03_probability_and_statistics/probability_distributions|Multivariate Gaussian (covariance must be PSD)]]
