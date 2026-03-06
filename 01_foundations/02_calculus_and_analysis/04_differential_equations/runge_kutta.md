---
layer: 01_foundations
type: concept
status: seed
tags: [algorithm]
created: 2026-03-01
---

# Runge-Kutta Methods

## Definition

**Runge-Kutta methods** are a family of numerical integration schemes for $\frac{dy}{dx}=f(x,y)$ that achieve higher accuracy than the Euler method by evaluating $f$ at intermediate points within each step.

## Intuition

Rather than using only the slope at the start of the step (Euler), Runge-Kutta methods take a weighted average of slopes sampled at several interior points. This cancels out higher-order error terms from the Taylor expansion, achieving accuracy $O(\Delta x^p)$ for a $p$th-order method — with no extra cost from reducing the step size. The classical RK4 achieves fourth-order accuracy with just four slope evaluations per step.

## Formal Description

### First-order Runge-Kutta (Euler's method)

$$
k_{1}=\Delta x\,f(x_{n},y_{n}),\quad y_{n+1}=y_{n}+k_{1}.
$$

### Second-order Runge-Kutta

$$
k_{1}=\Delta x\,f(x_{n},y_{n}),\quad k_{2}=\Delta x\,f(x_{n}+\alpha\Delta x,\,y_{n}+\beta k_{1}),\quad y_{n+1}=y_{n}+ak_{1}+bk_{2}.
$$
The parameters $\alpha,\beta,a,b$ must satisfy the consistency conditions
$$
a+b=1,\quad\alpha b=\beta b=\tfrac{1}{2}.
$$
Different choices of these parameters yield distinct second-order methods (e.g.\ the midpoint method with $\alpha=\beta=\frac{1}{2}$, $a=0$, $b=1$; or Heun's method with $\alpha=\beta=1$, $a=b=\frac{1}{2}$).

### Key Results

- The classical fourth-order Runge-Kutta (RK4) method uses four $k$-evaluations per step and achieves $O(\Delta x^{4})$ global error.
- Runge-Kutta methods are self-starting and do not require values at previous steps.
- They are the standard choice for numerically solving ODEs when higher accuracy than Euler's method is needed.

## Applications

- Standard workhorse for non-stiff ODE integration in scientific computing (`scipy.integrate.solve_ivp` default method is RK45).
- Trajectory simulation in physics, orbital mechanics, and robotics.
- Adaptive step-size control by comparing solutions of different orders (embedded Runge-Kutta pairs such as Dormand-Prince RK45).

## Trade-offs

- Each step requires multiple function evaluations of $f$; if $f$ is expensive to compute, methods that reuse previous evaluations (Adams-Bashforth) may be preferable.
- Standard Runge-Kutta methods are explicit and can be unstable for stiff equations; implicit variants (e.g.\ implicit RK) are needed for stiff problems but require solving a nonlinear system per step.
- Higher-order accuracy is only realised when the solution is sufficiently smooth; discontinuities in $f$ can degrade accuracy to first order.

## Links

- [[euler_method|Euler Method]]
