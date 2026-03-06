---
layer: 01_foundations
type: concept
status: growing
tags: [gradient, directional-derivative, vector-calculus, optimization, level-set]
created: 2026-03-06
---

# Gradient and Directional Derivative

## Definition

The **gradient** of a scalar field $f: \mathbb{R}^n \to \mathbb{R}$ at a point $\mathbf{x}$ is the vector of partial derivatives:

$$\nabla f(\mathbf{x}) = \begin{pmatrix} \partial f/\partial x_1 \\ \vdots \\ \partial f/\partial x_n \end{pmatrix} \in \mathbb{R}^n$$

It points in the direction of steepest ascent of $f$ at $\mathbf{x}$, with magnitude equal to the rate of steepest ascent.

## Intuition

The gradient generalises the derivative to multiple dimensions. For a 2D landscape $f(x,y)$ = altitude, $\nabla f$ is an arrow on the horizontal plane pointing uphill as steeply as possible. Its magnitude tells you how steep that slope is. Moving in the direction $-\nabla f$ descends as steeply as possible — the basis of gradient descent.

The directional derivative answers: "how fast does $f$ change if I walk in direction $\mathbf{u}$?" The gradient is the tool that computes this for any direction at once.

## Formal Description

**Partial derivative:** fix all variables except $x_i$ and differentiate:

$$\frac{\partial f}{\partial x_i}(\mathbf{x}) = \lim_{h \to 0} \frac{f(\mathbf{x} + h\mathbf{e}_i) - f(\mathbf{x})}{h}$$

**Directional derivative** in direction $\mathbf{u}$ ($\|\mathbf{u}\|=1$):

$$D_\mathbf{u} f(\mathbf{x}) = \lim_{h \to 0} \frac{f(\mathbf{x} + h\mathbf{u}) - f(\mathbf{x})}{h} = \nabla f(\mathbf{x}) \cdot \mathbf{u}$$

The directional derivative equals the dot product of the gradient with the unit direction. Maximised when $\mathbf{u} = \nabla f / \|\nabla f\|$, giving steepest ascent. Zero when $\mathbf{u} \perp \nabla f$ (moving along a level set).

**Level sets and gradient orthogonality:** the level set $\{x : f(\mathbf{x}) = c\}$ is a $(n-1)$-dimensional surface. The gradient $\nabla f(\mathbf{x})$ is **orthogonal to the level set** at every point $\mathbf{x}$.

**First-order Taylor approximation:**

$$f(\mathbf{x} + \delta) \approx f(\mathbf{x}) + \nabla f(\mathbf{x})^\top \delta \qquad \text{for small } \delta$$

This is the linear approximation; the gradient is the coefficient vector.

**Chain rule for scalar composition:** if $g: \mathbb{R}^n \to \mathbb{R}^m$ and $f: \mathbb{R}^m \to \mathbb{R}$:

$$\nabla_\mathbf{x}(f \circ g) = J_g(\mathbf{x})^\top \nabla_\mathbf{y} f(g(\mathbf{x}))$$

where $J_g$ is the Jacobian of $g$ (see [[jacobian_and_hessian]]).

**Gradient in Cartesian coordinates** ($n = 3$):

$$\nabla f = \frac{\partial f}{\partial x}\mathbf{i} + \frac{\partial f}{\partial y}\mathbf{j} + \frac{\partial f}{\partial z}\mathbf{k}$$

## Applications

| Application | Role of gradient |
|---|---|
| Gradient descent | Update $\theta \leftarrow \theta - \alpha \nabla_\theta L$ |
| Backpropagation | Accumulate gradients through the computation graph |
| Lagrange multipliers | Condition: $\nabla f = \lambda \nabla g$ at constrained optimum |
| Physics | Force = $-\nabla V$ (potential energy) |
| Image processing | Image gradient detects edges: $\nabla I = (\partial I/\partial x, \partial I/\partial y)$ |

## Trade-offs

- The gradient exists only where $f$ is differentiable. Non-smooth functions (e.g., ReLU) require **subgradients** at non-differentiable points.
- In high dimensions, computing the full gradient requires evaluating all $n$ partial derivatives; automatic differentiation handles this efficiently.
- Gradient direction is locally optimal but can lead to saddle points or local minima; second-order information (Hessian) is needed to distinguish these.

## Links

- [[jacobian_and_hessian|Jacobian and Hessian]] — multivariable analogue for vector-valued functions
- [[01_foundations/02_calculus_and_analysis/01_differentiation/chain_rule|Chain Rule]] — single-variable chain rule generalised here
- [[01_foundations/04_optimization/gradient_descent_optimization|Gradient Descent]] — gradient used in iterative optimization
- [[01_foundations/04_optimization/convex_optimization|Convex Optimization]] — first-order optimality condition: $\nabla f = 0$
- [[01_foundations/06_deep_learning_theory/backpropagation|Backpropagation]] — gradient of loss w.r.t. parameters
