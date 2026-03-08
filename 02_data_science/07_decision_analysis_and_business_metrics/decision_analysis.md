---
layer: 02_data_science
type: concept
status: seed
tags: [workflow, evaluation]
created: 2026-05-31
---

# Decision Analysis and Business Metrics

## Problem Context

A model with high AUC-ROC can still add zero business value if it is evaluated, thresholded, or deployed without considering the costs and benefits of each decision outcome. Decision analysis bridges the gap between statistical performance and operational utility.

## Analytical Goal

Choose a decision policy — typically a classification threshold — and success metrics that maximise (or satisfy) business value, accounting for the asymmetric costs of different error types.

## Data Considerations

The inputs to decision analysis are:
- A calibrated model that outputs probabilities $\hat{p} = P(y=1 \mid \mathbf{x})$.
- Estimates of the **cost matrix**: cost of false positives ($C_{FP}$) and false negatives ($C_{FN}$), and the benefit of true positives ($B_{TP}$).
- The **base rate** $\pi = P(y=1)$ in the deployment population.

If the model is not calibrated, the probabilities cannot be interpreted as true probabilities and the cost-benefit calculation will be wrong. Calibrate with Platt scaling or isotonic regression before decision analysis (see [[03_modeling/07_evaluation_and_model_selection/evaluation_and_validation|Evaluation and Validation]]).

## Method / Approach

### Expected Value Framework

At threshold $\tau$, predict $\hat{y} = 1$ iff $\hat{p} \geq \tau$. The **expected value per prediction** is:

$$
EV(\tau) = B_{TP} \cdot P(\hat{y}=1, y=1) - C_{FP} \cdot P(\hat{y}=1, y=0) - C_{FN} \cdot P(\hat{y}=0, y=1)
$$

Expanded using the confusion matrix probabilities:

$$
EV(\tau) = B_{TP} \cdot \text{TPR}(\tau) \cdot \pi - C_{FP} \cdot \text{FPR}(\tau) \cdot (1-\pi) - C_{FN} \cdot (1-\text{TPR}(\tau)) \cdot \pi
$$

The **optimal threshold** maximises $EV(\tau)$; it generally differs from 0.5.

**Special case — equal cost of FP and FN**: optimal threshold is 0.5. In most real problems costs are asymmetric.

**Example (fraud detection):**
- $B_{TP}$: fraudulent transaction amount saved ≈ £250
- $C_{FP}$: customer friction from blocked legitimate transaction ≈ £5
- $C_{FN}$: undetected fraud ≈ £250 (full loss)
- Optimal threshold much lower than 0.5 (false negatives are expensive).

### Threshold Selection in Practice

1. Compute precision–recall curve and F1 over the validation set.
2. Plot $EV(\tau)$ as a function of $\tau$ using estimated cost matrix.
3. Select $\tau^* = \argmax_\tau EV(\tau)$.
4. Validate stability of $\tau^*$ with sensitivity analysis (vary cost estimates ±50%).

```python
import numpy as np
from sklearn.metrics import precision_recall_curve

y_prob = model.predict_proba(X_val)[:, 1]
precisions, recalls, thresholds = precision_recall_curve(y_val, y_prob)

B_TP, C_FP, C_FN = 250, 5, 250
pi = y_val.mean()

ev = (B_TP * recalls[:-1] * pi
      - C_FP * (1 - precisions[:-1]) / precisions[:-1] * recalls[:-1] * pi
      - C_FN * (1 - recalls[:-1]) * pi)

optimal_threshold = thresholds[np.argmax(ev)]
```

### Metric Alignment

Choose the primary evaluation metric to match the deployment decision:

| Deployment context | Appropriate metric |
|---|---|
| Ranking / prioritisation (top-k alerts) | Precision @ K, Average Precision |
| Binary accept/reject at fixed threshold | F1, G-mean, expected value |
| Probabilistic risk score | Brier score, log-loss, calibration curve |
| Imbalanced class, recall critical | PR-AUC, F-beta (β > 1) |
| Regulatory requirement on FPR | TPR @ fixed FPR |

**Goodhart's Law:** once a metric becomes a target, it ceases to be a good measure. Define a primary metric for optimisation and secondary metrics as guardrails; monitor both.

### Cost-Benefit Analysis for Model Comparison

When comparing Model A vs Model B, compute:

$$
\Delta EV = EV_A(\tau^*_A) - EV_B(\tau^*_B)
$$

Both models should be evaluated at their own optimal thresholds, not a shared one.

## Validation / Risks

- **Cost matrix uncertainty**: rarely known precisely; perform sensitivity analysis.
- **Base rate shift**: deployment population may have different $\pi$ than training data; decision thresholds may need recalibration.
- **Proxy metrics**: offline metrics (AUC, F1) are proxies for business value; validate with A/B test before declaring model improvement.
- **Feedback loops**: decisions affect future data (e.g., denied applicants don't appear in future training data) → account for counterfactual outcomes.

## Communication Notes

- Frame results in business terms: "the model prevents £1.2M in annual fraud losses at a 3% increase in blocked legitimate transactions."
- Present sensitivity analysis to show robustness to cost estimates.
- For regulatory audiences: show the fairness implications of threshold choices (see [[02_data_science/06_interpretability_and_communication/fairness_metrics|Fairness Metrics]]).

## References

- Provost, F. & Fawcett, T. (2013). *Data Science for Business*. O'Reilly. Chapters 7–8.
- Drummond, C. & Holte, R. (2006). "Cost curves: An improved method for visualizing classifier performance." *Machine Learning*.

## Links

- [[02_data_science/01_problem_framing/problem_framing|Problem Framing — success criteria, loss choice]]
- [[03_modeling/07_evaluation_and_model_selection/evaluation_and_validation|Evaluation and Validation — calibration, metrics]]
- [[02_data_science/06_interpretability_and_communication/fairness_metrics|Fairness Metrics — threshold fairness implications]]
- [[01_foundations/03_probability_and_statistics/probability_theory|Probability Theory — expected value, Bayes optimal decision]]
