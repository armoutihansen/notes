---
layer: 01_foundations
type: concept
status: seed
tags: [theory]
created: 2026-03-01
---

# Trigonometric Functions

## Definition

The **cosine** $\cos x$ and **sine** $\sin x$ are defined using radians. An angle in radians equals arc length divided by radius; a full revolution is $2\pi$ radians ($360°$). Arguments of trigonometric functions are dimensionless.

## Intuition

Place a point on the unit circle at angle $\theta$ from the positive $x$-axis. The $x$-coordinate of that point is $\cos\theta$ and the $y$-coordinate is $\sin\theta$. As $\theta$ increases, the point travels around the circle, producing the familiar oscillating waves of cosine and sine.

## Formal Description

**Periodicity** (period $2\pi$):
$$
\cos(x+2\pi n)=\cos x,\quad \sin(x+2\pi n)=\sin x,\quad n\in\mathbb{Z}.
$$

**Phase shift:**
$$
\cos\!\left(x-\tfrac{\pi}{2}\right)=\sin x,\quad \sin\!\left(x+\tfrac{\pi}{2}\right)=\cos x.
$$

**Pythagorean identity:**
$$
\cos^{2}x+\sin^{2}x=1.
$$

**Addition formulas:**
$$
\sin(x+y)=\sin x\cos y+\cos x\sin y,\quad \cos(x+y)=\cos x\cos y-\sin x\sin y.
$$

**Other trigonometric functions:**
$$
\tan x=\frac{\sin x}{\cos x},\quad \cot x=\frac{\cos x}{\sin x},\quad \sec x=\frac{1}{\cos x},\quad \csc x=\frac{1}{\sin x}.
$$

**Polar parametrisation** of a circle of radius $r$:
$$
x=r\cos\theta,\quad y=r\sin\theta,\quad 0\leq\theta\leq 2\pi.
$$

**Inverse functions** are defined by restricting domains to ensure bijectivity:

| Function | Restricted domain | Range |
|---|---|---|
| $\arccos x$ | $[0,\pi]$ | $[0,\pi]$ |
| $\arcsin x$ | $[-\tfrac{\pi}{2},\tfrac{\pi}{2}]$ | $[-\tfrac{\pi}{2},\tfrac{\pi}{2}]$ |
| $\arctan x$ | $(-\tfrac{\pi}{2},\tfrac{\pi}{2})$ | $\mathbb{R}$ |

The arctangent has horizontal asymptotes: $\arctan x\to\tfrac{\pi}{2}$ as $x\to+\infty$ and $\arctan x\to-\tfrac{\pi}{2}$ as $x\to-\infty$.

Composite inverse-trig expressions (e.g. $\sin(\arctan x)$, $\cos(\arctan x)$) can be simplified using a right-triangle argument together with $\cos^{2}x+\sin^{2}x=1$.

## Applications

Trigonometric functions model periodic phenomena: oscillations in mechanics ($x(t)=A\cos\omega t$), electromagnetic waves, and audio signals. The addition formulas are the foundation of Fourier analysis, which decomposes arbitrary periodic functions into sinusoidal components.

## Trade-offs

Inverse trigonometric functions are only single-valued after domain restriction; different branches give different answers for the "same" inverse problem. Arguments must be in radians for calculus identities (derivatives, Taylor series) to hold without correction factors.

## Links

- [[complex_numbers|Complex Numbers]] — Euler's formula connects $e^{i\theta}$ to $\cos\theta$ and $\sin\theta$
- [[growth_decay_oscillation|Growth, Decay and Oscillation]] — sinusoidal solutions to $\ddot{x}+\omega^{2}x=0$
