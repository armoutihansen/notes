---
layer: 02_modeling
type: concept
status: seed
tags: [interpretability, algorithm]
created: 2026-03-02
---

# Model Interpretability Overview

## Definition

Model interpretability refers to the degree to which a human can understand the cause-and-effect relationships within a model — why a specific prediction was made and how input features influence outputs. Explainability is sometimes used interchangeably, though some authors distinguish: interpretability as the capacity to understand, explainability as the provision of human-comprehensible artifacts.

## Intuition

A high-accuracy black box is not always sufficient. Practitioners need to debug models, satisfy regulatory requirements, detect bias, and build user trust. Interpretability tools answer questions like: "Which features drove this credit rejection?", "Is the model using spurious correlations?", and "Does the model behave consistently across demographic groups?"

## Formal Description

### Taxonomy of Methods

**By model type:**

| Category | Description | Examples |
|---|---|---|
| Intrinsic / transparent | Model is interpretable by construction | Linear regression, decision trees, rule lists, GAMs |
| Post-hoc | Interpretation applied after training any model | SHAP, LIME, PDP, ICE, integrated gradients |

**By scope:**

| Scope | Description | Examples |
|---|---|---|
| Global | Explains overall model behaviour across all inputs | Feature importances, PDPs, global SHAP summary plots |
| Local | Explains a single prediction | LIME, SHAP for one instance, counterfactual explanations |

**By modality:**

- **Feature attribution**: how much does each feature contribute to a prediction? (SHAP, integrated gradients, permutation importance)
- **Example-based**: which training examples are most similar or influential? (k-NN explanations, influence functions)
- **Concept-based**: does the model represent human-defined concepts? (TCAV)
- **Counterfactual**: what minimal change to input would flip the prediction? (DiCE, watcher)

---

### Intrinsic vs Post-Hoc Trade-offs

Intrinsically interpretable models (linear, shallow trees) may sacrifice predictive performance. Post-hoc methods can explain any black box but may produce approximate or unfaithful explanations if the surrogate doesn't match the true model.

The **accuracy–interpretability trade-off** is context-dependent: in high-stakes domains (medicine, credit, criminal justice) interpretability may be required by regulation; in low-stakes offline settings a black box may be fine.

---

### Why Interpretability Matters

1. **Debugging**: identify data leakage, spurious correlations, distribution shift artifacts
2. **Trust**: operators and end-users need confidence that the model reasons correctly
3. **Fairness**: detect differential feature usage across demographic groups
4. **Regulatory compliance**: GDPR right-to-explanation, EU AI Act, SR 11-7 require explanations for automated decisions
5. **Scientific discovery**: understand which features or inputs are causally important

## Applications

- Credit scoring (which applicant attributes drove a denial)
- Medical diagnosis (which imaging features activated a classifier)
- Fraud detection (explainability for analyst review)
- Natural language processing (attention visualization, saliency maps)

## Trade-offs

- Simpler explanations are more digestible but may be less faithful
- Global methods may not reflect local behaviour for specific inputs
- Post-hoc explanation fidelity is often untested; different methods can disagree on feature importance rankings

## Links

- [[shap_and_feature_attribution|SHAP and Feature Attribution]]
- [[partial_dependence_ice|PDP and ICE]]
- [[fairness_metrics|Fairness Metrics]]
- [[model_risk_considerations|Model Risk Considerations]]
- [[02_modeling/index|Modeling]]
- [[01_foundations/05_statistical_learning_theory/bias_variance_analysis|Bias–Variance Analysis — understanding model error sources]]
- [[01_foundations/03_probability_and_statistics/probability_theory|Probability Theory — conditional expectation, SHAP foundation]]
