---
layer: 01_foundations
type: concept
status: seed
tags: [algorithm]
created: 2026-03-01
---

# Trapezoidal Rule

## Definition

The **trapezoidal rule** is a numerical integration method that approximates $\int_{a}^{b}f(x)\,dx$ by summing the areas of trapezoids formed between consecutive function evaluations.

## Intuition

Rather than approximating the function by a flat horizontal line (rectangle rule), the trapezoidal rule connects neighbouring sample points with straight-line segments. Each trapezoid captures the slope of the function locally, giving a better approximation with the same number of evaluations.

## Formal Description

**Single interval $[0,h]$:** Approximate $f$ by the line through $(0,f(0))$ and $(h,f(h))$:
$$
\int_{0}^{h}f(x)\,dx \approx \frac{h}{2}\bigl(f(0)+f(h)\bigr).
$$

**Composite rule on $[a,b]$:** Divide into $n$ subintervals of width $h=\frac{b-a}{n}$, with nodes $f_{k}=f(a+kh)$:
$$
\int_{a}^{b}f(x)\,dx \approx \frac{h}{2}\bigl(f_{0}+2f_{1}+\cdots+2f_{n-1}+f_{n}\bigr).
$$
All interior evaluations are multiplied by 2; the result is scaled by $\frac{h}{2}$.

## Applications

Used whenever the antiderivative of $f$ is unknown or impractical to compute analytically — e.g. integrating tabulated experimental data or numerically solving ODEs (trapezoidal method for stiff problems). It is the simplest baseline for adaptive quadrature routines.

## Trade-offs

The global error is $O(h^{2})$: halving the step size quarters the error. Simpson's rule achieves $O(h^{4})$ at the same cost by fitting parabolas instead of lines, making it preferable for smooth functions. The trapezoidal rule is exact for linear functions and well-suited to periodic functions integrated over a full period (spectral convergence in that case).

## Links

- [[limits_and_continuity|Limits and Continuity]] — convergence of the approximation as $h\to 0$
