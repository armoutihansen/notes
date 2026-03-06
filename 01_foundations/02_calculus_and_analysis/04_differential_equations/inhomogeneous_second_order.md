---
layer: 01_foundations
type: concept
status: seed
tags: [theory]
created: 2026-03-01
---

# Inhomogeneous Second-Order ODE

## Definition

The **inhomogeneous linear second-order ODE**:
$$
\ddot{x}+p(t)\dot{x}+q(t)x=g(t),\quad x(t_{0})=x_{0},\quad\dot{x}(t_{0})=u_{0},
$$
where $g(t)\neq 0$ is the inhomogeneous (forcing) term.

## Intuition

Linearity guarantees that the total response equals the free (homogeneous) response plus a forced (particular) response. The homogeneous part encodes the system's natural behaviour; the particular part encodes how the system responds to the specific input $g(t)$. Discontinuous or impulsive forcing terms are most cleanly handled by converting to an algebraic problem via the [[laplace_transform|Laplace transform]].

## Formal Description

### Three-step solution method

1. Solve the associated homogeneous ODE $\ddot{x}+p\dot{x}+qx=0$ for two independent solutions $x_{1}(t)$, $x_{2}(t)$, and form
$$
x_{h}(t)=c_{1}x_{1}(t)+c_{2}x_{2}(t).
$$
2. Find a **particular solution** $x_{p}(t)$ satisfying the full inhomogeneous ODE. When $p,q$ are constants and $g$ is a polynomial, exponential, sine, or cosine, an undetermined-coefficients ansatz works.
3. Write the general solution $x(t)=x_{h}(t)+x_{p}(t)$ and apply the initial conditions to determine $c_{1},c_{2}$.

Linearity guarantees $x_{h}+x_{p}$ solves the ODE:
$$
(\ddot{x}_{h}+p\dot{x}_{h}+qx_{h})+(\ddot{x}_{p}+p\dot{x}_{p}+qx_{p})=0+g=g.
$$

### Discontinuous inhomogeneous term

For an inhomogeneous term involving the [[heaviside_and_dirac|Heaviside step function]], e.g.\ $g(t)=1-u_{1}(t)$, the [[laplace_transform|Laplace transform]] method is most efficient.

**Example.** Solve $\ddot{x}+3\dot{x}+2x=1-u_{1}(t)$, $x(0)=\dot{x}(0)=0$.

Taking the Laplace transform:
$$
(s^{2}+3s+2)X(s)=\frac{1-e^{-s}}{s},\quad X(s)=\frac{1-e^{-s}}{s(s+1)(s+2)}.
$$
Setting $F(s)=\frac{1}{s(s+1)(s+2)}=\frac{1}{2}\cdot\frac{1}{s}-\frac{1}{s+1}+\frac{1}{2}\cdot\frac{1}{s+2}$, the solution is
$$
x(t)=f(t)-u_{1}(t)f(t-1),\quad f(t)=\tfrac{1}{2}-e^{-t}+\tfrac{1}{2}e^{-2t}.
$$

### Impulsive inhomogeneous term

For a Dirac delta forcing $g(t)=\delta(t)$, the Laplace transform gives $\mathcal{L}\{\delta(t)\}=1$.

**Example.** Solve $\ddot{x}+3\dot{x}+2x=\delta(t)$, $x(0)=\dot{x}(0)=0$.

$$
(s^{2}+3s+2)X(s)=1,\quad X(s)=\frac{1}{(s+1)(s+2)}=\frac{1}{s+1}-\frac{1}{s+2}.
$$

$$
x(t)=e^{-t}-e^{-2t}.
$$
Note: $x(t)$ is continuous at $t=0$ but $\dot{x}$ is not — impulsive forcing produces a velocity discontinuity.

### Key Results

- The general solution is always a sum of the homogeneous and a particular solution.
- Laplace transforms are the preferred method for discontinuous or impulsive forcing terms.
- Impulsive forcing $\delta(t)$ causes a jump in the first derivative but not in $x$ itself.

## Applications

- Driven mechanical oscillators (springs with periodic or impulsive forcing).
- Electrical circuits with switched or pulsed voltage sources.
- Any control-theory system with a specified input signal $g(t)$.

## Trade-offs

- The undetermined-coefficients method is only systematic when $g(t)$ is a polynomial, exponential, sine, or cosine (or products thereof); variation of parameters is needed otherwise.
- For piecewise or impulsive $g(t)$, the Laplace method is far more efficient but requires facility with partial fractions and the transform table.
- Initial conditions are applied to the full solution $x_h + x_p$, not to $x_h$ alone — a common source of error.

## Links

- [[homogeneous_second_order|Homogeneous Second-Order ODE]]
- [[superposition|Principle of Superposition]]
- [[laplace_transform|Laplace Transform]]
- [[heaviside_and_dirac|Heaviside and Dirac Functions]]
