---
layer: 03_modeling
type: workflow
status: growing
tags: [algorithm, training]
created: 2026-06-01
---

# Representation Learning — Implementation

## Conceptual Counterpart

[[03_modeling/02_unsupervised_learning/04_representation_learning/representation_learning|Representation Learning]] — autoencoders, VAE (ELBO, reparameterisation), contrastive learning (SimCLR, NT-Xent)

## Purpose

Practical implementation of self-supervised representation learning in PyTorch. Covers a shallow autoencoder (reconstruction), a Variational Autoencoder (VAE) with ELBO loss and the reparameterisation trick, and a SimCLR-style contrastive learning sketch using NT-Xent loss. Uses tabular data (scaled 2D Gaussian blobs) so examples run without GPU.

### Examples

#### Setup

```bash
pip install torch numpy matplotlib scikit-learn
```

```python
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader, TensorDataset
import numpy as np
import matplotlib.pyplot as plt
from sklearn.preprocessing import StandardScaler
from sklearn.datasets import make_blobs

# Synthetic data
X_np, y_np = make_blobs(n_samples=2000, centers=4, n_features=32, random_state=42)
X_np = StandardScaler().fit_transform(X_np).astype(np.float32)

X_tensor = torch.from_numpy(X_np)
dataset   = TensorDataset(X_tensor)
loader    = DataLoader(dataset, batch_size=128, shuffle=True)

INPUT_DIM  = X_np.shape[1]   # 32
LATENT_DIM = 8
DEVICE     = torch.device("cuda" if torch.cuda.is_available() else "cpu")
```

#### Shallow Autoencoder

```python
class Autoencoder(nn.Module):
    def __init__(self, input_dim: int, latent_dim: int):
        super().__init__()
        self.encoder = nn.Sequential(
            nn.Linear(input_dim, 64), nn.ReLU(),
            nn.Linear(64, latent_dim)
        )
        self.decoder = nn.Sequential(
            nn.Linear(latent_dim, 64), nn.ReLU(),
            nn.Linear(64, input_dim)
        )

    def forward(self, x: torch.Tensor):
        z = self.encoder(x)
        x_hat = self.decoder(z)
        return x_hat, z


ae = Autoencoder(INPUT_DIM, LATENT_DIM).to(DEVICE)
optimizer_ae = optim.Adam(ae.parameters(), lr=1e-3)
criterion_ae = nn.MSELoss()

# Training loop
for epoch in range(50):
    ae.train()
    epoch_loss = 0.0
    for (batch,) in loader:
        batch = batch.to(DEVICE)
        optimizer_ae.zero_grad()
        x_hat, _ = ae(batch)
        loss = criterion_ae(x_hat, batch)
        loss.backward()
        optimizer_ae.step()
        epoch_loss += loss.item()
    if (epoch + 1) % 10 == 0:
        print(f"AE Epoch {epoch+1:3d} | Loss: {epoch_loss / len(loader):.4f}")

# Extract latent representations
ae.eval()
with torch.no_grad():
    _, Z_ae = ae(X_tensor.to(DEVICE))
Z_ae = Z_ae.cpu().numpy()

# Anomaly detection: flag points with high reconstruction error
ae.eval()
with torch.no_grad():
    X_hat, _ = ae(X_tensor.to(DEVICE))
recon_errors = ((X_tensor.to(DEVICE) - X_hat) ** 2).mean(dim=1).cpu().numpy()
threshold = np.percentile(recon_errors, 95)
anomalies = recon_errors > threshold
print(f"Autoencoder anomalies (top 5% recon error): {anomalies.sum()}")
```

#### Variational Autoencoder (VAE)

The VAE encoder outputs `mu` and `log_var` (log variance) rather than a point estimate of `z`. The **reparameterisation trick** enables gradients to flow through the sampling step: $\mathbf{z} = \boldsymbol{\mu} + \boldsymbol{\epsilon} \cdot \exp(\frac{1}{2}\log\boldsymbol{\sigma}^2)$, $\boldsymbol{\epsilon} \sim \mathcal{N}(\mathbf{0}, I)$.

**ELBO loss:**
$$
\mathcal{L} = \underbrace{\|x - \hat{x}\|^2}_{\text{reconstruction}} + \underbrace{\frac{1}{2}\sum_j(1 + \log\sigma_j^2 - \mu_j^2 - \sigma_j^2)}_{\text{KL term (negative)}}
$$

The KL term regularises the posterior toward $\mathcal{N}(\mathbf{0}, I)$.

```python
class VAE(nn.Module):
    def __init__(self, input_dim: int, latent_dim: int):
        super().__init__()
        self.encoder_shared = nn.Sequential(
            nn.Linear(input_dim, 64), nn.ReLU()
        )
        self.fc_mu      = nn.Linear(64, latent_dim)   # mean
        self.fc_log_var = nn.Linear(64, latent_dim)   # log variance

        self.decoder = nn.Sequential(
            nn.Linear(latent_dim, 64), nn.ReLU(),
            nn.Linear(64, input_dim)
        )

    def encode(self, x: torch.Tensor):
        h = self.encoder_shared(x)
        return self.fc_mu(h), self.fc_log_var(h)

    def reparameterise(self, mu: torch.Tensor, log_var: torch.Tensor) -> torch.Tensor:
        """z = mu + eps * std,  eps ~ N(0, I)"""
        if self.training:
            std = torch.exp(0.5 * log_var)
            eps = torch.randn_like(std)
            return mu + eps * std
        return mu   # use mean at inference time

    def decode(self, z: torch.Tensor) -> torch.Tensor:
        return self.decoder(z)

    def forward(self, x: torch.Tensor):
        mu, log_var = self.encode(x)
        z    = self.reparameterise(mu, log_var)
        x_hat = self.decode(z)
        return x_hat, mu, log_var


def elbo_loss(x: torch.Tensor, x_hat: torch.Tensor,
              mu: torch.Tensor, log_var: torch.Tensor,
              beta: float = 1.0) -> torch.Tensor:
    """
    ELBO = reconstruction_loss + beta * KL divergence
    KL(q(z|x) || p(z)) = -0.5 * sum(1 + log_var - mu^2 - exp(log_var))
    """
    recon_loss = nn.functional.mse_loss(x_hat, x, reduction='sum') / x.size(0)
    kl_loss    = -0.5 * torch.sum(1 + log_var - mu.pow(2) - log_var.exp()) / x.size(0)
    return recon_loss + beta * kl_loss


vae = VAE(INPUT_DIM, LATENT_DIM).to(DEVICE)
optimizer_vae = optim.Adam(vae.parameters(), lr=1e-3)

for epoch in range(80):
    vae.train()
    epoch_loss = 0.0
    for (batch,) in loader:
        batch = batch.to(DEVICE)
        optimizer_vae.zero_grad()
        x_hat, mu, log_var = vae(batch)
        loss = elbo_loss(batch, x_hat, mu, log_var, beta=1.0)
        loss.backward()
        optimizer_vae.step()
        epoch_loss += loss.item()
    if (epoch + 1) % 20 == 0:
        print(f"VAE Epoch {epoch+1:3d} | ELBO: {epoch_loss / len(loader):.4f}")

# Sample new points from the prior N(0, I)
vae.eval()
with torch.no_grad():
    z_sample = torch.randn(16, LATENT_DIM, device=DEVICE)
    X_generated = vae.decode(z_sample).cpu().numpy()
print(f"Generated samples shape: {X_generated.shape}")

# Extract deterministic latent representations (use mu, not sample)
with torch.no_grad():
    mu_all, _ = vae.encode(X_tensor.to(DEVICE))
Z_vae = mu_all.cpu().numpy()
```

**beta-VAE:** set `beta > 1` (e.g., 4.0) to increase disentanglement pressure at the cost of reconstruction quality.

#### Contrastive Learning — SimCLR-style NT-Xent Loss

SimCLR creates two augmented views of each sample and trains an encoder so that views of the same sample are close in latent space while views of different samples are pushed apart.

```python
def simclr_augment(x: torch.Tensor, noise_std: float = 0.1) -> torch.Tensor:
    """Minimal tabular augmentation: Gaussian noise + random feature dropout."""
    noise  = torch.randn_like(x) * noise_std
    dropout_mask = (torch.rand_like(x) > 0.1).float()   # 10% feature dropout
    return (x + noise) * dropout_mask


class ProjectionHead(nn.Module):
    """MLP projection head: maps encoder output to contrastive space."""
    def __init__(self, input_dim: int, proj_dim: int = 32):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(input_dim, input_dim), nn.ReLU(),
            nn.Linear(input_dim, proj_dim)
        )

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        return nn.functional.normalize(self.net(x), dim=1)


class SimCLREncoder(nn.Module):
    def __init__(self, input_dim: int, latent_dim: int, proj_dim: int = 32):
        super().__init__()
        self.backbone = nn.Sequential(
            nn.Linear(input_dim, 64), nn.ReLU(),
            nn.Linear(64, latent_dim), nn.ReLU()
        )
        self.proj_head = ProjectionHead(latent_dim, proj_dim)

    def forward(self, x: torch.Tensor):
        h = self.backbone(x)
        z = self.proj_head(h)
        return h, z   # h for downstream tasks, z for contrastive loss


def nt_xent_loss(z1: torch.Tensor, z2: torch.Tensor,
                 temperature: float = 0.5) -> torch.Tensor:
    """
    NT-Xent (Normalised Temperature-scaled Cross Entropy) loss.
    z1, z2: (N, proj_dim) L2-normalised projections of two views of the same batch.
    For each sample i, the positive pair is (z1_i, z2_i).
    All 2N-2 other pairs are negatives.
    """
    N = z1.size(0)
    z = torch.cat([z1, z2], dim=0)               # (2N, proj_dim)
    sim = torch.mm(z, z.T) / temperature          # (2N, 2N) cosine similarities
    # Mask out self-similarity diagonal
    mask = torch.eye(2 * N, device=z.device).bool()
    sim.masked_fill_(mask, float('-inf'))
    # Positive pairs: (i, i+N) and (i+N, i)
    labels = torch.cat([torch.arange(N, 2 * N), torch.arange(N)]).to(z.device)
    loss = nn.functional.cross_entropy(sim, labels)
    return loss


simclr = SimCLREncoder(INPUT_DIM, LATENT_DIM, proj_dim=32).to(DEVICE)
optimizer_cl = optim.Adam(simclr.parameters(), lr=1e-3, weight_decay=1e-4)

for epoch in range(60):
    simclr.train()
    epoch_loss = 0.0
    for (batch,) in loader:
        batch = batch.to(DEVICE)
        x1, x2 = simclr_augment(batch), simclr_augment(batch)
        _, z1 = simclr(x1)
        _, z2 = simclr(x2)
        loss = nt_xent_loss(z1, z2, temperature=0.5)
        optimizer_cl.zero_grad()
        loss.backward()
        optimizer_cl.step()
        epoch_loss += loss.item()
    if (epoch + 1) % 20 == 0:
        print(f"SimCLR Epoch {epoch+1:3d} | NT-Xent: {epoch_loss / len(loader):.4f}")

# Extract representations for downstream use (use backbone h, not projection z)
simclr.eval()
with torch.no_grad():
    Z_cl, _ = simclr(X_tensor.to(DEVICE))
Z_cl = Z_cl.cpu().numpy()
print(f"SimCLR representations shape: {Z_cl.shape}")
```

**Important:** use the **backbone output** `h` for downstream tasks, not the projection head output `z`. The projection head is trained to satisfy the contrastive objective and may discard task-relevant information.

## Architecture

```
Input x (n_samples, input_dim)
  │
  ├── Autoencoder
  │     Encoder: x → z  (bottleneck compression)
  │     Decoder: z → x̂  (reconstruction)
  │     Loss: MSE(x, x̂)
  │     Use: compression, denoising, anomaly detection (high recon error)
  │
  ├── VAE
  │     Encoder: x → (μ, log σ²)
  │     Reparameterise: z = μ + ε·σ,  ε ~ N(0,I)
  │     Decoder: z → x̂
  │     Loss: MSE(x, x̂) + β·KL(N(μ,σ²) || N(0,I))
  │     Use: generation (sample z ~ N(0,I)), latent interpolation
  │
  └── SimCLR (contrastive)
        Augment: x → x₁, x₂  (two views)
        Encoder + projection head: xᵢ → zᵢ (L2-normalised)
        Loss: NT-Xent (positive pairs close, negatives far)
        Use: pre-training; fine-tune backbone on downstream task
```

**When to use each:**

| Method | Objective | Use case |
|---|---|---|
| Autoencoder | Reconstruction (MSE) | Compression, denoising, anomaly detection |
| VAE | ELBO (reconstruction + KL) | Generation, latent space interpolation, disentanglement |
| Contrastive (SimCLR) | NT-Xent on augmented pairs | Pre-training transferable representations with no labels |

## References

- Kingma, D.P. & Welling, M. (2014). "Auto-Encoding Variational Bayes." *ICLR 2014*. arXiv:1312.6114.
- Chen, T. et al. (2020). "A Simple Framework for Contrastive Learning of Visual Representations." *ICML 2020*. arXiv:2002.05709.
- Higgins, I. et al. (2017). "beta-VAE: Learning Basic Visual Concepts with a Constrained Variational Framework." *ICLR 2017*.

## Links

- [[03_modeling/02_unsupervised_learning/04_representation_learning/index|Representation Learning Sublayer Index]]
- [[03_modeling/02_unsupervised_learning/04_representation_learning/representation_learning|Representation Learning (theory)]]
- [[01_foundations/06_deep_learning_theory/backpropagation|Backpropagation — gradient flow through encoder/decoder]]
- [[03_modeling/04_deep_learning/01_mlp_and_representation_learning/index|MLP and Representation Learning]]
- [[03_modeling/02_unsupervised_learning/03_density_estimation/density_estimation_implementation|Density Estimation — Implementation (anomaly detection comparison)]]
