---
layer: 01_foundations
type: concept
status: seed
tags: [theory]
created: 2026-03-01
---

# Systems of Linear Equations

## Definition

A **system of linear equations** is a collection of $m$ equations in $n$ unknowns $x_1, \dots, x_n$:
$$
\begin{align}
a_{11}x_1 + a_{12}x_2 + \cdots + a_{1n}x_n &= b_1, \\
a_{21}x_1 + a_{22}x_2 + \cdots + a_{2n}x_n &= b_2, \\
&\;\vdots \\
a_{m1}x_1 + a_{m2}x_2 + \cdots + a_{mn}x_n &= b_m.
\end{align}
$$

## Intuition

Each equation defines a hyperplane in $\mathbb{R}^n$; the solution set is the intersection of all those hyperplanes. The system either has no solution (planes don't all meet), exactly one solution (planes meet at a point), or infinitely many (planes overlap along a line or subspace) — the rank of $\rm A$ determines which case holds.

## Formal Description

In **matrix form**, the system is written as $\rm Ax = b$, where $\rm A \in \mathbb{R}^{m \times n}$ is the coefficient matrix, $\rm x \in \mathbb{R}^n$ is the vector of unknowns, and $\rm b \in \mathbb{R}^m$ is the right-hand side vector.

**Existence and uniqueness** of solutions depends on the rank of $\rm A$ and the augmented matrix $[\rm A \mid b]$:

- **No solution** (inconsistent): $\operatorname{rank}(\rm A) < \operatorname{rank}([\rm A \mid b])$.
- **Unique solution**: $\operatorname{rank}(\rm A) = \operatorname{rank}([\rm A \mid b]) = n$.
- **Infinitely many solutions**: $\operatorname{rank}(\rm A) = \operatorname{rank}([\rm A \mid b]) < n$.

For a square system ($m = n$), a unique solution exists if and only if $\rm A$ is invertible (i.e., $\det(\rm A) \neq 0$), in which case $\rm x = A^{-1}b$.

## Applications

- Foundation of virtually every quantitative field: physics, engineering, data fitting, and optimisation all reduce to solving $\rm Ax = b$.
- Underpins [[lu_decomposition|LU Decomposition]] and [[computing_inverses|Computing Inverses]].

## Trade-offs

- Consistency must be checked before assuming a solution exists; blind application of a solver may silently return a least-squares approximation rather than an exact solution.
- Direct inversion ($\rm x = A^{-1}b$) is numerically inferior to factorisation-based methods for large $n$.
- Rank determination is sensitive to floating-point errors; a numerically small pivot may reflect near-singularity.

## Links

- [[gaussian_elimination|Gaussian Elimination]]
- [[lu_decomposition|LU Decomposition]]
- [[computing_inverses|Computing Inverses]]
