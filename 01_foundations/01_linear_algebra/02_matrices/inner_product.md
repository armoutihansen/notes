---
layer: 01_foundations
type: concept
status: seed
tags: [theory]
created: 2026-03-01
---

# Inner Product

## Definition

The **inner product** (dot product, scalar product) of two vectors is the matrix product of a row vector with a column vector, yielding a scalar.

## Intuition

Geometrically, $\boldsymbol{u}^{\top}\boldsymbol{v}=\|\boldsymbol{u}\|\|\boldsymbol{v}\|\cos\theta$ where $\theta$ is the angle between the two vectors. It measures how much one vector "projects onto" the other. When the inner product is zero, the vectors point in completely perpendicular directions; when it equals the product of their norms, they are perfectly aligned.

## Formal Description

For $3\times 1$ column vectors $\boldsymbol{u},\boldsymbol{v}$:
$$
\boldsymbol{u}^{\top}\boldsymbol{v}=\begin{pmatrix}u_1 & u_2 & u_3\end{pmatrix}\begin{pmatrix}v_1\\ v_2\\ v_3\end{pmatrix}=u_1v_1+u_2v_2+u_3v_3
$$

- If $\boldsymbol{u}^{\top}\boldsymbol{v}=0$ for nonzero $\boldsymbol{u},\boldsymbol{v}$, the vectors are **orthogonal**.
- The **norm** (Euclidean length) of a vector: $\|\boldsymbol{u}\|=(\boldsymbol{u}^{\top}\boldsymbol{u})^{1/2}=(u_1^2+u_2^2+u_3^2)^{1/2}$.
- A vector with $\|\boldsymbol{u}\|=1$ is **normalised**.
- A set of vectors that are mutually orthogonal and each normalised is called **orthonormal**.
- The inner product is commutative: $\boldsymbol{u}^{\top}\boldsymbol{v}=\boldsymbol{v}^{\top}\boldsymbol{u}$.

## Applications

- Computing similarity between vectors (e.g. cosine similarity in NLP and recommendation systems).
- Defining the Euclidean norm and distance for optimisation and nearest-neighbour search.
- Checking orthogonality; forming orthonormal bases via Gram–Schmidt.

## Trade-offs

- The standard inner product assumes the Euclidean metric; in weighted or non-Euclidean spaces a different inner product (e.g. $\boldsymbol{u}^{\top}\boldsymbol{M}\boldsymbol{v}$ for positive-definite $\boldsymbol{M}$) may be more appropriate.
- For complex vectors, the inner product is $\boldsymbol{u}^{*}\boldsymbol{v}$ (conjugate transpose), not $\boldsymbol{u}^{\top}\boldsymbol{v}$; confusing the two breaks conjugate symmetry and positive-definiteness.
- The inner product collapses two vectors to a single number, discarding directional information — use the outer product when the full rank-1 interaction matrix is needed.

## Links

- [[matrix_operations|Matrix Operations]]
- [[outer_product|Outer Product]]
- [[special_matrices|Special Matrices]] (orthogonal matrices)
