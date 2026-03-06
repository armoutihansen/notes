---
layer: 01_foundations
type: concept
status: seed
tags: [algorithm]
created: 2026-03-01
---

# Ratio Test

## Definition

The **ratio test** determines convergence of an infinite series $\sum_{n=0}^{\infty} a_n$ by examining the limit of successive term ratios:
$$
L = \lim_{n \to \infty} \left|\frac{a_{n+1}}{a_n}\right|.
$$

## Intuition

If the ratio of successive terms approaches a constant $L$, the series eventually behaves like a [[geometric_series|geometric series]] with ratio $L$. Convergence follows when $L < 1$.

## Formal Description

The geometric series $\sum x^n$ converges iff $|x| < 1$, with ratio of successive terms equal to $x$. By comparison:

- $L < 1$: series **converges** (absolutely).
- $L > 1$: series **diverges**.
- $L = 1$: **indeterminate** — the test is inconclusive.

### Key Results

The ratio test is especially effective when $a_n$ involves factorials or exponentials.

**Example.** For $\displaystyle\sum_{n=0}^{\infty} \frac{1}{n!}$:
$$
\lim_{n\to\infty} \frac{\frac{1}{(n+1)!}}{\frac{1}{n!}} = \lim_{n\to\infty} \frac{1}{n+1} = 0 < 1.
$$
The series converges (its sum is $e$).

## Applications

The ratio test is used to find the **radius of convergence** of a [[power_series|power series]]:
$$
\left|x\right| \lim_{n\to\infty}\left|\frac{c_{n+1}}{c_n}\right| < 1 \implies R = \lim_{n\to\infty}\left|\frac{c_n}{c_{n+1}}\right|.
$$

## Trade-offs

The test is inconclusive when $L = 1$: both the convergent p-series $\sum 1/n^2$ and the divergent harmonic series $\sum 1/n$ give $L = 1$. Polynomial or rational $a_n$ always yield $L = 1$; comparison or integral tests are needed instead.

## Links

- [[geometric_series|Geometric Series]]
- [[power_series|Power Series]]
- [[sequences_and_series|Sequences and Series]]
