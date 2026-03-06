---
layer: 01_foundations
type: concept
status: seed
tags: [algorithm]
created: 2026-03-01
---

# Solving (LU)x = b

## Definition

Given the [[lu_decomposition|LU Decomposition]] $\rm A = LU$, solving $\rm Ax = b$ reduces to two triangular solves, which is efficient for multiple right-hand sides.

## Intuition

Introduce an intermediate vector $\rm y = Ux$ so that $\rm Ly = b$. Solving $\rm Ly = b$ top-to-bottom (forward substitution) and then $\rm Ux = y$ bottom-to-top (back substitution) each cost only $O(n^2)$, because every triangular system has a direct formula for each unknown.

## Formal Description

Write $\rm (LU)x = L(Ux) = b$. Set $\rm y = Ux$, then:

1. **Forward substitution**: Solve $\rm Ly = b$ for $\rm y$ (starting from the first equation).
2. **Back substitution**: Solve $\rm Ux = y$ for $\rm x$ (starting from the last equation).

**Example.** With
$$
\rm L=\begin{pmatrix}1&0&0\\-2&1&0\\-1&1&1\end{pmatrix},\quad
\rm U=\begin{pmatrix}-3&2&-1\\0&-2&5\\0&0&-2\end{pmatrix},\quad
b=\begin{pmatrix}-1\\-7\\-6\end{pmatrix},
$$
forward substitution on $\rm Ly = b$ gives:
$$
\begin{align}
y_1 &= -1, \\
y_2 &= -7 + 2y_1 = -9, \\
y_3 &= -6 + y_1 - y_2 = 2.
\end{align}
$$
Back substitution on $\rm Ux = y$ gives:
$$
\begin{align}
x_3 &= -1, \\
x_2 &= -\tfrac{1}{2}(-9 - 5x_3) = 2, \\
x_1 &= -\tfrac{1}{3}(-1 - 2x_2 + x_3) = 2.
\end{align}
$$
Hence $\rm x = (2,\, 2,\, -1)^\top$.

Each triangular solve costs $O(n^2)$, far cheaper than re-running Gaussian elimination ($O(n^3)$) for each new $\rm b$.

## Applications

Standard computational method for solving [[systems_of_linear_equations|Systems of Linear Equations]] with a fixed coefficient matrix and multiple right-hand sides (e.g., finite-element solvers, iterative refinement).

## Trade-offs

- Requires the LU factorisation to already be computed; if $\rm A$ changes, the $O(n^3)$ factorisation must be redone.
- Numerical accuracy depends on whether pivoting was used during the LU phase; without pivoting, forward/back substitution can amplify errors.
- Not applicable directly if $\rm A$ is singular; the system must be analysed for consistency first.

## Links

- [[lu_decomposition|LU Decomposition]]
- [[systems_of_linear_equations|Systems of Linear Equations]]
