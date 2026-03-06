---
layer: 01_foundations
type: concept
status: seed
tags: [theory]
created: 2026-03-01
---

# Limits and Continuity

## Definition

The **limit** of a function $f(x)$ as $x$ approaches $a$ is the value that $f(x)$ approaches:
$$
\lim_{x\to a}f(x).
$$

A function $f$ is **continuous at $a$** if $f(a)$ is well-defined and finite, and
$$
\lim_{x\to a}f(x)=f(a).
$$
A function is continuous over an interval if its graph forms an unbroken curve — no holes, no jumps, no infinite divergence at any finite point.

## Intuition

A limit captures the behaviour of a function near a point without requiring the function to be defined there. Continuity says the function has no surprises: zooming in on the graph always reveals a connected curve, and evaluating at the point gives the same answer as approaching from either side.

## Formal Description

Direct substitution often yields an indeterminate form $\frac{0}{0}$. Algebraic manipulation can resolve this. For example:
$$
\lim_{h\to 0}\frac{(x+h)^{2}-x^{2}}{h}=\lim_{h\to 0}\frac{h(2x+h)}{h}=2x.
$$

**Intermediate Value Theorem:** If $f$ is continuous on $[a,b]$, then $f$ takes every value between $f(a)$ and $f(b)$ on that interval.

## Applications

Limits underpin the definition of the derivative ($\lim_{h\to 0}$) and the Riemann integral. Continuity is a prerequisite for many theorems in calculus, including the Intermediate Value Theorem and the Extreme Value Theorem.

## Trade-offs

Limits can fail to exist if the left- and right-hand limits differ (jump discontinuity) or if the function oscillates infinitely near the point. Continuity at a point does not imply differentiability there (e.g. $|x|$ at $x=0$).

## Links

- [[newtons_method|Newton's Method]] — uses limits implicitly via derivatives
