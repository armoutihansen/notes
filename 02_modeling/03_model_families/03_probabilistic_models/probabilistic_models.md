---
layer: 02_modeling
type: concept
status: growing
tags: [algorithm, classification, clustering]
created: 2026-03-06
---

# Probabilistic Models

## Definition

Models that explicitly represent probability distributions over outputs (and sometimes latent variables), enabling calibrated uncertainty estimates and principled inference.

## Intuition

Unlike discriminative models that directly learn $P(y|\mathbf{x})$, generative probabilistic models model the joint distribution $P(\mathbf{x}, y)$ or the full data distribution $P(\mathbf{x})$. This enables not just prediction but density estimation, anomaly detection, and sampling.

## Formal Description

### Naive Bayes Classifier

**Generative model:** $P(y, \mathbf{x}) = P(y)\prod_{j=1}^d P(x_j | y)$ — assumes features are conditionally independent given the class.

**Prediction:** apply Bayes' theorem:

$$P(y|\mathbf{x}) \propto P(y)\prod_{j=1}^d P(x_j|y)$$

**Variants by feature type:**

| Variant | $P(x_j|y)$ model | Use case |
|---|---|---|
| Gaussian NB | $\mathcal{N}(\mu_{jy}, \sigma_{jy}^2)$ | Continuous features |
| Multinomial NB | Multinomial with $\theta_{jy}$ | Text (word counts) |
| Bernoulli NB | Bernoulli with $p_{jy}$ | Binary features |
| Complement NB | Complement classes | Text (better for imbalanced) |

**Laplace smoothing** prevents zero probabilities: $\hat{P}(x_j|y) = (c_{jy} + \alpha)/(c_y + \alpha d)$.

**Strength:** extremely fast, works well on high-dimensional sparse text; robust to irrelevant features. **Limitation:** conditional independence is almost always violated, producing poor probability calibration (needs Platt scaling or isotonic regression).

### Gaussian Mixture Model (GMM)

Models data as a mixture of $K$ Gaussians:

$$p(\mathbf{x}) = \sum_{k=1}^K \pi_k\,\mathcal{N}(\mathbf{x};\boldsymbol{\mu}_k, \boldsymbol{\Sigma}_k), \qquad \sum_k \pi_k = 1, \quad \pi_k \geq 0$$

**EM Algorithm for GMM:**

*E-step:* compute responsibilities (posterior probability of each component given data point):

$$r_{ik} = \frac{\pi_k\,\mathcal{N}(\mathbf{x}_i;\boldsymbol{\mu}_k,\boldsymbol{\Sigma}_k)}{\sum_j \pi_j\,\mathcal{N}(\mathbf{x}_i;\boldsymbol{\mu}_j,\boldsymbol{\Sigma}_j)}$$

*M-step:* update parameters using responsibilities as soft assignments:

$$\pi_k^{\text{new}} = \frac{1}{n}\sum_i r_{ik}, \qquad \boldsymbol{\mu}_k^{\text{new}} = \frac{\sum_i r_{ik}\mathbf{x}_i}{\sum_i r_{ik}}$$

$$\boldsymbol{\Sigma}_k^{\text{new}} = \frac{\sum_i r_{ik}(\mathbf{x}_i - \boldsymbol{\mu}_k^{\text{new}})(\mathbf{x}_i - \boldsymbol{\mu}_k^{\text{new}})^\top}{\sum_i r_{ik}}$$

**Covariance types** (`covariance_type` in sklearn):

| Type | Parameters | Constraint |
|---|---|---|
| `full` | $K$ full matrices | Most flexible, most parameters |
| `tied` | 1 shared matrix | Shared shape across components |
| `diag` | $K$ diagonal matrices | Axis-aligned ellipsoids |
| `spherical` | $K$ scalars | Spherical components |

**Model selection:** use BIC to choose $K$: $\text{BIC} = k\ln n - 2\ln\hat{L}$; penalises model complexity.

**Relation to k-means:** k-means is a special case of GMM EM with spherical equal-variance components and hard assignments (0/1 instead of soft $r_{ik}$).

### Bayesian Linear Regression

Places a Gaussian prior over weights: $\mathbf{w} \sim \mathcal{N}(\mathbf{0}, \sigma_0^2 I)$.

With Gaussian likelihood, the posterior is Gaussian:

$$\mathbf{w}|\mathcal{D} \sim \mathcal{N}\!\left(\boldsymbol{\mu}_n, \boldsymbol{\Sigma}_n\right)$$

$$\boldsymbol{\Sigma}_n = \left(\sigma_0^{-2}I + \sigma^{-2}X^\top X\right)^{-1}, \qquad \boldsymbol{\mu}_n = \sigma^{-2}\boldsymbol{\Sigma}_n X^\top\mathbf{y}$$

The MAP estimate $\boldsymbol{\mu}_n$ is identical to ridge regression with $\lambda = \sigma^2/\sigma_0^2$. The full posterior gives predictive uncertainty that increases away from training data — richer than point estimates.

## Applications

- Naive Bayes: spam filtering, document classification, text categorisation
- GMM: density estimation, soft clustering, anomaly detection, speaker diarisation
- Bayesian linear regression: uncertainty-aware prediction, active learning

## Trade-offs

- Naive Bayes: strong independence assumption → poor calibration; but fast, interpretable, good baseline.
- GMM: sensitive to initialisation; EM can converge to local optima; `n_init > 1` is recommended.
- Bayesian regression: analytical but scales as $O(d^3)$ for covariance inversion; use sparse priors for high-dimensional problems.

## Links

- [[01_foundations/03_probability_and_statistics/bayesian_inference|Bayesian Inference]]
- [[01_foundations/03_probability_and_statistics/probability_distributions|Probability Distributions]]
- [[02_modeling/03_model_families/06_unsupervised_learning/unsupervised_learning|Unsupervised Learning (k-means vs GMM)]]
- [[02_modeling/04_training_dynamics/optimization_algorithms|Optimization Algorithms (EM as coordinate ascent)]]
