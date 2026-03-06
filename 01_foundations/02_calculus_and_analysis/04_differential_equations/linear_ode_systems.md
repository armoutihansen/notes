---
layer: 01_foundations
type: concept
status: seed
tags: [algorithm]
created: 2026-03-01
---

# Linear ODE Systems

## Definition

A **system of homogeneous linear first-order ODEs** with constant coefficients:
$$
\dot{x}_{1}=ax_{1}+bx_{2},\quad\dot{x}_{2}=cx_{1}+dx_{2},
$$
written in matrix form as $\dot{\mathbf{x}}=\mathbf{A}\mathbf{x}$.

## Intuition

The matrix $\mathbf{A}$ stretches and rotates the state vector $\mathbf{x}$; the natural directions of stretching are the eigenvectors. Along each eigendirection the system decouples into a scalar ODE $\dot{v}=\lambda v$, so the solution is a sum of exponentials weighted by eigenvalues — a direct vector-space generalisation of the scalar [[characteristic_roots|characteristic root]] cases.

## Formal Description

The exponential ansatz $\mathbf{x}(t)=\mathbf{v}e^{\lambda t}$ converts the system into the **eigenvalue problem**
$$
\mathbf{A}\mathbf{v}=\lambda\mathbf{v}.
$$
The characteristic equation for the $2\times 2$ matrix is
$$
\det(\mathbf{A}-\lambda\mathbf{I})=\lambda^{2}-(a+d)\lambda+(ad-bc)=0.
$$
Each eigenvalue $\lambda_{k}$ with eigenvector $\mathbf{v}_{k}$ gives a solution $\mathbf{v}_{k}e^{\lambda_{k}t}$.

### Key Results

Three cases arise, analogous to the scalar [[characteristic_roots|characteristic root]] cases:

| Eigenvalues | General solution |
|---|---|
| Distinct real $\lambda_{1}\neq\lambda_{2}$ | $\mathbf{x}=c_{1}\mathbf{v}_{1}e^{\lambda_{1}t}+c_{2}\mathbf{v}_{2}e^{\lambda_{2}t}$ |
| Complex conjugates $\lambda=\alpha\pm i\beta$ | Separate real and imaginary parts using Euler's formula |
| Repeated eigenvalue | Second solution involves $t e^{\lambda t}$ (as in scalar case) |

The [[superposition|principle of superposition]] applies since the system is linear.

## Applications

- Phase-plane analysis and stability of equilibria in two-dimensional dynamical systems.
- Coupled mechanical or electrical oscillators.
- Reducing any $n$th-order scalar ODE to a first-order system (companion matrix form) for numerical or analytical treatment.

## Trade-offs

- Finding eigenvectors requires solving $(\mathbf{A}-\lambda\mathbf{I})\mathbf{v}=0$; repeated eigenvalues may yield defective matrices requiring generalised eigenvectors.
- For complex eigenvalues, Euler's formula is needed to extract real-valued solution components.
- Scales well to $n\times n$ systems in principle, but eigendecomposition becomes expensive for large $n$.

## Links

- [[characteristic_roots|Characteristic Roots]]
- [[homogeneous_second_order|Homogeneous Second-Order ODE]]
- [[superposition|Principle of Superposition]]
