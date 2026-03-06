---
layer: 01_foundations
type: concept
status: seed
tags: [evaluation]
created: 2026-03-02
---

# Evaluation Metrics

## Definition

Scalar performance measures used to guide model selection and compare systems; properly structured metrics enable fast iteration.

## Intuition

If you cannot rank two systems with a single number, iteration slows down; the key insight is to separate what you maximize (optimizing metric) from what must merely be acceptable (satisficing metrics).

## Formal Description

**Single-number metric:** any scalar that ranks candidate models — accuracy, F1, AUC, BLEU, WER, etc.; choose one primary metric per project and don't average incomparable metrics without care.

**Satisficing vs optimizing:**
- Pick one metric to maximize (**optimizing**)
- Set threshold constraints on others (**satisficing**)
- Example: maximize accuracy s.t. inference latency < 100 ms, model size < 50 MB

**Trade-off surfaces:** when two metrics are genuinely in tension (precision–recall, accuracy–latency), the satisficing framework makes the trade-off explicit rather than hiding it in a weighted sum.

**When to combine metrics:** a weighted combination $J = \alpha \cdot \text{accuracy} - \beta \cdot \text{latency}$ is reasonable if relative importance is stable across the project lifetime; otherwise prefer satisficing constraints.

## Applications

- NLP: BLEU, ROUGE, perplexity
- Vision: top-1 accuracy, mAP
- Speech: WER
- Production systems: latency + accuracy jointly constrained

## Trade-offs

- Any single metric is a proxy and can be gamed
- Metrics should be re-evaluated as project requirements mature
- Business constraints (latency, cost, fairness) often dominate raw accuracy in deployment

## Links
- [[data_splits_and_distribution]]
- [[01_foundations/05_statistical_learning_theory/error_analysis]]
- [[orthogonalization]]
