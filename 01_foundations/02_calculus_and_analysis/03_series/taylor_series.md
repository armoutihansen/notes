---
layer: 01_foundations
type: concept
status: seed
tags: [theory, algorithm]
created: 2026-03-01
---

# Taylor Series

## Definition

A **Taylor series** is a [[power_series|power series]] representation of a function $f(x)$ about a point $x = a$, constructed so that all derivatives of the series match those of $f$ at $a$.

## Intuition

A Taylor series approximates a smooth function by matching its value and all its derivatives at a single point. The more terms included, the better the approximation over a wider region. The linear term recovers the familiar tangent-line approximation.

## Formal Description

Writing $f(x) = \sum_{n=0}^\infty c_n (x-a)^n$ and differentiating repeatedly at $x = a$ gives $c_n = f^{(n)}(a)/n!$, so

$$
f(x) = \sum_{n=0}^{\infty} \frac{f^{(n)}(a)}{n!}(x-a)^n = f(a) + f'(a)(x-a) + \frac{f''(a)}{2!}(x-a)^2 + \frac{f'''(a)}{3!}(x-a)^3 + \dots
$$

The special case $a = 0$ is called a **Maclaurin series**:
$$
f(x) = f(0) + f'(0)\,x + \frac{f''(0)}{2!}x^2 + \frac{f'''(0)}{3!}x^3 + \dots
$$

**Small-increment form.** For small $\epsilon$:
$$
f(x + \epsilon) = f(x) + f'(x)\,\epsilon + \frac{f''(x)}{2!}\epsilon^2 + \dots
$$

### Key Results — Common Taylor Series

All series below are centered at $0$.

| Function | Taylor series | Radius of convergence |
|---|---|---|
| $e^x$ | $\displaystyle\sum_{n=0}^{\infty}\frac{x^n}{n!} = 1 + x + \frac{x^2}{2!} + \frac{x^3}{3!} + \cdots$ | $\infty$ |
| $\sin x$ | $\displaystyle\sum_{n=0}^{\infty}\frac{(-1)^n x^{2n+1}}{(2n+1)!} = x - \frac{x^3}{3!} + \frac{x^5}{5!} - \cdots$ | $\infty$ |
| $\cos x$ | $\displaystyle\sum_{n=0}^{\infty}\frac{(-1)^n x^{2n}}{(2n)!} = 1 - \frac{x^2}{2!} + \frac{x^4}{4!} - \cdots$ | $\infty$ |
| $\dfrac{1}{1-x}$ | $\displaystyle\sum_{n=0}^{\infty} x^n = 1 + x + x^2 + x^3 + \cdots$ | $1$ |
| $\dfrac{1}{1+x}$ | $\displaystyle\sum_{n=0}^{\infty}(-1)^n x^n = 1 - x + x^2 - x^3 + \cdots$ | $1$ |
| $\ln(1+x)$ | $\displaystyle\sum_{n=1}^{\infty}\frac{(-1)^{n+1} x^n}{n} = x - \frac{x^2}{2} + \frac{x^3}{3} - \cdots$ | $1$ |
| $\arctan x$ | $\displaystyle\sum_{n=0}^{\infty}\frac{(-1)^n x^{2n+1}}{2n+1} = x - \frac{x^3}{3} + \frac{x^5}{5} - \cdots$ | $1$ |
| $(1+x)^p$ | $\displaystyle 1 + px + \frac{p(p-1)}{2!}x^2 + \frac{p(p-1)(p-2)}{3!}x^3 + \cdots$ | $1$ |
| $\sqrt{1+x}$ | $1 + \tfrac{1}{2}x - \tfrac{1}{8}x^2 + \cdots$ | $1$ |

**Derivation notes:**
- $\cos x$ is obtained by differentiating the $\sin x$ series term-by-term.
- $\ln(1+x)$ is obtained by integrating the series for $\tfrac{1}{1+x}$, using $\ln 1 = 0$.
- $\arctan x$ is obtained by integrating the series for $\tfrac{1}{1+x^2}$ (substitute $x \mapsto x^2$ in $\tfrac{1}{1+x}$), using $\arctan 0 = 0$.

**Remarkable special values:**
$$
\ln 2 = 1 - \tfrac{1}{2} + \tfrac{1}{3} - \tfrac{1}{4} + \dots, \qquad \frac{\pi}{4} = 1 - \tfrac{1}{3} + \tfrac{1}{5} - \tfrac{1}{7} + \dots
$$

**Euler's formula** follows from substituting $x = i\theta$ into the exponential series and separating real and imaginary parts using the $\sin$ and $\cos$ series:
$$
e^{i\theta} = \cos\theta + i\sin\theta.
$$

## Applications

Taylor series provide polynomial approximations to smooth functions. The linear approximation $f(x+\epsilon)\approx f(x)+f'(x)\epsilon$ underlies [[lhospitals_rule|L'Hôpital's Rule]].

**Example.** $\dfrac{1}{1+\epsilon} \approx 1 - \epsilon$ for small $\epsilon$ (linear Taylor approximation with $f(x)=1/x$, $x=1$).

## Trade-offs

A Taylor series converges to $f(x)$ only within its radius of convergence and only for analytic functions. Functions like $e^{-1/x^2}$ are smooth but not analytic at $0$ — all Taylor coefficients vanish yet the function is nonzero, so the series fails to recover $f$.

## Links

- [[power_series|Power Series]]
- [[geometric_series|Geometric Series]]
- [[lhospitals_rule|L'Hôpital's Rule]]
- [[harmonic_and_p_series|Harmonic and p-Series]] — alternating harmonic series sums to $\ln 2$
- [[complex_numbers|Complex Numbers]] — Euler's formula $e^{i\theta}=\cos\theta+i\sin\theta$ follows from the exponential Taylor series
