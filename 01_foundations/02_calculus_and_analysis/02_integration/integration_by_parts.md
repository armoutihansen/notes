---
layer: 01_foundations
type: concept
status: seed
tags: [algorithm]
created: 2026-03-01
---

# Integration by Parts

## Definition

**Integration by parts** is based on the product rule for differentiation. Since $(fg)' = f'g + fg'$, rearranging and integrating gives
$$
\int f'(x)\,g(x)\,dx = f(x)\,g(x) - \int f(x)\,g'(x)\,dx.
$$

## Intuition

Integration by parts is the product rule run backwards: it trades one integral for another by shifting a derivative from one factor to the other. The goal is to choose $u$ and $dv$ so that the new integral $\int v\,du$ is simpler than the original. A useful mnemonic for choosing $u$ is **LIATE** (Logarithm, Inverse trig, Algebraic, Trigonometric, Exponential) — pick $u$ from the category that appears earliest in the list.

## Formal Description

Setting $u = g(x)$, $dv = f'(x)\,dx$, $du = g'(x)\,dx$, $v = f(x)$, the formula becomes
$$
\int u\,dv = uv - \int v\,du.
$$
The key idea is that $\int v\,du$ should be simpler than $\int u\,dv$.

For **definite integrals**:
$$
\int_{a}^{b}u\,dv = uv\Big|_{a}^{b} - \int_{a}^{b}v\,du.
$$

**Example.** $\displaystyle\int xe^x\,dx$. Let $u=x$, $dv=e^x\,dx$, so $du=dx$, $v=e^x$:
$$
\int xe^x\,dx = xe^x - \int e^x\,dx = xe^x - e^x + C = e^x(x-1)+C.
$$

## Applications

- Integrals of the form $\int x^n e^x\,dx$, $\int x^n \sin x\,dx$, $\int x^n \cos x\,dx$ (reduce the power of $x$ by repeated application).
- Integrals involving $\ln x$ or inverse trigonometric functions, where these are chosen as $u$.
- Deriving reduction formulas for $\int \sin^n x\,dx$, $\int \cos^n x\,dx$, and similar families.

## Trade-offs

- The technique requires a good choice of $u$ and $dv$; a poor choice yields a more complicated integral.
- Some integrals require applying the formula twice (or more), and occasionally the original integral reappears on the right — it can then be solved algebraically rather than integrated again.
- Integration by parts does not apply when the integrand cannot be written as a product of two functions in a useful way.
- For definite integrals, both the boundary term $uv\big|_a^b$ and the remaining integral must be evaluated; errors in either part invalidate the result.

## Links

- Product Rule
- [[fundamental_theorem|Fundamental Theorem of Calculus]]
- [[integrals|Integrals]]
