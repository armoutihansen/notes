---
layer: 01_foundations
type: concept
status: growing
tags: [algorithm, training]
created: 2026-03-06
---

# Layer Normalization

## Definition

**Layer Normalization (LN)** normalises each sample's activations across the feature dimension. For an activation vector $\mathbf{x} \in \mathbb{R}^d$ within a single sample:

$$\text{LN}(\mathbf{x}) = \gamma \odot \frac{\mathbf{x} - \mu}{\sqrt{\sigma^2 + \epsilon}} + \beta$$

where $\mu = \frac{1}{d}\sum_{i=1}^d x_i$ is the mean, $\sigma^2 = \frac{1}{d}\sum_{i=1}^d (x_i - \mu)^2$ is the variance, $\gamma, \beta \in \mathbb{R}^d$ are learned scale and shift parameters, and $\epsilon$ is a small constant for numerical stability.

## Intuition

LN stabilises training by ensuring each sample's feature vector has approximately zero mean and unit variance before entering the next layer. Unlike Batch Normalization, LN normalises **within a single sample** over features — so it is unaffected by batch size and can be applied to sequences of varying length.

This makes LN the natural choice for transformers and RNNs, where batch statistics are unreliable (small batches, variable-length sequences, autoregressive generation with batch size 1).

## Formal Description

**Comparison with Batch Normalization:**

| Property | Batch Norm | Layer Norm |
|---|---|---|
| Normalisation axis | Batch dimension (per feature) | Feature dimension (per sample) |
| Batch size dependence | Yes — breaks with small batches | No |
| Sequence/variable length | Problematic | Works naturally |
| Primary use | CNNs, MLPs, ResNets | Transformers, RNNs |
| Train/test discrepancy | Yes (uses running stats at test) | No |

**Pre-LN vs Post-LN:**

- **Post-LN** (original Transformer, "Add & Norm"): $\text{LN}(\mathbf{x} + F(\mathbf{x}))$. Residual is added, then normalised. Training can be unstable without warmup.
- **Pre-LN**: $\mathbf{x} + F(\text{LN}(\mathbf{x}))$. LN is applied before the sub-layer. Empirically more stable; used in GPT-2 onward.

Modern large language models (GPT-3, LLaMA, Mistral) typically use **Pre-LN** with **RMSNorm**.

**RMSNorm (Root Mean Square Layer Normalization):** simplifies LN by removing the mean-centering step:

$$\text{RMSNorm}(\mathbf{x}) = \gamma \odot \frac{\mathbf{x}}{\text{RMS}(\mathbf{x}) + \epsilon}, \qquad \text{RMS}(\mathbf{x}) = \sqrt{\frac{1}{d}\sum_{i=1}^d x_i^2}$$

Computationally cheaper (no mean computation), and empirically matches LN on most benchmarks. Used in LLaMA, Gemma, Mistral.

**Transformer block with Pre-LN:**

```
y = x + Attention(LN(x))
z = y + FFN(LN(y))
```

Each sub-layer receives normalised input, improving gradient flow through deep stacks.

**Gradient flow argument:** without normalisation, activations can grow or shrink exponentially with depth. LN bounds activation norms to approximately 1, keeping gradients well-scaled throughout.

**Learnable parameters:** each LN layer has $2d$ trainable parameters ($\gamma$ and $\beta$). At initialisation: $\gamma = \mathbf{1}$, $\beta = \mathbf{0}$ (identity map).

## Applications

| Model family | Normalization used |
|---|---|
| Original Transformer | Post-LN |
| GPT-2, GPT-3 | Pre-LN |
| LLaMA, Mistral, Gemma | Pre-RMSNorm |
| BERT | Post-LN |
| Vision Transformer (ViT) | Pre-LN |
| LSTM (modern) | LN on gates |

## Trade-offs

- LN has higher per-step cost than BN (cannot cache running statistics) but eliminates batch size constraints entirely.
- Pre-LN is more stable than Post-LN for very deep models but slightly changes the representation (residual path has no normalisation).
- For very large $d$ (e.g., $d = 4096$ in LLaMA), RMSNorm is a meaningful speed improvement over full LN.

## Links

- [[batch_normalization|Batch Normalization]] — normalises over batch; complementary to LN
- [[dropout|Dropout]] — alternative / complementary regularisation
- [[residual_connections|Residual Connections]] — LN always used in conjunction with residuals in transformers
- [[01_foundations/06_deep_learning_theory/weight_initialization|Weight Initialization]] — normalisation reduces sensitivity to init scale
- [[05_ai_engineering/00_foundation_models/foundation_model_overview|Foundation Model Overview]] — LN / RMSNorm are standard in all modern LLMs
