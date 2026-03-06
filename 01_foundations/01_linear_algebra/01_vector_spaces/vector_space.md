---
layer: 01_foundations
type: concept
status: seed
tags: [theory]
created: 2026-03-01
---

# Vector Space

## Definition

A _vector space_ consists of a set of vectors and a set of scalars. Although vectors can be quite general, we focus here on vectors that are real column matrices and scalars that are real numbers.

For the set of vectors and scalars to form a vector space, the set must be **closed under vector addition and scalar multiplication**: for any two vectors $\mathrm{u}, \mathrm{v}$ in the set and scalars $a, b$, the linear combination $a\mathrm{u} + b\mathrm{v}$ must still be in the set.

## Intuition

A vector space is the natural setting for linear algebra: any object you can scale and add, while staying "in the same universe," lives in a vector space. Geometrically, $\mathbb{R}^2$ is the plane and $\mathbb{R}^3$ is ordinary 3D space — both are closed under stretching and combining arrows. Subspaces are flat (they always contain the origin) and represent constraints like "all solutions to $\mathrm{Ax}=0$."

## Formal Description

A subset $\mathrm{W}$ of a vector space $\mathrm{V}$ is a **vector subspace** if it is itself a vector space under the same operations — equivalently, if it is closed under linear combinations.

**Example.** The set of all $3 \times 1$ real matrices forms the vector space $\mathbb{R}^3$, since for any $\mathrm{u}, \mathrm{v} \in \mathbb{R}^3$ and scalars $a, b$:
$$
a\mathrm{u}+b\mathrm{v}=
\begin{pmatrix}
au_1+bv_1\\
au_2+bv_2\\
au_3+bv_3
\end{pmatrix} \in \mathbb{R}^3.
$$

There are four fundamental vector spaces associated with an $m \times n$ matrix $\mathrm{A}$:

- [[null_space|Null Space]] $\mathrm{Null(A)}$: all $\mathrm{x}$ with $\mathrm{Ax}=0$, a subspace of $\mathbb{R}^n$.
- [[column_space|Column Space]] $\mathrm{Col(A)}$: span of the columns of $\mathrm{A}$, a subspace of $\mathbb{R}^m$.
- [[row_space_and_rank|Row Space and Left Null Space]]: column space and null space of $\mathrm{A}^T$.

## Applications

- Representing and reasoning about solution sets of linear systems.
- Defining function spaces, polynomial spaces, and other infinite-dimensional settings in analysis.
- Foundation for all matrix decompositions (QR, SVD, eigendecomposition).

## Trade-offs

The abstract definition (closure under linear combinations) is minimal and powerful, but it elides metric structure — notions of angle and distance require additional structure (an inner product). Working over the reals is sufficient for most ML applications; complex vector spaces arise in signal processing and quantum computing.

## Links

- [[null_space|Null Space]]
- [[column_space|Column Space]]
- [[row_space_and_rank|Row Space and Rank]]
- [[span_basis_dimension|Span, Basis and Dimension]]
