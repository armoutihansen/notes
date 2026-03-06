---
layer: 01_foundations
type: concept
status: seed
tags: [theory]
created: 2026-03-01
---

# Partial Derivatives

## Definition

For a function $f(x, y)$ of several variables, the partial derivative with respect to one variable is computed by differentiating with respect to that variable while holding all others fixed.

## Intuition

A partial derivative measures the slope of the function's graph along a slice parallel to one coordinate axis — it is the ordinary derivative in one direction, treating the other variables as constants.

## Formal Description

$$
\frac{\partial f}{\partial x} = \lim_{h \to 0} \frac{f(x+h, y) - f(x, y)}{h}, \qquad \frac{\partial f}{\partial y} = \lim_{h \to 0} \frac{f(x, y+h) - f(x, y)}{h}.
$$

**Example:** For $f(x, y) = 2x^{3}y^{2} + y^{3}$:
$$
\frac{\partial f}{\partial x} = 6x^{2}y^{2}, \qquad \frac{\partial f}{\partial y} = 4x^{3}y + 3y^{2}.
$$

Second partial derivatives:
$$
\frac{\partial^{2}f}{\partial x^{2}} = 12xy^{2}, \qquad \frac{\partial^{2}f}{\partial y^{2}} = 4x^{3} + 6y.
$$

**Symmetry of mixed partials** (Clairaut's theorem, when partials are continuous):
$$
\frac{\partial^{2}f}{\partial x\,\partial y} = \frac{\partial^{2}f}{\partial y\,\partial x}.
$$

**Total differential:**
$$
df = \frac{\partial f}{\partial x}\,dx + \frac{\partial f}{\partial y}\,dy.
$$

**Multivariable chain rule** — if $f = f(x(t), y(t))$:
$$
\frac{df}{dt} = \frac{\partial f}{\partial x}\frac{dx}{dt} + \frac{\partial f}{\partial y}\frac{dy}{dt}.
$$

If $f = f(x(r,\theta), y(r,\theta))$:
$$
\frac{\partial f}{\partial r} = \frac{\partial f}{\partial x}\frac{\partial x}{\partial r} + \frac{\partial f}{\partial y}\frac{\partial y}{\partial r}, \qquad \frac{\partial f}{\partial \theta} = \frac{\partial f}{\partial x}\frac{\partial x}{\partial \theta} + \frac{\partial f}{\partial y}\frac{\partial y}{\partial \theta}.
$$

## Applications

- Gradient vector $\nabla f = \left(\frac{\partial f}{\partial x}, \frac{\partial f}{\partial y}\right)$ used in optimisation and gradient descent.
- Hessian matrix of second partials used in second-order optimisation methods.
- PDEs (heat equation, wave equation, etc.) are expressed in terms of partial derivatives.

## Trade-offs

- Existence of all partial derivatives at a point does not imply differentiability or even continuity there.
- Clairaut's theorem (symmetry of mixed partials) requires continuity of the second partials; pathological counterexamples exist without this condition.

## Links

- [[chain_rule|Chain Rule]]
- [[definition_of_derivative|Definition of the Derivative]]
