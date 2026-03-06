---
layer: 01_foundations
type: concept
status: seed
tags: [theory]
created: 2026-03-01
---

# Vectors

## Definition

A **vector** is a quantity with both magnitude and direction. Vectors can be added to one another and multiplied by scalars (real numbers). A **Cartesian coordinate system** assigns three mutually perpendicular axes ($x$, $y$, $z$) to three-dimensional space; placing the tail of a vector at the origin, its head points to a coordinate triple that gives the vector's components.

## Intuition

Think of a vector as an arrow: its length is the magnitude and its pointing direction captures the direction. The same arrow can be described abstractly (magnitude + direction) or concretely via components measured along chosen axes. Changing axes changes the numbers but not the arrow itself.

## Formal Description

**Standard unit vectors** $\mathbf{i},\mathbf{j},\mathbf{k}$ point along the positive $x$-, $y$-, $z$-axes respectively, each with length 1. A vector $\mathbf{A}$ with components $(A_{1},A_{2},A_{3})$ is written
$$
\mathbf{A}=A_{1}\mathbf{i}+A_{2}\mathbf{j}+A_{3}\mathbf{k}.
$$

**Length** (independent of coordinate orientation):
$$
|\mathbf{A}|=\sqrt{A_{1}^{2}+A_{2}^{2}+A_{3}^{2}}.
$$

**Vector addition** is commutative and associative:
$$
\mathbf{A}+\mathbf{B}=\mathbf{B}+\mathbf{A},\qquad(\mathbf{A}+\mathbf{B})+\mathbf{C}=\mathbf{A}+(\mathbf{B}+\mathbf{C}).
$$

**Component-wise operations** (with $\mathbf{B}=B_{1}\mathbf{i}+B_{2}\mathbf{j}+B_{3}\mathbf{k}$):
$$
\mathbf{A}+\mathbf{B}=(A_{1}+B_{1})\mathbf{i}+(A_{2}+B_{2})\mathbf{j}+(A_{3}+B_{3})\mathbf{k},\qquad c\mathbf{A}=cA_{1}\mathbf{i}+cA_{2}\mathbf{j}+cA_{3}\mathbf{k}.
$$

**Scalar multiplication** is distributive:
$$
k(\mathbf{A}+\mathbf{B})=k\mathbf{A}+k\mathbf{B}.
$$
Multiplying by a positive scalar changes the length but not the direction of a vector.

**Geometric interpretation:** Vector addition is represented by placing the tail of one vector at the head of the other. For $\mathbf{A}$ and $\mathbf{B}$ sharing a tail, the vector $\mathbf{B}-\mathbf{A}$ points from the head of $\mathbf{A}$ to the head of $\mathbf{B}$.

**Position vector:**
$$
\mathbf{r}=x\mathbf{i}+y\mathbf{j}+z\mathbf{k}.
$$

**Displacement vector** from $\mathbf{r}_{1}$ to $\mathbf{r}_{2}$:
$$
\mathbf{r}_{2}-\mathbf{r}_{1}=(x_{2}-x_{1})\mathbf{i}+(y_{2}-y_{1})\mathbf{j}+(z_{2}-z_{1})\mathbf{k}.
$$

## Applications

Newton's equation for a mass $m$ acted on by forces $\mathbf{F}_{1}$ and $\mathbf{F}_{2}$:
$$
m\mathbf{a}=\mathbf{F}_{1}+\mathbf{F}_{2}.
$$

Cartesian components are used throughout physics, computer graphics, and machine learning (feature vectors, weight vectors).

## Trade-offs

Component representation depends on the choice of axes; abstract vector properties (length, addition, scalar multiplication) are basis-independent. In higher dimensions or curved spaces, other coordinate systems may be more natural than Cartesian ones.

## Links

- [[position_velocity_acceleration|Position, Velocity and Acceleration]] — velocity and acceleration as vectors
- [[trigonometric_functions|Trigonometric Functions]] — polar/Cartesian conversion via $\cos$ and $\sin$
