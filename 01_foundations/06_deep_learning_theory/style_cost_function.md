---
layer: 01_foundations
type: concept
status: seed
tags: [algorithm, vision, training]
created: 2026-03-02
---

# Style Cost Function

## Definition

A loss that measures the stylistic difference between two images by comparing correlations between feature map channels (Gram matrices) at multiple layers of a pretrained CNN. Used in neural style transfer to optimize pixel values so a generated image matches the style of a reference image.

## Intuition

Style is captured by co-activation patterns across channels — which texture patterns tend to occur together — not by exact spatial positions. The Gram matrix computes these second-order channel statistics by removing spatial structure. Two images share the same style when their channel correlations match, regardless of where those textures appear.

## Formal Description

**Feature maps at layer $l$:** reshape to $A^{[l]} \in \mathbb{R}^{n_C^{[l]} \times (n_H^{[l]} \cdot n_W^{[l]})}$ (channels × flattened spatial positions).

**Gram matrix:** 

$$G^{[l]} = A^{[l]}\left(A^{[l]}\right)^\top \in \mathbb{R}^{n_C^{[l]} \times n_C^{[l]}}$$

Entry $G^{[l]}_{ij}$ is the inner product between feature maps $i$ and $j$, capturing how correlated those two channel activations are across all spatial locations.

**Style cost for layer $l$:**

$$J_\text{style}^{[l]} = \frac{1}{\left(2\, n_C^{[l]}\, n_H^{[l]}\, n_W^{[l]}\right)^2} \left\|G^{[l]}_S - G^{[l]}_G\right\|_F^2$$

where $S$ = style image, $G$ = generated image.

**Total style cost** (weighted sum over layers):

$$J_\text{style} = \sum_l \lambda^{[l]}\, J_\text{style}^{[l]}$$

**Combined NST objective:**

$$J = \alpha\, J_\text{content} + \beta\, J_\text{style}$$

Optimize the pixel values of the generated image (not network weights) via gradient descent.

## Applications

Neural style transfer; texture synthesis; measuring perceptual similarity between images. The Gram matrix approach has also influenced perceptual loss functions used in super-resolution and image-to-image translation.

## Trade-offs

Gram matrix discards all spatial information by design — good for style, bad if spatial structure matters. Computation grows with the square of the number of channels. The choice of which layers to use for style vs. content significantly affects results (lower layers → fine textures; higher layers → semantic style). Optimization in pixel space is slow compared to feed-forward style transfer networks.

## Links

- [[neural_style_transfer]] (in 02_modeling/computer_vision)
- [[01_foundations/_legacy/deep_learning_theory/backpropagation]]
