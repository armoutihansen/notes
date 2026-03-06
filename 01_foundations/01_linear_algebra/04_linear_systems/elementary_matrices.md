---
layer: 01_foundations
type: concept
status: seed
tags: [theory]
created: 2026-03-01
---

# Elementary Matrices

## Definition

An **elementary matrix** is a matrix that differs from the identity matrix by a single elementary row operation. Left-multiplying a matrix $\rm A$ by an elementary matrix applies that row operation to $\rm A$.

## Intuition

Every row operation is secretly a matrix multiplication. Writing elimination as $\rm M_k \cdots M_1 A = U$ turns the algorithmic sequence of steps into an algebraic identity, which then immediately yields the LU factorisation by inverting each $\rm M_i$.

## Formal Description

Each step of [[gaussian_elimination|Gaussian Elimination]] can be expressed as left-multiplication by an elementary matrix. For the matrix
$$
\rm A=
\begin{pmatrix}-3 & 2 & -1\\ 6 & -6 & 7\\ 3 & -4 & 4\end{pmatrix},
$$
the three elimination steps give $\rm M_3 M_2 M_1 A = U$, where:

$$
\rm M_1=\begin{pmatrix}1&0&0\\2&1&0\\0&0&1\end{pmatrix},\quad
\rm M_2=\begin{pmatrix}1&0&0\\0&1&0\\1&0&1\end{pmatrix},\quad
\rm M_3=\begin{pmatrix}1&0&0\\0&1&0\\0&-1&1\end{pmatrix},
$$

and $\rm U$ is the resulting upper-triangular matrix. $\rm M_1$ adds twice row 1 to row 2; $\rm M_2$ adds row 1 to row 3; $\rm M_3$ subtracts row 2 from row 3.

Elementary matrices are always **invertible**; the inverse reverses the row operation (e.g., if $\rm M$ adds $c$ times row $i$ to row $j$, then $\rm M^{-1}$ adds $-c$ times row $i$ to row $j$). The factorisation $\rm M_k \cdots M_1 A = U$ is the foundation of [[lu_decomposition|LU Decomposition]].

## Applications

- Provides the algebraic proof that Gaussian elimination produces an LU factorisation.
- Used to derive properties of determinants under row operations.

## Trade-offs

- Elementary matrices are conceptually useful but are rarely formed explicitly in practice; storing and multiplying $n$ separate $n\times n$ matrices costs far more than simply recording the multipliers.
- Only applicable when row operations suffice; column operations require right-multiplication, which is distinct.

## Links

- [[gaussian_elimination|Gaussian Elimination]]
- [[lu_decomposition|LU Decomposition]]
