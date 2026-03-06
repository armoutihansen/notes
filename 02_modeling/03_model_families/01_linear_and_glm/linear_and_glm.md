---
layer: 02_modeling
type: concept
status: growing
tags: [algorithm, tabular, regression, classification]
created: 2026-03-06
---

# Linear and Generalised Linear Models

## Definition

Linear models predict a target as a linear function of input features (possibly after a link function transformation). GLMs extend this to non-Gaussian response distributions via a link function connecting the linear predictor to the response mean.

## Intuition

Linear models are the baseline of statistical modelling: interpretable, fast, well-understood theoretically, and often surprisingly competitive. GLMs generalise them to count data, binary outcomes, and positive continuous outcomes — making them the workhorse of insurance pricing, clinical research, and any domain requiring interpretable predictions.

## Formal Description

### Linear Regression

Model: $\hat{y} = \mathbf{w}^\top \mathbf{x} + b$

**OLS solution:** minimises $\mathrm{RSS} = \|\mathbf{y} - X\mathbf{w}\|^2$; closed form: $\hat{\mathbf{w}} = (X^\top X)^{-1}X^\top\mathbf{y}$ (requires $X^\top X$ invertible).

**Assumptions (Gauss-Markov):**
1. Linearity: $\mathbb{E}[y|\mathbf{x}] = \mathbf{w}^\top\mathbf{x}$
2. No multicollinearity: $X^\top X$ has full rank
3. Homoscedasticity: $\mathrm{Var}[\varepsilon] = \sigma^2$ constant
4. No autocorrelation: $\mathrm{Cov}[\varepsilon_i, \varepsilon_j] = 0$

**Regularised variants:**

| Model | Penalty | Effect |
|---|---|---|
| Ridge (L2) | $\lambda\|\mathbf{w}\|_2^2$ | Shrinks coefficients, handles multicollinearity |
| Lasso (L1) | $\lambda\|\mathbf{w}\|_1$ | Sparse solutions (feature selection) |
| Elastic Net | $\lambda_1\|\mathbf{w}\|_1 + \lambda_2\|\mathbf{w}\|_2^2$ | Combines sparsity and stability |

### Logistic Regression

Binary classification via the logistic function:

$$P(y=1|\mathbf{x}) = \sigma(\mathbf{w}^\top\mathbf{x}) = \frac{1}{1+e^{-\mathbf{w}^\top\mathbf{x}}}$$

Training: minimise negative log-likelihood (log-loss):

$$\mathcal{L} = -\frac{1}{n}\sum_i [y_i \log \hat{p}_i + (1-y_i)\log(1-\hat{p}_i)]$$

No closed form; solved via gradient descent or iteratively reweighted least squares (IRLS). The decision boundary $\{\mathbf{x}: \mathbf{w}^\top\mathbf{x} = 0\}$ is a hyperplane.

**Coefficient interpretation:** $e^{w_j}$ is the multiplicative change in odds for a one-unit increase in $x_j$.

### Generalised Linear Models (GLMs)

Three components:
1. **Random component:** $y \sim$ exponential family (Gaussian, Poisson, Gamma, Binomial, Tweedie, ...)
2. **Linear predictor:** $\eta = \mathbf{w}^\top\mathbf{x}$
3. **Link function:** $g(\mu) = \eta$, where $\mu = \mathbb{E}[y|\mathbf{x}]$

**Common GLM families:**

| Family | Link | Use case |
|---|---|---|
| Gaussian | Identity | Continuous symmetric targets |
| Binomial | Logit | Binary classification |
| Poisson | Log | Count data (e.g., claims frequency) |
| Gamma | Log/Inverse | Positive continuous (e.g., claim severity) |
| Tweedie | Log | Zero-inflated positive continuous (pure premium) |

**Tweedie** with variance power $p \in (1,2)$ is the standard model for insurance pure premium (compound Poisson-Gamma).

**Estimation:** maximum likelihood via IRLS. Parameter covariance matrix enables Wald tests and confidence intervals for each coefficient.

**Deviance:** GLM analogue of RSS; measures goodness-of-fit relative to a saturated model. Null deviance − model deviance analogous to $R^2$.

### Multiplicative GLM (Insurance Standard)

With log link on Poisson or Gamma: $\mu = e^{\mathbf{w}^\top\mathbf{x}} = \prod_j e^{w_j x_j}$

Each coefficient $e^{w_j}$ is a **rating factor** multiplying the base rate — directly interpretable and auditable for regulatory purposes.

## Applications

- Ridge/Lasso: high-dimensional feature matrices where $d \gg n$
- Logistic regression: credit scoring, fraud scoring baselines
- Poisson GLM: claim frequency modelling (claims per year of exposure)
- Gamma GLM: claim severity modelling (average cost per claim)
- Tweedie GLM: direct pure premium modelling

## Trade-offs

- Linear models assume the true relationship is (approximately) linear or linearisable via the link function — violated by complex non-linear patterns.
- Interpretability advantage over trees disappears when many interaction terms are added.
- Lasso is brittle with highly correlated features; Elastic Net is more stable.

## Links

- [[01_foundations/01_linear_algebra/01_vector_spaces/least_squares|Least Squares]]
- [[01_foundations/03_probability_and_statistics/bayesian_inference|Bayesian Inference (MAP = regularised MLE)]]
- [[01_foundations/04_optimization/convex_optimization|Convex Optimization]]
- [[02_modeling/04_training_dynamics/regularization|Regularization]]
- [[02_modeling/05_evaluation_and_validation/index|Evaluation and Validation]]
