---
layer: 01_foundations
type: concept
status: seed
tags: [theory]
created: 2026-03-01
---

# Sequences and Series

## Definition

A **sequence** is an ordered list of numbers $a_1, a_2, a_3, \dots$ A **series** is the sum of a sequence. The **partial sums** $s_n = \sum_{k=1}^{n} a_k$ define convergence: an **infinite series** $\sum_{k=1}^\infty a_k$ **converges** if $\lim_{n\to\infty} s_n$ exists and is finite; otherwise it **diverges**.

## Intuition

A series converges when the running total "settles down" to a fixed value. Adding infinitely many positive terms can still yield a finite result if the terms shrink fast enough — but shrinking to zero is necessary, not sufficient.

## Formal Description

A sequence may be defined explicitly (e.g. $a_n = 1/n$) or by a **recursion relation**. The Fibonacci sequence is defined by
$$
F_{n+2} = F_{n+1} + F_n, \quad F_1 = F_2 = 1.
$$

The partial sums $s_1, s_2, \dots, s_n$ are defined as
$$
s_1 = a_1, \quad s_2 = a_1 + a_2, \quad \dots, \quad s_n = a_1 + a_2 + \dots + a_n.
$$

The partial sums of the Fibonacci sequence telescope:
$$
\sum_{k=1}^{n} F_k = F_{n+2} - 1.
$$

### Key Results

- A series converges if and only if its sequence of partial sums converges.
- Convergence depends only on the tail of the series (finitely many terms do not affect convergence).

## Applications

Sequences and series appear throughout analysis, probability (expected values), and discrete mathematics. They are the foundation for [[power_series|Power Series]] and [[taylor_series|Taylor Series]].

## Trade-offs

The definition of convergence via partial sums is precise but gives no direct method for finding the sum. Separate convergence tests ([[ratio_test|Ratio Test]], integral test, comparison) are needed in practice.

## Links

- [[geometric_series|Geometric Series]]
- [[harmonic_and_p_series|Harmonic and p-Series]]
- [[ratio_test|Ratio Test]]
- [[power_series|Power Series]]
