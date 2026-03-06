---
layer: 01_foundations
type: concept
status: seed
tags: [theory]
created: 2026-03-01
---

# Linear Independence

## Definition

The vectors $\{u_1, \dots, u_n\}$ are _linearly independent_ if the equation
$$
c_1 u_1 + \dots + c_n u_n = 0
$$
has only the trivial solution $c_1 = \dots = c_n = 0$. Equivalently, no vector in the set can be written as a linear combination of the others.

If a non-trivial solution exists, the vectors are _linearly dependent_.

## Intuition

Independent vectors point in genuinely different directions — none can be "explained" by the others. Geometrically, two vectors in $\mathbb{R}^2$ are dependent if and only if they are collinear (one is a rescaling of the other). Independence is the key property that makes a spanning set a basis: it guarantees unique coordinates.

## Formal Description

**Example (dependent).** The vectors
$$
\mathrm{u}=\begin{pmatrix}1\\0\\0\end{pmatrix},\quad
\mathrm{v}=\begin{pmatrix}0\\1\\0\end{pmatrix},\quad
\mathrm{w}=\begin{pmatrix}2\\3\\0\end{pmatrix}
$$
are linearly dependent since $\mathrm{w} = 2\mathrm{u} + 3\mathrm{v}$.

**Example (independent).** The standard basis vectors $\mathrm{e}_1, \mathrm{e}_2, \mathrm{e}_3$ are linearly independent since
$$
a\mathrm{e}_1 + b\mathrm{e}_2 + c\mathrm{e}_3 =
\begin{pmatrix}a\\b\\c\end{pmatrix} = \begin{pmatrix}0\\0\\0\end{pmatrix}
$$
forces $a = b = c = 0$.

**Algorithmic check.** Place the vectors as rows of a matrix and compute the reduced row echelon form. If any row becomes all zeros, the vectors are linearly dependent.

A set of $n$ linearly independent vectors in an $n$-dimensional space forms a [[span_basis_dimension|basis]] for that space.

## Applications

- Determining whether columns of $\mathrm{A}$ form a basis (and hence whether $\mathrm{A}$ is full column rank).
- Feature selection: redundant (dependent) features contribute no new information.
- Verifying that a proposed basis is valid before using it for decompositions.

## Trade-offs

Checking independence via RREF is exact but $O(mn^2)$ for an $m \times n$ matrix. For large matrices, approximate methods (rank estimation via SVD) are preferred in practice but can misclassify near-dependent vectors due to floating-point tolerances.

## Links

- [[span_basis_dimension|Span, Basis and Dimension]]
- [[null_space|Null Space]]
