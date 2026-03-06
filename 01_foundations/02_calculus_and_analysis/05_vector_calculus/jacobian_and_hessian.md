---
layer: 01_foundations
type: concept
status: growing
tags: [theory]
created: 2026-03-06
---

# Jacobian and Hessian

## Definition

**Jacobian:** the matrix of all first-order partial derivatives of a vector-valued function $\mathbf{f}: \mathbb{R}^n \to \mathbb{R}^m$:

$$J_\mathbf{f}(\mathbf{x}) = \frac{\partial \mathbf{f}}{\partial \mathbf{x}} \in \mathbb{R}^{m \times n}, \qquad (J_\mathbf{f})_{ij} = \frac{\partial f_i}{\partial x_j}$$

**Hessian:** the matrix of all second-order partial derivatives of a scalar function $f: \mathbb{R}^n \to \mathbb{R}$:

$$H_f(\mathbf{x}) = \nabla^2 f(\mathbf{x}) \in \mathbb{R}^{n \times n}, \qquad H_{ij} = \frac{\partial^2 f}{\partial x_i\, \partial x_j}$$

When $f$ has continuous second derivatives (Schwarz's theorem), $H$ is symmetric: $H_{ij} = H_{ji}$.

## Intuition

**Jacobian** is the "derivative" of a multi-input, multi-output function — it captures how each output changes as each input changes. A $1 \times n$ Jacobian is the gradient row vector; an $m \times n$ Jacobian generalises to multiple outputs. The Jacobian is the best linear approximation of $\mathbf{f}$ near $\mathbf{x}$.

**Hessian** encodes the curvature of a scalar landscape at a point. Just as the second derivative tells you whether a 1D function is concave or convex, the Hessian's eigenvalues tell you the same in each direction. Positive definite Hessian → local minimum; indefinite Hessian → saddle point.

## Formal Description

**Jacobian: vector chain rule.** If $\mathbf{h} = \mathbf{f}(\mathbf{g}(\mathbf{x}))$, then:

$$J_\mathbf{h} = J_\mathbf{f}(g(\mathbf{x}))\, J_\mathbf{g}(\mathbf{x}) \in \mathbb{R}^{p \times n}$$

This is the multivariable chain rule; it underlies backpropagation's layer-by-layer gradient computation.

**Jacobian determinant:** for a square map $\mathbf{f}: \mathbb{R}^n \to \mathbb{R}^n$, $|\det J_\mathbf{f}|$ measures the local volume scaling factor of the transformation. Used in change-of-variables for integration and in normalising flows.

**Second-order Taylor expansion:**

$$f(\mathbf{x} + \delta) \approx f(\mathbf{x}) + \nabla f(\mathbf{x})^\top \delta + \frac{1}{2} \delta^\top H_f(\mathbf{x})\, \delta$$

**Stationary points:** at a critical point $\nabla f(\mathbf{x}^*) = 0$, the Hessian classifies the point:

| $H$ signature | Type |
|---|---|
| All eigenvalues $> 0$ (PD) | Strict local minimum |
| All eigenvalues $< 0$ (ND) | Strict local maximum |
| Mixed signs (indefinite) | Saddle point |
| Some zero eigenvalues (PSD) | Degenerate — need higher-order terms |

**Newton's method** uses the Hessian for faster convergence than gradient descent:

$$\mathbf{x}_{t+1} = \mathbf{x}_t - H_f(\mathbf{x}_t)^{-1}\, \nabla f(\mathbf{x}_t)$$

Converges quadratically near a minimum but costs $O(n^3)$ per step (Hessian inversion); impractical for large $n$ (e.g., neural network parameters). Quasi-Newton methods (L-BFGS) approximate $H^{-1}$ without explicitly computing it.

**Hessian in deep learning:** the Hessian of the loss w.r.t. parameters has $O(n^2)$ entries — intractable for modern networks with millions of parameters. Second-order information is instead captured implicitly through adaptive optimizers (Adam/RMSProp) or through the Fisher information matrix.

## Applications

| Application | Role |
|---|---|
| Backpropagation | Jacobian of each layer's output w.r.t. input is the layer's local gradient |
| Change of variables | Jacobian determinant scales probability density under transformation |
| Normalising flows | Bijective map with tractable Jacobian determinant |
| Newton / quasi-Newton optimisation | Hessian (or its approximation) gives curvature information |
| Curvature analysis | Hessian eigenvalues diagnose saddle points, condition number |
| Sensitivity analysis | $\partial \hat y / \partial x_j$ (Jacobian column) measures feature importance |

## Trade-offs

- Full Hessian computation: $O(n^2)$ storage, $O(n^3)$ inversion — infeasible for large networks.
- Jacobian-vector products (JVP, forward mode) and vector-Jacobian products (VJP, reverse mode) can be computed without materialising the full Jacobian — this is the basis of automatic differentiation.
- Saddle points dominate in high-dimensional loss landscapes; second-order analysis reveals them, but first-order methods (SGD, Adam) escape them through gradient noise.

## Links

- [[gradient_and_directional_derivative|Gradient and Directional Derivative]] — gradient is the $1 \times n$ Jacobian for scalar functions
- [[01_foundations/02_calculus_and_analysis/01_differentiation/chain_rule|Chain Rule]] — scalar chain rule; Jacobian is its vector generalisation
- [[01_foundations/04_optimization/convex_optimization|Convex Optimization]] — PD Hessian ↔ strictly convex
- [[01_foundations/04_optimization/lagrangian_and_constrained_optimization|Constrained Optimization]] — second-order conditions for KKT
- [[01_foundations/06_deep_learning_theory/backpropagation|Backpropagation]] — uses Jacobian chain rule throughout
- [[01_foundations/01_linear_algebra/03_eigenvalues/spectral_theorem|Spectral Theorem]] — Hessian is symmetric; eigenvalues determine its definiteness
