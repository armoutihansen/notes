---
layer: 01_foundations
type: concept
status: seed
tags: [theory]
created: 2026-03-01
---

# Homogeneous Second-Order ODE

## Definition

The **homogeneous second-order ODE with constant coefficients**:
$$
a\ddot{x}+b\dot{x}+cx=0,
$$
where $a,b,c$ are constants.

## Intuition

The equation models a damped harmonic oscillator: $a$ is inertia, $b$ is damping, and $c$ is restoring force. Trying a solution of the form $e^{rt}$ is natural because exponentials are eigenfunctions of the derivative operator. The discriminant $b^2-4ac$ then determines whether the system oscillates, decays monotonically, or sits at the boundary.

## Formal Description

The exponential ansatz $x=e^{rt}$ converts the ODE into an algebraic **characteristic equation**:
$$
ar^{2}+br+c=0,
$$
with roots
$$
r_{\pm}=\frac{-b\pm \sqrt{ b^{2}-4ac }}{2a}.
$$

The general solution is a linear superposition of two linearly independent solutions (verified by the [[wronskian|Wronskian]]); the two free constants are determined from initial values on $x$ and $\dot{x}$.

### Key Results

Three cases arise depending on the discriminant $b^{2}-4ac$:

| Discriminant | Root type | General solution |
|---|---|---|
| $b^{2}-4ac>0$ | Distinct real $r_{1},r_{2}$ | $x(t)=c_{1}e^{r_{1}t}+c_{2}e^{r_{2}t}$ |
| $b^{2}-4ac<0$ | Complex conjugates $\lambda\pm i\mu$ | $x(t)=e^{\lambda t}(A\cos\mu t+B\sin\mu t)$ |
| $b^{2}-4ac=0$ | Repeated root $r$ | $x(t)=(c_{1}+c_{2}t)e^{rt}$ |

See [[characteristic_roots|Characteristic Roots]] for derivations of all three cases.

## Applications

- Mechanical spring-mass-damper systems.
- Electrical RLC circuits ($L\ddot{q}+R\dot{q}+q/C=0$).
- Foundation for the [[inhomogeneous_second_order|inhomogeneous]] solution and [[linear_ode_systems|linear ODE systems]].

## Trade-offs

- Restricted to constant coefficients; variable-coefficient second-order ODEs generally require [[series_solution|series solutions]] or numerical methods.
- The exponential ansatz always works for constant-coefficient homogeneous linear ODEs; no guessing about the solution form is needed.
- Linear independence of the two solutions must be confirmed via the [[wronskian|Wronskian]] before forming the general solution.

## Links

- [[characteristic_roots|Characteristic Roots]]
- [[superposition|Principle of Superposition]]
- [[wronskian|Wronskian]]
- [[inhomogeneous_second_order|Inhomogeneous Second-Order ODE]]
