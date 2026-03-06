---
layer: 01_foundations
type: concept
status: seed
tags: [theory]
created: 2026-03-01
---

# Position, Velocity, and Acceleration

## Definition

For a mass moving along the $x$-axis, the **position** $x(t)$, **velocity** $v(t)$, and **acceleration** $a(t)$ are related by differentiation with respect to time.

## Intuition

Each quantity is the rate of change of the previous one: velocity measures how quickly position changes, acceleration measures how quickly velocity changes. Integration runs the chain in reverse — given acceleration, integrate to get velocity, then integrate again to get position (plus constants of integration set by initial conditions).

## Formal Description

$$
v(t)=\dot{x}(t),\qquad a(t)=\dot{v}(t)=\ddot{x}(t).
$$

Dot notation denotes time derivatives: one dot for first derivative, two dots for second.

**Newton's second law** ($F=ma$) gives an ODE for position:
$$
\ddot{x}=\frac{F}{m}.
$$

**Velocity as a function of position** — applying the chain rule:
$$
a = \frac{dv}{dt} = \frac{dv}{dx}\frac{dx}{dt} = v\frac{dv}{dx}.
$$

Sign conventions: positive velocity means motion in the positive $x$-direction; positive acceleration increases velocity (speeds up a positive-velocity mass, slows a negative-velocity mass).

## Applications

Foundation of classical mechanics: projectile motion, orbital dynamics, vibrations. The same differentiation structure applies to any quantity and its rate of change (e.g. charge and current in circuit theory).

## Trade-offs

The one-dimensional treatment assumes motion along a single axis; in 2D/3D, position, velocity, and acceleration become vectors. The framework assumes smooth, differentiable motion — discontinuous forces (impulses) require special handling via momentum-impulse relations.

## Links

- [[growth_decay_oscillation|Growth, Decay and Oscillation]] — standard ODEs for position under constant force, drag, or restoring force
- [[chain_rule|Chain Rule]] — used to express acceleration as $v\,dv/dx$
