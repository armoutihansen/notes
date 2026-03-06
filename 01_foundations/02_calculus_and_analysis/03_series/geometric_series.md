---
layer: 01_foundations
type: concept
status: seed
tags: [theory]
created: 2026-03-01
---

# Geometric Series

## Definition

A **geometric series** is a series where each term is a constant multiple of the previous term:
$$
\sum_{k=0}^{\infty} x^k = 1 + x + x^2 + x^3 + \dots
$$

## Intuition

Multiplying the partial sum by $x$ and subtracting from itself cancels all middle terms, leaving a closed-form expression. For $|x| < 1$ the terms shrink to zero and the sum converges; for $|x| \geq 1$ they do not.

## Formal Description

Multiplying by $x$ and subtracting gives the finite sum formula:
$$
(1 - x)\sum_{k=0}^{n} x^k = 1 - x^{n+1} \implies \sum_{k=0}^{n} x^k = \frac{1 - x^{n+1}}{1 - x}.
$$

Taking $n \to \infty$: when $|x| < 1$, $x^{n+1} \to 0$, yielding the **infinite geometric series**:
$$
\sum_{k=0}^{\infty} x^k = \frac{1}{1-x}, \quad (|x| < 1).
$$

The series diverges for $|x| \geq 1$.

### Key Results

- Converges if and only if $|x| < 1$, with sum $\dfrac{1}{1-x}$.
- Useful identity: $\dfrac{1}{1+x} = \sum_{k=0}^{\infty} (-x)^k = 1 - x + x^2 - x^3 + \dots$, $|x|<1$.

**Example.** $1 + \tfrac{1}{2} + \tfrac{1}{4} + \tfrac{1}{8} + \dots = \dfrac{1}{1 - \frac{1}{2}} = 2$.

## Applications

The geometric series is the foundation for the [[ratio_test|Ratio Test]] and for deriving [[power_series|Power Series]] and [[taylor_series|Taylor Series]] of elementary functions such as $\ln(1+x)$ and $\arctan x$.

## Trade-offs

The formula $\frac{1}{1-x}$ requires $|x| < 1$; at the boundary ($|x| = 1$) the series diverges. For complex $x$, convergence still requires $|x| < 1$.

## Links

- [[sequences_and_series|Sequences and Series]]
- [[ratio_test|Ratio Test]]
- [[power_series|Power Series]]
- [[taylor_series|Taylor Series]]
