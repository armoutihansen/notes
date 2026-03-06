---
layer: 01_foundations
type: concept
status: growing
tags: [svd, singular-value-decomposition, matrix-factorization, pca, pseudoinverse]
created: 2026-03-06
---

# Singular Value Decomposition (SVD)

## Definition

Every real matrix $A \in \mathbb{R}^{m \times n}$ can be factorised as

$$A = U \Sigma V^\top$$

where $U \in \mathbb{R}^{m \times m}$ and $V \in \mathbb{R}^{n \times n}$ are orthogonal matrices and $\Sigma \in \mathbb{R}^{m \times n}$ is a diagonal matrix with non-negative entries $\sigma_1 \geq \sigma_2 \geq \cdots \geq \sigma_r > 0$ (the **singular values**), $r = \text{rank}(A)$.

## Intuition

Every linear map decomposes into three geometric steps: (1) rotate/reflect input space ($V^\top$), (2) scale along each axis ($\Sigma$), and (3) rotate/reflect output space ($U$). The singular values $\sigma_i$ measure how strongly the map stretches in each direction — the first is the largest and captures the most "action".

This is the most general matrix factorisation: it always exists, requires no special structure (unlike eigendecomposition), and applies to rectangular matrices.

## Formal Description

**Thin (economy) SVD:** keep only the $r$ non-zero singular values:

$$A = U_r \Sigma_r V_r^\top \qquad U_r \in \mathbb{R}^{m \times r},\ \Sigma_r \in \mathbb{R}^{r \times r},\ V_r \in \mathbb{R}^{n \times r}$$

**Relationship to eigendecomposition:**

$$A^\top A = V \Sigma^\top \Sigma V^\top \qquad (\text{eigendecomp of } A^\top A)$$

$$A A^\top = U \Sigma \Sigma^\top U^\top \qquad (\text{eigendecomp of } A A^\top)$$

The singular values are $\sigma_i = \sqrt{\lambda_i(A^\top A)}$; the columns of $V$ are eigenvectors of $A^\top A$ (right singular vectors); columns of $U$ are eigenvectors of $A A^\top$ (left singular vectors).

**Rank-$k$ approximation (Eckart-Young theorem):**

$$A_k = U_k \Sigma_k V_k^\top = \sum_{i=1}^k \sigma_i\, u_i v_i^\top$$

minimises $\|A - B\|_F$ (Frobenius) and $\|A - B\|_2$ (spectral) over all rank-$k$ matrices $B$. The approximation error is $\|A - A_k\|_F^2 = \sum_{i=k+1}^r \sigma_i^2$.

**Moore-Penrose pseudoinverse:**

$$A^+ = V \Sigma^+ U^\top$$

where $\Sigma^+$ replaces each non-zero $\sigma_i$ by $1/\sigma_i$. Gives the minimum-norm least-squares solution to $A\mathbf{x} = \mathbf{b}$: $\hat{\mathbf{x}} = A^+\mathbf{b}$.

**Condition number:**

$$\kappa(A) = \frac{\sigma_{\max}}{\sigma_{\min}}$$

A large condition number indicates a near-singular (ill-conditioned) matrix; small perturbations in $\mathbf{b}$ can cause large changes in $\hat{\mathbf{x}}$.

**Numerical computation:** the Golub-Reinsch algorithm computes SVD in $O(\min(m,n) \cdot mn)$ time via bidiagonalization followed by QR iteration.

## Applications

| Application | How SVD is used |
|---|---|
| PCA | Principal components = right singular vectors of centred data matrix $X$; $\sigma_i^2 / (n-1)$ = variance explained |
| Low-rank approximation | Compress images, text co-occurrence matrices, recommendation systems |
| Pseudoinverse / least squares | Solve overdetermined or rank-deficient systems $A\mathbf{x} \approx \mathbf{b}$ |
| Latent Semantic Analysis (LSA) | Compress term-document matrix to $k$ latent semantic dimensions |
| Noise reduction | Truncate small singular values to remove noise |
| Numerical linear algebra | Matrix condition estimation, regularization |

## Trade-offs

- Full SVD costs $O(m^2 n)$ for $m \geq n$; use randomized SVD (e.g., `sklearn.utils.extmath.randomized_svd`) for large matrices when only the top-$k$ factors are needed.
- Numerically more stable than eigendecomposition of $A^\top A$ (squaring the condition number).
- Rank-$k$ truncation is the globally optimal low-rank approximation — no other rank-$k$ factorisation is better in Frobenius or spectral norm.

## Links

- [[eigenvalues_and_eigenvectors|Eigenvalues and Eigenvectors]] — singular values are eigenvalues of $A^\top A$
- [[matrix_diagonalization|Matrix Diagonalization]] — SVD generalises eigendecomposition to rectangular matrices
- [[01_foundations/01_linear_algebra/01_vector_spaces/orthogonal_projections|Orthogonal Projections]] — columns of $U_k$, $V_k$ are orthonormal bases for column/row spaces
- [[02_modeling/03_model_families/06_unsupervised_learning/unsupervised_learning|PCA and Unsupervised Learning]]
