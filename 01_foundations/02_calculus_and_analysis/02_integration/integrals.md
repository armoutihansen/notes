---
layer: 01_foundations
type: concept
status: seed
tags: [theory]
created: 2026-03-01
---

# Integrals

## Definition

The **indefinite integral** of $f(x)$ is defined as
$$
\int f(x)\,dx = F(x) + C, \quad \text{when } F'(x) = f(x),
$$
where $C$ is an arbitrary constant.

The **definite integral** of $f$ over $[a,b]$ is
$$
\int_{a}^{b}f(x)\,dx = F(b)-F(a),
$$
where $F$ is any antiderivative of $f$, i.e.\ $F'(x)=f(x)$. The difference $F(b)-F(a)$ is also written $\bigl[F(x)\bigr]_{a}^{b}$.

## Intuition

The indefinite integral reverses differentiation: it finds a function whose derivative is the given integrand. The constant $C$ reflects the fact that infinitely many antiderivatives exist (shifted vertically), so integration without limits yields a family of functions.

The definite integral measures the signed area between the graph of $f$ and the $x$-axis over $[a,b]$. Riemann sums make this geometric picture precise: partition the interval into thin vertical strips, approximate each strip's area by a rectangle, and take the limit as the strips become infinitely thin.

## Formal Description

### Indefinite Integrals

**Linearity** follows directly from the corresponding differentiation rules:
$$
\int a f(x)\,dx = a\int f(x)\,dx,
$$

$$
\int \bigl[f(x)+g(x)\bigr]\,dx = \int f(x)\,dx + \int g(x)\,dx.
$$
Combined:
$$
\int \bigl[a_1 f_1(x)+\cdots+a_n f_n(x)\bigr]\,dx = a_1\int f_1(x)\,dx+\cdots+a_n\int f_n(x)\,dx.
$$

**Standard antiderivatives:**

| Integrand | Integral |
|-----------|----------|
| $x^a$ ($a\neq -1$) | $\dfrac{x^{a+1}}{a+1}+C$ |
| $x^{-1}$ | $\ln\lvert x\rvert+C$ |
| $e^{ax}$ ($a\neq 0$) | $\dfrac{1}{a}e^{ax}+C$ |
| $a^x$ ($a>0,\,a\neq 1$) | $\dfrac{1}{\ln a}a^x+C$ |

The $x^{-1}$ case uses $\ln(-x)$ for $x<0$ (chain rule gives derivative $x^{-1}$), hence $\ln|x|$ covers both signs.

Differentiating confirms: $\dfrac{d}{dx}\int f(x)\,dx = f(x)$.

### Definite Integrals

**Properties.** For $f$ continuous on an interval containing $a$, $b$, $c$:
$$
\begin{align}
\int_{a}^{b}f(x)\,dx &= -\int_{b}^{a}f(x)\,dx, \\
\int_{a}^{a}f(x)\,dx &= 0, \\
\int_{a}^{b}\alpha f(x)\,dx &= \alpha\int_{a}^{b}f(x)\,dx, \\
\int_{a}^{b}f(x)\,dx &= \int_{a}^{c}f(x)\,dx+\int_{c}^{b}f(x)\,dx.
\end{align}
$$

**Riemann integral.** For a bounded function on $[a,b]$, partition $[a,b]$ into $n$ subintervals with widths $\Delta x_i = x_{i+1}-x_i$ and choose $\xi_i\in[x_i,x_{i+1}]$. If the Riemann sum
$$
\sum_{i=0}^{n-1}f(\xi_i)\,\Delta x_i
$$
converges as $n\to\infty$ and $\max\Delta x_i\to 0$, then $f$ is *Riemann integrable* and
$$
\int_{a}^{b}f(x)\,dx = \lim\sum_{i=0}^{n-1}f(\xi_i)\,\Delta x_i.
$$
Every continuous function is Riemann integrable.

**Differentiation with respect to the limits.** If $F'(x)=f(x)$:
$$
\frac{d}{dt}\int_{a}^{t}f(x)\,dx = f(t), \qquad
\frac{d}{dt}\int_{t}^{b}f(x)\,dx = -f(t).
$$
More generally, for differentiable $a(t)$, $b(t)$ and continuous $f$:
$$
\frac{d}{dt}\int_{a(t)}^{b(t)}f(x)\,dx = f(b(t))\,b'(t) - f(a(t))\,a'(t).
$$

## Applications

- Computing areas, volumes, arc lengths, and surface areas in geometry.
- Finding displacement from velocity (and velocity from acceleration) in physics.
- Evaluating probability densities and cumulative distribution functions.
- Solving differential equations via direct integration.

## Trade-offs

- Antiderivatives always exist for continuous functions but need not be expressible in closed form using elementary functions (e.g.\ $\int e^{x^2}\,dx$, $\int e^{-x^2}\,dx$, $\int \frac{e^x}{x}\,dx$).
- The Riemann integral requires boundedness; unbounded functions or infinite intervals require improper integrals with separate convergence analysis.
- The dummy variable $x$ in $\int_a^b f(x)\,dx$ carries no meaning outside the integral; confusing it with a free variable is a common error.
- Splitting limits at a point $c$ outside $[a,b]$ is valid but requires $f$ to be integrable on the larger interval.

## Links

- [[fundamental_theorem|Fundamental Theorem of Calculus]]
- [[integration_by_substitution|Integration by Substitution]]
- [[integration_by_parts|Integration by Parts]]
