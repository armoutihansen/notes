---
layer: 01_foundations
type: concept
status: seed
tags: [theory]
created: 2026-03-01
---

# Scalars, Vectors, Matrices, and Tensors

## Definition

- **Scalar**: a single number; denoted by a lower-case variable, e.g. $s\in\mathbb{R}$ or $n\in\mathbb{N}$.
- **Vector**: an ordered array of numbers. Denoted in bold lower-case, e.g. $\boldsymbol{x}$; individual elements in italic with subscript, e.g. $x_1, x_2, \ldots$. A vector with $n$ real elements lies in $\mathbb{R}^n$.
- **Matrix**: a 2-D array of numbers. $\boldsymbol{A}\in\mathbb{R}^{m\times n}$ has $m$ rows and $n$ columns; element $(i,j)$ is written $A_{i,j}$.
- **Tensor**: a multi-dimensional array of numbers arranged on a regular grid; denoted $\boldsymbol{\mathsf{A}}$, with element at coordinates $(i,j,k)$ written $\mathsf{A}_{i,j,k}$.

## Intuition

Think of scalars as points, vectors as arrows in space, matrices as rectangular grids of numbers that represent linear maps between spaces, and tensors as higher-dimensional generalisations of matrices. Each level adds one more axis of indexing. The transpose mirrors a matrix along its main diagonal — rotating it 90° and flipping — and broadcasting lets a lower-dimensional object (vector) act on every slice of a higher-dimensional one (matrix) without explicit copying.

## Formal Description

**Vector** (column form):
$$
\boldsymbol{x}=\begin{bmatrix}
x_1\\
x_2\\
\vdots\\
x_n
\end{bmatrix}
$$

**Matrix**:
$$
\boldsymbol{A}=\begin{bmatrix}
A_{1,1} & A_{1,2}\\
A_{2,1} & A_{2,2}
\end{bmatrix}
$$

- $\boldsymbol{A}_{i,:}$ denotes the $i$-th row; $\boldsymbol{A}_{:,j}$ denotes the $j$-th column.
- $\boldsymbol{x}_S$ denotes the sub-vector indexed by the set $S\subseteq\{1,\ldots,n\}$.

**Transpose**: $(\boldsymbol{A}^{\top})_{i,j}=A_{j,i}$. Flips rows and columns:
$$
\boldsymbol{A}=\begin{bmatrix}
A_{1,1} & A_{1,2}\\
A_{2,1} & A_{2,2}\\
A_{3,1} & A_{3,2}
\end{bmatrix} \implies \boldsymbol{A}^{\top}=\begin{bmatrix}
A_{1,1} & A_{2,1} & A_{3,1}\\
A_{1,2} & A_{2,2} & A_{3,2}
\end{bmatrix}
$$

A vector is a matrix with one column; its transpose is a row vector: $\boldsymbol{x}=[x_1,x_2,x_3]^{\top}$. A scalar satisfies $a=a^{\top}$.

**Addition** (same-shape matrices): $C_{i,j}=A_{i,j}+B_{i,j}$.

**Scalar multiplication**: $D_{i,j}=a\cdot B_{i,j}+c$ for $\boldsymbol{D}=a\cdot\boldsymbol{B}+c$.

**Broadcasting** (deep learning convention): $\boldsymbol{C}=\boldsymbol{A}+\boldsymbol{b}$ adds vector $\boldsymbol{b}$ to every row of $\boldsymbol{A}$, i.e. $C_{i,j}=A_{i,j}+b_j$.

## Applications

- Building block for all of linear algebra and matrix calculus.
- Representing datasets (rows = samples, columns = features) and linear transformations.
- Broadcasting is pervasive in numerical computing frameworks (NumPy, PyTorch).

## Trade-offs

- Addition and scalar multiplication require identical shapes (or broadcastable shapes); mixing shapes without broadcasting raises dimension errors.
- Broadcasting can silently produce unexpected shapes if dimensions are mismatched — always verify shapes explicitly.
- Tensors beyond rank 2 can be hard to visualise; index notation is essential to reason about them correctly.

## Links

- [[matrix_operations|Matrix Operations]]
- [[special_matrices|Special Matrices]]
