---
layer: 01_foundations
type: concept
status: seed
tags: [algorithm]
created: 2026-03-01
---

# Newton's Method

## Definition

**Newton's method** is an iterative root-finding algorithm for solving $f(x)=0$. It requires an analytical derivative $f'(x)$ and is among the fastest methods when applicable.

## Intuition

At each step, replace the curve $f$ with its tangent line at the current guess and use where that line crosses zero as the next guess. Because the tangent is a good local approximation, each iterate is typically much closer to the root than the previous one — convergence is quadratic near the root.

## Formal Description

At each iterate $x_{n}$, approximate $f$ by its tangent line:
$$
y - f(x_{n}) = f'(x_{n})(x - x_{n}).
$$
Setting $y=0$ gives the next iterate:
$$
x_{n+1} = x_{n} - \frac{f(x_{n})}{f'(x_{n})}.
$$
An initial guess $x_{0}$ close to the root $r$ is required.

**Example — estimating $\sqrt{2}$:** Solve $f(x)=x^{2}-2=0$ with $f'(x)=2x$:
$$
x_{n+1} = x_{n} - \frac{x_{n}^{2}-2}{2x_{n}} = \frac{x_{n}^{2}+2}{2x_{n}}.
$$
Starting from $x_{0}=1$, this converges rapidly to $\sqrt{2}\approx 1.41421$.

## Applications

Used in optimisation (solving $\nabla f = 0$), in numerical solvers for ODEs and PDEs, and in computational geometry. Variants (quasi-Newton methods) approximate $f'$ when it is expensive to compute exactly.

## Trade-offs

Convergence is not guaranteed globally: a poor initial guess can cause divergence, cycling, or convergence to the wrong root. The method requires $f'(x_{n})\neq 0$ at each step and knowledge of $f'$ analytically or numerically. Near a root with multiplicity $>1$, convergence degrades from quadratic to linear.

## Links

- [[limits_and_continuity|Limits and Continuity]] — convergence relies on continuity of $f$ and $f'$
