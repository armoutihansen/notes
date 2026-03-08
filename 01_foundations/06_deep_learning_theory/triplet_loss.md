---
layer: 01_foundations
type: concept
status: seed
tags: [algorithm, training]
created: 2026-03-02
---

# Triplet Loss

## Definition

A metric learning objective that trains an embedding function $f$ to satisfy: anchor is closer to positive (same class) than to negative (different class) by at least a margin $\alpha$. The network learns a distance function in embedding space rather than a classifier over fixed classes.

## Intuition

Instead of classifying into predefined categories, the network learns a geometry where similar items cluster together and dissimilar items are pushed apart. This is useful when classes are not predefined at training time (e.g., verifying identity of unseen people) or when fine-grained similarity matters more than hard class boundaries.

## Formal Description

Given anchor $a$, positive $p$, negative $n$:

$$
\ell(a, p, n) = \max\!\left(0,\; \|f(a) - f(p)\|_2^2 - \|f(a) - f(n)\|_2^2 + \alpha\right)
$$

**Margin $\alpha > 0$** prevents the trivial solution of collapsing all embeddings to the same point (which would satisfy $\|f(a)-f(p)\|^2 = \|f(a)-f(n)\|^2$).

**Hard triplet mining:** Once training progresses, random triplets are mostly easy (loss = 0) and produce no gradient. Semi-hard negatives — negatives that are currently closer to the anchor than the positive, or just beyond the positive — provide the most useful signal.

**Online mining:** Compute all pairwise distances within a batch, then select hard or semi-hard pairs. Enables mining without a separate offline mining step.

## Applications

Face verification and recognition (FaceNet), one-shot learning, image similarity search, person re-identification. Any task where the goal is measuring similarity rather than classification.

## Trade-offs

Requires careful triplet mining strategy; random sampling leads to training stagnation. Too-hard negatives (outliers or label noise) can destabilize training. Requires larger batch sizes for effective online mining. Contrastive loss and NT-Xent (SimCLR) are related alternatives that can be easier to tune.

## Links

- [[face_recognition]] (in 03_modeling/computer_vision)
- [[01_foundations/06_deep_learning_theory/cross_entropy_loss]]
