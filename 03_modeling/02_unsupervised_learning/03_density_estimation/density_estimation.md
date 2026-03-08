---
layer: 03_modeling
type: concept
status: seed
tags: [algorithm, tabular, clustering]
created: 2026-05-31
---

# Density Estimation

## Core Idea

Density estimation constructs an estimate $\hat{p}(\mathbf{x})$ of the underlying data-generating distribution from observed samples. It is used for anomaly detection (low-density points are anomalous), generative modelling, and as a component of Bayesian classifiers.

## Mathematical Formulation

### Kernel Density Estimation (KDE)

Non-parametric: places a kernel $K$ centred at each training point and sums:

$$
\hat{p}(\mathbf{x}) = \frac{1}{n h^d} \sum_{i=1}^n K\left(\frac{\mathbf{x} - \mathbf{x}_i}{h}\right)
$$

where $h > 0$ is the **bandwidth** and $d$ is the dimensionality.

**Common kernels:** Gaussian ($K(u) = \frac{1}{\sqrt{2\pi}} e^{-u^2/2}$), Epanechnikov (optimal in MSE sense), tophat.

**Bandwidth selection:** Silverman's rule of thumb: $h = 1.06\, \hat\sigma\, n^{-1/5}$ (Gaussian kernel, univariate). Cross-validated bandwidth selection is preferred in practice.

```python
from sklearn.neighbors import KernelDensity
import numpy as np

kde = KernelDensity(bandwidth=0.5, kernel='gaussian')
kde.fit(X_train)

log_density = kde.score_samples(X_test)  # log p(x) for each test point
anomaly_mask = log_density < np.percentile(log_density, 5)  # bottom 5%
```

### Gaussian Mixture Model (GMM)

Parametric: models the data as a mixture of $K$ Gaussian components:

$$
p(\mathbf{x}) = \sum_{k=1}^K \pi_k \, \mathcal{N}(\mathbf{x}; \boldsymbol{\mu}_k, \Sigma_k)
$$

Parameters: mixing weights $\pi_k \geq 0$, $\sum_k \pi_k = 1$; means $\boldsymbol{\mu}_k$; covariances $\Sigma_k$.

**EM algorithm** alternates between:
- **E-step:** compute posterior responsibilities $r_{ik} = P(z_i = k \mid \mathbf{x}_i)$.
- **M-step:** update $\pi_k$, $\boldsymbol{\mu}_k$, $\Sigma_k$ as weighted sufficient statistics.

EM converges to a local maximum of the log-likelihood $\sum_i \log p(\mathbf{x}_i)$.

```python
from sklearn.mixture import GaussianMixture

gmm = GaussianMixture(n_components=3, covariance_type='full', random_state=42)
gmm.fit(X_train)

log_probs = gmm.score_samples(X_test)    # log p(x)
labels    = gmm.predict(X_test)           # most likely component
```

**Covariance types:**

| Type | Constraint | Parameters |
|---|---|---|
| `full` | Each component has its own $\Sigma_k$ | Most flexible |
| `tied` | All components share $\Sigma$ | Fewer parameters |
| `diag` | Diagonal $\Sigma_k$ (uncorrelated features) | Fast, scalable |
| `spherical` | $\Sigma_k = \sigma_k^2 I$ | Simplest |

## Inductive Bias

- **KDE**: makes no parametric assumption about shape; density is always positive; smooth with sufficient bandwidth.
- **GMM**: assumes the distribution is a finite mixture of Gaussians; unimodal within each component.

## Training Objective

- **KDE**: no training; density is computed directly from data.
- **GMM**: maximise log-likelihood $\sum_i \log p(\mathbf{x}_i)$ via EM; selects $K$ using BIC or AIC.

### Model Selection for GMM

```python
bic_scores = []
for k in range(1, 11):
    gmm = GaussianMixture(n_components=k, covariance_type='full', random_state=42)
    gmm.fit(X_train)
    bic_scores.append(gmm.bic(X_train))

optimal_k = 1 + bic_scores.index(min(bic_scores))
```

## Strengths

- KDE makes no parametric assumption and estimates any shape.
- GMM provides a fully probabilistic model with interpretable components.
- Both can be used for anomaly detection (low $\hat{p}$ → anomalous).

## Weaknesses

- KDE scales poorly with dimensionality (curse of dimensionality: bandwidth must grow exponentially with $d$).
- GMM requires specifying $K$ and assumes Gaussian components.
- EM can converge to poor local optima; use multiple random initialisations.

## Variants

- **Variational Bayes GMM** (`BayesianGaussianMixture`): places priors on mixture weights, automatically shrinks unused components — effectively selects $K$.
- **Normalising Flows**: learned invertible transformations that map a simple distribution (Gaussian) to a complex one, giving exact density.
- **Kernel Mixture Models**: KDE with learnable bandwidths per component.

## References

- Silverman, B.W. (1986). *Density Estimation for Statistics and Data Analysis*. Chapman & Hall.
- Bishop, C. (2006). *Pattern Recognition and Machine Learning*. §9 (EM and GMMs).

## Links

- [[03_modeling/02_unsupervised_learning/01_clustering/unsupervised_learning|Clustering — GMM as soft clustering; k-means as hard limit]]
- [[03_modeling/03_probabilistic_models/03_bayesian_modeling/probabilistic_models|Probabilistic Models — GMM, Naive Bayes]]
- [[01_foundations/03_probability_and_statistics/probability_distributions|Probability Distributions]]
- [[02_data_science/03_exploratory_data_analysis/exploratory_data_analysis|EDA — KDE for distribution visualisation]]
