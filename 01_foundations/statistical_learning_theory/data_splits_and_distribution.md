---
layer: 01_foundations
type: concept
status: seed
tags: [data-splits, train-dev-test, distribution-mismatch, evaluation]
created: 2026-03-02
---

# Data Splits and Distribution

## Definition

The practice of partitioning data into train/dev/test sets to enable unbiased model iteration and evaluation, and managing the consequences of distribution mismatch between sets.

## Intuition

The dev set guides model selection; the test set estimates real-world performance; they must reflect the actual deployment distribution — otherwise you're optimizing for the wrong target.

## Formal Description

**Train/dev/test partition:**
- **Train**: used for optimization (gradient updates, parameter fitting)
- **Dev (validation)**: used for hyperparameter selection and model comparison; may be evaluated many times
- **Test**: used for final unbiased evaluation — touch only once

**Sizes:**
- Traditional split: 60/20/20 for small datasets
- With millions of examples, 98/1/1 is reasonable — dev/test only need to be large enough for statistical significance

**Distribution requirements:**
- Dev and test must come from the same distribution
- Train can be augmented from other sources (web crawl, synthetic data, etc.)

**Distribution mismatch:** if $P_\text{train} \neq P_\text{dev/test}$, improvements in dev/test error may not reflect real-world improvement; models may overfit to the training distribution.

**Training-dev set:** a held-out set sampled from the *training* distribution (not dev/test distribution); used to distinguish variance from mismatch:

| Gap | Interpretation |
|-----|----------------|
| Train error ≈ training-dev error ≪ dev error | Mismatch (not variance) |
| Train error ≪ training-dev error ≈ dev error | Variance (not mismatch) |
| Train error ≪ training-dev error ≪ dev error | Both variance and mismatch |

## Applications

All supervised ML projects; especially important in industry where training data is crawled or synthetic but deployment is user-facing.

## Trade-offs

- Too-small dev set → noisy model comparisons
- Mismatched dev/test → optimizing for the wrong distribution
- Leaking test set → overly optimistic reported performance

## Links
- [[bias_variance_analysis]]
- [[error_analysis]]
