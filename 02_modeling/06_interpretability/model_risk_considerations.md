---
layer: 02_modeling
type: concept
status: seed
tags: [model-risk, model-governance, validation, backtesting, concept-drift, sr11-7, eu-ai-act, regulation]
created: 2026-03-02
---

# Model Risk Considerations

## Definition

Model risk is the potential for adverse consequences arising from decisions based on incorrect or misused models. Model risk management (MRM) encompasses policies, processes, and controls that govern model development, validation, deployment, and monitoring throughout a model's lifecycle.

## Intuition

Every deployed model is an approximation of reality built on historical data. The world can drift, the training data can be unrepresentative, and the model can be used outside its intended scope. Without governance structures, model failures may go undetected until they cause financial loss, regulatory action, reputational damage, or harm to individuals.

## Formal Description

### Model Risk Sources

| Source | Description |
|---|---|
| Conceptual error | Wrong modelling approach for the problem (e.g., linear model for highly non-linear phenomena) |
| Data quality | Biased, incomplete, stale, or incorrectly processed training data |
| Implementation error | Bugs in feature pipelines, scoring code, or deployment infrastructure |
| Overfitting / underfitting | Poor generalization from training to production population |
| Concept drift | Statistical distribution of inputs $P(X)$ or $P(Y|X)$ changes post-deployment |
| Scope creep | Model used for purposes beyond its validated intended use |

---

### Regulatory Frameworks

**SR 11-7 (Federal Reserve / OCC, 2011)**: the foundational US regulatory guidance on model risk management for banks. Defines a model as a quantitative method with inputs, processing, and outputs. Requires:
1. **Model development** with sound methodology, conceptual review, and documentation
2. **Independent model validation** — separate team reviews conceptual soundness, ongoing monitoring, and outcomes analysis
3. **Effective challenge**: validators must have sufficient authority, incentive, and expertise to critically assess models
4. Governance: inventory, tiering by risk, periodic revalidation schedule

**EU AI Act (2024)**: risk-based regulation. High-risk AI systems (credit scoring, employment, critical infrastructure, law enforcement) require conformity assessment, technical documentation, human oversight provisions, transparency to affected persons, and registration in an EU database. Prohibited AI: social scoring by public authorities, real-time biometric identification in public spaces (with exceptions).

---

### Validation Framework

**Conceptual soundness review**: Does the theoretical basis support the use case? Are assumptions reasonable and documented?

**Statistical backtesting**: evaluate model performance on a holdout period or an out-of-time sample. Metrics depend on use case:
- Classification: AUROC, Gini coefficient, Kolmogorov-Smirnov statistic, PSI (Population Stability Index)
- Regression: RMSE, MAE, bias

**Population Stability Index (PSI)**: measures distributional shift in input features between development and current population. $\text{PSI} = \sum_i (A_i - E_i)\ln(A_i/E_i)$. Thresholds: PSI < 0.1 (stable), 0.1–0.2 (monitor), > 0.2 (investigate/retrain).

**Outcome analysis**: compare actual outcomes vs. predicted; track calibration, discrimination, and performance over time segments.

---

### Concept Drift

**Covariate shift**: $P(X)$ changes but $P(Y|X)$ stays the same. Solution: reweigh samples or retrain.
**Label drift**: $P(Y|X)$ changes (the relationship between features and outcome shifts). Requires model retraining. Harder to detect early because true labels often arrive with delay.
**Concept drift detection methods**: Page-Hinkley test, ADWIN (Adaptive Windowing), CUSUM, drift detectors in River/scikit-multiflow.

---

### Model Governance Lifecycle

1. **Development**: documented methodology, data lineage, assumptions, limitations
2. **Independent validation**: conceptual review, benchmarking, sensitivity analysis, backtesting
3. **Approval and tiering**: risk-based classification (tier 1–3 by materiality)
4. **Deployment**: champion/challenger setup; shadow mode evaluation before full rollout
5. **Ongoing monitoring**: scheduled performance reports, drift alerts, exception escalation
6. **Periodic revalidation**: triggered by material changes (new data, new use case, significant drift)
7. **Retirement**: decommission process; retain documentation per record-keeping requirements

## Applications

- Credit risk models (PD/LGD/EAD, IFRS 9 ECL models) — SR 11-7 primary domain
- Stress testing models (CCAR/DFAST)
- Any ML model used in hiring, lending, benefits eligibility, or medical decisions — EU AI Act high-risk category
- Fraud detection, AML transaction monitoring

## Trade-offs

- Heavier governance slows innovation; lighter governance increases risk — calibrate by tier/materiality
- Independent validation adds cost and time but is essential for detecting implementation errors and conceptual gaps
- Automated drift monitoring reduces manual review burden but can produce false alarms requiring triage

## Links

- [[fairness_metrics|Fairness Metrics]]
- [[interpretability_overview|Interpretability Overview]]
- [[02_modeling/index|Modeling]]
- [[01_foundations/05_statistical_learning_theory/generalization_bounds|Generalization Bounds — out-of-distribution and deployment risk]]
- [[01_foundations/05_statistical_learning_theory/bias_variance_analysis|Bias–Variance Analysis — model complexity and overfitting risk]]
