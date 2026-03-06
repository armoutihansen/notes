---
layer: 01_foundations
type: concept
status: seed
tags: [theory]
created: 2026-03-01
---

# Span, Basis, and Dimension

## Definition

**Span.** The _span_ of a set of vectors $\{\mathrm{v}_1, \dots, \mathrm{v}_n\}$ is the vector space consisting of all linear combinations of $\mathrm{v}_1, \dots, \mathrm{v}_n$. We say the set _spans_ the vector space.

**Basis.** The smallest set of vectors needed to span a vector space forms a _basis_ for that space. Equivalently, a basis is a spanning set of [[linear_independence|linearly independent]] vectors.

**Dimension.** The number of vectors in a basis gives the _dimension_ of the vector space.

## Intuition

A basis is a "coordinate system" for a vector space — the minimal set of directions from which every point can be reached by scaling and adding. Removing any basis vector shrinks the reachable space; adding one introduces redundancy. Dimension captures how many genuinely independent degrees of freedom exist, regardless of which particular basis you choose.

## Formal Description

**Example.** The set
$$
\left\{
\begin{pmatrix}1\\0\\0\end{pmatrix},
\begin{pmatrix}0\\1\\0\end{pmatrix},
\begin{pmatrix}2\\3\\0\end{pmatrix}
\right\}
$$
spans the subspace of $3 \times 1$ matrices with zero in the third row. Since the third vector is linearly dependent on the first two, a basis needs only two vectors, e.g.:
$$
\left\{
\begin{pmatrix}1\\0\\0\end{pmatrix},
\begin{pmatrix}0\\1\\0\end{pmatrix}
\right\}.
$$
This subspace has dimension 2.

- A basis is not unique; any two bases for the same vector space have the same number of vectors (the dimension).
- An **orthonormal basis** consists of vectors that are mutually orthogonal and of unit norm — this is the preferred form and is constructed via the [[gram_schmidt|Gram-Schmidt Process]].
- The first $k$ orthonormal vectors from Gram-Schmidt span the same subspace as the first $k$ original basis vectors.

## Applications

- Counting degrees of freedom in a linear system (rank, nullity).
- Expressing any vector as coordinates relative to a chosen basis (change of basis).
- Dimensionality reduction techniques (PCA, SVD) find low-dimensional subspaces that capture most variance.

## Trade-offs

A non-orthogonal basis is valid but makes projection and inner-product computations more expensive. Orthonormal bases eliminate those costs at the price of running Gram-Schmidt or an equivalent procedure upfront.

## Links

- [[vector_space|Vector Space]]
- [[linear_independence|Linear Independence]]
- [[gram_schmidt|Gram-Schmidt Process]]
