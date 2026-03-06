---
layer: 01_foundations
type: concept
status: seed
tags: [algorithm]
created: 2026-03-01
---

# Series Solution of ODEs

## Definition

The **series solution method** seeks a solution to an ODE as a power series
$$
y(x)=\sum_{n=0}^{\infty}a_{n}x^{n}.
$$

## Intuition

If the ODE has analytic (variable) coefficients, its solutions are themselves analytic near an ordinary point, and can be written as convergent power series. Substituting the series into the ODE and matching powers of $x$ turns the differential equation into a recurrence relation on the coefficients — reducing "solve the ODE" to "find a pattern in the $a_n$". The two free constants $a_0$ and $a_1$ correspond to the two independent solutions guaranteed by linearity.

## Formal Description

Differentiate term-by-term:
$$
y'(x)=\sum_{n=1}^{\infty}na_{n}x^{n-1},\quad y''(x)=\sum_{n=2}^{\infty}n(n-1)a_{n}x^{n-2}.
$$
Substitute into the ODE and shift the summation index (e.g. $n\mapsto n+2$ in the $y''$ sum) to align powers of $x$:
$$
\sum_{n=2}^{\infty}n(n-1)a_{n}x^{n-2}=\sum_{n=0}^{\infty}(n+2)(n+1)a_{n+2}x^{n}.
$$
Combine all sums into a single power series and set each coefficient to zero to obtain a **recurrence relation** for $a_{n}$.

**Example** ($y''+y=0$):
$$
a_{n+2}=-\frac{a_{n}}{(n+2)(n+1)},\quad n=0,1,\dots
$$
Even and odd coefficients decouple, giving two independent sequences starting from $a_{0}$ and $a_{1}$. The resulting series are recognised as
$$
y(x)=a_{0}\cos x+a_{1}\sin x.
$$

### Key Results

- The method applies to ODEs with variable coefficients where closed-form solutions may be unavailable.
- Two independent series solutions arise from the two free constants $a_{0}$ and $a_{1}$, in accordance with the [[superposition|principle of superposition]].
- Convergence of the power series must be verified separately.

## Applications

- Solving ODEs with polynomial coefficients near ordinary points (e.g.\ Legendre, Hermite, Chebyshev equations arising in physics).
- Deriving special functions (Bessel functions via Frobenius method at regular singular points).
- Verifying or discovering closed-form solutions for simple equations (as in the $y''+y=0$ example above).

## Trade-offs

- Only valid near ordinary points; singular points require the Frobenius method (series with a non-integer leading power).
- The radius of convergence is limited to the distance to the nearest singular point of the coefficients.
- Computing many terms of the recurrence is straightforward but can be tedious; closed-form recognition of the resulting series is not always possible.

## Links

- [[superposition|Principle of Superposition]]
- [[homogeneous_second_order|Homogeneous Second-Order ODE]]
