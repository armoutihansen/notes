---
layer: 03_modeling
type: concept
status: seed
tags: [algorithm, vision]
created: 2026-03-02
---

# Face Recognition

## Definition

The task of verifying or identifying a person from their face image; framed as a metric learning problem where the model learns an embedding space where same-identity faces cluster together.

## Intuition

Training a separate classifier per identity requires retraining when new identities are added. Metric learning instead trains an embedding where distance directly measures similarity — enabling one-shot recognition of unseen identities.

## Formal Description

**Verification vs identification:**
- Verification: given two images, are they the same person? (binary)
- Identification: given a probe image, who is this person? (retrieval)

**Embedding network** $f: \mathbb{R}^{H \times W \times 3} \to \mathbb{R}^d$, typically $d = 128$ or $512$; trained so $\|f(x_i) - f(x_j)\|_2$ is small for same identity, large for different.

**Triplet loss:** anchor-positive-negative triplets with margin $\alpha$:

$$
\mathcal{L} = \sum_i \left[\|f(A_i) - f(P_i)\|^2 - \|f(A_i) - f(N_i)\|^2 + \alpha\right]_+
$$

Hard triplet mining (selecting difficult negatives) is critical for convergence.

**Siamese networks:** two branches with shared weights processing two images; outputs compared directly.

**Deployment:** compare probe embedding against gallery embeddings; threshold on distance for accept/reject.

## Applications

Phone unlock, building access control, photo management (tagging), law enforcement (surveillance).

## Trade-offs

- Requires large labeled datasets of identity pairs/triplets
- Hard triplet mining is critical for convergence
- Bias/fairness concerns (demographic performance disparities)
- Privacy implications

## Links

- [[object_detection]]
- [[cnn_architecture]]
- [[03_modeling/04_deep_learning/index|Deep Learning]]
- [[01_foundations/06_deep_learning_theory/triplet_loss|Triplet Loss — anchor/positive/negative metric learning]]
- [[01_foundations/01_linear_algebra/03_eigenvalues/svd|SVD — embedding distance geometry, low-rank face spaces]]
