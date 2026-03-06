---
layer: 01_foundations
type: proof
status: seed
tags: [theory]
created: 2026-03-01
---

# Fundamental Theorem of Calculus

## Statement

The **Fundamental Theorem of Calculus** establishes that differentiation and integration are inverse operations. It has two parts:

- **First Theorem:** If $F(x)=\int_{a}^{x}f(s)\,ds$, then $F'(x)=f(x)$.
- **Second Theorem:** For any antiderivative $F$ of $f$, $\displaystyle\int_{a}^{b}f(s)\,ds = F(b)-F(a)$.

## Assumptions

- $f$ is continuous on the closed interval $[a,b]$.
- $F$ is any function satisfying $F'(x)=f(x)$ on $[a,b]$ (an antiderivative of $f$).

## Proof Sketch

**First Theorem.** The integral $\int_{a}^{x}f(s)\,ds$ accumulates area as $x$ increases. Incrementing $x$ by a small $h$ adds a thin strip of width $h$ and height $\approx f(x)$; dividing by $h$ and taking the limit recovers $f(x)$.

**Second Theorem.** Because $F(x) = \int_{a}^{x}f(s)\,ds + C$ is the general antiderivative, evaluating at the endpoints and subtracting eliminates $C$, yielding $\int_{a}^{b}f(s)\,ds = F(b)-F(a)$.

## Full Proof

**First Fundamental Theorem.** Define the accumulation function
$$
y(x) = \int_{a}^{x}f(s)\,ds.
$$
By the definition of the derivative,
$$
y'(x) = \lim_{h\to 0}\frac{\int_{a}^{x+h}f(s)\,ds - \int_{a}^{x}f(s)\,ds}{h} = \lim_{h\to 0}\frac{\int_{x}^{x+h}f(s)\,ds}{h}.
$$
Since $f$ is continuous, for small $h$ the integral $\int_{x}^{x+h}f(s)\,ds$ approximates a rectangle of width $h$ and height $f(x)$. Therefore
$$
\frac{d}{dx}\int_{a}^{x}f(s)\,ds = f(x).
$$

**Second Fundamental Theorem.** By the First Theorem, $y(x) = \int_{a}^{x}f(s)\,ds$ is an antiderivative of $f$, so the general antiderivative is
$$
F(x) = \int_{a}^{x}f(s)\,ds + C.
$$
Evaluating at $x=a$ gives $F(a) = 0 + C$, so $C = F(a)$. Evaluating at $x=b$:
$$
F(b) = \int_{a}^{b}f(s)\,ds + F(a),
$$
and therefore
$$
\int_{a}^{b}f(s)\,ds = F(b) - F(a).
$$
Equivalently,
$$
\int_{a}^{b}f'(x)\,dx = f(x)\Big|_{a}^{b} = f(b) - f(a).
$$

## Notes / Intuition

- Differentiation and integration are inverse operations; the theorem is the formal bridge between them.
- Every continuous function has an antiderivative (given by the accumulation function), even when no closed-form expression exists.
- The Second Theorem justifies the standard technique of computing definite integrals by finding an antiderivative and evaluating at the endpoints, rather than computing a limit of Riemann sums directly.
- Geometrically, the First Theorem says the rate of change of accumulated area equals the height of the curve at the moving boundary.

## Links

- [[integrals|Integrals]]
