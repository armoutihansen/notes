---
layer: 01_foundations
type: concept
status: seed
tags: [theory]
created: 2026-03-01
---

# Matrix Operations

## Definition

Matrix operations include addition, scalar multiplication, matrix-matrix multiplication, matrix-vector multiplication, and the transpose. They form the algebraic foundation for linear algebra.

## Intuition

Matrix–matrix multiplication computes all dot products between rows of the left matrix and columns of the right matrix — think of it as composing two linear transformations. The transpose geometrically "reflects" a matrix across its diagonal. The Hadamard product is purely element-wise and has no linear-map interpretation on its own.

## Formal Description

### Matrix Addition

Matrices can be added only if they have the same dimensions; addition is element-wise:
$$
\begin{pmatrix}a & b\\ c & d\end{pmatrix}+\begin{pmatrix}e & f\\ g & h\end{pmatrix}=\begin{pmatrix}a+e & b+f\\ c+g & d+h\end{pmatrix}
$$
i.e. $C_{i,j}=A_{i,j}+B_{i,j}$.

### Scalar Multiplication

Every element of the matrix is multiplied by the scalar $k$:
$$
k\begin{pmatrix}a & b\\ c & d\end{pmatrix}=\begin{pmatrix}ka & kb\\ kc & kd\end{pmatrix}
$$

### Matrix–Matrix Multiplication

An $m\times n$ matrix $\boldsymbol{A}$ can be multiplied by an $n\times p$ matrix $\boldsymbol{B}$; the result $\boldsymbol{C}=\boldsymbol{AB}$ is $m\times p$:
$$
C_{i,j}=\sum_{k=1}^{n}A_{i,k}B_{k,j}
$$

Example ($2\times 2$):
$$
\begin{pmatrix}a & b\\ c & d\end{pmatrix}\begin{pmatrix}e & f\\ g & h\end{pmatrix}=\begin{pmatrix}ae+bg & af+bh\\ ce+dg & cf+dh\end{pmatrix}
$$

The **Hadamard product** (element-wise product) is denoted $\boldsymbol{A}\odot\boldsymbol{B}$ and is distinct from standard matrix multiplication.

### Matrix–Vector Multiplication

For $\boldsymbol{A}\in\mathbb{R}^{m\times n}$ and $\boldsymbol{x}\in\mathbb{R}^n$, the product $\boldsymbol{Ax}=\boldsymbol{b}$ with $\boldsymbol{b}\in\mathbb{R}^m$ represents a system of linear equations.

The **dot product** of two vectors $\boldsymbol{x},\boldsymbol{y}\in\mathbb{R}^n$ is the matrix product $\boldsymbol{x}^{\top}\boldsymbol{y}$.

### Transpose

The transpose $\boldsymbol{A}^{\top}$ switches rows and columns: $(\boldsymbol{A}^{\top})_{i,j}=A_{j,i}$. For an $m\times n$ matrix, $\boldsymbol{A}^{\top}$ is $n\times m$.
$$
\boldsymbol{A}=\begin{pmatrix}a_{11} & a_{12} & \cdots & a_{1n}\\ a_{21} & a_{22} & \cdots & a_{2n}\\ \vdots & \vdots & \ddots & \vdots\\ a_{m1} & a_{m2} & \cdots & a_{mn}\end{pmatrix}\implies \boldsymbol{A}^{\top}=\begin{pmatrix}a_{11} & a_{21} & \cdots & a_{m1}\\ a_{12} & a_{22} & \cdots & a_{m2}\\ \vdots & \vdots & \ddots & \vdots\\ a_{1n} & a_{2n} & \cdots & a_{mn}\end{pmatrix}
$$

**Matrix multiplication properties:**
- Distributive: $\boldsymbol{A}(\boldsymbol{B}+\boldsymbol{C})=\boldsymbol{AB}+\boldsymbol{AC}$
- Associative: $\boldsymbol{A}(\boldsymbol{BC})=(\boldsymbol{AB})\boldsymbol{C}$
- Not commutative in general: $\boldsymbol{AB}\neq\boldsymbol{BA}$

**Transpose properties:**
- $(\boldsymbol{A}^{\top})^{\top}=\boldsymbol{A}$
- $(\boldsymbol{A}+\boldsymbol{B})^{\top}=\boldsymbol{A}^{\top}+\boldsymbol{B}^{\top}$
- $(\boldsymbol{AB})^{\top}=\boldsymbol{B}^{\top}\boldsymbol{A}^{\top}$
- Dot product commutativity: $\boldsymbol{x}^{\top}\boldsymbol{y}=\boldsymbol{y}^{\top}\boldsymbol{x}$

**Symmetry:**
- $\boldsymbol{A}$ is *symmetric* if $\boldsymbol{A}^{\top}=\boldsymbol{A}$.
- $\boldsymbol{A}$ is *skew-symmetric* if $\boldsymbol{A}^{\top}=-\boldsymbol{A}$ (diagonal elements must be zero).

Examples:
$$
\text{symmetric: }\begin{pmatrix}a & b & c\\ b & d & e\\ c & e & f\end{pmatrix},\quad\text{skew-symmetric: }\begin{pmatrix}0 & b & c\\ -b & 0 & e\\ -c & -e & 0\end{pmatrix}
$$

## Applications

- Expressing systems of linear equations as $\boldsymbol{Ax}=\boldsymbol{b}$, where $\boldsymbol{A}\in\mathbb{R}^{m\times n}$, $\boldsymbol{b}\in\mathbb{R}^m$, $\boldsymbol{x}\in\mathbb{R}^n$.
- Broadcasting in deep learning: $C_{i,j}=A_{i,j}+b_j$ adds a vector to every row of a matrix.

## Trade-offs

- Matrix multiplication is **not commutative**: $\boldsymbol{AB}\neq\boldsymbol{BA}$ in general, even when both products are defined. Reversing operand order is a common source of bugs.
- Dimensions must be compatible for multiplication ($m\times n$ by $n\times p$); addition requires identical shapes.
- The Hadamard product $\boldsymbol{A}\odot\boldsymbol{B}$ is commutative but does not correspond to function composition — mixing it with standard multiplication requires care.

## Links

- [[scalars_vectors_matrices|Scalars, Vectors, Matrices and Tensors]]
- [[inverse_matrix|Inverse Matrix]]
- [[inner_product|Inner Product]]
- [[outer_product|Outer Product]]
- [[special_matrices|Special Matrices]]
