---
layer: 01_foundations
type: concept
status: seed
tags: [algorithm]
created: 2026-03-01
---

# Linear First-Order ODE

## Definition

A linear first-order equation with initial condition in standard form:
$$
\frac{dy}{dx}+p(x)y=g(x),\quad y(x_{0})=y_{0}.
$$

## Intuition

Multiplying by the integrating factor "balances" the left-hand side so it collapses into a single derivative $\frac{d}{dx}[\mu y]$, after which the equation can be integrated directly. Geometrically, the factor $\mu$ warps the solution space so that the ODE becomes a trivially solvable exact derivative.

## Formal Description

All linear first-order ODEs can be integrated using an **integrating factor** $\mu = \mu(x)$. Multiplying the equation by $\mu(x)$ and requiring
$$
\mu(x)\left[ \frac{dy}{dx}+p(x)y \right] =\frac{d}{dx}\left[ \mu(x)y \right]
$$
yields a directly integrable equation. The integrating factor is
$$
\mu(x)=e^{\int_{x_{0}}^{x}p(x)\,dx},
$$
and the solution is
$$
y(x)=\frac{1}{\mu(x)}\left( y_{0}+\int_{x_{0}}^x \mu(x)g(x)\,dx \right).
$$

### Key Results

- Any linear first-order ODE can be solved by the integrating factor method.
- The integrating factor satisfies $\frac{d\mu}{dx}=p(x)\mu$, which is itself a [[separable_equations|separable equation]].
- The method applies even when the equation is not separable.

## Applications

- Mixing and decay problems (e.g.\ concentration in a tank, radioactive decay with external input).
- RC circuit analysis: $\dot{q}+q/(RC)=V(t)/R$.
- Population models with immigration or harvesting terms.

## Trade-offs

- Requires $p(x)$ and $g(x)$ to be continuous on the interval of interest.
- The integrals $\int p\,dx$ and $\int \mu g\,dx$ may not have closed forms, requiring numerical integration.
- Only applies to linear equations; nonlinear first-order ODEs need separate methods (e.g.\ [[separable_equations|separation of variables]]).

## Links

- [[separable_equations|Separable First-Order Equations]]
- [[inhomogeneous_second_order|Inhomogeneous Second-Order ODE]]
