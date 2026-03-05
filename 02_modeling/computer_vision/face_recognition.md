---
layer: 02_modeling
type: concept
status: seed
tags: [computer_vision, representation_learning, metric_learning]
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

**Triplet loss:** see [[01_foundations/_legacy/deep_learning_theory 1/triplet_loss]] in 01_foundations/deep_learning_theory; anchor-positive-negative triplets with margin $\alpha$.

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

- [[01_foundations/_legacy/deep_learning_theory 1/triplet_loss]] (in 01_foundations/deep_learning_theory)
- [[object_detection]]
- [[cnn_architecture]]
