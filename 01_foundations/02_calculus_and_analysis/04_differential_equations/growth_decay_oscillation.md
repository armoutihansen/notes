---
layer: 01_foundations
type: concept
status: seed
tags: [theory]
created: 2026-03-01
---

# Growth, Decay, and Oscillation

## Definition

Three fundamental behaviours — linear growth, exponential growth/decay, and oscillation — each correspond to a simple ODE. The **order** of a differential equation is the highest-order derivative it contains.

## Intuition

These three ODEs are the simplest non-trivial differential equations and serve as building blocks for understanding more complex systems. Exponential growth arises whenever the rate of change is proportional to the current value; oscillation arises when a restoring force is proportional to displacement.

## Formal Description

**Linear growth:**
$$
x(t)=at+b\quad\Longleftrightarrow\quad\ddot{x}=0,\quad x(0)=b,\;\dot{x}(0)=a.
$$

**Exponential growth ($r>0$) / decay ($r<0$):**
$$
x(t)=Ae^{rt}\quad\Longleftrightarrow\quad\dot{x}=rx,\quad x(0)=A.
$$

**Oscillation:**
$$
x(t)=A\cos\omega t+B\sin\omega t\quad\Longleftrightarrow\quad\ddot{x}+\omega^{2}x=0.
$$

Initial conditions select the specific solution:
- $x(0)=A$, $\dot{x}(0)=0$ $\;\Rightarrow\;$ $x(t)=A\cos\omega t$.
- $x(0)=0$, $\dot{x}(0)=\omega B$ $\;\Rightarrow\;$ $x(t)=B\sin\omega t$.

## Applications

Exponential growth models population growth and compound interest; exponential decay models radioactive decay, cooling, and discharge of a capacitor. Oscillation describes mass-spring systems, pendulums (small angles), and LC circuits.

## Trade-offs

These are idealised models: real systems rarely exhibit pure exponential growth indefinitely (resource limits apply) or pure undamped oscillation (friction dissipates energy). More realistic models combine these behaviours, e.g. damped oscillation $\ddot{x}+2\gamma\dot{x}+\omega^{2}x=0$.

## Links

- [[trigonometric_functions|Trigonometric Functions]] — $\cos$ and $\sin$ as periodic solutions
- [[natural_logarithm|Natural Logarithm]] — exponential function and its inverse
- [[position_velocity_acceleration|Position, Velocity and Acceleration]] — physical interpretation of $\dot{x}$ and $\ddot{x}$
