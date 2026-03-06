---
layer: 01_foundations
type: concept
status: seed
tags: [algorithm]
created: 2026-03-01
---

# Centered Differences

## Definition

A centered difference approximates the derivative of $f$ at $x$ using a symmetric interval $[x-h, x+h]$, which is more accurate than a one-sided difference.

## Intuition

By using points on both sides of $x$, the odd-order error terms cancel in the Taylor expansion, giving second-order accuracy $O(h^2)$ instead of the $O(h)$ accuracy of a forward or backward difference.

## Formal Description

**First derivative (centered difference):**
$$
f'(x) = \lim_{h \to 0} \frac{f(x+h) - f(x-h)}{2h}.
$$

**Second derivative:** Applying the centered difference formula for $f'$ with step $h/2$ and differencing again:
$$
f''(x) = \lim_{h \to 0} \frac{f'\!\left(x+\tfrac{h}{2}\right) - f'\!\left(x-\tfrac{h}{2}\right)}{h} = \lim_{h \to 0} \frac{f(x+h) - 2f(x) + f(x-h)}{h^{2}}.
$$

Finite-step approximations:
$$
f'(x) \approx \frac{f(x+h) - f(x-h)}{2h}, \qquad f''(x) \approx \frac{f(x+h) - 2f(x) + f(x-h)}{h^{2}}.
$$

The centered difference for $f'$ has $O(h^2)$ error, compared to $O(h)$ for the forward difference $\frac{f(x+h)-f(x)}{h}$.

## Applications

- Numerical differentiation when only discrete function evaluations are available.
- Finite-difference methods for solving ODEs and PDEs on a grid.
- Gradient checking in machine learning to verify analytic gradients.

## Trade-offs

- Requires two function evaluations per point (vs. one for forward difference), which may matter when evaluations are expensive.
- Accuracy degrades for very small $h$ due to floating-point cancellation errors; an optimal $h \approx \varepsilon^{1/3}$ (where $\varepsilon$ is machine epsilon) balances truncation and round-off error.
- Not applicable at boundary points of a domain where one-sided values are unavailable.

## Links

- [[definition_of_derivative|Definition of the Derivative]]
