---
layer: 04_ml_engineering
type: engineering
tool: general
status: growing
tags: [ml-lifecycle, mlops, project-scoping, iterative-development]
created: 2026-03-05
---

# ML Lifecycle

## Purpose

The ML lifecycle describes the end-to-end, iterative process of taking a business problem through data acquisition, model development, deployment, and ongoing monitoring. Unlike a waterfall software project, the ML lifecycle is inherently circular: each deployed model generates new signal that informs the next iteration. Understanding the lifecycle prevents teams from over-investing in a single stage (e.g., model tuning) while neglecting others (e.g., monitoring infrastructure).

## Architecture

### Iterative Development Cycle

The lifecycle has five major stages, each feeding into the next—and back into earlier stages:

```
Data → Model → Deployment → Monitoring → Retrain → (Data → ...)
```

**1. Data**
Collect, explore, and validate training data. Define the labeling strategy, establish data quality contracts, and version the dataset. Data exploration often reveals that the original problem framing was wrong, sending the project back to scoping. Data is the highest-leverage stage: improving data quality typically yields larger accuracy gains than architecture changes.

**2. Model**
Select a model family, engineer features, train, tune hyperparameters, and evaluate against held-out data. The key output of this stage is not just model weights but a reproducible training pipeline. Model development is itself iterative: start with the simplest reasonable baseline, measure its shortcomings, and add complexity only where the data justifies it.

**3. Deployment**
Package the model, expose it via API or batch job, and release to production traffic. Deployment decisions made here—serving infrastructure, input validation, caching, fallback logic—are hard to change later. Shadow mode and canary deployment reduce risk by exposing the new model to a fraction of traffic before full rollout.

**4. Monitoring**
Track model performance in production. Monitor both technical metrics (latency, error rate, throughput) and ML metrics (prediction distribution, feature drift, label distribution shift, business KPIs). Monitoring closes the loop: degradation signals trigger retraining, and unexpected input patterns surface new data collection opportunities.

**5. Retrain**
Retrain on fresh data when performance degrades or at a scheduled cadence. Retraining is not a one-shot fix—it must be automated, tested, and gated behind the same evaluation criteria as the original deployment. Continuous training pipelines treat retraining as a first-class software operation.

### When to Use ML vs. Heuristics

Before committing to the ML lifecycle, evaluate whether ML is warranted:

| Situation                                           | Recommendation                               |
| --------------------------------------------------- | -------------------------------------------- |
| Patterns are too complex for hand-crafted rules     | Use ML                                       |
| Data is abundant and labels are available           | Use ML                                       |
| Rules can be enumerated and maintained              | Use heuristics                               |
| Data volume is small (<1,000 labelled examples)     | Heuristics or transfer learning              |
| Latency budget is <10 ms                            | Consider lookup tables or lightweight models |
| Regulatory environment requires full explainability | Linear/tree models or heuristics             |

A common mistake is building an ML system when a well-maintained rule engine would be cheaper, faster to audit, and easier to update.

### Project Scoping

Good project scoping answers four questions before development begins:

1. **Business objective**: What metric is the business trying to move, and by how much? Tie the ML objective directly to this.
2. **Constraints**: Latency budget, compute budget, fairness requirements, explainability requirements, data access limitations.
3. **Data inventory**: What labelled data exists? What can be collected? What labelling strategy is feasible?
4. **Evaluation protocol**: How will the model be evaluated before and after deployment? What is the acceptance threshold?

Scoping should also produce a risk register: what are the ways this project fails, and how would we detect each failure mode early?

### Baseline Establishment

Every ML project must establish a baseline before training anything complex. Useful baselines include:

- **Random baseline**: What accuracy does random guessing achieve? Sets the floor.
- **Majority class baseline**: Always predict the most common class. Crucial for imbalanced datasets.
- **Human performance**: Benchmark against what a skilled human achieves. Sets the ceiling for tasks where human labels are available.
- **Simple model baseline**: Logistic regression or a shallow decision tree on raw features. Quantifies how much complexity is worth adding.

The gap between the simple model and human performance is the "ML opportunity". If this gap is small, complex models are rarely worth the operational cost.

## Implementation Notes

- Use experiment tracking tools (MLflow, Weights & Biases, or similar) from the first experiment to ensure reproducibility across the full lifecycle.
- Version datasets and model artifacts together; a model trained on dataset `v3` is not valid evidence for dataset `v4`.
- Define "done" for each lifecycle stage with explicit exit criteria before starting, not after.
- In the monitoring stage, distinguish between data drift (input distribution shift) and concept drift (the relationship between inputs and outputs has changed); they require different responses.

## Trade-offs

| Decision | Trade-off |
|---|---|
| Automated retraining vs. manual gating | Speed of adaptation vs. risk of silent regression |
| Complex feature engineering vs. raw inputs | Accuracy vs. pipeline fragility |
| Online learning vs. batch retraining | Freshness vs. stability and reproducibility |
| Single deployment target vs. multi-environment | Simplicity vs. risk containment |

## References

- Huyen, C. (2022). *Designing Machine Learning Systems*. O'Reilly. Chapters 1–2.
- Sculley et al. (2015). *Hidden Technical Debt in Machine Learning Systems*. NeurIPS.
- Ng, A. (2022). *Machine Learning Yearning* (draft). deeplearning.ai.

## Links
- [[ml_system_design|ML System Design]]
- [[data_pipeline_patterns|Data Pipeline Patterns]]
- [[data_labeling|Data Labeling Strategies]]
- [[hyperparameter_tuning|Hyperparameter Tuning]]
- [[regularization|Regularization]]
