---
layer: 01_foundations
type: concept
status: seed
tags: [algorithm]
created: 2026-03-01
---

# Orthogonal Projections

## Definition

Let $\mathrm{V}$ be an $n$-dimensional vector space and $\mathrm{W}$ a $p$-dimensional subspace with orthonormal [[span_basis_dimension|basis]] $\{\mathrm{s}_1, \dots, \mathrm{s}_p\}$. The **orthogonal projection** of $\mathrm{v} \in \mathrm{V}$ onto $\mathrm{W}$ is the unique vector in $\mathrm{W}$ closest to $\mathrm{v}$.

## Intuition

Orthogonal projection drops a perpendicular from $\mathrm{v}$ onto the subspace $\mathrm{W}$: the foot of that perpendicular is the projection. The residual $\mathrm{v} - \mathrm{v}_{\operatorname{proj}}$ is orthogonal to every vector in $\mathrm{W}$, which is exactly what "closest point" means. When $\mathrm{W} = \mathrm{Col(A)}$, projecting $\mathrm{b}$ finds the nearest achievable output to $\mathrm{b}$, which is the least-squares solution.

## Formal Description

Any $\mathrm{v} \in \mathrm{V}$ can be written using a full orthonormal basis $\{\mathrm{s}_1, \dots, \mathrm{s}_p, \mathrm{t}_1, \dots, \mathrm{t}_{n-p}\}$ as
$$
\mathrm{v} = a_1\mathrm{s}_1 + \dots + a_p\mathrm{s}_p + b_1\mathrm{t}_1 + \dots + b_{n-p}\mathrm{t}_{n-p}.
$$
The orthogonal projection onto $\mathrm{W}$ retains only the $\mathrm{W}$-components:
$$
\mathrm{v}_{\operatorname{proj}_{\mathrm{W}}} = a_1\mathrm{s}_1 + \dots + a_p\mathrm{s}_p,
$$
where the coefficients are computed via inner products: $a_k = \mathrm{v}^T \mathrm{s}_k$. Therefore:
$$
\mathrm{v}_{\operatorname{proj}_{\mathrm{W}}} = (\mathrm{v}^T\mathrm{s}_1)\mathrm{s}_1 + \dots + (\mathrm{v}^T\mathrm{s}_p)\mathrm{s}_p.
$$

**Minimality.** $\mathrm{v}_{\operatorname{proj}_{\mathrm{W}}}$ is the closest point in $\mathrm{W}$ to $\mathrm{v}$. For any $\mathrm{w} \in \mathrm{W}$ with expansion $\mathrm{w} = c_1\mathrm{s}_1 + \dots + c_p\mathrm{s}_p$:
$$
\|\mathrm{v} - \mathrm{w}\|^2 = (a_1-c_1)^2 + \dots + (a_p-c_p)^2 + b_1^2 + \dots + b_{n-p}^2 \geq \|\mathrm{v} - \mathrm{v}_{\operatorname{proj}_{\mathrm{W}}}\|^2.
$$

**Projection matrix.** When $\mathrm{W} = \mathrm{Col(A)}$ and the columns of $\mathrm{A}$ are linearly independent, the projection of $\mathrm{b}$ onto $\mathrm{Col(A)}$ is given by
$$
\mathrm{b}_{\operatorname{proj}_{\mathrm{Col(A)}}} = \mathrm{A}(\mathrm{A}^T\mathrm{A})^{-1}\mathrm{A}^T\mathrm{b}.
$$
The matrix $\mathrm{P} = \mathrm{A}(\mathrm{A}^T\mathrm{A})^{-1}\mathrm{A}^T$ is idempotent: $\mathrm{P}^2 = \mathrm{P}$.

## Applications

- Least-squares regression projects the target $\mathrm{b}$ onto the column space of the design matrix.
- Gram-Schmidt uses successive projections to build an orthonormal basis.
- Signal processing: projecting a signal onto a subspace removes noise components outside the subspace.

## Trade-offs

The formula $\mathrm{P} = \mathrm{A}(\mathrm{A}^T\mathrm{A})^{-1}\mathrm{A}^T$ requires $\mathrm{A}^T\mathrm{A}$ to be invertible (full column rank). If $\mathrm{A}$ is rank-deficient, use the pseudoinverse $\mathrm{A}^+$ instead. Explicitly forming $\mathrm{P}$ is $O(m^2 n)$ and memory-intensive; in practice, solve the normal equations directly rather than materialising $\mathrm{P}$.

## Links

- [[span_basis_dimension|Span, Basis and Dimension]]
- [[gram_schmidt|Gram-Schmidt Process]]
- [[least_squares|Least Squares]]
- [[column_space|Column Space]]
