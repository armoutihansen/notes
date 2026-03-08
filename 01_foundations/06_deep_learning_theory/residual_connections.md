---
layer: 01_foundations
type: concept
status: growing
tags: [algorithm]
created: 2026-03-06
---

# Residual Connections

## Definition

A **residual connection** (skip connection) adds the input of a sub-layer directly to its output:

$$
\mathbf{y} = F(\mathbf{x}) + \mathbf{x}
$$

where $F(\mathbf{x})$ is the sub-layer transformation (e.g., a convolutional block or a transformer FFN) and $\mathbf{x}$ is the identity shortcut. The block learns the **residual** $F(\mathbf{x}) = \mathbf{y} - \mathbf{x}$ rather than a direct mapping.

## Intuition

Training very deep networks ($>20$ layers) with plain stacked layers was empirically nearly impossible before residual connections — both gradients and activations degraded. The residual shortcut creates a **gradient highway**: during backpropagation, gradient can flow directly through the identity path without passing through any non-linearities. This solves the vanishing gradient problem for depth.

The "residual" framing is also conceptually appealing: if the identity is already a good approximation, $F(\mathbf{x})$ only needs to learn small corrections, which is easier than learning the full mapping from scratch.

## Formal Description

**ResNet block (He et al., 2016):**

$$
\mathbf{y} = F(\mathbf{x}; W_1, W_2) + \mathbf{x}
$$

$$
F(\mathbf{x}) = W_2\, \text{ReLU}(W_1\, \mathbf{x})
$$

When dimensions don't match (different channel/feature sizes), a projection shortcut $\mathbf{x}' = W_s \mathbf{x}$ is used:

$$
\mathbf{y} = F(\mathbf{x}) + W_s \mathbf{x}
$$

**Gradient flow analysis:** backpropagation through a residual block:

$$
\frac{\partial \mathcal{L}}{\partial \mathbf{x}} = \frac{\partial \mathcal{L}}{\partial \mathbf{y}} \cdot \frac{\partial \mathbf{y}}{\partial \mathbf{x}} = \frac{\partial \mathcal{L}}{\partial \mathbf{y}} \left(1 + \frac{\partial F}{\partial \mathbf{x}}\right)
$$

The "1" term means gradient flows to layer $l$ directly from any deeper layer $L$ without multiplicative shrinkage:

$$
\frac{\partial \mathcal{L}}{\partial \mathbf{x}_l} = \frac{\partial \mathcal{L}}{\partial \mathbf{x}_L} \prod_{i=l}^{L-1}\left(1 + \frac{\partial F_i}{\partial \mathbf{x}_i}\right)
$$

The product can no longer vanish as long as the individual $\partial F_i / \partial \mathbf{x}_i$ terms are small.

**Ensemble interpretation (Veit et al., 2016):** unrolling a residual network with $L$ blocks produces $2^L$ paths through the network. The output is effectively an ensemble of networks of varying depth. Most of the gradient flows through shorter paths, making the network robust to depth.

**Transformer residual structure:**

```
# Pre-LN transformer block
y = x + Attention(LN(x))
z = y + FFN(LN(y))
```

Every transformer block has two residual connections: one around the attention sub-layer and one around the FFN. This is why transformers can scale to hundreds of layers.

**Dense connections (DenseNet):** every layer receives input from all previous layers:

$$
\mathbf{x}_l = H_l([\mathbf{x}_0, \mathbf{x}_1, \ldots, \mathbf{x}_{l-1}])
$$

More extreme form of skip connections; useful for tasks requiring multi-scale features (segmentation, detection).

**Highway networks:** learned gating of the skip connection:

$$
\mathbf{y} = T(\mathbf{x}) \cdot F(\mathbf{x}) + (1 - T(\mathbf{x})) \cdot \mathbf{x}
$$

where $T(\mathbf{x}) = \sigma(W_T \mathbf{x} + b_T)$ is a trainable gate. Precursor to ResNets; less popular because ungated residuals work equally well.

## Applications

| Architecture | Residual variant |
|---|---|
| ResNet (image classification) | Standard $F(\mathbf{x}) + \mathbf{x}$ blocks |
| Transformer (all LLMs) | Two residuals per block (attn + FFN) |
| U-Net (segmentation) | Skip connections between encoder/decoder |
| DenseNet | Dense skip connections across all layers |
| EfficientNet | MBConv blocks with residuals |

## Trade-offs

- Residual connections add essentially zero parameter cost (the identity shortcut has no parameters unless a projection is needed).
- Memory cost: during backpropagation, activations at each residual junction must be stored. Gradient checkpointing trades memory for recompute.
- For very shallow networks (2–3 layers), residuals provide minimal benefit.
- Pre-LN placement combined with residuals is currently the most stable configuration for large language models.

## Links

- [[layer_normalization|Layer Normalization]] — always paired with residuals in transformers (Pre-LN or Post-LN)
- [[batch_normalization|Batch Normalization]] — paired with residuals in ResNets (BN→ReLU→Conv→BN→ReLU→Conv + shortcut)
- [[backpropagation|Backpropagation]] — residuals fundamentally change the gradient flow computation graph
- [[01_foundations/06_deep_learning_theory/weight_initialization|Weight Initialization]] — initialise $F$ close to zero so residual block starts as near-identity
- [[activation_functions|Activation Functions]] — GELU/ReLU used inside residual blocks
