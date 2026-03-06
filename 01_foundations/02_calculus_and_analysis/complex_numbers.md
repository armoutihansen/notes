---
layer: 01_foundations
type: concept
status: seed
tags: [theory]
created: 2026-03-01
---

# Complex Numbers

## Definition

The imaginary unit $i$ is defined as one of the two numbers (the other being $-i$) satisfying $z^{2}+1=0$, i.e. $i=\sqrt{-1}$. A **complex number** $z$ and its **complex conjugate** $\bar{z}$ are
$$
z=x+iy,\quad \bar{z}=x-iy,
$$
where $x,y\in\mathbb{R}$. We write $x=\mathrm{Re}\,z$ and $y=\mathrm{Im}\,z$.

## Intuition

The real line is extended to a two-dimensional plane: the real part moves left/right, the imaginary part moves up/down. Multiplication by $i$ rotates a point 90° counterclockwise, and the polar form makes this geometric action explicit — multiplying two complex numbers adds their angles and multiplies their magnitudes.

## Formal Description

**Extracting real and imaginary parts:**
$$
\mathrm{Re}\,z=\frac{1}{2}(z+\bar{z}),\quad \mathrm{Im}\,z=\frac{1}{2i}(z-\bar{z}).
$$

**Modulus (absolute value):**
$$
|z|=(z\bar{z})^{1/2}=\sqrt{x^{2}+y^{2}}.
$$

**Arithmetic:** Addition, subtraction, and multiplication follow from $i^{2}=-1$. Division uses $\frac{z}{w}=\frac{z\bar{w}}{w\bar{w}}$.

**Polar form:** Representing $z$ in the complex plane with $x=r\cos\theta$, $y=r\sin\theta$ gives
$$
z = x+iy = r(\cos\theta+i\sin\theta) = re^{i\theta},
$$
where $r=|z|$ and $\theta=\arg z$. The angle $\theta$ is not unique; the principal value satisfies $-\pi<\theta\leq\pi$.

Polar multiplication: if $z_{1}=r_{1}e^{i\theta_{1}}$ and $z_{2}=r_{2}e^{i\theta_{2}}$, then
$$
z_{1}z_{2}=r_{1}r_{2}e^{i(\theta_{1}+\theta_{2})}.
$$
Multiplying by $e^{i\theta_{2}}$ (with $r_{2}=1$) rotates $z_{1}$ counterclockwise by $\theta_{2}$.

**Euler's formula** (derived from the Taylor series of $e^x$, $\cos x$, $\sin x$):
$$
e^{i\theta}=\cos\theta+i\sin\theta,\quad e^{-i\theta}=\cos\theta-i\sin\theta.
$$

**Cosine and sine from exponentials:**
$$
\cos x=\frac{e^{ix}+e^{-ix}}{2},\quad \sin x=\frac{e^{ix}-e^{-ix}}{2i}.
$$

**Euler's identity** (setting $\theta=\pi$):
$$
e^{i\pi}+1=0.
$$

## Applications

Complex numbers are used to represent 2D rotations and scaling in a single multiplication, to analyse AC circuits via phasors, and to solve polynomial equations over $\mathbb{C}$. Fourier analysis relies on $e^{i\omega t}$ as a basis of oscillatory functions.

## Trade-offs

$\mathbb{C}$ is not an ordered field — there is no consistent way to compare complex numbers as greater or less than one another. The argument $\theta$ is multi-valued; choosing a branch cut (e.g. the principal value) introduces a discontinuity.

## Links

- [[trigonometric_functions|Trigonometric Functions]]
- Taylor Series — Euler's formula is derived from the Taylor expansions of $e^x$, $\cos x$, and $\sin x$
