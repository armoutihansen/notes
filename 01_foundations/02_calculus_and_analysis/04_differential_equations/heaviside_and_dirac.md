---
layer: 01_foundations
type: concept
status: seed
tags: [theory]
created: 2026-03-01
---

# Heaviside and Dirac Functions

## Definition

### Heaviside step function

The **Heaviside (unit step) function** $u_{c}(t)$ is zero before $t=c$ and one from $t=c$ onward:
$$
u_{c}(t)=\begin{cases}0 & t<c,\\1 & t\geq c.\end{cases}
$$

### Dirac delta function

The **Dirac delta function** $\delta(t)$ is defined by the property
$$
\int_{-\infty}^{\infty}f(t)\,\delta(t)\,dt=f(0)
$$
for any function $f(t)$. It is technically a distribution, not a function.

## Intuition

The Heaviside function models an instantaneous switch: a system that is "off" before time $c$ and "on" after. The Dirac delta models an idealised instantaneous impulse — infinite magnitude, zero duration, unit area. Physically, $\delta(t)$ is the derivative of $u_0(t)$: a sudden jump in value corresponds to an infinite spike in its rate of change.

## Formal Description

### Heaviside properties

The Laplace transform is
$$
\mathcal{L}\left\{u_{c}(t)\right\}=\frac{e^{-cs}}{s}.
$$
The Heaviside function represents a time translation of $f(t)$ by $c$:
$$
u_{c}(t)f(t-c)=\begin{cases}0 & t<c,\\f(t-c) & t\geq c,\end{cases}\quad\mathcal{L}\left\{u_{c}(t)f(t-c)\right\}=e^{-cs}F(s).
$$
A piecewise function $f(t)=f_{1}(t)$ for $t<c$ and $f_{2}(t)$ for $t\geq c$ can be written as
$$
f(t)=f_{1}(t)+(f_{2}(t)-f_{1}(t))u_{c}(t).
$$

### Dirac delta properties

The shifted delta function can be represented as a limit of step functions:
$$
\delta(t-c)=\lim_{\epsilon\to 0}\frac{1}{2\epsilon}\left(u_{c-\epsilon}(t)-u_{c+\epsilon}(t)\right).
$$
Its Laplace transform (with $c>0$) is
$$
\mathcal{L}\left\{\delta(t-c)\right\}=e^{-cs}.
$$

### Key Results

- The Heaviside function is the primitive (antiderivative) of the Dirac delta: $\frac{d}{dt}u_{c}(t)=\delta(t-c)$.
- Impulsive forcing via $\delta(t)$ produces a jump in the first derivative of the solution but not in the solution itself.
- Both functions appear in the [[laplace_transform|Laplace transform table]] (entries 12–14).

## Applications

- Encoding piecewise-defined forcing terms in ODE problems as a single expression.
- Modelling instantaneous forces (hammer blow, short circuit) in physics and engineering.
- Signal processing: the unit step and impulse are the canonical test inputs for linear time-invariant systems.

## Trade-offs

- The Dirac delta is not a function in the classical sense; rigorous treatment requires the theory of distributions.
- Piecewise expressions using $u_c(t)$ can look compact but become unwieldy with many switch times.
- In numerical computation, approximating $\delta(t)$ requires care to avoid large discretisation errors.

## Links

- [[laplace_transform|Laplace Transform]]
- [[inhomogeneous_second_order|Inhomogeneous Second-Order ODE]]
