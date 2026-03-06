---
layer: 01_foundations
type: concept
status: seed
tags: [theory]
created: 2026-03-01
---

# Row Space and Rank

## Definition

For an $m \times n$ matrix $\mathrm{A}$, there are two additional fundamental vector spaces beyond the [[column_space|column space]] and [[null_space|null space]]:

- **Row space** $\mathrm{Row(A)}$: the column space of $\mathrm{A}^T$ — the span of the rows of $\mathrm{A}$, a subspace of $\mathbb{R}^n$.
- **Left null space** $\mathrm{Null(A}^T)$: the null space of $\mathrm{A}^T$ — all $\mathrm{y}$ with $\mathrm{A}^T\mathrm{y} = 0$, a subspace of $\mathbb{R}^m$.

## Intuition

A matrix has four fundamental subspaces that come in two orthogonal pairs: the row space and null space partition $\mathbb{R}^n$ (the input space), while the column space and left null space partition $\mathbb{R}^m$ (the output space). Rank is the "effective dimensionality" of the mapping — how many independent output directions the matrix can reach.

## Formal Description

A [[span_basis_dimension|basis]] for the row space can be read directly from $\mathrm{rref(A)}$: the non-zero rows (written as column vectors) with pivot entries form a basis.

The null space of $\mathrm{A}$ is orthogonal to the row space: every vector in $\mathrm{Null(A)}$ is orthogonal to every vector in $\mathrm{Row(A)}$. These two subspaces are _orthogonal complements_ within $\mathbb{R}^n$, i.e. their dimensions sum to $n$. Similarly, $\mathrm{Col(A)}$ and $\mathrm{Null(A^T)}$ are orthogonal complements within $\mathbb{R}^m$.

The dimensions of the column space and row space are equal:
$$
\dim(\mathrm{Col(A)}) = \dim(\mathrm{Row(A)}).
$$
This common value is the **rank** of $\mathrm{A}$:
$$
\mathrm{rank(A)} \leq \min(m, n).
$$
When equality holds, $\mathrm{A}$ is of **full rank**. For a square matrix of full rank, $\dim(\mathrm{Null(A)}) = 0$ and $\mathrm{A}$ is invertible.

## Applications

- Rank reveals the number of linearly independent constraints or equations in a system.
- Full column rank is necessary and sufficient for $\mathrm{A}^T\mathrm{A}$ to be invertible (uniqueness in least squares).
- Rank deficiency signals multicollinearity in regression or redundant features in a dataset.

## Trade-offs

Rank computed via RREF can be numerically unreliable for nearly-rank-deficient matrices. Numerical rank is better estimated as the number of singular values exceeding a threshold $\varepsilon$ (SVD-based), at the cost of $O(\min(m,n) \cdot mn)$ computation.

## Links

- [[column_space|Column Space]]
- [[null_space|Null Space]]
- [[span_basis_dimension|Span, Basis and Dimension]]
- [[vector_space|Vector Space]]
