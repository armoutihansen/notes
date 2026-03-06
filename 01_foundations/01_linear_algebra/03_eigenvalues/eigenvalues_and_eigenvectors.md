---
layer: 01_foundations
type: concept
status: seed
tags: [theory, algorithm]
created: 2026-03-01
---

# Eigenvalues and Eigenvectors

## Definition

Let $\mathrm{A}$ be an $n\times n$ matrix. A scalar $\lambda$ is an **eigenvalue** of $\mathrm{A}$ if there exists a nonzero column vector $\mathrm{x}$ — called an **eigenvector** — such that
$$
\mathrm{Ax} = \lambda\mathrm{x}.
$$
Multiplication by $\mathrm{A}$ leaves the direction of $\mathrm{x}$ unchanged (or reverses it), scaling it by $\lambda$. If $\mathrm{x}$ is an eigenvector then so is any nonzero scalar multiple $s\mathrm{x}$; eigenvectors are therefore only determined up to scale.

## Intuition

Eigenvectors are the special directions in space that a linear transformation does not rotate — it only stretches or compresses them (and possibly flips them if $\lambda < 0$). Every other vector gets both rotated and scaled, but eigenvectors stay on their own lines through the origin. The eigenvalue $\lambda$ tells you exactly how much stretching occurs along that direction. A transformation can be fully understood once all its eigendirections and associated scales are known.

## Formal Description

**Characteristic equation.** Rewriting the eigenvalue equation using the identity matrix $\mathrm{I}$,
$$
(\mathrm{A} - \lambda\mathrm{I})\,\mathrm{x} = 0.
$$
For nonzero solutions to exist the matrix $(\mathrm{A} - \lambda\mathrm{I})$ must be singular:
$$
\det(\mathrm{A} - \lambda\mathrm{I}) = 0.
$$
This is the **characteristic equation** of $\mathrm{A}$. By the Leibniz formula it is a degree-$n$ polynomial in $\lambda$, so an $n\times n$ matrix has exactly $n$ eigenvalues counted with multiplicity (over $\mathbb{C}$). Once an eigenvalue $\lambda_i$ is found, the corresponding eigenvector $\mathrm{x}_i$ is obtained by solving $(\mathrm{A} - \lambda_i\mathrm{I})\mathrm{x} = 0$.

**2×2 case.** For $\mathrm{A} = \begin{pmatrix} a & b \\ c & d \end{pmatrix}$ the characteristic equation is
$$
\lambda^2 - \operatorname{Tr}\mathrm{A}\,\lambda + \det\mathrm{A} = 0,
$$
where $\operatorname{Tr}\mathrm{A} = a + d$ is the trace. The discriminant $\Delta = (\operatorname{Tr}\mathrm{A})^2 - 4\det\mathrm{A}$ determines the nature of the roots.

**Distinct real eigenvalues** ($\Delta > 0$). The two roots $\lambda_1 \neq \lambda_2$ are real. Eigenvectors corresponding to distinct eigenvalues are linearly independent, so $\mathrm{A}$ is diagonalisable over $\mathbb{R}$.

**Complex conjugate eigenvalues** ($\Delta < 0$). The roots form a conjugate pair $\lambda = \alpha \pm \beta i$ with $\alpha, \beta \in \mathbb{R}$, $\beta \neq 0$. The corresponding eigenvectors are also complex conjugates of each other. A real $2\times 2$ matrix with complex eigenvalues represents a rotation-scaling transformation; it is diagonalisable over $\mathbb{C}$ but not over $\mathbb{R}$.

**Repeated eigenvalue** ($\Delta = 0$). The single eigenvalue $\lambda = \tfrac{1}{2}\operatorname{Tr}\mathrm{A}$ may or may not yield two linearly independent eigenvectors; the matrix may or may not be diagonalisable.

More generally, an $n\times n$ matrix may have fewer than $n$ distinct eigenvalues, and eigenvalues can be real or complex.

## Applications

- Diagonalisation of matrices for efficient computation (e.g., matrix powers, differential equations).
- Principal Component Analysis (PCA): eigenvectors of the covariance matrix are the principal components.
- Stability analysis of dynamical systems: eigenvalues determine whether perturbations grow or decay.
- Spectral graph theory: eigenvalues of the graph Laplacian encode connectivity structure.

## Trade-offs

- Solving the characteristic polynomial is exact for $n \leq 4$ (closed-form roots exist) but numerically unstable for large $n$; iterative methods (QR algorithm, power iteration) are used instead.
- Complex eigenvalues require working over $\mathbb{C}$ even when $\mathrm{A}$ is real, adding computational overhead.
- Repeated eigenvalues (defective matrices) may lack a full set of linearly independent eigenvectors, making diagonalisation impossible; Jordan normal form is required in that case.

## Links

- [[determinants|Determinants]] — characteristic equation is evaluated via $\det(\mathrm{A} - \lambda\mathrm{I})$
- [[matrix_diagonalization|Matrix Diagonalization]] — eigenvectors form the columns of the diagonalising matrix $\mathrm{S}$
