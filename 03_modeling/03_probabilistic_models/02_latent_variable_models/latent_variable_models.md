---
layer: 03_modeling
type: concept
status: seed
tags: [algorithm, theory]
created: 2026-05-31
---

# Latent Variable Models

## Core Idea

Latent variable models introduce unobserved (hidden) variables $\mathbf{z}$ to explain structure in the observed data $\mathbf{x}$. The joint model $p(\mathbf{x}, \mathbf{z} \mid \theta)$ is richer than any direct model of $\mathbf{x}$ alone, and marginalising over $\mathbf{z}$ gives a flexible marginal $p(\mathbf{x} \mid \theta) = \int p(\mathbf{x} \mid \mathbf{z}, \theta)\, p(\mathbf{z} \mid \theta)\, d\mathbf{z}$.

The core algorithmic challenge is **inference**: computing the posterior $p(\mathbf{z} \mid \mathbf{x}, \theta)$, which is generally intractable and requires approximation via EM, variational inference, or MCMC.

## Mathematical Formulation

### The EM Algorithm (General Form)

**Goal:** maximise $\log p(\mathbf{x} \mid \theta) = \log \int p(\mathbf{x}, \mathbf{z} \mid \theta)\, d\mathbf{z}$.

**EM iterates:**
- **E-step:** compute expected complete-data log-likelihood:
  $$
  Q(\theta \mid \theta^{(t)}) = \mathbb{E}_{\mathbf{z} \mid \mathbf{x}, \theta^{(t)}}[\log p(\mathbf{x}, \mathbf{z} \mid \theta)]
  $$
- **M-step:** update parameters:
  $$
  \theta^{(t+1)} = \arg\max_\theta Q(\theta \mid \theta^{(t)})
  $$

EM is guaranteed to monotonically increase $\log p(\mathbf{x} \mid \theta)$ at each iteration and converges to a local maximum.

### Hidden Markov Model (HMM)

An HMM models a sequence $\mathbf{x}_{1:T} = (x_1, \ldots, x_T)$ by positing a hidden Markov chain $z_{1:T}$:

$$
p(\mathbf{x}, \mathbf{z}) = p(z_1) \prod_{t=2}^T p(z_t \mid z_{t-1}) \prod_{t=1}^T p(x_t \mid z_t)
$$

**Parameters:**
- **Initial distribution** $\pi_k = P(z_1 = k)$
- **Transition matrix** $A_{jk} = P(z_t = k \mid z_{t-1} = j)$
- **Emission distribution** $p(x_t \mid z_t = k)$ (Gaussian for continuous observations)

**Inference algorithms:**
- **Forward-backward** (Baum-Welch): computes $P(z_t \mid \mathbf{x}_{1:T})$ — marginal posterior at each step; used in EM.
- **Viterbi**: finds the MAP sequence $\arg\max_{\mathbf{z}} P(\mathbf{z} \mid \mathbf{x})$ — the most probable hidden state path.

**Applications:** speech recognition (phoneme states), gene finding (exon/intron states), regime detection in time series.

### Connection to VAE

The Variational Autoencoder (VAE) is a latent variable model with:
- Prior $p(\mathbf{z}) = \mathcal{N}(\mathbf{0}, I)$
- Likelihood $p_\theta(\mathbf{x} \mid \mathbf{z})$ = decoder neural network
- Approximate posterior $q_\phi(\mathbf{z} \mid \mathbf{x})$ = encoder neural network

EM is intractable here because the posterior is complex; instead, variational inference maximises the ELBO (see [[03_modeling/02_unsupervised_learning/04_representation_learning/representation_learning|Representation Learning]]).

## Inductive Bias

- Latent variables encode structured uncertainty: the model commits to a prior over what is unobserved and infers its value from data.
- HMMs assume Markov property: the future is conditionally independent of the past given the current hidden state.

## Training Objective

Maximise marginal log-likelihood $\sum_i \log p(\mathbf{x}_i \mid \theta)$ via:
- Exact EM for finite discrete $\mathbf{z}$ (GMMs, HMMs)
- Variational EM / ELBO maximisation for continuous or complex $\mathbf{z}$ (VAE)

## Strengths

- Principled handling of uncertainty and missing observations.
- HMMs model temporal dynamics with interpretable state structure.
- Can separate generative factors into latent dimensions.

## Weaknesses

- EM converges to local optima; sensitive to initialisation.
- HMM inference is $O(K^2 T)$ — expensive for many states or long sequences.
- Choosing the number of hidden states / latent dimensions requires model selection.

## Variants

- **Factorial HMM**: multiple independent Markov chains; exponential state space.
- **Input-Output HMM (IO-HMM)**: transitions conditioned on inputs.
- **Deep latent Gaussian models**: multi-layer latent variable models.
- **Normalising flows**: exact inference via invertible transformations.

## References

- Rabiner, L.R. (1989). "A tutorial on hidden Markov models and selected applications in speech recognition." *Proc. IEEE*.
- Dempster, A.P., Laird, N.M. & Rubin, D.B. (1977). "Maximum likelihood from incomplete data via the EM algorithm." *JRSS-B*.
- Kingma, D.P. & Welling, M. (2014). "Auto-Encoding Variational Bayes." ICLR.

## Links

- [[03_modeling/03_probabilistic_models/01_graphical_models/graphical_models|Graphical Models — HMM as a dynamic Bayesian network]]
- [[03_modeling/02_unsupervised_learning/03_density_estimation/density_estimation|Density Estimation — GMM / EM]]
- [[03_modeling/02_unsupervised_learning/04_representation_learning/representation_learning|Representation Learning — VAE]]
- [[01_foundations/03_probability_and_statistics/probability_theory|Probability Theory — marginalisation, Bayes theorem]]
