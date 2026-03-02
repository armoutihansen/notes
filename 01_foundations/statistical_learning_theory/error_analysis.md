---
layer: 01_foundations
type: concept
status: seed
tags: [error-analysis, diagnostics, dev-set, mislabeling]
created: 2026-03-02
---

# Error Analysis

## Definition

A systematic process of manually inspecting misclassified dev examples to identify and quantify error categories, enabling prioritized model improvement.

## Intuition

Raw error rate tells you how bad the model is, but not why; error analysis reveals the largest tractable sources of error so effort goes to the highest-impact fixes.

## Formal Description

**Minimal workflow:**
1. Sample ~100 misclassified dev examples
2. Define error categories (e.g., blurry image, label noise, unusual lighting, occlusion)
3. Count each category
4. Compute ceiling: if fixing category X would eliminate p% of errors, it's worth at most a p% relative improvement
5. Prioritize by expected impact × implementation effort

**Handling mislabeled examples:** count mislabeled examples as a separate category; only worth fixing if they represent a significant fraction of total errors; apply corrections consistently to dev and test sets (never selectively to one).

## Applications

Image classification, NLP tasks, speech recognition — anywhere you have interpretable dev errors.

## Trade-offs

- Time-consuming for very large error sets
- Human judgment introduces bias in category definitions
- Categories should be mutually exclusive to avoid double-counting

## Links
- [[bias_variance_analysis]]
- [[evaluation_metrics]]
- [[data_splits_and_distribution]]
