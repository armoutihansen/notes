---
layer: 01_foundations
type: concept
status: seed
tags: [algorithm]
created: 2026-03-01
---

# LU Decomposition

## Definition

The **LU decomposition** of a matrix $\rm A$ is a factorization $\rm A = LU$, where $\rm L$ is unit lower-triangular (ones on the diagonal) and $\rm U$ is upper-triangular.

## Intuition

Gaussian elimination simultaneously produces two matrices: the upper-triangular result $\rm U$ and the record of elimination multipliers $\rm L$. LU decomposition simply names and stores both, amortising the $O(n^3)$ elimination cost across all future solves with the same $\rm A$.

## Formal Description

From [[elementary_matrices|Elementary Matrices]], Gaussian elimination gives
$$
\rm M_3 M_2 M_1 A = U \implies A = M_1^{-1} M_2^{-1} M_3^{-1} U = LU.
$$

Inverting each elementary matrix simply negates the off-diagonal multiplier:
$$
\rm M_1^{-1}=\begin{pmatrix}1&0&0\\-2&1&0\\0&0&1\end{pmatrix},\quad
\rm M_2^{-1}=\begin{pmatrix}1&0&0\\0&1&0\\-1&0&1\end{pmatrix},\quad
\rm M_3^{-1}=\begin{pmatrix}1&0&0\\0&1&0\\0&1&1\end{pmatrix}.
$$

The lower-triangular factor $\rm L = M_1^{-1}M_2^{-1}M_3^{-1}$ is assembled by simply placing the elimination multipliers into their respective positions:
$$
\rm L=\begin{pmatrix}1&0&0\\-2&1&0\\-1&1&1\end{pmatrix}.
$$

The full decomposition for the example matrix is:
$$
\begin{pmatrix}-3&2&-1\\6&-6&7\\3&-4&4\end{pmatrix}
=
\begin{pmatrix}1&0&0\\-2&1&0\\-1&1&1\end{pmatrix}
\begin{pmatrix}-3&2&-1\\0&-2&5\\0&0&-2\end{pmatrix}.
$$

The off-diagonal entries of $\rm L$ are exactly the **elimination multipliers** used during Gaussian elimination. LU decomposition requires $O(n^3)$ operations; once computed, each solve $\rm Ax=b$ costs only $O(n^2)$.

## Applications

Solving $\rm Ax = b$ for many vectors $\rm b$ — see [[solving_lu|Solving (LU)x=b]]. Also used internally for determinant computation and matrix inversion.

## Trade-offs

- Requires $\rm A$ to be non-singular (or at minimum that no zero pivot is encountered without row swaps); a singular matrix has no LU factorisation without permutation.
- Without pivoting the decomposition exists only if no pivot is zero; **partial pivoting** (producing $\rm PA = LU$) is required for general non-singular matrices.
- Storing both $\rm L$ and $\rm U$ uses $O(n^2)$ memory, the same as $\rm A$ itself (they can be packed into a single matrix).

## Links

- [[elementary_matrices|Elementary Matrices]]
- [[gaussian_elimination|Gaussian Elimination]]
- [[solving_lu|Solving (LU)x=b]]
