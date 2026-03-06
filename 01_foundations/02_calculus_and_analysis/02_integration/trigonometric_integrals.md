---
layer: 01_foundations
type: concept
status: seed
tags: [algorithm]
created: 2026-03-01
---

# Trigonometric Integrals

## Definition

**Trigonometric integrals** are integrals involving powers of sine and cosine. **Trigonometric substitution** is a technique for integrals containing square roots of quadratic expressions, using inverse substitutions to convert them into trigonometric integrals.

## Intuition

Powers of sine and cosine interact through the Pythagorean identity $\sin^2\theta+\cos^2\theta=1$ and the half-angle formulas. These identities let us reduce high powers to lower ones or swap between the two functions, creating something amenable to substitution. For square-root integrands, a trigonometric substitution works because a trig identity collapses the square root: $\sqrt{a^2-x^2}$ becomes $a\cos\theta$ when $x=a\sin\theta$, eliminating the radical entirely.

## Formal Description

**Trigonometric integrals** of the form $\int\sin^m\theta\cos^n\theta\,d\theta$:

- If both $m$ and $n$ are **even**, apply the half-angle (reduction) formulas:
$$
\sin^2\theta = \tfrac{1}{2}(1-\cos 2\theta), \qquad \cos^2\theta = \tfrac{1}{2}(1+\cos 2\theta).
$$
- If either $m$ or $n$ is **odd**, apply the Pythagorean identity $\sin^2\theta+\cos^2\theta=1$ to reduce to a single trigonometric function, then use [[integration_by_substitution|substitution]].

**Trigonometric substitution** handles integrands containing:
$$
\sqrt{a^2-x^2},\quad\sqrt{a^2+x^2},\quad\sqrt{x^2-a^2}.
$$
Use the respective inverse substitutions:
$$
x = a\sin\theta,\qquad x = a\tan\theta,\qquad x = a\sec\theta,
$$
then find $dx$ by differentiation and change the limits of integration accordingly.

**Example 1** (even powers). $\displaystyle\int_{0}^{2\pi}\cos^2\theta\,d\theta$:
$$
\int_{0}^{2\pi}\cos^2\theta\,d\theta = \frac{1}{2}\int_{0}^{2\pi}(1+\cos 2\theta)\,d\theta = \frac{1}{2}\left[\theta+\frac{1}{2}\sin 2\theta\right]_{0}^{2\pi} = \pi.
$$

**Example 2** (odd power). $\displaystyle\int_{0}^{\pi/2}\cos^3\theta\,d\theta$. Write $\cos^3\theta = (1-\sin^2\theta)\cos\theta$ and let $u=\sin\theta$, $du=\cos\theta\,d\theta$:
$$
\int_{0}^{\pi/2}\cos^3\theta\,d\theta = \int_{0}^{1}(1-u^2)\,du = \left[u-\tfrac{1}{3}u^3\right]_{0}^{1} = \frac{2}{3}.
$$

**Example 3** (trig substitution). $\displaystyle\int_{0}^{1}\sqrt{1-x^2}\,dx$. Let $x=\sin\theta$, $dx=\cos\theta\,d\theta$; limits: $\theta\in[0,\tfrac{\pi}{2}]$:
$$
\int_{0}^{1}\sqrt{1-x^2}\,dx = \int_{0}^{\pi/2}\cos^2\theta\,d\theta = \frac{1}{2}\int_{0}^{\pi/2}(1+\cos 2\theta)\,d\theta = \frac{\pi}{4}.
$$

## Applications

- Computing areas and arc lengths of curves defined by circles, ellipses, and other conic sections.
- Evaluating integrals arising in Fourier analysis (orthogonality of $\sin$ and $\cos$).
- Solving integrals in physics involving oscillatory motion or circular geometry.

## Trade-offs

- The even/odd strategy requires recognising the parity of both exponents before choosing a method; mixing up the cases leads to dead ends.
- After a trigonometric substitution, back-substituting from $\theta$ to $x$ requires care with signs (e.g.\ $\sqrt{1-\sin^2\theta} = |\cos\theta|$, not always $\cos\theta$).
- Trig substitution introduces a change of variable that must be invertible on the integration domain; checking the domain of $\theta$ is essential.
- For mixed products $\sin^m\theta\cos^n\theta$ with large even exponents, repeated application of half-angle formulas can become tedious; reduction formulas or a CAS may be preferable.

## Links

- [[integration_by_substitution|Integration by Substitution]]
- [[integrals|Integrals]]
