---
layer: 01_foundations
type: concept
status: growing
tags: [theory, algorithm]
created: 2026-03-06
---

# Bayesian Inference

## Definition

A statistical framework that treats model parameters as random variables and updates a prior belief distribution with observed data to produce a posterior distribution using Bayes' theorem.

## Intuition

Frequentist inference asks "what does this data say about a fixed unknown parameter?" Bayesian inference asks "after seeing this data, what should I believe about the parameter?" The prior encodes what is known before the data; the likelihood encodes what the data says; Bayes' theorem combines them into the posterior.

## Formal Description

**Bayes' theorem for inference:**

$$
\underbrace{P(\theta \mid \mathcal{D})}_{\text{posterior}} = \frac{\underbrace{P(\mathcal{D} \mid \theta)}_{\text{likelihood}} \cdot \underbrace{P(\theta)}_{\text{prior}}}{\underbrace{P(\mathcal{D})}_{\text{evidence}}}
$$

The evidence $P(\mathcal{D}) = \int P(\mathcal{D} \mid \theta)\,P(\theta)\,d\theta$ normalises the posterior but is often intractable.

**Maximum Likelihood Estimation (MLE):** finds the $\theta$ that maximises the likelihood, ignoring the prior:

$$
\hat{\theta}_{\mathrm{MLE}} = \arg\max_\theta P(\mathcal{D} \mid \theta) = \arg\max_\theta \sum_i \log p(x_i \mid \theta)
$$

**Maximum A Posteriori (MAP):** finds the mode of the posterior:

$$
\hat{\theta}_{\mathrm{MAP}} = \arg\max_\theta \log P(\theta \mid \mathcal{D}) = \arg\max_\theta \left[\log P(\mathcal{D} \mid \theta) + \log P(\theta)\right]
$$

MAP with a Gaussian prior is equivalent to L2-regularised MLE (the prior acts as a regulariser).

**Posterior predictive distribution:**

$$
P(\tilde{x} \mid \mathcal{D}) = \int P(\tilde{x} \mid \theta)\,P(\theta \mid \mathcal{D})\,d\theta
$$

Integrates over parameter uncertainty — more principled than plugging in a point estimate.

### Conjugate Priors

A prior $P(\theta)$ is **conjugate** to a likelihood $P(\mathcal{D} \mid \theta)$ if the posterior is in the same distributional family as the prior. This makes updates analytical:

| Likelihood | Conjugate prior | Posterior |
|---|---|---|
| Bernoulli($p$) | Beta($\alpha,\beta$) | Beta($\alpha + k,\, \beta + n - k$) |
| Poisson($\lambda$) | Gamma($\alpha,\beta$) | Gamma($\alpha + \sum x_i,\, \beta + n$) |
| Gaussian (known $\sigma$) | Gaussian($\mu_0, \sigma_0^2$) | Gaussian |
| Categorical($\boldsymbol{\pi}$) | Dirichlet($\boldsymbol{\alpha}$) | Dirichlet($\boldsymbol{\alpha} + \mathbf{c}$) |

**Beta-Binomial example:** after observing $k$ successes in $n$ trials with prior Beta($\alpha,\beta$):

$$
P(p \mid k,n) = \mathrm{Beta}(\alpha + k,\; \beta + n - k)
$$

The posterior mean is $(\alpha + k) / (\alpha + \beta + n)$, which interpolates between the prior mean and the MLE as $n$ grows.

### Approximate Inference

When the posterior is intractable:
- **MCMC (Markov Chain Monte Carlo):** sample from the posterior (Metropolis-Hastings, Hamiltonian MC via Stan/PyMC).
- **Variational inference (VI):** approximate the posterior with a tractable family $q(\theta)$ by minimising $\mathrm{KL}(q \| p)$.
- **Laplace approximation:** fit a Gaussian at the MAP estimate.

## Applications

- **Naive Bayes classifier:** uses class-conditional independence to compute $P(y \mid \mathbf{x}) \propto P(y) \prod_j P(x_j \mid y)$.
- **Bayesian linear regression:** places a Gaussian prior on weights; posterior is analytical and gives uncertainty estimates over predictions.
- **Topic models (LDA):** Dirichlet-Multinomial hierarchy.
- **Bayesian hyperparameter optimisation (Gaussian processes):** prior over functions, update with evaluated points.
- **Online learning:** sequential Bayesian updates as new data arrives.

## Trade-offs

| Approach | Advantage | Limitation |
|---|---|---|
| Full Bayesian | Calibrated uncertainty, principled | Expensive (MCMC/VI), prior choice matters |
| MAP | Cheap, equivalent to regularised MLE | Point estimate, no uncertainty |
| MLE | Simplest, unbiased in large $n$ | Overconfident, no regularisation |

- Prior choice can dominate with small datasets; with large datasets the likelihood overwhelms the prior and results converge to MLE.

## Links

- [[probability_theory|Probability Theory]]
- [[probability_distributions|Probability Distributions]]
- [[hypothesis_testing|Hypothesis Testing]]
- [[01_foundations/05_statistical_learning_theory/bias_variance_analysis|Bias–Variance Analysis]]
- [[03_modeling/03_probabilistic_models/index|Probabilistic Models]]
