---
layer: 02_modeling
type: concept
status: seed
tags: [interpretability, evaluation, safety]
created: 2026-03-02
---

# Fairness Metrics

## Definition

Quantitative criteria that operationalize notions of equitable treatment or equal impact across demographic groups or individuals in a machine learning system's predictions.

## Intuition

Fairness is multi-dimensional: should a model make equally accurate predictions for every group (equalized odds)? Should it select members of each group at the same rate (demographic parity)? Should predicted probabilities reflect true probabilities equally across groups (calibration)? These criteria are mathematically incompatible in general — choosing among them requires explicit value judgments about which type of fairness is most important in the deployment context.

## Formal Description

Let $A$ be a protected attribute (e.g., race, gender), $Y$ the true outcome, and $\hat Y$ the model prediction (or $\hat P$ the predicted probability).

---

### Group Fairness Criteria

**Demographic Parity (Statistical Parity)**

$$P(\hat Y = 1 \mid A = 0) = P(\hat Y = 1 \mid A = 1)$$

The positive prediction rate is equal across groups. Does not condition on true labels; can be satisfied even by a model that systematically misclassifies one group.

**Equalized Odds** (Hardt et al., 2016)

$$P(\hat Y = 1 \mid Y = y, A = 0) = P(\hat Y = 1 \mid Y = y, A = 1) \quad \forall\, y \in \{0, 1\}$$

Both true positive rate (TPR) and false positive rate (FPR) are equal across groups. A weaker variant, **equal opportunity**, requires only equal TPR (group members who deserve a positive outcome get it equally).

**Calibration (Predictive Parity)**

$$P(Y = 1 \mid \hat P = p, A = 0) = P(Y = 1 \mid \hat P = p, A = 1) \quad \forall\, p$$

Predicted probabilities mean the same thing regardless of group. Required for probability outputs used in decision support; critical in criminal risk assessment tools (COMPAS controversy).

**Impossibility Theorem** (Chouldechova 2017, Kleinberg et al. 2016): when base rates differ across groups, demographic parity, equalized odds, and calibration cannot all be simultaneously satisfied (except in degenerate cases). Choosing a metric encodes a normative judgment.

---

### Individual Fairness

Proposed by Dwork et al. (2012): similar individuals (according to a task-specific similarity metric $d$) should receive similar predictions:

$$d(x_i, x_j) < \epsilon \implies |\hat f(x_i) - \hat f(x_j)| < \delta$$

Requires defining a domain-appropriate similarity metric, which is itself a normative choice. Difficult to operationalize but avoids the group homogeneity assumption.

---

### Disparate Impact

A proxy metric from US employment law (80% rule): the selection rate for a protected group is at least 80% of the rate for the most-selected group. Used in EEOC guidelines and as a legal safe harbor.

$$\frac{\min_a P(\hat Y=1 \mid A=a)}{\max_a P(\hat Y=1 \mid A=a)} \geq 0.8$$

---

### Measuring and Mitigating Bias

**Pre-processing**: reweigh training examples, resampling (Reweighing algorithm, SMOTE variants for fairness)
**In-processing**: add fairness constraints or regularization terms to the training objective (adversarial debiasing, fairness-aware ERM)
**Post-processing**: threshold calibration per group after training (Hardt et al. 2016); simpler but does not address representation within the model

## Applications

- Credit scoring and lending (Fair Housing Act, ECOA)
- Hiring and HR systems (EEOC)
- Criminal justice risk assessment (COMPAS, bail decisions)
- Healthcare triage and resource allocation
- Advertising and recommendation systems

## Trade-offs

- **Fairness–accuracy tradeoff**: enforcing group parity constraints generally reduces overall accuracy; the size of this tradeoff depends on base rate differences and task correlation
- **Group vs. individual fairness**: group metrics may be satisfied while individual injustices persist; neither implies the other
- **Proxy discrimination**: removing protected attributes doesn't prevent discrimination through correlated proxies (e.g., ZIP code as a proxy for race)
- **Metric choice is normative**: no metric is universally "correct"; regulatory context and stakeholder values should guide selection

## Links

- [[interpretability_overview|Interpretability Overview]]
- [[model_risk_considerations|Model Risk Considerations]]
- [[02_modeling/index|Modeling]]
- [[01_foundations/03_probability_and_statistics/probability_theory|Probability Theory — conditional probability, calibration, disparate impact]]
- [[01_foundations/05_statistical_learning_theory/evaluation_metrics|Evaluation Metrics — precision, recall, AUC-ROC across groups]]
