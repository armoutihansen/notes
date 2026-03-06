---
layer: 04_ml_engineering
type: engineering
tool: general
status: growing
tags: [ml-systems, system-design, technical-debt, production-ml]
created: 2026-03-05
---

# ML System Design

## Purpose

ML system design covers the principles, constraints, and failure modes that distinguish machine learning systems from traditional software. Understanding these differences is prerequisite to building production-grade ML: systems that don't just work in notebooks, but hold up under real-world data distributions, operational load, and evolving requirements.

## Architecture

### Four Properties of Production ML

**Reliability** means the system continues to produce correct outputs under adversarial conditions—hardware failures, malformed inputs, distributional shift, and upstream data outages. Unlike traditional software where correctness is deterministic, ML systems have stochastic behavior and can degrade silently (no exception thrown when accuracy drops).

**Scalability** addresses growth in data volume, feature count, model complexity, and request throughput. ML systems must scale along multiple axes simultaneously: training compute, serving latency (p50/p99), feature computation, and monitoring overhead. Design decisions made early (e.g., synchronous vs. asynchronous inference, batch vs. real-time features) compound at scale.

**Maintainability** is the hardest property to retrofit. ML systems have dual codebases—application code and data/model artifacts—that evolve on different timescales. A model trained six months ago may be incompatible with a refactored feature pipeline today. Maintainability requires reproducible training pipelines, versioned artifacts, comprehensive logging, and explicit contracts between components.

**Adaptability** recognises that the world changes. Customer behaviour shifts, products evolve, regulations tighten. ML systems must be designed to retrain, fine-tune, or replace models without a full rewrite. This demands modularity: decouple data ingestion, feature engineering, training, serving, and monitoring so each can evolve independently.

### Hidden Technical Debt (Sculley et al.)

The 2015 Google paper "Hidden Technical Debt in Machine Learning Systems" identifies several ML-specific debt patterns:

- **Boundary erosion**: ML components lack clear input/output contracts; changing one model silently degrades another because they share features or interact through feedback loops.
- **Entanglement (CACE principle)**: Changing Anything Changes Everything. Adding a feature, removing a feature, or altering preprocessing alters the learned function in non-local, hard-to-predict ways.
- **Data dependencies**: Unlike code dependencies, data dependencies are invisible to static analysis. Unstable upstream data sources, undeclared transformations, and implicit assumptions about schema silently break models.
- **Feedback loops**: A deployed model influences the data it will be retrained on. A recommendation model shapes user behaviour; a fraud model changes attacker strategy. Left unmanaged, feedback loops cause runaway bias or system ossification.
- **Glue code and pipeline jungles**: Wrapping general-purpose libraries with bespoke glue code accumulates rapidly. Pipeline jungles—ad hoc preprocessing scripts scattered across repos—make it impossible to reproduce training runs.

### Before Starting a Project

Before writing a single line of model code, answer:

1. **Is ML the right tool?** If a deterministic rule, a lookup table, or a regression with hand-crafted features achieves 90% of the business value, use that. ML introduces complexity that must be justified.
2. **What is the objective?** Define the proxy metric (e.g., click-through rate), the business metric (e.g., revenue), and the ethical constraints (e.g., demographic parity). Misalignment between these is the most common source of deployed model failure.
3. **What data exists?** Inventory data sources: volume, freshness, label availability, access latency, and legal permissions. A model is only as good as its training signal.
4. **What is the baseline?** Establish a simple baseline (random, majority class, linear model, human performance) before any deep learning. The baseline defines the minimum bar for the ML system to be worth deploying.
5. **What are the failure modes?** Enumerate ways the system can fail silently (e.g., data pipeline outage returning stale features), loudly (e.g., serving infrastructure crash), or subtly (e.g., performance degrading on a demographic subgroup).

## Implementation Notes

- Document data schemas with explicit types, nullable fields, and expected value ranges. Treat schema changes as API-breaking changes.
- Use feature flags or shadow mode to deploy new models without full traffic exposure.
- Separate business logic from ML logic: business rules (price floors, regulatory constraints) should not be baked into model weights.
- Instrument everything from day one: prediction distributions, feature statistics, label lag, and serving latency. Retrofitting monitoring is expensive.

## Trade-offs

| Concern | Trade-off |
|---|---|
| Model complexity | Accuracy vs. latency, interpretability, maintainability |
| Retraining frequency | Freshness vs. compute cost and stability risk |
| Online vs. batch inference | Latency vs. throughput and infrastructure cost |
| Single model vs. ensemble | Accuracy vs. serving complexity and debugging difficulty |

## References

- Sculley et al. (2015). *Hidden Technical Debt in Machine Learning Systems*. NeurIPS.
- Huyen, C. (2022). *Designing Machine Learning Systems*. O'Reilly.
- Kleppmann, M. (2017). *Designing Data-Intensive Applications*. O'Reilly.

## Links
- [[ml_lifecycle|ML Lifecycle]]
- [[regularization|Regularization]]
- [[interpretability_overview|Interpretability Overview]]
- [[model_risk_considerations|Model Risk Considerations]]
- [[fairness_metrics|Fairness Metrics]]
- [[drift_detection|Drift Detection]]
- [[ml_observability|ML Observability]]
- [[data_pipeline_patterns|Data Pipeline Patterns]]
