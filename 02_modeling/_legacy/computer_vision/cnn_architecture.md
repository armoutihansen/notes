---
layer: 02_modeling
type: concept
status: seed
tags: [cnn, computer_vision, resnet, architecture]
created: 2026-03-02
---

# CNN Architecture

## Definition

A feedforward architecture for processing grid-structured data (images, time-frequency spectrograms) using stacked convolution, pooling, and fully connected layers; with residual connections to enable very deep networks.

## Intuition

Early layers learn low-level features (edges, colors); middle layers combine them into textures and parts; late layers form semantic concepts. Residual connections allow gradients to flow directly to early layers, making depth tractable.

## Formal Description

**Canonical structure:** INPUT → [CONV → BN → ReLU]* → POOL → ... → FC → OUTPUT

**Feature map dimensions:** as depth increases, $H \times W$ decreases (via pooling/stride) while $C$ increases; a typical progression:

$$224 \times 224 \times 3 \;\to\; 112 \times 112 \times 64 \;\to\; \cdots \;\to\; 7 \times 7 \times 512 \;\to\; 4096 \;\to\; K$$

**Why convolutions work:**
- **Parameter sharing:** the same filter detects the same pattern everywhere → massive parameter reduction vs. FC
- **Sparse connections:** each output unit depends only on a local receptive field
- **Translation equivariance:** convolving then shifting = shifting then convolving

**Residual (skip) connections (ResNet):**

$$y = F(x, \{W_i\}) + x$$

where $F$ is a stack of convolutions. If $F \approx 0$, the block reduces to identity — making it easy for optimization to preserve input. Enables training networks with 50–1000+ layers without vanishing gradients.

**Projection shortcut:** when dimensions change ($x$ and $F(x)$ have different shapes), use $1 \times 1$ convolution on the skip connection.

## Applications

Image classification (ResNet, VGG, EfficientNet), object detection backbones, feature extraction for downstream tasks.

## Trade-offs

- Deep CNNs require significant compute and memory
- Fixed-size receptive fields miss very long-range dependencies (Vision Transformers address this)
- ResNets still dominate many production vision tasks despite ViT advances

## Links

- [[convolution]]
- [[object_detection]]
- [[01_foundations/_legacy/deep_learning_theory 1/weight_initialization]] (in 01_foundations/deep_learning_theory)
