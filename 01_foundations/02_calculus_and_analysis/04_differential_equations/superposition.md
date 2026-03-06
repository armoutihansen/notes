---
layer: 01_foundations
type: proof
status: seed
tags: [theory]
created: 2026-03-01
---

# Principle of Superposition

## Statement

Any linear combination of solutions to a homogeneous linear ODE is also a solution.

## Assumptions

- The ODE is **linear** and **homogeneous**: $\ddot{x}+p(t)\dot{x}+q(t)x=0$.
- $X_{1}(t)$ and $X_{2}(t)$ are solutions on a common interval.
- $c_{1},c_{2}$ are arbitrary constants.

## Proof Sketch

Because differentiation is linear, substituting $x=c_{1}X_{1}+c_{2}X_{2}$ into the ODE separates into two copies of the original equation — each equal to zero — whose sum is zero. No nonlinear terms appear, so the combination is itself a solution.

## Full Proof

Let $X_{1}$ and $X_{2}$ satisfy the ODE:
$$
\ddot{X}_{i}+p(t)\dot{X}_{i}+q(t)X_{i}=0,\quad i=1,2.
$$
Set $x=c_{1}X_{1}+c_{2}X_{2}$. Then
$$
\ddot{x}+p\dot{x}+qx
=c_{1}\!\left(\ddot{X}_{1}+p\dot{X}_{1}+qX_{1}\right)+c_{2}\!\left(\ddot{X}_{2}+p\dot{X}_{2}+qX_{2}\right)
=c_{1}\cdot 0+c_{2}\cdot 0=0.
$$
Hence $x=c_{1}X_{1}+c_{2}X_{2}$ satisfies the ODE. $\square$

## Notes / Intuition

- Superposition holds for any linear homogeneous ODE; it fails for nonlinear ODEs.
- The set of all solutions to a second-order homogeneous linear ODE forms a two-dimensional vector space; any two linearly independent solutions form a basis.
- Superposition is the foundation for building the general solution $x_{h}=c_{1}X_{1}+c_{2}X_{2}$ used in the [[inhomogeneous_second_order|inhomogeneous solution method]].
- Linear independence of $X_{1}$ and $X_{2}$ is verified by the [[wronskian|Wronskian]]: if $W\neq 0$, the two solutions span the full solution space.

## Links

- [[homogeneous_second_order|Homogeneous Second-Order ODE]]
- [[wronskian|Wronskian]]
- [[inhomogeneous_second_order|Inhomogeneous Second-Order ODE]]
