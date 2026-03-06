---
layer: 01_foundations
type: concept
status: growing
tags: [theory]
created: 2026-03-06
---

# Lagrangian and Constrained Optimization

## Definition

The Lagrangian method converts a constrained optimization problem into an unconstrained one by incorporating constraints into the objective via **Lagrange multipliers**. The KKT conditions characterise optima of constrained problems.

## Intuition

A constraint restricts the feasible region. The Lagrange multiplier $\lambda$ measures the sensitivity of the optimal value to relaxing that constraint — if $\lambda$ is large, tightening the constraint costs a lot; if $\lambda = 0$, the constraint is inactive at the optimum. The method works because at a constrained optimum, the gradient of the objective is parallel to the gradient of the active constraint.

## Formal Description

### Equality Constraints (Classical Lagrange Multipliers)

Problem: $\min f(\mathbf{x})$ subject to $h_j(\mathbf{x}) = 0$, $j = 1,\ldots,p$.

**Lagrangian:**

$$\mathcal{L}(\mathbf{x}, \boldsymbol{\nu}) = f(\mathbf{x}) + \sum_{j=1}^p \nu_j h_j(\mathbf{x})$$

**Necessary condition** (Lagrange condition): at a local minimum $\mathbf{x}^*$:

$$\nabla_\mathbf{x} \mathcal{L} = \nabla f(\mathbf{x}^*) + \sum_j \nu_j^* \nabla h_j(\mathbf{x}^*) = \mathbf{0}$$

Geometrically: $\nabla f$ lies in the span of $\{\nabla h_j\}$.

### Inequality Constraints (KKT Conditions)

Problem: $\min f(\mathbf{x})$ s.t. $g_i(\mathbf{x}) \leq 0$, $i=1,\ldots,m$; $h_j(\mathbf{x})=0$, $j=1,\ldots,p$.

**Lagrangian:**

$$\mathcal{L}(\mathbf{x}, \boldsymbol{\lambda}, \boldsymbol{\nu}) = f(\mathbf{x}) + \sum_{i=1}^m \lambda_i g_i(\mathbf{x}) + \sum_{j=1}^p \nu_j h_j(\mathbf{x})$$

**KKT conditions** (necessary for local optimum under constraint qualification; sufficient for convex problems):

| Condition | Equation |
|---|---|
| Stationarity | $\nabla_\mathbf{x}\mathcal{L} = \mathbf{0}$ |
| Primal feasibility | $g_i(\mathbf{x}^*) \leq 0$, $h_j(\mathbf{x}^*) = 0$ |
| Dual feasibility | $\lambda_i^* \geq 0$ |
| Complementary slackness | $\lambda_i^* g_i(\mathbf{x}^*) = 0$ for all $i$ |

**Complementary slackness** means: either the constraint is active ($g_i = 0$) or the multiplier is zero ($\lambda_i = 0$). This classifies constraints as active (binding) or inactive.

### Dual Problem

The **Lagrange dual function**: $g(\boldsymbol{\lambda}, \boldsymbol{\nu}) = \inf_\mathbf{x}\,\mathcal{L}(\mathbf{x}, \boldsymbol{\lambda}, \boldsymbol{\nu})$

Dual problem: $\max_{\boldsymbol{\lambda} \geq 0, \boldsymbol{\nu}} g(\boldsymbol{\lambda}, \boldsymbol{\nu})$

- **Weak duality:** $d^* \leq p^*$ always holds.
- **Strong duality:** $d^* = p^*$ holds for convex problems satisfying Slater's condition (feasible interior point exists).

The dual is always concave (max of a concave function) and often easier to solve.

### Sensitivity Interpretation

At the optimum, $\nu_j^* = -\partial p^*/\partial b_j$ where $b_j$ is the RHS of equality constraint $h_j(\mathbf{x}) = b_j$. Lagrange multipliers are **shadow prices** — the marginal value of relaxing a constraint.

## Applications

**Support Vector Machine (SVM):** primal problem is a QP; dual formulation:
$$\max_{\boldsymbol{\alpha}} \sum_i \alpha_i - \frac{1}{2}\sum_{i,j}\alpha_i\alpha_j y_i y_j \mathbf{x}_i^\top\mathbf{x}_j \quad \text{s.t.} \sum_i \alpha_i y_i = 0,\; 0 \leq \alpha_i \leq C$$

The kernel trick enters naturally in the dual, replacing $\mathbf{x}_i^\top\mathbf{x}_j$ with $k(\mathbf{x}_i, \mathbf{x}_j)$.

**Lasso/Constrained regression:** $\min \|y - X\beta\|^2$ s.t. $\|\beta\|_1 \leq t$ — Lagrangian gives the penalised form.

**Portfolio optimisation:** maximise expected return subject to variance constraint.

## Trade-offs

- KKT conditions identify candidates for optima but do not guarantee finding a global optimum for non-convex problems.
- The dual can be much lower-dimensional than the primal (e.g., SVM dual has $n$ variables, primal has $d+1$).

## Links

- [[convex_optimization|Convex Optimization]]
- [[gradient_descent_optimization|Gradient Descent and Variants]]
- [[01_foundations/02_calculus_and_analysis/01_differentiation/partial_derivatives|Partial Derivatives]]
- [[02_modeling/03_model_families/05_kernel_methods/index|Kernel Methods (SVM)]]
