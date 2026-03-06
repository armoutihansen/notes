---
layer: 01_foundations
type: concept
status: seed
tags: [algorithm]
created: 2026-03-01
---

# Computing Inverses

## Definition

The **inverse** $\rm A^{-1}$ of an invertible square matrix $\rm A$ satisfies $\rm A A^{-1} = I$. It can be computed by reducing the augmented matrix $[\rm A \mid I]$ to $[\rm I \mid A^{-1}]$ via [[reduced_row_echelon|Reduced Row Echelon Form]].

## Intuition

$\rm A^{-1}$ is the matrix whose columns solve $\rm Ax = e_i$ for each standard basis vector $\rm e_i$. Augmenting $\rm A$ with the full identity and row-reducing solves all $n$ systems simultaneously in a single pass.

## Formal Description

Let the columns of $\rm A^{-1}$ be $\rm a_1^{-1}, a_2^{-1}, \dots$ and the columns of $\rm I$ be $\rm e_1, e_2, \dots$. Then $\rm A a_i^{-1} = e_i$ for each $i$. Solving all $n$ systems simultaneously via row reduction on the $n \times 2n$ augmented matrix $[\rm A \mid I]$ and continuing until $\rm rref(A) = I$ yields $\rm A^{-1}$ on the right-hand side.

**Example.** Starting from
$$
\begin{pmatrix}-3&2&-1&1&0&0\\6&-6&7&0&1&0\\3&-4&4&0&0&1\end{pmatrix},
$$
row reduction proceeds:
$$
\begin{align}
&\to\begin{pmatrix}-3&2&-1&1&0&0\\0&-2&5&2&1&0\\0&-2&3&1&0&1\end{pmatrix}\\
&\to\begin{pmatrix}-3&2&-1&1&0&0\\0&-2&5&2&1&0\\0&0&-2&-1&-1&1\end{pmatrix}\\
&\to\begin{pmatrix}-3&0&4&3&1&0\\0&-2&5&2&1&0\\0&0&-2&-1&-1&1\end{pmatrix}\\
&\to\begin{pmatrix}-3&0&0&1&-1&2\\0&-2&0&-\tfrac{1}{2}&-\tfrac{3}{2}&\tfrac{5}{2}\\0&0&-2&-1&-1&1\end{pmatrix}\\
&\to\begin{pmatrix}1&0&0&-\tfrac{1}{3}&\tfrac{1}{3}&-\tfrac{2}{3}\\0&1&0&\tfrac{1}{4}&\tfrac{3}{4}&-\tfrac{5}{4}\\0&0&1&\tfrac{1}{2}&\tfrac{1}{2}&-\tfrac{1}{2}\end{pmatrix}.
\end{align}
$$
Hence,
$$
\rm A^{-1}=\begin{pmatrix}-\tfrac{1}{3}&\tfrac{1}{3}&-\tfrac{2}{3}\\\tfrac{1}{4}&\tfrac{3}{4}&-\tfrac{5}{4}\\\tfrac{1}{2}&\tfrac{1}{2}&-\tfrac{1}{2}\end{pmatrix}.
$$

$\rm A^{-1}$ exists if and only if $\rm rref(A) = I$ (equivalently, $\det(\rm A) \neq 0$). If $\rm A$ is not invertible, the reduction will produce a zero row on the left side before reaching $\rm I$.

## Applications

- Used when the inverse itself is needed explicitly (e.g., symbolic manipulation, deriving normal equations in statistics).
- Confirms invertibility: if reduction fails to reach $\rm I$ on the left, $\rm A$ is singular.

## Trade-offs

- Computing $\rm A^{-1}$ explicitly costs $O(n^3)$ and requires $O(n^2)$ storage; for solving $\rm Ax = b$ it is numerically less stable and no faster than LU factorisation.
- In floating-point arithmetic, the inverse accumulates rounding errors; solving via LU is preferred whenever the inverse is only needed to apply to vectors.
- Only applicable to square, non-singular matrices.

## Links

- [[reduced_row_echelon|Reduced Row Echelon Form]]
- [[systems_of_linear_equations|Systems of Linear Equations]]
