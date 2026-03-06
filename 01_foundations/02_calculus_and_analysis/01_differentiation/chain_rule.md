---
layer: 01_foundations
type: concept
status: seed
tags: [theory]
created: 2026-03-01
---

# Chain Rule

## Definition

The chain rule gives the derivative of a composite function $y = f(g(x))$.

## Intuition

"Derivative of the outside times derivative of the inside." If $g$ stretches or compresses its input by a factor $g'(x)$, and $f$ further scales by $f'(g(x))$, the net rate of change is the product of these two local scaling factors.

## Formal Description

If $y = y(x(t))$, the derivative of $y$ with respect to $t$ is:
$$
\frac{dy}{dt} = \frac{dy}{dx}\,\frac{dx}{dt},
$$
treating the Leibniz derivatives as ratios of differentials.

In terms of function composition:
$$
\bigl[f(g(x))\bigr]' = f'(g(x))\,g'(x).
$$

$$
\frac{d}{dx}f(g(x)) = f'(g(x))\cdot g'(x).
$$

## Applications

- Differentiating composite functions such as $e^{x^2}$, $\sin(3x)$, $\ln(\cos x)$.
- Underlies implicit differentiation and the general power rule for real exponents.
- Extends to the multivariable chain rule for partial derivatives.

## Trade-offs

The rule requires $g$ to be differentiable at $x$ and $f$ to be differentiable at $g(x)$. In the multivariable setting the scalar product $g'(x)$ becomes a Jacobian matrix product.

## Links

- [[differentiation_rules|Differentiation Rules]]
- [[partial_derivatives|Partial Derivatives]]
- [[common_derivatives|Common Derivatives]]
