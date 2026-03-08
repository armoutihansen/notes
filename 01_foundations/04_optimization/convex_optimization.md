---
layer: 01_foundations
type: concept
status: growing
tags: [theory]
created: 2026-03-06
---

# Convex Optimization

## Definition

An optimization problem is **convex** if the objective function is convex and the feasible set is a convex set. Any local minimum of a convex problem is a global minimum.

## Intuition

Convexity is the "nice" case of optimization: the loss landscape has no local minima traps. This is why methods like logistic regression, SVMs, and lasso are tractable at scale — their objectives are convex. Neural network training is non-convex, but understanding convex problems illuminates why certain algorithms work and fail.

## Formal Description

### Convex Sets and Functions

**Convex set** $\mathcal{C} \subseteq \mathbb{R}^n$: for all $\mathbf{x}, \mathbf{y} \in \mathcal{C}$ and $\lambda \in [0,1]$:
$$
\lambda\mathbf{x} + (1-\lambda)\mathbf{y} \in \mathcal{C}
$$

Examples: halfspaces, polyhedra, norm balls, the positive semidefinite cone.

**Convex function** $f: \mathcal{C} \to \mathbb{R}$: for all $\mathbf{x}, \mathbf{y} \in \mathcal{C}$ and $\lambda \in [0,1]$:
$$
f(\lambda\mathbf{x} + (1-\lambda)\mathbf{y}) \leq \lambda f(\mathbf{x}) + (1-\lambda)f(\mathbf{y})
$$

Equivalently (for differentiable $f$): $f(\mathbf{y}) \geq f(\mathbf{x}) + \nabla f(\mathbf{x})^\top(\mathbf{y} - \mathbf{x})$ (first-order condition).

For twice-differentiable $f$: $f$ is convex iff $\nabla^2 f(\mathbf{x}) \succeq 0$ (PSD Hessian) everywhere.

**Strictly convex:** strict inequality above; unique global minimum.

**Strongly convex** with parameter $m > 0$: $f(\mathbf{y}) \geq f(\mathbf{x}) + \nabla f(\mathbf{x})^\top(\mathbf{y}-\mathbf{x}) + \frac{m}{2}\|\mathbf{y}-\mathbf{x}\|^2$. Guarantees linear convergence of gradient descent.

**Preservation rules:**
- Non-negative linear combinations of convex functions are convex.
- The composition $g(f(\mathbf{x}))$ is convex if $g$ is convex and non-decreasing and $f$ is convex.
- Maximum of convex functions is convex.

### Convex Optimization Problem

$$
\min_{\mathbf{x}} f(\mathbf{x}) \quad \text{s.t.} \quad g_i(\mathbf{x}) \leq 0,\; h_j(\mathbf{x}) = 0
$$

where $f, g_i$ are convex and $h_j$ are affine.

**Key result:** every local minimum is a global minimum.

**First-order optimality (unconstrained):** $\nabla f(\mathbf{x}^*) = 0$.

### KKT Conditions (Constrained Problems)

For the problem $\min f(\mathbf{x})$ s.t. $g_i(\mathbf{x}) \leq 0$, $h_j(\mathbf{x}) = 0$, the **Lagrangian** is:

$$
\mathcal{L}(\mathbf{x}, \boldsymbol{\lambda}, \boldsymbol{\nu}) = f(\mathbf{x}) + \sum_i \lambda_i g_i(\mathbf{x}) + \sum_j \nu_j h_j(\mathbf{x})
$$

**KKT necessary conditions** for optimality $(\mathbf{x}^*, \boldsymbol{\lambda}^*, \boldsymbol{\nu}^*)$:
1. **Stationarity:** $\nabla f + \sum_i \lambda_i^* \nabla g_i + \sum_j \nu_j^* \nabla h_j = \mathbf{0}$
2. **Primal feasibility:** $g_i(\mathbf{x}^*) \leq 0$, $h_j(\mathbf{x}^*) = 0$
3. **Dual feasibility:** $\lambda_i^* \geq 0$
4. **Complementary slackness:** $\lambda_i^* g_i(\mathbf{x}^*) = 0$

For convex problems, KKT conditions are **sufficient** as well as necessary (under Slater's condition for strong duality).

### Common Convex Problems in ML

| Problem | Convex? | Notes |
|---|---|---|
| Linear regression (squared loss) | ✅ Strongly convex | Closed-form solution |
| Logistic regression | ✅ Convex | No closed form; gradient descent works |
| Lasso ($\ell_1$ regularisation) | ✅ Convex, non-smooth | Requires proximal methods or coordinate descent |
| Ridge regression ($\ell_2$) | ✅ Strongly convex | Closed-form solution |
| Hard-margin SVM | ✅ QP | Kernelizable via dual |
| Neural network training | ❌ Non-convex | Multiple local minima, saddle points |

### Duality

The **dual problem**: $\max_{\boldsymbol{\lambda} \geq 0, \boldsymbol{\nu}} g(\boldsymbol{\lambda}, \boldsymbol{\nu})$ where $g(\boldsymbol{\lambda}, \boldsymbol{\nu}) = \inf_\mathbf{x} \mathcal{L}(\mathbf{x}, \boldsymbol{\lambda}, \boldsymbol{\nu})$.

**Weak duality:** $d^* \leq p^*$ always.

**Strong duality** (Slater's condition): if the primal is convex and strictly feasible, $d^* = p^*$. SVMs are solved via their dual, which is often more convenient.

## Applications

- Logistic regression, lasso, ridge, elastic net: convex, efficient global optimisation
- SVM training: quadratic program (QP), solved in dual
- Convex relaxations of combinatorial problems (e.g., LP relaxation)
- Neural tangent kernel theory assumes convex-like analysis in overparameterised regime

## Trade-offs

- Convex problems are tractable but often insufficient to capture the representational power needed for complex tasks (hence neural networks).
- Non-convex objectives like neural networks often still converge to good solutions in practice due to overparameterisation and benign loss landscape structure.

## Links

- [[gradient_descent_optimization|Gradient Descent and Variants]]
- [[lagrangian_and_constrained_optimization|Lagrangian and Constrained Optimization]]
- [[01_foundations/02_calculus_and_analysis/01_differentiation/partial_derivatives|Partial Derivatives]]
- [[01_foundations/02_calculus_and_analysis/01_differentiation/chain_rule|Chain Rule]]
- [[03_modeling/06_training_and_regularization/optimization_algorithms|Optimization Algorithms]]
- [[03_modeling/01_supervised_learning/03_kernel_methods/index|Kernel Methods]]
