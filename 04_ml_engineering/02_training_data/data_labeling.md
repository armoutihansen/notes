---
layer: 04_ml_engineering
type: engineering
tool: general
status: growing
tags: [data-labeling, weak-supervision, active-learning, annotation]
created: 2026-03-05
---

# Data Labeling Strategies

## Purpose

Labels are the signal that supervised ML learns from. The quality, quantity, and consistency of labels determine an upper bound on model performance. Yet labeling is expensive, slow, and error-prone. Understanding the trade-offs between labeling strategies allows practitioners to acquire the highest-quality signal within time and budget constraints.

## Architecture

### Hand Labeling

Human annotation by domain experts or crowdworkers (e.g., Scale AI, Labelbox, Amazon Mechanical Turk) is the most reliable source of ground truth. Key dimensions:

- **Quality**: Subject matter experts produce higher-quality labels but cost 10–100× more than crowdworkers. Use experts for complex tasks (medical imaging, legal documents); crowdworkers for tasks with clear visual or textual criteria.
- **Cost**: Hand labeling costs range from $0.01/label (simple classification, crowd) to $50+/label (specialist annotation, legal/medical). Budget drives the feasible dataset size.
- **Speed**: Labeling campaigns for 100K examples typically take weeks to months. This creates a bottleneck in the ML lifecycle: model iteration is blocked on data.
- **Label ambiguity**: Many tasks have inherent ambiguity. A sentiment label of "neutral" vs. "slightly negative" is subjective. Ambiguity should be measured (inter-annotator agreement, Cohen's κ) and handled explicitly rather than averaged away.

**Labeling guidelines** are the most under-invested component of labeling programs. A well-written guideline document:
- Provides 20–50 worked examples covering edge cases.
- Defines decision trees for ambiguous cases.
- Specifies what information annotators should and should not use.
- Is versioned alongside the dataset.

Poor guidelines are the primary cause of low inter-annotator agreement. Target κ > 0.7 for most tasks before trusting the labels.

### Weak Supervision (Snorkel-Style)

Weak supervision generates noisy, programmatic labels at scale without manual annotation, then denoises them using a generative model. The Snorkel framework (Ratner et al., 2017) operationalised this approach.

**Labeling functions (LFs)**: Python functions that map an input to a label or abstain. Each LF encodes a heuristic, a distant supervision signal, or a pattern:

```python
@labeling_function()
def lf_keyword_fraud(x):
    fraud_keywords = ["urgent", "wire transfer", "gift card"]
    return FRAUD if any(kw in x.text.lower() for kw in fraud_keywords) else ABSTAIN
```

Individual LFs have low accuracy and limited coverage; their outputs are combined via a **label model** that learns each LF's accuracy and correlation structure. The label model produces probabilistic labels that are typically used as soft targets for a downstream discriminative model.

**Majority vote** is the simplest aggregation: assign the label chosen by the most LFs. Works well when LFs are diverse and independent; degrades when LFs are correlated.

Weak supervision is most effective when: (1) labeling is expensive or slow, (2) domain knowledge is encodable as rules, and (3) you have ≥10 LFs with diverse coverage.

### Natural and Behavioural Labels

**Natural labels** are implicit ground truth signals embedded in user behaviour:

- A click on a recommended item is a positive label for the recommender.
- A return/refund is a negative label for purchase intent prediction.
- A "not spam" action in Gmail is a negative label for the spam classifier.

Natural labels are abundant and low-cost but biased: they reflect what users chose to do, not what they would have done under all counterfactual conditions. They also have **label lag**: the true outcome (e.g., a purchase that occurs 7 days after a recommendation) is only observed after a delay. Choosing the right observation window is a critical modelling decision.

### Semi-Supervised Labeling

Semi-supervised methods use a small pool of labelled data alongside a large pool of unlabelled data. Common approaches:

- **Self-training**: Train on labelled data, pseudo-label high-confidence predictions on unlabelled data, retrain. Iterate.
- **Label propagation**: Propagate labels through a similarity graph; nearby unlabelled points inherit labels.
- **Consistency regularisation** (e.g., UDA, FixMatch): Unlabelled examples must produce consistent predictions under data augmentation.

Semi-supervised learning is most effective when the model's decision boundary can be inferred from the unlabelled data's structure—i.e., when cluster assumption or manifold assumption holds.

### Active Learning for Label Efficiency

Active learning selects the most informative unlabelled examples to label next, rather than labeling at random. Strategies include:

- **Uncertainty sampling**: Label examples where the model is least confident (lowest max probability, highest entropy).
- **Query by committee**: Maintain an ensemble; label where members disagree most.
- **Core-set selection**: Choose examples that maximally cover the embedding space.
- **Expected model change**: Select examples whose labeling would maximally change the model.

Active learning can reduce the required label budget by 50–90% on many tasks. The practical constraint is that it requires a human in the loop during training, which complicates automation.

### Labeling Quality Metrics

| Metric | Use |
|---|---|
| **Cohen's κ** | Pairwise inter-annotator agreement for categorical labels |
| **Krippendorff's α** | Multi-rater agreement, handles missing annotations |
| **Label noise rate** | Fraction of labels that disagree with a reference |
| **Coverage per class** | Detects under-represented classes early |
| **LF coverage / accuracy** | Weak supervision LF diagnostics |

## Implementation Notes

- Always hold out a "gold set" of expert-annotated examples to validate crowdworker quality over time. Reject workers below the gold set threshold.
- When using natural labels, explicitly define the label horizon (e.g., "a click within 24 hours counts as positive") and document it alongside the dataset.
- In weak supervision pipelines, track LF coverage and accuracy on the labelled validation set. Remove LFs that reduce label model accuracy.
- Store raw annotations (before aggregation) alongside aggregated labels. Disagreement is a signal, not just noise—it may indicate genuinely ambiguous examples worth examining.

## Trade-offs

| Strategy | Quality | Cost | Speed | Scale |
|---|---|---|---|---|
| Expert hand labeling | Highest | Highest | Slowest | Low |
| Crowdsourced labeling | Medium | Medium | Medium | High |
| Weak supervision | Lower | Low | Fast | Very high |
| Natural labels | Task-dependent | Very low | Instant | Very high |
| Semi-supervised | Medium | Low | Medium | High |

## References

- Ratner, A. et al. (2017). *Snorkel: Rapid Training Data Creation with Weak Supervision*. VLDB.
- Huyen, C. (2022). *Designing Machine Learning Systems*. O'Reilly. Chapter 4.
- Settles, B. (2009). *Active Learning Literature Survey*. UW-Madison Technical Report.

## Links
- [[class_imbalance_and_augmentation|Class Imbalance and Data Augmentation]]
- [[dataset_versioning|Dataset Versioning and Lineage]]
- [[ml_lifecycle|ML Lifecycle]]
- [[fairness_metrics|Fairness Metrics]]
