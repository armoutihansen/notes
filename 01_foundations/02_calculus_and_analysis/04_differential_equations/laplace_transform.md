---
layer: 01_foundations
type: concept
status: seed
tags: [algorithm]
created: 2026-03-01
---

# Laplace Transform

## Definition

The **Laplace transform** of $f(t)$ is
$$
F(s)=\mathcal{L}\left\{f(t)\right\}=\int_{0}^{\infty}e^{-st}f(t)\,dt.
$$

## Intuition

The Laplace transform converts differentiation into multiplication by $s$, turning a linear ODE with initial conditions into an algebraic equation in $s$. Solving algebraically and inverting (using partial fractions and the table below) recovers the time-domain solution — including initial conditions — without having to integrate the ODE directly.

## Formal Description

### Linearity

$$
\mathcal{L}\left\{c_{1}f_{1}(t)+c_{2}f_{2}(t)\right\}=c_{1}\mathcal{L}\left\{f_{1}\right\}+c_{2}\mathcal{L}\left\{f_{2}\right\}.
$$

### Transforms of derivatives (via integration by parts)

$$
\mathcal{L}\left\{\dot{x}\right\}=sX(s)-x(0),\quad \mathcal{L}\left\{\ddot{x}\right\}=s^{2}X(s)-sx(0)-\dot{x}(0).
$$

### Solving a constant-coefficient ODE

For $a\ddot{x}+b\dot{x}+cx=g(t)$ with $x(0)=x_{0}$, $\dot{x}(0)=u_{0}$, the Laplace transform yields the algebraic equation
$$
a\!\left(s^{2}X-sx_{0}-u_{0}\right)+b\!\left(sX-x_{0}\right)+cX=G(s).
$$
Solve for $X(s)$, then invert using the table below (with partial fractions as needed) to obtain $x(t)$.

### Table of Laplace Transforms

| # | $f(t)=\mathcal{L}^{-1}\{F(s)\}$ | $F(s)=\mathcal{L}\{f(t)\}$ |
|---|---|---|
| 1 | $e^{at}f(t)$ | $F(s-a)$ |
| 2 | $1$ | $\dfrac{1}{s}$ |
| 3 | $e^{at}$ | $\dfrac{1}{s-a}$ |
| 4 | $t^{n}$ | $\dfrac{n!}{s^{n+1}}$ |
| 5 | $t^{n}e^{at}$ | $\dfrac{n!}{(s-a)^{n+1}}$ |
| 6a | $\sin bt$ | $\dfrac{b}{s^{2}+b^{2}}$ |
| 6b | $\sinh bt$ | $\dfrac{b}{s^{2}-b^{2}}$ |
| 7a | $\cos bt$ | $\dfrac{s}{s^{2}+b^{2}}$ |
| 7b | $\cosh bt$ | $\dfrac{s}{s^{2}-b^{2}}$ |
| 8 | $e^{at}\sin bt$ | $\dfrac{b}{(s-a)^{2}+b^{2}}$ |
| 9 | $e^{at}\cos bt$ | $\dfrac{s-a}{(s-a)^{2}+b^{2}}$ |
| 10 | $t\sin bt$ | $\dfrac{2bs}{(s^{2}+b^{2})^{2}}$ |
| 11 | $t\cos bt$ | $\dfrac{s^{2}-b^{2}}{(s^{2}+b^{2})^{2}}$ |
| 12 | $u_{c}(t)$ | $\dfrac{e^{-cs}}{s}$ |
| 13 | $u_{c}(t)f(t-c)$ | $e^{-cs}F(s)$ |
| 14 | $\delta(t-c)$ | $e^{-cs}$ |
| 15 | $\dot{x}(t)$ | $sX(s)-x(0)$ |
| 16 | $\ddot{x}(t)$ | $s^{2}X(s)-sx(0)-\dot{x}(0)$ |

## Applications

- Solving [[inhomogeneous_second_order|inhomogeneous ODEs]] with discontinuous or impulsive forcing (Heaviside, Dirac delta).
- Analysing stability and frequency response in control systems (transfer functions).
- Computing convolutions via the convolution theorem $\mathcal{L}\{f*g\}=F(s)G(s)$.

## Trade-offs

- Most powerful when $g(t)$ involves Heaviside or Dirac delta terms; for smooth $g(t)$, undetermined coefficients may be simpler.
- Inverting $X(s)$ requires partial fraction decomposition and matching against the table; complex poles can make this tedious.
- The standard table assumes zero-state initial conditions are encoded in the transform; non-zero initial conditions add extra $s$-domain terms that must be tracked carefully.

## Links

- [[inhomogeneous_second_order|Inhomogeneous Second-Order ODE]]
- [[heaviside_and_dirac|Heaviside and Dirac Functions]]
