---
layer: 01_foundations
type: concept
status: seed
tags: [algorithm]
created: 2026-03-01
---

# Reduced Row Echelon Form

## Definition

A matrix is in **reduced row echelon form (rref)** if:
1. The first nonzero entry in every row (the **leading 1**) is equal to one.
2. All entries above and below each leading 1 are zero.
3. Any all-zero rows appear at the bottom.

## Intuition

rref is the "fully simplified" form of a matrix: each pivot column becomes a standard basis vector, making free variables and dependent variables immediately visible. It is the canonical form that uniquely encodes the row space of the matrix.

## Formal Description

The row elimination procedure of [[gaussian_elimination|Gaussian Elimination]] is continued past upper-triangular form — eliminating entries *above* each pivot and scaling pivot rows — to reach rref.

**Example.** For the $3\times 4$ matrix
$$
\rm A =
\begin{pmatrix}
1 & 2 & 3 & 4 \\
4 & 5 & 6 & 7 \\
6 & 7 & 8 & 9
\end{pmatrix},
$$
row reduction proceeds as
$$
\begin{align}
\begin{pmatrix}1 & 2 & 3 & 4 \\ 4 & 5 & 6 & 7 \\ 6 & 7 & 8 & 9\end{pmatrix}
&\stackrel{R_2=R_2-4R_1}{\to}
\begin{pmatrix}1 & 2 & 3 & 4 \\ 0 & -3 & -6 & -9 \\ 6 & 7 & 8 & 9\end{pmatrix} \\
&\stackrel{R_3=R_3-6R_1}{\to}
\begin{pmatrix}1 & 2 & 3 & 4 \\ 0 & -3 & -6 & -9 \\ 0 & -5 & -10 & -15\end{pmatrix} \\
&\stackrel{R_2=-R_2/3}{\to}
\begin{pmatrix}1 & 2 & 3 & 4 \\ 0 & 1 & 2 & 3 \\ 0 & -5 & -10 & -15\end{pmatrix} \\
&\stackrel{R_1=R_1-2R_2}{\to}
\begin{pmatrix}1 & 0 & -1 & -2 \\ 0 & 1 & 2 & 3 \\ 0 & -5 & -10 & -15\end{pmatrix} \\
&\stackrel{R_3=R_3+5R_2}{\to}
\begin{pmatrix}1 & 0 & -1 & -2 \\ 0 & 1 & 2 & 3 \\ 0 & 0 & 0 & 0\end{pmatrix}
= \rm rref(A).
\end{align}
$$
Here $\rm A$ has two **pivot columns**.

The rref of a matrix $\rm A$ is **unique**. Rows may need to be exchanged during the computation. If $\rm A$ is square and invertible, then $\rm rref(A) = I$.

## Applications

- Determines the rank and null space of a matrix directly from the pivot structure.
- Foundation for [[computing_inverses|Computing Inverses]] via augmented matrix $[\rm A \mid I]$.

## Trade-offs

- More expensive than stopping at upper-triangular form (requires an additional upward-elimination pass and row scaling).
- Numerically, rref can amplify round-off errors when pivots are small; in practice, LU with pivoting is preferred for solving linear systems.
- Row exchanges during reduction affect the pivot column order but not uniqueness of the final form.

## Links

- [[gaussian_elimination|Gaussian Elimination]]
- [[computing_inverses|Computing Inverses]]
