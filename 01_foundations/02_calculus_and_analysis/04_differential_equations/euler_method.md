---
layer: 01_foundations
type: concept
status: seed
tags: [algorithm]
created: 2026-03-01
---

# Euler Method

## Definition

**Euler's method** is the simplest numerical scheme for solving an initial-value problem. Given $\frac{dy}{dx}=f(x,y)$ with $y(x_{0})=y_{0}$, a single step of size $\Delta x$ is
$$
y_{n+1}=y_{n}+\Delta x\,f(x_{n},y_{n}).
$$
The tangent-line slope at $(x_{n},y_{n})$ is used to march the solution forward.

## Intuition

Euler's method is simply the repeated application of the first-order Taylor approximation: the solution curve is replaced by its tangent line at each step. The smaller the step $\Delta x$, the less the tangent departs from the true curve before the next correction. The accumulated error grows as $O(\Delta x)$ — one power of $\Delta x$ is "spent" on the approximation at each step, giving a first-order method.

## Formal Description

### First-order ODE

Repeat the step $y_{n+1}=y_{n}+\Delta x\,f(x_{n},y_{n})$ until the desired $x$ is reached. For small enough $\Delta x$ the numerical solution converges to the exact solution.

### Higher-order ODEs

A second-order ODE $\ddot{x}=f(t,x,\dot{x})$ is first reduced to a first-order system by setting $u=\dot{x}$:
$$
\dot{x}=u,\quad\dot{u}=f(t,x,u).
$$
Starting from $(x_{0},u_{0})$ at $t_{0}$, each step advances both variables simultaneously:
$$
x_{n+1}=x_{n}+\Delta t\,u_{n},\quad u_{n+1}=u_{n}+\Delta t\,f(t_{n},x_{n},u_{n}).
$$
The same reduction applies to ODEs of any order: introduce one new variable per derivative to obtain a first-order system.

### Key Results

- Euler's method is a **first-order** method: the global error is $O(\Delta t)$.
- The method is simple but generally less accurate than higher-order methods such as [[runge_kutta|Runge-Kutta]].
- Any $n$th-order ODE can be converted to a system of $n$ first-order ODEs and then integrated numerically.

## Applications

- Quick prototyping and teaching of numerical ODE concepts.
- Baseline reference against which higher-order methods are compared.
- Embedded in more sophisticated adaptive solvers as the predictor step.

## Trade-offs

- First-order accuracy means halving the step size only halves the error; achieving high accuracy requires very small $\Delta t$ and many steps.
- Can be numerically unstable for stiff ODEs unless the step size is very small.
- [[runge_kutta|Runge-Kutta]] methods achieve much higher accuracy with the same number of function evaluations.

## Links

- [[runge_kutta|Runge-Kutta Methods]]
- [[linear_first_order|Linear First-Order ODE]]
