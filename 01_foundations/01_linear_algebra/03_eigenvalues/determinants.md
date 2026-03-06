---
layer: 01_foundations
type: concept
status: seed
tags: [theory]
created: 2026-03-01
---

# Determinants

## Definition

The **determinant** of a square matrix $\mathrm{A}$ is a scalar that is nonzero if and only if $\mathrm{A}$ is invertible. When $\det \mathrm{A} = 0$ the matrix is called **singular**.

## Intuition

The determinant measures the factor by which a linear transformation scales volumes. A $2\times 2$ determinant gives the signed area of the parallelogram formed by the column vectors; a $3\times 3$ determinant gives the signed volume of the parallelepiped. A negative determinant indicates that the transformation includes a reflection (orientation reversal). When $\det \mathrm{A} = 0$ the transformation collapses space onto a lower-dimensional subspace, making inversion impossible.

## Formal Description

**2×2 determinant.** For $\mathrm{A} = \begin{pmatrix} a & b \\ c & d \end{pmatrix}$,
$$
\det \mathrm{A} = \begin{vmatrix} a & b \\ c & d \end{vmatrix} = ad - bc.
$$

**3×3 determinant.** For a $3\times 3$ matrix,
$$
\det \mathrm{A} = \begin{vmatrix} a & b & c \\ d & e & f \\ g & h & i \end{vmatrix} = aei + bfg + cdh - ceg - bdi - afh.
$$

**Laplace (cofactor) expansion.** The determinant can be expanded across any row or down any column. Expanding the $3\times 3$ determinant along the first row:
$$
\begin{vmatrix} a & b & c \\ d & e & f \\ g & h & i \end{vmatrix} = a\begin{vmatrix} e & f \\ h & i \end{vmatrix} - b\begin{vmatrix} d & f \\ g & i \end{vmatrix} + c\begin{vmatrix} d & e \\ g & h \end{vmatrix}.
$$
Each $2\times 2$ block is a **minor** — the submatrix obtained by deleting the element's row and column. The sign of the term for position $(i,j)$ is $(-1)^{i+j}$, forming the checkerboard pattern
$$
\begin{pmatrix} + & - & + \\ - & + & - \\ + & - & + \end{pmatrix}.
$$
This rule generalises to $n\times n$ matrices: expand along any row or column, multiply each element by its signed minor, and sum the results. The practical strategy is to expand along the row or column containing the most zeros.

**Leibniz formula.** An equivalent formulation expresses the $n\times n$ determinant as a sum over all $n!$ permutations $\sigma$ of $\{1,\dots,n\}$:
$$
\det \mathrm{A} = \sum_{\sigma} \operatorname{sgn}(\sigma)\, a_{1,\sigma(1)}\, a_{2,\sigma(2)} \cdots a_{n,\sigma(n)},
$$
where each term contains exactly one element from each row and each column. The sign $\operatorname{sgn}(\sigma)$ is $+1$ for even permutations and $-1$ for odd permutations. For a $3\times 3$ matrix the six terms are
$$
+aei,\; +bfg,\; +cdh,\; -afh,\; -bdi,\; -ceg,
$$
corresponding to even permutations $\{1,2,3\},\{2,3,1\},\{3,1,2\}$ and odd permutations $\{1,3,2\},\{2,1,3\},\{3,2,1\}$ of the columns.

## Applications

- Testing matrix invertibility: $\mathrm{A}$ is invertible iff $\det \mathrm{A} \neq 0$.
- Computing the characteristic polynomial $\det(\mathrm{A} - \lambda\mathrm{I})$ to find eigenvalues.
- Computing areas and volumes under linear transformations (change-of-variables in integration).
- Cramer's rule for solving small linear systems.

## Trade-offs

- Cofactor expansion has $O(n!)$ complexity in the naive form; LU decomposition reduces determinant computation to $O(n^3)$ for large matrices, making direct expansion impractical beyond $4\times 4$.
- Numerical determinants of large matrices are susceptible to floating-point overflow/underflow; log-determinants are preferred in practice.
- Determinant alone does not characterise a matrix fully — two matrices can share the same determinant while being structurally very different.

## Links

- [[eigenvalues_and_eigenvectors|Eigenvalues and Eigenvectors]] — the characteristic equation $\det(\mathrm{A} - \lambda\mathrm{I}) = 0$ requires evaluating a determinant
