---
layer: 01_foundations
type: concept
status: growing
tags: [theory]
created: 2026-03-06
---

# Probability Distributions

## Definition

A probability distribution specifies how probability mass (discrete) or density (continuous) is assigned over the possible values of a random variable.

## Intuition

Distributions are the vocabulary of probabilistic modelling. Different processes generate data with characteristic shapes: coin flips are Bernoulli, counts of rare events are Poisson, natural measurements often cluster near a mean (Gaussian), and proportions live on the unit interval (Beta).

## Formal Description

### Discrete Distributions

**Bernoulli** — single binary trial with success probability $p$:
$$
P(X = k) = p^k(1-p)^{1-k}, \quad k \in \{0,1\}, \quad \mathbb{E}[X] = p, \quad \operatorname{Var}(X) = p(1-p)
$$

**Binomial** — number of successes in $n$ independent Bernoulli($p$) trials:
$$
P(X = k) = \binom{n}{k}p^k(1-p)^{n-k}, \quad \mathbb{E}[X] = np, \quad \operatorname{Var}(X) = np(1-p)
$$

**Poisson** — number of events in a fixed interval at rate $\lambda$:
$$
P(X = k) = \frac{\lambda^k e^{-\lambda}}{k!}, \quad \mathbb{E}[X] = \lambda, \quad \operatorname{Var}(X) = \lambda
$$

**Categorical** — generalisation of Bernoulli to $K$ categories with probabilities $(\pi_1,\ldots,\pi_K)$, $\sum_k \pi_k = 1$.

### Continuous Distributions

**Uniform** $\operatorname{Unif}(a,b)$:
$$
f(x) = \frac{1}{b-a}, \quad \mathbb{E}[X] = \frac{a+b}{2}, \quad \operatorname{Var}(X) = \frac{(b-a)^2}{12}
$$

**Gaussian (Normal)** $\mathcal{N}(\mu, \sigma^2)$ — the most important distribution in statistics:
$$
f(x) = \frac{1}{\sigma\sqrt{2\pi}}\exp\!\left(-\frac{(x-\mu)^2}{2\sigma^2}\right), \quad \mathbb{E}[X] = \mu, \quad \operatorname{Var}(X) = \sigma^2
$$

Standard normal: $Z = (X - \mu)/\sigma \sim \mathcal{N}(0,1)$.

**Multivariate Gaussian** $\mathcal{N}(\boldsymbol{\mu}, \boldsymbol{\Sigma})$:
$$
f(\mathbf{x}) = \frac{1}{(2\pi)^{d/2}|\boldsymbol{\Sigma}|^{1/2}}\exp\!\left(-\tfrac{1}{2}(\mathbf{x}-\boldsymbol{\mu})^\top\boldsymbol{\Sigma}^{-1}(\mathbf{x}-\boldsymbol{\mu})\right)
$$

$\boldsymbol{\Sigma}$ is the covariance matrix; marginals and conditionals of joint Gaussians are Gaussian.

**Exponential** $\operatorname{Exp}(\lambda)$ — time until first event at rate $\lambda$, memoryless:
$$
f(x) = \lambda e^{-\lambda x}, \quad x \geq 0, \quad \mathbb{E}[X] = 1/\lambda, \quad \operatorname{Var}(X) = 1/\lambda^2
$$

**Gamma** $\operatorname{Gamma}(\alpha, \beta)$ — sum of $\alpha$ independent $\operatorname{Exp}(\beta)$ variables; conjugate prior for Poisson rate:
$$
f(x) = \frac{\beta^\alpha}{\Gamma(\alpha)}x^{\alpha-1}e^{-\beta x}, \quad x > 0
$$

**Beta** $\operatorname{Beta}(\alpha,\beta)$ — distribution over $[0,1]$; conjugate prior for Bernoulli/Binomial $p$:
$$
f(x) = \frac{x^{\alpha-1}(1-x)^{\beta-1}}{B(\alpha,\beta)}, \quad x \in [0,1]
$$

Symmetric when $\alpha=\beta$; $\alpha=\beta=1$ is Uniform; concentrates near extremes when $\alpha,\beta < 1$.

**Dirichlet** $\operatorname{Dir}(\boldsymbol{\alpha})$ — multivariate generalisation of Beta; conjugate prior for Categorical/Multinomial:
$$
f(\mathbf{x}) \propto \prod_{k=1}^K x_k^{\alpha_k - 1}, \quad \sum_k x_k = 1, \quad x_k \geq 0
$$

### Central Limit Theorem

For i.i.d. $X_1, \ldots, X_n$ with mean $\mu$ and variance $\sigma^2 < \infty$:

$$
\frac{\bar{X}_n - \mu}{\sigma/\sqrt{n}} \xrightarrow{d} \mathcal{N}(0,1) \quad \text{as } n \to \infty
$$

Justifies Gaussian approximations in large-sample inference.

### Exponential Family

Many distributions share the form:
$$
f(x \mid \boldsymbol{\eta}) = h(x)\exp\!\left(\boldsymbol{\eta}^\top T(x) - A(\boldsymbol{\eta})\right)
$$

where $\boldsymbol{\eta}$ are natural parameters, $T(x)$ are sufficient statistics, and $A(\boldsymbol{\eta})$ is the log-partition function. Conjugate priors exist for exponential family likelihoods. Examples: Gaussian, Bernoulli, Poisson, Gamma, Beta, Dirichlet.

## Applications

- Gaussian: linear regression errors, natural measurements, approximate posteriors
- Bernoulli/Binomial: classification, A/B testing
- Poisson: count data in insurance (claims frequency), NLP (word counts)
- Beta/Dirichlet: Bayesian priors for probabilities and topic models (LDA)
- Exponential/Gamma: survival analysis, insurance claim severity

## Trade-offs

- The Gaussian assumption is often violated in practice (heavy tails, skewness); always check empirically before assuming normality.
- Conjugate priors give analytical posteriors but may not match domain knowledge; non-conjugate priors require MCMC or variational inference.

## Links

- [[probability_theory|Probability Theory]]
- [[bayesian_inference|Bayesian Inference]]
- [[01_foundations/05_statistical_learning_theory/evaluation_metrics|Evaluation Metrics]]
- [[03_modeling/03_probabilistic_models/index|Probabilistic Models]]
- [[03_modeling/06_training_and_regularization/loss_functions|Loss Functions]]
