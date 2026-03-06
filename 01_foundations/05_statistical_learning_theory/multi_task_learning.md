---
layer: 01_foundations
type: concept
status: seed
tags: [algorithm, training]
created: 2026-03-02
---

# Multi-Task Learning

## Definition

Training a single model to perform multiple related tasks simultaneously, sharing representations across tasks to improve generalization and data efficiency.

## Intuition

Related tasks share useful low-level structure; learning them jointly forces the shared layers to represent features that are broadly useful, acting as implicit regularization.

## Formal Description

**Architecture:** shared encoder/backbone + separate task-specific heads; loss is a weighted sum:

$$J = \sum_{k=1}^K w_k J_k$$

**Hard parameter sharing:** shared hidden layers with separate output heads (standard approach).

**Soft parameter sharing:** separate models with regularization encouraging similar parameters (less common).

**Unlike transfer learning:** all tasks are trained simultaneously; there is no separate pre-training phase.

**Works well when:**
1. Tasks share low- or mid-level features
2. Each task has insufficient data alone
3. Tasks have similar levels of difficulty

**Requirement for beneficial MTL:** tasks must share useful features; if not, separate models perform better.

## Applications

- Autonomous driving: simultaneously predict lane lines, pedestrians, traffic signs
- NLP: POS tagging + NER + parsing in a single model
- Medical imaging: multi-label classification over pathologies

## Trade-offs

- Negative transfer if tasks are too dissimilar
- Loss weighting $w_k$ requires tuning; a dominant task can crowd out others
- Harder to debug than single-task models
- Task difficulty imbalance can cause uneven convergence

## Links
- [[transfer_learning]]
- [[data_splits_and_distribution]]
