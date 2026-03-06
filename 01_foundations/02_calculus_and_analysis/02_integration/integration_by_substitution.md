---
layer: 01_foundations
type: concept
status: seed
tags: [algorithm]
created: 2026-03-01
---

# Integration by Substitution

## Definition

**Integration by substitution** is a technique based on the [[chain_rule|chain rule]]:
$$
\frac{d}{dx}f(g(x)) = f'(g(x))\,g'(x).
$$
It transforms an integral of a composite function into a simpler integral by introducing a substitution $u = g(x)$.

## Intuition

Just as the chain rule tells us how to differentiate a composition, substitution is the chain rule run backwards. By renaming $g(x)$ as $u$, a complicated integrand is rewritten in a new variable where a standard form is visible. For definite integrals, the limits travel with the substitution — they convert to values of $u$, so we never need to undo the substitution at the end.

## Formal Description

Given an integral of the form $\int f'(g(x))\,g'(x)\,dx$, let $u = g(x)$ with differential $du = g'(x)\,dx$. Then
$$
\int f'(g(x))\,g'(x)\,dx = \int f'(u)\,du = f(u)+C = f(g(x))+C.
$$
For **definite integrals**, the limits of integration must be transformed accordingly:
$$
\int_{a}^{b}f'(g(x))\,g'(x)\,dx = \int_{g(a)}^{g(b)}f'(u)\,du = f(g(b))-f(g(a)).
$$

**Example 1.** $\displaystyle\int xe^{-x^2}\,dx$. Let $u=x^2$, $du=2x\,dx$:
$$
\int xe^{-x^2}\,dx = \frac{1}{2}\int e^{-u}\,du = -\frac{1}{2}e^{-u}+C = -\frac{1}{2}e^{-x^2}+C.
$$

**Example 2.** $\displaystyle\int_{0}^{\pi/2}\sin^3\theta\cos\theta\,d\theta$. Let $u=\sin\theta$, $du=\cos\theta\,d\theta$; limits change to $u=0$ and $u=1$:
$$
\int_{0}^{\pi/2}\sin^3\theta\cos\theta\,d\theta = \int_{0}^{1}u^3\,du = \frac{u^4}{4}\Big|_{0}^{1} = \frac{1}{4}.
$$

## Applications

- Simplifying integrals of composite functions wherever $g'(x)$ appears as a factor.
- Evaluating trigonometric, exponential, and logarithmic integrals by choosing $u$ to be the inner function.
- Converting definite integrals over one interval to equivalent integrals over a transformed interval.

## Trade-offs

- The factor $g'(x)$ must appear (possibly up to a constant multiple) in the integrand; substitution fails if it is absent.
- Choosing the right $u$ is not algorithmic — experience and pattern recognition are required.
- For definite integrals, forgetting to transform the limits is a common error; alternatively, back-substituting before evaluation avoids this but adds steps.
- Substitution handles one layer of composition at a time; nested compositions may require iterated substitutions.

## Links

- [[chain_rule|Chain Rule]]
- [[integrals|Integrals]]
- [[trigonometric_integrals|Trigonometric Integrals]]
