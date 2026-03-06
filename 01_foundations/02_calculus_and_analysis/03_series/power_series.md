---
layer: 01_foundations
type: concept
status: seed
tags:
  - math/calculus/series
created: 2026-03-01
---

# Power Series

## Definition

A **power series** is an infinite series of the form
$$
\sum_{n=0}^{\infty} c_n x^n = c_0 + c_1 x + c_2 x^2 + c_3 x^3 + \dots
$$

## Intuition

A power series is a polynomial with infinitely many terms. Within its radius of convergence it behaves exactly like a smooth function — it can be differentiated and integrated term-by-term, making it a powerful tool for defining and computing with functions analytically.

## Formal Description

The [[ratio_test|ratio test]] gives convergence when
$$
|x| \lim_{n\to\infty}\left|\frac{c_{n+1}}{c_n}\right| < 1 \iff |x| < R, \quad R = \lim_{n\to\infty}\left|\frac{c_n}{c_{n+1}}\right|.
$$

$R$ is the **radius of convergence**. The series converges for $|x| < R$, diverges for $|x| > R$, and the boundary $|x| = R$ requires separate analysis.

Within $|x| < R$, the power series may be **differentiated** or **integrated term-by-term**:
$$
\frac{d}{dx}\sum_{n=0}^{\infty} c_n x^n = \sum_{n=1}^{\infty} n\, c_n x^{n-1},
$$

$$
\int \left(\sum_{n=0}^{\infty} c_n x^n\right) dx = C + \sum_{n=0}^{\infty} \frac{c_n\, x^{n+1}}{n+1}.
$$

### Key Results

**Example.** The series $\displaystyle\sum_{n=0}^{\infty} \frac{x^n}{n!}$ has infinite radius of convergence ($R = \infty$) and is its own derivative — it equals $e^x$.

## Applications

Power series are the analytic framework underlying [[taylor_series|Taylor Series]]. Differentiating and integrating known power series (e.g. the [[geometric_series|geometric series]]) is a standard technique for obtaining new series.

## Trade-offs

Convergence at the boundary $|x| = R$ must be checked separately for each series; the ratio test is inconclusive there. Power series centered at $a = 0$ may not converge for all $x$ of interest — shifting the center or using a different representation may be needed.

## Links

- [[ratio_test|Ratio Test]]
- [[geometric_series|Geometric Series]]
- [[taylor_series|Taylor Series]]
