---
layer: 01_foundations
type: proof
status: seed
tags: [theory]
created: 2026-03-01
---

# Wronskian

## Statement

Two solutions $X_{1}(t)$ and $X_{2}(t)$ of a homogeneous linear second-order ODE are linearly independent if and only if their Wronskian
$$
W=\begin{vmatrix}
X_{1}(t_{0}) & X_{2}(t_{0}) \\
\dot{X}_{1}(t_{0}) & \dot{X}_{2}(t_{0})
\end{vmatrix}=X_{1}(t_{0})\dot{X}_{2}(t_{0})-\dot{X}_{1}(t_{0})X_{2}(t_{0})
$$
is nonzero at some (equivalently, any) point $t_{0}$.

## Assumptions

- The ODE $\ddot{x}+p(t)\dot{x}+q(t)x=0$ has continuous coefficients $p,q$ on an interval $I$.
- $X_{1},X_{2}$ are solutions on $I$.

## Proof Sketch

The general solution $x=c_{1}X_{1}+c_{2}X_{2}$ can be matched to arbitrary initial conditions $(x_{0},u_{0})$ by solving a $2\times 2$ linear system whose coefficient matrix has determinant $W$. By the invertibility criterion, a unique solution $(c_1, c_2)$ exists if and only if $W\neq 0$, which is equivalent to linear independence of $X_1$ and $X_2$.

## Full Proof

Imposing $x(t_{0})=x_{0}$ and $\dot{x}(t_{0})=u_{0}$ on $x=c_{1}X_{1}+c_{2}X_{2}$ gives the system
$$
c_{1}X_{1}(t_{0})+c_{2}X_{2}(t_{0})=x_{0},\quad c_{1}\dot{X}_{1}(t_{0})+c_{2}\dot{X}_{2}(t_{0})=u_{0}.
$$
In matrix form:
$$
\begin{pmatrix}X_{1}(t_{0})&X_{2}(t_{0})\\\dot{X}_{1}(t_{0})&\dot{X}_{2}(t_{0})\end{pmatrix}\begin{pmatrix}c_{1}\\c_{2}\end{pmatrix}=\begin{pmatrix}x_{0}\\u_{0}\end{pmatrix}.
$$
This system has a unique solution for any $(x_{0},u_{0})$ if and only if the coefficient matrix is invertible, i.e.\ $W\neq 0$.

If $W=0$, then either $X_{1}$ and $X_{2}$ are linearly dependent (one is a scalar multiple of the other), or the pair fails to span the solution space, and initial conditions cannot always be satisfied uniquely. $\square$

## Notes / Intuition

- $W\neq 0$ if and only if $X_{1}$ and $X_{2}$ are **linearly independent** as functions on $I$.
- The solution space of a second-order homogeneous linear ODE is two-dimensional; two independent solutions form a basis.
- **Example:** $X_{1}=\cos\omega t$, $X_{2}=\sin\omega t$ ($\omega\neq 0$) give $W=\omega\neq 0$ for all $t$, confirming independence.
- The Wronskian is used in [[homogeneous_second_order|homogeneous second-order ODEs]] to verify that the two solutions obtained from the [[characteristic_roots|characteristic roots]] are genuinely independent.

## Links

- [[homogeneous_second_order|Homogeneous Second-Order ODE]]
- [[superposition|Principle of Superposition]]
- [[characteristic_roots|Characteristic Roots]]
