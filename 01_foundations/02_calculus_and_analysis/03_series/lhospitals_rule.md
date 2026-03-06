---
layer: 01_foundations
type: concept
status: seed
tags: [algorithm]
created: 2026-03-01
---

# L'Hôpital's Rule

## Definition

> **L'Hôpital's Rule.** If $f(a) = g(a) = 0$ and $g'(a) \neq 0$, then
> $$\lim_{x \to a} \frac{f(x)}{g(x)} = \frac{f'(a)}{g'(a)}.$$

## Intuition

Near $x = a$, both $f$ and $g$ are approximated by their linear terms. When both vanish at $a$, the ratio reduces to the ratio of slopes — i.e. the ratio of first derivatives evaluated at $a$.

## Formal Description

L'Hôpital's rule handles limits that produce the indeterminate forms $\tfrac{0}{0}$ or $\tfrac{\infty}{\infty}$ under direct substitution.

**Derivation via Taylor series.** Suppose $f$ and $g$ have [[taylor_series|Taylor series]] around $x = a$:
$$
f(x) = f(a) + (x-a)f'(a) + \frac{(x-a)^2}{2!}f''(a) + \dots
$$

$$
g(x) = g(a) + (x-a)g'(a) + \frac{(x-a)^2}{2!}g''(a) + \dots
$$

When $f(a) = g(a) = 0$, both numerator and denominator contain a factor of $(x-a)$:
$$
\lim_{x \to a} \frac{f(x)}{g(x)} = \lim_{x \to a} \frac{(x-a)\!\left(f'(a) + \frac{x-a}{2!}f''(a) + \dots\right)}{(x-a)\!\left(g'(a) + \frac{x-a}{2!}g''(a) + \dots\right)} = \frac{f'(a)}{g'(a)}.
$$

### Key Results

- **Repeated application:** if $f'(a) = g'(a) = 0$, apply the rule again to $f'/g'$.
- **$\tfrac{\infty}{\infty}$ form:** the rule applies equally.
- **Other indeterminate forms** ($0 \cdot \infty$, $\infty - \infty$, $0^0$, etc.) can often be rewritten algebraically as $\tfrac{0}{0}$ or $\tfrac{\infty}{\infty}$ before applying the rule.

**Example.**
$$
\lim_{x \to 0} \frac{\sin(ax)}{x} \xrightarrow{\;0/0\;} \lim_{x \to 0} \frac{a\cos(ax)}{1} = a.
$$

## Applications

L'Hôpital's rule is used to evaluate limits that arise in computing Taylor series coefficients, verifying series convergence, and simplifying indeterminate expressions in analysis.

## Trade-offs

The rule requires $f$ and $g$ to be differentiable near $a$. Repeated application can cycle or fail to simplify; direct Taylor expansion is often more efficient in such cases. Forms like $\infty^0$ or $1^\infty$ require algebraic rewriting (e.g. via logarithms) before the rule can be applied.

## Links

- [[taylor_series|Taylor Series]]
- [[sequences_and_series|Sequences and Series]]
