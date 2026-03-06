---
layer: 01_foundations
type: concept
status: seed
tags: [algorithm]
created: 2026-03-01
---

# Characteristic Roots

## Definition

The **characteristic equation** of $a\ddot{x}+b\dot{x}+cx=0$ is the quadratic
$$
ar^{2}+br+c=0,\quad r_{\pm}=\frac{-b\pm\sqrt{b^{2}-4ac}}{2a}.
$$
The nature of the roots determines the form of the general solution.

## Intuition

The three cases correspond directly to physical behaviour: two distinct real roots give pure exponential growth/decay (overdamped); complex conjugate roots give oscillations wrapped in an exponential envelope (underdamped); a repeated root is the critical transition between the two (critically damped). Euler's formula $e^{i\mu t}=\cos\mu t+i\sin\mu t$ is the bridge from complex roots to real oscillatory solutions.

## Formal Description

### Case 1 — Distinct Real Roots ($b^{2}-4ac>0$)

Two distinct real roots $r_{1}\neq r_{2}$ give two independent exponential solutions. The general solution is
$$
x(t)=c_{1}e^{r_{1}t}+c_{2}e^{r_{2}t}.
$$

### Case 2 — Complex Conjugate Roots ($b^{2}-4ac<0$)

Write the roots as $r=\lambda+i\mu$ and $\bar{r}=\lambda-i\mu$. Using Euler's formula $e^{i\mu t}=\cos\mu t+i\sin\mu t$, two real independent solutions are
$$
x_{1}(t)=e^{\lambda t}\cos\mu t,\quad x_{2}(t)=e^{\lambda t}\sin\mu t.
$$
The general solution is
$$
x(t)=e^{\lambda t}\left(A\cos\mu t+B\sin\mu t\right).
$$
The real part of the roots appears in the exponential envelope; the imaginary part appears in the oscillatory terms.

### Case 3 — Repeated Root ($b^{2}-4ac=0$)

The ansatz yields only one independent solution $e^{rt}$. Taking the limit $\mu\to 0$ of the complex-conjugate solution (using $\lim_{\mu\to 0}\mu^{-1}\sin\mu t=t$) reveals a second independent solution $te^{rt}$. The general solution is
$$
x(t)=\left(c_{1}+c_{2}t\right)e^{rt}.
$$
The repeated root $r$ satisfies $r=-b/(2a)$.

### Key Results

- All three cases follow from the single exponential ansatz $x=e^{rt}$.
- The [[superposition|principle of superposition]] justifies forming general solutions as linear combinations.
- Independence of the two solutions is confirmed by the [[wronskian|Wronskian]] being nonzero.

## Applications

- Directly gives the transient response of any linear constant-coefficient system.
- The eigenvalue analogue underpins [[linear_ode_systems|linear ODE systems]].
- Stability analysis: the system is stable if and only if all roots have strictly negative real part.

## Trade-offs

- Only applies to constant-coefficient equations; variable-coefficient ODEs require other approaches.
- For complex roots, Euler's formula is required to recover real-valued solutions from complex exponentials.
- The repeated-root case requires a separate argument (limit or reduction of order) to find the second independent solution $te^{rt}$.

## Links

- [[homogeneous_second_order|Homogeneous Second-Order ODE]]
- [[superposition|Principle of Superposition]]
- [[wronskian|Wronskian]]
- [[linear_ode_systems|Linear ODE Systems]] (analogous eigenvalue cases)
