---
layer: 01_foundations
type: concept
status: seed
tags: [theory]
created: 2026-03-01
---

# Definition of the Derivative

## Definition

The derivative of $f(x)$ is the slope of the tangent line to the curve $y = f(x)$ at a point, defined as the limit of the slope of the secant line through $(x, f(x))$ and $(x+h, f(x+h))$ as $h \to 0$.

## Intuition

As the second point $(x+h, f(x+h))$ slides toward $(x, f(x))$, the secant line rotates into the tangent line. The derivative captures the instantaneous rate of change — how steeply the function is rising or falling at exactly that point.

## Formal Description

**Limit definition:**
$$
f'(x) = \lim_{h \to 0} \frac{f(x+h) - f(x)}{h}.
$$

**Leibniz notation:** Writing $h = \Delta x$ and $f(x+\Delta x) - f(x) = \Delta y$,
$$
\frac{dy}{dx} = \lim_{\Delta x \to 0} \frac{\Delta y}{\Delta x}.
$$
The differential of $y = f(x)$ is $dy = f'(x)\,dx$, treating $dx$ as an infinitesimal increment.

Common notation for first and second derivatives:
$$
f'(x),\quad \frac{df}{dx},\quad y',\quad \frac{dy}{dx},\qquad f''(x),\quad \frac{d^{2}f}{dx^{2}},\quad y'',\quad \frac{d^{2}y}{dx^{2}}.
$$

The second derivative in Leibniz notation:
$$
\frac{d^{2}y}{dx^{2}} = \frac{d}{dx}\!\left(\frac{dy}{dx}\right).
$$

For functions of time, dot notation is used: $\dot{x} = \frac{dx}{dt}$, $\ddot{x} = \frac{d^{2}x}{dt^{2}}$.

## Applications

- Foundation for all differentiation rules (power, product, chain, etc.).
- Tangent-line approximations (linearisation) of smooth functions.
- Velocity and acceleration as first and second derivatives of position.

## Trade-offs

A function is **differentiable** at $x$ if it is continuous and has no sharp corners there (i.e., the limit is the same from both sides). Continuity is necessary but not sufficient for differentiability.

## Links

- [[centered_differences|Centered Differences]]
- [[differentiation_rules|Differentiation Rules]]
