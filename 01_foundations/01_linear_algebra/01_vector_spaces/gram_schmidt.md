---
layer: 01_foundations
type: proof
status: seed
tags: [algorithm]
created: 2026-03-01
---

# Gram-Schmidt Process

## Statement

Given any [[span_basis_dimension|basis]] $\{\mathrm{v}_1, \dots, \mathrm{v}_n\}$ for an $n$-dimensional [[vector_space|vector space]], the Gram-Schmidt process produces an orthonormal basis $\{\hat{\mathrm{u}}_1, \dots, \hat{\mathrm{u}}_n\}$ for the same space such that $\operatorname{span}\{\hat{\mathrm{u}}_1, \dots, \hat{\mathrm{u}}_k\} = \operatorname{span}\{\mathrm{v}_1, \dots, \mathrm{v}_k\}$ for all $k$.

## Assumptions

- $\{\mathrm{v}_1, \dots, \mathrm{v}_n\}$ is a basis (linearly independent spanning set) for a finite-dimensional inner product space over $\mathbb{R}$.
- No $\mathrm{v}_k$ is zero (guaranteed by linear independence).

## Proof Sketch

At each step $k$, subtract from $\mathrm{v}_k$ its projections onto all previously constructed orthogonal vectors $\mathrm{u}_1, \dots, \mathrm{u}_{k-1}$. The residual is nonzero (because $\mathrm{v}_k$ is not in $\operatorname{span}\{\mathrm{v}_1,\dots,\mathrm{v}_{k-1}\}$) and orthogonal to all prior vectors by construction. Normalising gives a unit vector. Since each $\mathrm{u}_k$ is a linear combination of $\mathrm{v}_1, \dots, \mathrm{v}_k$, the intermediate spans are preserved.

## Full Proof

**Algorithm.** Set $\mathrm{u}_1 = \mathrm{v}_1$. For $k = 2, \dots, n$:
$$
\mathrm{u}_k = \mathrm{v}_k - \sum_{j=1}^{k-1} \frac{\mathrm{u}_j^T\mathrm{v}_k}{\mathrm{u}_j^T\mathrm{u}_j}\,\mathrm{u}_j.
$$
Normalize:
$$
\hat{\mathrm{u}}_k = \frac{\mathrm{u}_k}{(\mathrm{u}_k^T\mathrm{u}_k)^{1/2}}.
$$

**Orthogonality.** For $i < k$:
$$
\mathrm{u}_i^T\mathrm{u}_k = \mathrm{u}_i^T\mathrm{v}_k - \frac{\mathrm{u}_i^T\mathrm{v}_k}{\mathrm{u}_i^T\mathrm{u}_i}\,(\mathrm{u}_i^T\mathrm{u}_i) - \sum_{j \neq i} \frac{\mathrm{u}_j^T\mathrm{v}_k}{\mathrm{u}_j^T\mathrm{u}_j}\,(\mathrm{u}_i^T\mathrm{u}_j) = \mathrm{u}_i^T\mathrm{v}_k - \mathrm{u}_i^T\mathrm{v}_k = 0,
$$
where $\mathrm{u}_i^T\mathrm{u}_j = 0$ for $j \neq i$ by induction.

**Non-degeneracy.** $\mathrm{u}_k \neq 0$ because $\mathrm{v}_k \notin \operatorname{span}\{\mathrm{v}_1, \dots, \mathrm{v}_{k-1}\} = \operatorname{span}\{\mathrm{u}_1, \dots, \mathrm{u}_{k-1}\}$; if $\mathrm{u}_k = 0$ then $\mathrm{v}_k$ would be a linear combination of $\mathrm{u}_1, \dots, \mathrm{u}_{k-1}$, contradicting linear independence.

**Span preservation.** Each $\mathrm{u}_k = \mathrm{v}_k - \sum_{j<k}(\cdot)\mathrm{u}_j$ is a linear combination of $\mathrm{v}_1, \dots, \mathrm{v}_k$, so $\operatorname{span}\{\mathrm{u}_1,\dots,\mathrm{u}_k\} \subseteq \operatorname{span}\{\mathrm{v}_1,\dots,\mathrm{v}_k\}$. Both sets have the same cardinality $k$ and are linearly independent (orthogonal vectors are independent), so the spans are equal.

**Example.** For the basis $\left\{\begin{pmatrix}1\\1\\1\end{pmatrix}, \begin{pmatrix}0\\1\\1\end{pmatrix}\right\}$:
$$
\mathrm{u}_1 = \begin{pmatrix}1\\1\\1\end{pmatrix}, \quad
\mathrm{u}_2 = \begin{pmatrix}0\\1\\1\end{pmatrix} - \frac{2}{3}\begin{pmatrix}1\\1\\1\end{pmatrix} = \frac{1}{3}\begin{pmatrix}-2\\1\\1\end{pmatrix},
$$
giving the orthonormal basis $\left\{\frac{1}{\sqrt{3}}\begin{pmatrix}1\\1\\1\end{pmatrix},\ \frac{1}{\sqrt{6}}\begin{pmatrix}-2\\1\\1\end{pmatrix}\right\}$.

## Notes / Intuition

Each step is a projection-and-subtract: we remove from $\mathrm{v}_k$ everything it "shares" with the already-built directions, leaving only a genuinely new orthogonal component. This is the constructive heart of the QR decomposition ($\mathrm{A} = \mathrm{QR}$, where $\mathrm{Q}$ has the $\hat{\mathrm{u}}_k$ as columns). Classical Gram-Schmidt is numerically unstable; the **modified** variant (subtracting projections one at a time) is preferred in practice.

## Links

- [[span_basis_dimension|Span, Basis and Dimension]]
- [[orthogonal_projections|Orthogonal Projections]]
- [[vector_space|Vector Space]]
