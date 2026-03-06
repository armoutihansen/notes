---
layer: 01_foundations
type: proof
status: seed
tags: [algorithm, regression]
created: 2026-03-01
---

# Least Squares and Normal Equations

## Statement

Given an over-determined system $\mathrm{Ax} = \mathrm{b}$ (where $\mathrm{b} \notin \mathrm{Col(A)}$), the vector $\hat{\mathrm{x}}$ that minimises $\|\mathrm{Ax} - \mathrm{b}\|_2$ satisfies the **normal equations**:
$$
\mathrm{A}^T\mathrm{A}\,\hat{\mathrm{x}} = \mathrm{A}^T\mathrm{b}.
$$
When the columns of $\mathrm{A}$ are linearly independent, the solution is unique: $\hat{\mathrm{x}} = (\mathrm{A}^T\mathrm{A})^{-1}\mathrm{A}^T\mathrm{b}$.

## Assumptions

- $\mathrm{A}$ is $m \times n$ with $m > n$ (over-determined).
- The columns of $\mathrm{A}$ are linearly independent (full column rank), so $\mathrm{A}^T\mathrm{A}$ is invertible.
- $\mathrm{b} \in \mathbb{R}^m$.

## Proof Sketch

The minimum of $\|\mathrm{Ax} - \mathrm{b}\|_2$ is achieved when $\mathrm{Ax}$ is as close to $\mathrm{b}$ as possible inside $\mathrm{Col(A)}$ — i.e., when $\mathrm{Ax} = \mathrm{b}_{\operatorname{proj}}$, the [[orthogonal_projections|orthogonal projection]] of $\mathrm{b}$ onto $\mathrm{Col(A)}$. The residual $\mathrm{b} - \mathrm{A}\hat{\mathrm{x}}$ must be orthogonal to every column of $\mathrm{A}$, which gives $\mathrm{A}^T(\mathrm{b} - \mathrm{A}\hat{\mathrm{x}}) = 0$, i.e., the normal equations.

## Full Proof

**Step 1: Decompose $\mathrm{b}$.** Write $\mathrm{b} = \mathrm{b}_{\operatorname{proj}} + \mathrm{r}$, where $\mathrm{b}_{\operatorname{proj}} \in \mathrm{Col(A)}$ and $\mathrm{r} = \mathrm{b} - \mathrm{b}_{\operatorname{proj}}$ is orthogonal to $\mathrm{Col(A)}$. Since $\mathrm{r} \perp \mathrm{Col(A)}$, we have $\mathrm{r} \in \mathrm{Null(A^T)}$, so $\mathrm{A}^T\mathrm{r} = 0$.

**Step 2: Optimality condition.** For any $\mathrm{x}$:
$$
\|\mathrm{Ax} - \mathrm{b}\|^2 = \|\mathrm{Ax} - \mathrm{b}_{\operatorname{proj}} - \mathrm{r}\|^2 = \|\mathrm{Ax} - \mathrm{b}_{\operatorname{proj}}\|^2 + \|\mathrm{r}\|^2,
$$
using the Pythagorean theorem (since $\mathrm{Ax} - \mathrm{b}_{\operatorname{proj}} \in \mathrm{Col(A)}$ and $\mathrm{r} \perp \mathrm{Col(A)}$). This is minimised when $\mathrm{Ax} = \mathrm{b}_{\operatorname{proj}}$, i.e., $\|\mathrm{Ax} - \mathrm{b}_{\operatorname{proj}}\|^2 = 0$.

**Step 3: Derive normal equations.** Multiply $\mathrm{A}\hat{\mathrm{x}} = \mathrm{b}_{\operatorname{proj}}$ on the left by $\mathrm{A}^T$:
$$
\mathrm{A}^T\mathrm{A}\hat{\mathrm{x}} = \mathrm{A}^T\mathrm{b}_{\operatorname{proj}} = \mathrm{A}^T(\mathrm{b} - \mathrm{r}) = \mathrm{A}^T\mathrm{b} - \mathrm{A}^T\mathrm{r} = \mathrm{A}^T\mathrm{b}.
$$

**Step 4: Unique solution.** By full column rank, $\mathrm{A}^T\mathrm{A}$ is symmetric positive definite and hence invertible:
$$
\hat{\mathrm{x}} = (\mathrm{A}^T\mathrm{A})^{-1}\mathrm{A}^T\mathrm{b}.
$$

**Projection matrix.** The projection $\mathrm{b}_{\operatorname{proj}} = \mathrm{A}\hat{\mathrm{x}} = \mathrm{A}(\mathrm{A}^T\mathrm{A})^{-1}\mathrm{A}^T\mathrm{b} =: \mathrm{P}\mathrm{b}$ satisfies $\mathrm{P}^2 = \mathrm{P}$ (idempotent). If $\mathrm{A}$ is square and invertible, $\mathrm{P} = \mathrm{I}$.

**Example (linear regression).** Fitting a line through $(1,1),(2,3),(3,2)$ gives
$$
\begin{pmatrix}3 & 6 \\ 6 & 14\end{pmatrix}\begin{pmatrix}\beta_0 \\ \beta_1\end{pmatrix} = \begin{pmatrix}6 \\ 13\end{pmatrix},
$$
yielding $\beta_0 = 1$, $\beta_1 = \tfrac{1}{2}$, so the least-squares line is $y = 1 + \tfrac{x}{2}$.

## Notes / Intuition

The normal equations say: the residual $\mathrm{b} - \mathrm{A}\hat{\mathrm{x}}$ must be orthogonal to the column space — which is exactly what "nearest point in a subspace" means. Numerically, solving the normal equations by forming $\mathrm{A}^T\mathrm{A}$ squares the condition number; using QR factorisation ($\mathrm{A} = \mathrm{QR}$, then solving $\mathrm{R}\hat{\mathrm{x}} = \mathrm{Q}^T\mathrm{b}$) is preferred for numerical stability.

## Links

- [[orthogonal_projections|Orthogonal Projections]]
- [[column_space|Column Space]]
- [[null_space|Null Space]]
