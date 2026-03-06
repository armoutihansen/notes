---
layer: 01_foundations
type: concept
status: seed
tags: [algorithm]
created: 2026-03-01
---

# Separable First-Order Equations

## Definition

A first-order ODE is **separable** if it can be written in the form
$$
g(y) \frac{dy}{dx}=f(x),\quad y(x_{0})=y_{0},
$$
where $g(y)$ is independent of $x$ and $f(x)$ is independent of $y$.

## Intuition

Separability means the $x$-dependence and $y$-dependence can be pulled to opposite sides of the equation. The differential $dy/dx$ is treated as a fraction, and each "side" is integrated independently — turning a differential equation into two ordinary integrals.

## Formal Description

Integration from $x_{0}$ to $x$ and the substitution $u=y(x)$ yield
$$
\int_{y_{0}}^{y}g(u)\,du=\int_{x_{0}}^{x}f(x)\,dx.
$$
This can often be solved analytically for $y=y(x)$.

A practical shorthand: treat $\frac{dy}{dx}$ as a fraction to obtain the separated form
$$
g(y)\,dy=f(x)\,dx,
$$
which is then integrated directly on each side.

### Key Results

- Separability is a sufficient condition for an analytic solution by direct integration.
- The resulting algebraic equation may or may not be solvable explicitly for $y$.

## Applications

- Exponential growth and decay: $\dot{y}=ky$.
- Logistic population growth: $\dot{y}=ky(1-y/N)$.
- Deriving the [[linear_first_order|integrating factor]] $\mu$, which itself satisfies $d\mu/dx = p(x)\mu$.

## Trade-offs

- Only applicable when the equation can be separated; many physically relevant ODEs are not separable.
- Even when separated, the resulting integrals may lack closed forms.
- The implicit relation obtained after integration may be difficult or impossible to solve explicitly for $y(x)$.

## Links

- [[linear_first_order|Linear First-Order ODE]]
