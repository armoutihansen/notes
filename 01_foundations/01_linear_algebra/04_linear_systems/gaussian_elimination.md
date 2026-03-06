---
layer: 01_foundations
type: concept
status: seed
tags: [algorithm]
created: 2026-03-01
---

# Gaussian Elimination

## Definition

**Gaussian elimination** is the standard algorithm for solving a system of linear equations $\rm Ax = b$. It reduces the augmented matrix $[\rm A \mid b]$ to upper-triangular form via row operations, followed by **back substitution**.

## Intuition

Work column by column from left to right: use the current diagonal entry (the pivot) to zero out everything below it. After $n-1$ steps the matrix is upper-triangular and the unknowns can be read off from the bottom row upward by back substitution.

## Formal Description

The three allowed **elementary row operations** are:
1. Interchange any two rows.
2. Multiply a row by a nonzero constant.
3. Add a multiple of one row to another.

These operations do not change the solution set. The goal is to reduce $\rm A$ to upper-triangular form $\rm U$.

**Example.** For the system
$$
\begin{pmatrix}
-3 & 2 & -1 \\
6 & -6 & 7 \\
3 & -4 & 4
\end{pmatrix}
\begin{pmatrix} x_1 \\ x_2 \\ x_3 \end{pmatrix}
=
\begin{pmatrix} -1 \\ -7 \\ -6 \end{pmatrix},
$$
form the augmented matrix and reduce:
$$
\begin{pmatrix}-3 & 2 & -1 & -1 \\ 6 & -6 & 7 & -7 \\ 3 & -4 & 4 & -6\end{pmatrix}
\to
\begin{pmatrix}-3 & 2 & -1 & -1 \\ 0 & -2 & 5 & -9 \\ 0 & -2 & 3 & -7\end{pmatrix}
\to
\begin{pmatrix}-3 & 2 & -1 & -1 \\ 0 & -2 & 5 & -9 \\ 0 & 0 & -2 & 2\end{pmatrix}.
$$
Back substitution then gives $\rm x = (2,\, 2,\, -1)^\top$.

The matrix element used to eliminate entries below it is called the **pivot**. Elimination fails if a pivot is zero; row interchange must be performed first. For numerical stability on large systems, **partial pivoting** (always swapping to bring the largest-magnitude entry to the pivot position) is used.

## Applications

- Direct method for solving $\rm Ax = b$ for a single right-hand side.
- Basis for [[lu_decomposition|LU Decomposition]] and [[reduced_row_echelon|Reduced Row Echelon Form]].

## Trade-offs

- A zero pivot requires a row interchange; without pivoting the algorithm may fail even for non-singular $\rm A$.
- Without partial pivoting, small pivots can cause catastrophic cancellation and large rounding errors.
- Costs $O(n^3)$ for an $n \times n$ system; inefficient when the same $\rm A$ must be used with many right-hand sides (use LU instead).

## Links

- [[systems_of_linear_equations|Systems of Linear Equations]]
- [[reduced_row_echelon|Reduced Row Echelon Form]]
- [[elementary_matrices|Elementary Matrices]]
- [[lu_decomposition|LU Decomposition]]
