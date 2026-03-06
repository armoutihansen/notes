---
layer: 01_foundations
type: concept
status: seed
tags: [algorithm]
created: 2026-03-01
---

# Partial Fractions

## Definition

**Integration by partial fractions** uses partial fraction decomposition to split a rational integrand into simpler fractions that can each be integrated directly.

## Intuition

A rational function $\frac{p(x)}{q(x)}$ is generally hard to integrate as written, but once its denominator is factored, the fraction can be split into a sum of simple pieces — each with a single linear or irreducible quadratic factor in the denominator. These pieces integrate to logarithms or arctangents, both of which are elementary. Partial fractions is essentially the reverse of adding fractions over a common denominator.

## Formal Description

To integrate a rational function $\frac{p(x)}{q(x)}$, decompose it into a sum of partial fractions whose denominators are the factors of $q(x)$. Coefficients are found by the **cover-up method**: to find the coefficient for a linear factor $(x-r)$, multiply both sides by $(x-r)$ and set $x=r$.

**Example.** $\displaystyle\int_{x_0}^{x}\frac{ds}{s(1-s)}$, where $0<x_0,x<1$.

Decompose: $\dfrac{1}{s(1-s)} = \dfrac{A}{s}+\dfrac{B}{1-s}$.

Cover-up gives $A=1$, $B=1$, so $\dfrac{1}{s(1-s)}=\dfrac{1}{s}+\dfrac{1}{1-s}$.

Therefore,
$$
\begin{align}
\int_{x_0}^{x}\frac{ds}{s(1-s)}
&= \int_{x_0}^{x}\frac{ds}{s}+\int_{x_0}^{x}\frac{ds}{1-s} \\
&= \ln s\Big|_{x_0}^{x}-\ln(1-s)\Big|_{x_0}^{x} \\
&= \ln\!\left(\frac{x}{x_0}\right)-\ln\!\left(\frac{1-x}{1-x_0}\right) \\
&= \ln\!\left(\frac{x(1-x_0)}{x_0(1-x)}\right).
\end{align}
$$

## Applications

- Integrating rational functions arising in differential equations (e.g.\ logistic growth).
- Computing Laplace and Z-transform inverses.
- Evaluating integrals that arise in probability and statistics with rational-function densities.

## Trade-offs

- Requires $q(x)$ to be fully factored; finding roots of high-degree polynomials may be difficult or impossible analytically.
- When $\deg p \geq \deg q$, polynomial long division must be performed first before decomposing.
- Repeated factors and irreducible quadratic factors require additional partial fraction forms (e.g.\ $\frac{A}{(x-r)^k}$ for repeated roots, $\frac{Bx+C}{x^2+bx+c}$ for quadratics), increasing complexity.
- The method only applies to rational integrands; non-rational functions require other techniques.

## Links

- [[integrals|Integrals]]
- [[integration_by_substitution|Integration by Substitution]]
