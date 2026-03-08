---
layer: 03_modeling
type: concept
status: seed
tags: [algorithm, theory]
created: 2026-05-31
---

# Representation Learning

## Core Idea

Representation learning discovers compact, informative encodings of raw data that capture structure useful for downstream tasks. Rather than hand-engineering features, the model learns what to extract. Autoencoders do this through reconstruction; contrastive methods do it by pulling similar examples together and pushing dissimilar examples apart.

## Mathematical Formulation

### Autoencoder

An autoencoder comprises an **encoder** $f_\phi: \mathbb{R}^d \to \mathbb{R}^k$ ($k \ll d$) and a **decoder** $g_\theta: \mathbb{R}^k \to \mathbb{R}^d$, trained to minimise reconstruction error:

$$
\mathcal{L} = \frac{1}{n}\sum_{i=1}^n \|g_\theta(f_\phi(\mathbf{x}_i)) - \mathbf{x}_i\|^2
$$

The bottleneck layer $\mathbf{z} = f_\phi(\mathbf{x}) \in \mathbb{R}^k$ is the **latent representation**.

```python
import torch
import torch.nn as nn

class Autoencoder(nn.Module):
    def __init__(self, input_dim, latent_dim):
        super().__init__()
        self.encoder = nn.Sequential(
            nn.Linear(input_dim, 256), nn.ReLU(),
            nn.Linear(256, latent_dim)
        )
        self.decoder = nn.Sequential(
            nn.Linear(latent_dim, 256), nn.ReLU(),
            nn.Linear(256, input_dim)
        )

    def forward(self, x):
        z = self.encoder(x)
        return self.decoder(z), z
```

### Variational Autoencoder (VAE)

The VAE places a prior $p(\mathbf{z}) = \mathcal{N}(\mathbf{0}, I)$ on the latent space and trains an approximate posterior $q_\phi(\mathbf{z}|\mathbf{x}) = \mathcal{N}(\boldsymbol{\mu}_\phi(\mathbf{x}), \text{diag}(\boldsymbol{\sigma}^2_\phi(\mathbf{x})))$.

**Evidence Lower BOund (ELBO):**

$$
\mathcal{L}_{\text{VAE}} = \underbrace{\mathbb{E}_{q_\phi(\mathbf{z}|\mathbf{x})}[\log p_\theta(\mathbf{x}|\mathbf{z})]}_{\text{reconstruction}} - \underbrace{D_{\text{KL}}(q_\phi(\mathbf{z}|\mathbf{x}) \| p(\mathbf{z}))}_{\text{regularisation}}
$$

The KL term forces the posterior to match the prior, regularising the latent space. Reparameterisation trick enables backprop through the sampling step: $\mathbf{z} = \boldsymbol{\mu} + \boldsymbol{\sigma} \odot \boldsymbol{\epsilon}$, $\boldsymbol{\epsilon} \sim \mathcal{N}(\mathbf{0}, I)$.

### Contrastive Learning

Learns representations such that similar (positive) pairs have nearby embeddings and dissimilar (negative) pairs have distant embeddings.

**InfoNCE / NT-Xent loss** (SimCLR):

$$
\mathcal{L} = -\log \frac{\exp(\text{sim}(\mathbf{z}_i, \mathbf{z}_j) / \tau)}{\sum_{k \neq i} \exp(\text{sim}(\mathbf{z}_i, \mathbf{z}_k) / \tau)}
$$

where $\text{sim}(\mathbf{u}, \mathbf{v}) = \mathbf{u}^\top \mathbf{v} / (\|\mathbf{u}\|\|\mathbf{v}\|)$ and $\tau$ is a temperature parameter.

Positive pairs are created via data augmentation (crop, flip, colour jitter for images; token masking for text).

## Inductive Bias

- **Autoencoder**: assumes the data lies on a low-dimensional manifold; all information must pass through the bottleneck.
- **VAE**: assumes a smooth, continuous latent space with a Gaussian prior; supports generation via sampling $\mathbf{z} \sim \mathcal{N}(\mathbf{0}, I)$.
- **Contrastive**: assumes that augmented views of the same instance are semantically equivalent; requires careful negative mining.

## Training Objective

| Method | Objective |
|---|---|
| Autoencoder | Minimise reconstruction loss (MSE or BCE) |
| VAE | Maximise ELBO = reconstruction − KL divergence |
| Contrastive (SimCLR) | Maximise agreement between augmented views |

## Strengths

- Pre-trained representations transfer to downstream tasks with little labelled data.
- Autoencoders provide interpretable compression and can detect anomalies via high reconstruction error.
- VAEs enable controllable generation and latent-space interpolation.

## Weaknesses

- Representations may not capture task-relevant structure if the reconstruction/contrastive objective is misaligned with the downstream task.
- VAEs often produce blurry reconstructions (blurred by the expected reconstruction objective).
- Contrastive learning requires large batch sizes or memory banks for sufficient negatives.

## Variants

- **Denoising autoencoder**: corrupts inputs before encoding; forces the encoder to learn robust representations.
- **Sparse autoencoder**: adds L1 penalty on activations; encourages sparse, disentangled representations.
- **β-VAE**: scales the KL term by $\beta > 1$; encourages disentanglement.
- **BYOL / SimSiam**: contrastive-free methods that avoid the need for negative pairs.

## References

- Kingma, D.P. & Welling, M. (2014). "Auto-Encoding Variational Bayes." ICLR.
- Chen, T. et al. (2020). "A Simple Framework for Contrastive Learning of Visual Representations." ICML.
- Bengio, Y. et al. (2013). "Representation Learning: A Review and New Perspectives." *IEEE TPAMI*.

## Links

- [[03_modeling/02_unsupervised_learning/03_density_estimation/density_estimation|Density Estimation — VAE as generative model]]
- [[03_modeling/03_probabilistic_models/02_latent_variable_models/latent_variable_models|Latent Variable Models — VAE connection to EM and HMMs]]
- [[03_modeling/04_deep_learning/01_mlp_and_representation_learning/mpl|MLP — encoder and decoder are MLPs]]
- [[01_foundations/03_probability_and_statistics/probability_theory|Probability Theory — ELBO, KL divergence]]
