---
layer: 01_foundations
type: concept
status: seed
tags: [algorithm, fine-tuning]
created: 2026-03-02
---

# Transfer Learning

## Definition

Leveraging representations or parameters learned on a source task/domain to improve learning on a related target task/domain, typically by fine-tuning a pretrained model.

## Intuition

Lower-level features (edges, textures, syntax) are shared across tasks; it's wasteful to relearn them from scratch; the more similar the source and target, the more transferable the representations.

## Formal Description

**Full fine-tuning:** initialize with pretrained weights $\theta_\text{pre}$, then train all layers on the target task with a small learning rate.

**Head-only fine-tuning (feature extraction):** freeze pretrained layers, train only a new output head; useful when target data is small.

**When to use transfer:**
1. Source and target have overlapping low-level features
2. Source dataset is much larger than target
3. Target data is scarce
4. Compute budget is limited

**Pre-training → fine-tuning workflow:**
1. Train on large source dataset
2. Replace/reinitialize output layer for target task
3. Optionally freeze early layers
4. Fine-tune with lower learning rate than initial training

**Negative transfer:** if source and target distributions are too different, pretrained features may hurt rather than help.

## Applications

- Vision: ImageNet → medical imaging, autonomous driving
- NLP: BERT/GPT → downstream classification, QA, summarization
- Speech: large ASR model → specific accents or domains

## Trade-offs

- Requires compatible architectures between source and target
- Fine-tuning all layers needs non-trivial target data to avoid catastrophic forgetting
- Hyperparameter sensitivity (learning rate, number of layers to freeze)
- Pretrained models can encode biases from source data

## Links
- [[multi_task_learning]]
- [[data_splits_and_distribution]]
- [[bias_variance_analysis]]
