---
layer: 04_ml_engineering
type: engineering
tool: general
status: growing
tags: [retraining, continual-learning, data-drift, mlops]
created: 2026-03-05
---

# Retraining Strategies

## Purpose

Models degrade in production when the world changes faster than the training data. Three core failure modes drive this need: **data drift** (input feature distributions shift — e.g., user demographics change), **label shift** (class priors change — e.g., fraud rate spikes post-policy-change), and **concept drift** (the relationship between inputs and labels changes — e.g., "good credit" means something different in a recession). Without periodic retraining, accuracy, calibration, and business metrics silently erode.

## Architecture

### Stateless Retraining (Train from Scratch)

A new model is trained on a fresh data window every cycle, discarding all prior model weights.

- **Pros:** Simple, robust against catastrophic forgetting, no dependency on checkpoint history, easy to audit.
- **Cons:** Expensive compute cost; slow for large models; requires a sufficiently large recent dataset.
- **Typical use case:** Tabular models, fraud detection, recommendation systems with fast-changing item catalogs.

### Stateful Retraining (Fine-tuning)

Training resumes from the last production checkpoint on a recent data increment.

- **Pros:** Much faster convergence; lower compute cost; leverages previously learned representations.
- **Cons:** Risk of **catastrophic forgetting** (old distribution underrepresented in new window); accumulates instability over many cycles; requires robust checkpoint management.
- **Mitigation:** Replay buffers (mix old and new data); elastic weight consolidation (EWC) penalizes large changes to weights important for prior tasks.

## Implementation Notes

### Trigger Strategies

| Trigger | Mechanism | Suitable For |
|---|---|---|
| **Scheduled** | Fixed cadence (daily, weekly) | Stable domains, low operational overhead |
| **Performance-based** | Retrain when metric drops below threshold (e.g., AUC < 0.82) | When evaluation feedback loop is fast |
| **Drift-based** | Statistical test on input/output distribution (PSI, KS-test, MMD) | When labels are delayed or expensive |
| **Data-volume-based** | Retrain when N new labeled examples accumulate | Active learning pipelines |

Performance-based triggers require a reliable online evaluation signal. Drift-based triggers are useful when ground truth labels are delayed by weeks (e.g., default prediction in lending — the label arrives months later). Population Stability Index (PSI) is a common threshold: PSI > 0.2 indicates significant shift.

### Data Freshness and Weighting

Not all historical data is equally valuable. Two common approaches:

1. **Fixed recency window:** Train only on the last *T* days/months. Simple but discards potentially useful older signal.
2. **Exponential decay weighting:** Weight sample *i* as `w_i = exp(-λ · age_i)` where `λ` controls decay rate. Balances recency with data volume; requires a tuned `λ`.

### Lineage and Reproducibility

Every retrained model must be linked to: (1) the exact training data snapshot (versioned dataset URI), (2) the code commit hash, (3) the environment/container image digest, (4) hyperparameters, and (5) the parent checkpoint (for stateful). MLflow, DVC, and SageMaker Experiments all support this provenance graph. Without it, debugging a regression in a retrained model becomes intractable.

## Trade-offs

- **Freshness vs. stability:** Retraining too frequently on small windows increases variance; retraining too infrequently allows drift to accumulate.
- **Stateful speed vs. forgetting risk:** Fine-tuning is 5–10× cheaper per cycle but requires careful data mixing to avoid distribution collapse.
- **Trigger cost vs. lag:** Performance-based triggers react fastest but require continuous evaluation infrastructure. Scheduled triggers are cheapest operationally but react slowest.
- **Automation vs. oversight:** Fully automated retraining pipelines reduce latency but require strong guardrails (shadow evaluation, automatic rollback) to prevent silent model degradation from a bad retraining run.

## References

- Sculley et al., "Hidden Technical Debt in Machine Learning Systems" (NeurIPS 2015)
- Gama et al., "A Survey on Concept Drift Adaptation" (ACM Computing Surveys, 2014)
- Kirkpatrick et al., "Overcoming Catastrophic Forgetting in Neural Networks" (PNAS, 2017)
- Klaise et al., "Monitoring and Explainability of Models in Production" (ICML Workshop, 2020)

## Links
- [[testing_in_production|Testing in Production]]
- [[ml_platform_architecture|ML Platform Architecture]]
- [[drift_detection|Drift Detection]]
- [[ml_observability|ML Observability]]
- [[experiment_tracking|Experiment Tracking]]
- [[rollout_strategies|Model Rollout Strategies]]
