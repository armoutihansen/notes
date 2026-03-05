---
layer: 02_modeling
type: concept
status: seed
tags: [cnn, computer_vision, convolution, pooling]
created: 2026-03-02
---

# Convolution, Padding, Stride, and Pooling

## Definition

The core spatial operations in CNNs — convolution applies a learnable filter over local receptive fields; padding preserves spatial dimensions; stride controls the step size; pooling downsamples feature maps.

## Intuition

Convolution detects local patterns (edges, textures) using shared weights, making the operation translation-equivariant and vastly more parameter-efficient than fully connected layers. Pooling introduces translation invariance and reduces computation. Many deep learning libraries implement cross-correlation (no kernel flip), though the term "convolution" is used throughout.

## Formal Description

**Convolution (cross-correlation in practice):** output element $(i,j)$ is computed as:

$$\sum_{r=0}^{f-1}\sum_{c=0}^{f-1} W_{rc} \cdot X_{i\cdot s+r,\, j\cdot s+c}$$

Filter $W \in \mathbb{R}^{f \times f \times C_\text{in}}$ is shared across all spatial positions.

**Output size:**

$$n_\text{out} = \left\lfloor\frac{n + 2p - f}{s}\right\rfloor + 1$$

where $n$ = input size, $p$ = padding, $f$ = filter size, $s$ = stride.

**Padding modes:**
- "Valid" convolution: $p=0$, output shrinks
- "Same" convolution: $p=(f-1)/2$, output same size as input (for stride 1)

**Multiple output channels:** $C_\text{out}$ filters each producing one output channel; total parameters = $f \times f \times C_\text{in} \times C_\text{out} + C_\text{out}$ (with biases).

**3D input (color images):** $X \in \mathbb{R}^{H \times W \times C_\text{in}}$, filter $\in \mathbb{R}^{f \times f \times C_\text{in}}$.

**Pooling:** max pooling takes the maximum over an $f \times f$ window; average pooling takes the mean; typically $f=2, s=2$ halves spatial dimensions; no learnable parameters.

## Applications

All CNN-based vision models; feature extraction for images. Modern variants include depthwise separable convolution (MobileNet), dilated convolution, and transposed convolution (upsampling).

## Trade-offs

- Parameter sharing assumes translation invariance — fails for tasks with position-specific patterns
- Pooling loses spatial precision (problematic for detection/segmentation, which use feature pyramids instead)
- Max pooling is not differentiable at ties (but in practice this is ignored)

## Links

- [[cnn_architecture]]
- [[01_foundations/_legacy/deep_learning_theory 1/backpropagation]] (in 01_foundations/deep_learning_theory)
