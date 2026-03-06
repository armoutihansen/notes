---
layer: 01_foundations
type: concept
status: seed
tags: [theory]
created: 2026-03-01
---

# Harmonic and p-Series

## Definition

The **p-series** is:
$$
\sum_{n=1}^{\infty} \frac{1}{n^p} = 1 + \frac{1}{2^p} + \frac{1}{3^p} + \frac{1}{4^p} + \dots
$$

The special case $p = 1$ is the **harmonic series**:
$$
\sum_{n=1}^{\infty} \frac{1}{n} = 1 + \frac{1}{2} + \frac{1}{3} + \frac{1}{4} + \dots
$$

## Intuition

The harmonic series diverges despite its terms going to zero — the terms decrease too slowly. Grouping them into blocks of doubling size shows each block contributes at least $\tfrac{1}{2}$. For $p > 1$ the terms shrink fast enough that the integral comparison yields a finite bound.

## Formal Description

**Convergence via integral comparison.** The integral test gives
$$
\int_{1}^{\infty} \frac{1}{x^p}\,dx < \sum_{n=1}^{\infty} \frac{1}{n^p} < 1 + \int_{1}^{\infty} \frac{1}{x^p}\,dx.
$$

For $p \neq 1$:
$$
\int_{1}^{\infty} x^{-p}\,dx = \frac{1}{1-p}\!\left(\lim_{x\to\infty} x^{1-p} - 1\right),
$$
which converges when $p > 1$ and diverges when $p < 1$.

For $p = 1$: $\int_1^\infty \frac{1}{x}\,dx = \ln x\big|_1^\infty = \infty$ (diverges).

**Grouping proof of harmonic divergence:**
$$
\begin{align}
&1 + \tfrac{1}{2} + \bigl(\tfrac{1}{3}+\tfrac{1}{4}\bigr) + \bigl(\tfrac{1}{5}+\cdots+\tfrac{1}{8}\bigr) + \cdots \\
\geq{}&1 + \tfrac{1}{2} + \tfrac{1}{2} + \tfrac{1}{2} + \cdots \to \infty.
\end{align}
$$

### Key Results

$$
\sum_{n=1}^{\infty} \frac{1}{n^p} \begin{cases} \text{converges} & p > 1, \\ \text{diverges} & p \leq 1. \end{cases}
$$

The **alternating harmonic series** converges (by the alternating series test):
$$
\sum_{n=1}^{\infty} \frac{(-1)^{n+1}}{n} = 1 - \frac{1}{2} + \frac{1}{3} - \frac{1}{4} + \dots = \ln 2.
$$

## Applications

The p-series is a standard benchmark for comparison tests. The harmonic series and its alternating form appear in the [[taylor_series|Taylor Series]] of $\ln(1+x)$ evaluated at $x = 1$.

## Trade-offs

The integral test requires a monotone decreasing positive function; it gives convergence or divergence but not the sum. The boundary case $p = 1$ (harmonic) is the canonical example where term decay is necessary but insufficient for convergence.

## Links

- [[sequences_and_series|Sequences and Series]]
- [[taylor_series|Taylor Series]] — the value $\ln 2$ follows from the Taylor series of $\ln(1+x)$ at $x=1$
