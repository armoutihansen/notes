---
layer: 01_foundations
type: concept
status: seed
tags: [theory]
created: 2026-03-01
---

# Column Space

## Definition

The _column space_ of a matrix $\mathrm{A}$, denoted $\mathrm{Col(A)}$, is the [[span_basis_dimension|span]] of the columns of $\mathrm{A}$. For an $m \times n$ matrix, $\mathrm{Col(A)}$ is a subspace of $\mathbb{R}^m$.

## Intuition

$\mathrm{Ax}$ is just a weighted sum of the columns of $\mathrm{A}$, with weights given by $\mathrm{x}$. So the column space is exactly the set of all outputs the matrix can produce — the range of the linear map. The system $\mathrm{Ax} = \mathrm{b}$ is solvable precisely when $\mathrm{b}$ lands inside this output set.

## Formal Description

For any vector $\mathrm{x}$, the product $\mathrm{Ax}$ is a linear combination of the columns of $\mathrm{A}$:
$$
\begin{pmatrix}a & b \\ c & d\end{pmatrix}\begin{pmatrix}x \\ y\end{pmatrix}
= x\begin{pmatrix}a \\ c\end{pmatrix} + y\begin{pmatrix}b \\ d\end{pmatrix}.
$$
Thus $\mathrm{Ax} \in \mathrm{Col(A)}$ for all $\mathrm{x}$, and the system $\mathrm{Ax} = \mathrm{b}$ is consistent if and only if $\mathrm{b} \in \mathrm{Col(A)}$.

**Finding a basis.** Row operations preserve the linear dependence relations among columns. The pivot columns of $\mathrm{rref(A)}$ identify which columns of $\mathrm{A}$ (not $\mathrm{rref(A)}$) form a basis for $\mathrm{Col(A)}$.

**Example.**
$$
\mathrm{A}=\begin{pmatrix}-3 & 6 & -1 & 1 & -7 \\ 1 & -2 & 2 & 3 & -1 \\ 2 & -4 & 5 & 8 & -4\end{pmatrix},\quad
\mathrm{rref(A)}=\begin{pmatrix}1 & -2 & 0 & -1 & 3 \\ 0 & 0 & 1 & 2 & -2 \\ 0 & 0 & 0 & 0 & 0\end{pmatrix}.
$$
The pivot columns are 1 and 3, so a basis for $\mathrm{Col(A)}$ is
$$
\left\{\begin{pmatrix}-3 \\ 1 \\ 2\end{pmatrix},\begin{pmatrix}-1 \\ 2 \\ 5\end{pmatrix}\right\}.
$$

**Rank-nullity theorem.** For an $m \times n$ matrix $\mathrm{A}$:
$$
\dim(\mathrm{Col(A)}) + \dim(\mathrm{Null(A)}) = n.
$$
The dimension of $\mathrm{Col(A)}$ equals the number of pivot columns; the dimension of $\mathrm{Null(A)}$ equals the number of non-pivot (free) columns.

## Applications

- Determining solvability of $\mathrm{Ax} = \mathrm{b}$: consistent iff $\mathrm{b} \in \mathrm{Col(A)}$.
- Least-squares problems project $\mathrm{b}$ onto $\mathrm{Col(A)}$ when no exact solution exists.
- In neural networks, each layer maps inputs into the column space of its weight matrix.

## Trade-offs

The column space depends on which columns are chosen as a basis — basis vectors are not unique, though the subspace they span is. Using the actual pivot columns of $\mathrm{A}$ (not $\mathrm{rref(A)}$) preserves the geometric meaning of the original data.

## Links

- [[null_space|Null Space]]
- [[span_basis_dimension|Span, Basis and Dimension]]
- [[row_space_and_rank|Row Space and Rank]]
- [[vector_space|Vector Space]]
