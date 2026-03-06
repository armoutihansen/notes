---
layer: 01_foundations
type: concept
status: seed
tags: [theory]
created: 2026-03-01
---

# Null Space

## Definition

The _null space_ of a matrix $\mathrm{A}$, denoted $\mathrm{Null(A)}$, is the set of all column vectors $\mathrm{x}$ satisfying
$$
\mathrm{Ax} = 0.
$$
$\mathrm{Null(A)}$ is a [[vector_space|vector subspace]]: if $\mathrm{x}, \mathrm{y} \in \mathrm{Null(A)}$, then $a\mathrm{x} + b\mathrm{y} \in \mathrm{Null(A)}$. For an $m \times n$ matrix $\mathrm{A}$, the null space is a subspace of $\mathbb{R}^n$.

## Intuition

The null space answers: "what inputs does $\mathrm{A}$ completely destroy (map to zero)?" It captures all the "invisible" directions — perturbations in the input that produce no change in the output. A large null space means the matrix has many directions it ignores, implying high redundancy or underdetermination. A trivial null space ($\{0\}$ only) means the mapping is injective.

## Formal Description

**Finding a basis.** Bring $\mathrm{A}$ to reduced row echelon form. The variables associated with pivot columns are _basic variables_; those with non-pivot columns are _free variables_. Express each basic variable in terms of the free variables and collect vectors.

**Example.** For
$$
\mathrm{A}=\begin{pmatrix}-3 & 6 & -1 & 1 & -7 \\ 1 & -2 & 2 & 3 & -1 \\ 2 & -4 & 5 & 8 & -4\end{pmatrix},\quad
\mathrm{rref(A)}=\begin{pmatrix}1 & -2 & 0 & -1 & 3 \\ 0 & 0 & 1 & 2 & -2 \\ 0 & 0 & 0 & 0 & 0\end{pmatrix},
$$
basic variables are $x_1, x_3$; free variables are $x_2, x_4, x_5$. Solving:
$$
x_1 = 2x_2 + x_4 - 3x_5, \qquad x_3 = -2x_4 + 2x_5.
$$
The general null-space vector is
$$
x_2\begin{pmatrix}2\\1\\0\\0\\0\end{pmatrix}
+x_4\begin{pmatrix}1\\0\\-2\\1\\0\end{pmatrix}
+x_5\begin{pmatrix}-3\\0\\2\\0\\1\end{pmatrix},
$$
giving a basis for $\mathrm{Null(A)}$. The null space is a 3-dimensional subspace of $\mathbb{R}^5$.

- $\dim(\mathrm{Null(A)})$ equals the number of non-pivot columns of $\mathrm{rref(A)}$.
- If $\mathrm{A}$ is square and invertible, $\mathrm{Null(A)} = \{0\}$.
- By the rank-nullity theorem: $\dim(\mathrm{Col(A)}) + \dim(\mathrm{Null(A)}) = n$.

## Applications

**Underdetermined systems.** If $\mathrm{u}$ is the general vector in $\mathrm{Null(A)}$ and $\mathrm{v}$ is any particular solution to $\mathrm{Av} = \mathrm{b}$, then the general solution is $\mathrm{x} = \mathrm{u} + \mathrm{v}$, since $\mathrm{A(u+v)} = 0 + \mathrm{b} = \mathrm{b}$.

**Regularisation.** In ill-posed inverse problems, solutions are constrained to be orthogonal to the null space to ensure uniqueness.

## Trade-offs

Computing the full null space via RREF is exact but numerically unstable for ill-conditioned matrices. The SVD provides a numerically stable null-space basis via the right singular vectors corresponding to zero (or near-zero) singular values.

## Links

- [[column_space|Column Space]]
- [[span_basis_dimension|Span, Basis and Dimension]]
- [[row_space_and_rank|Row Space and Rank]]
- [[vector_space|Vector Space]]
