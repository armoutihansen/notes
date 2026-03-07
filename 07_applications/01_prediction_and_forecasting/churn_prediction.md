---
layer: 07_applications
type: application
status: growing
tags: [classification, tabular, forecasting]
created: 2026-03-11
---

# Churn Prediction

## Problem

Identify subscription customers who are at elevated risk of cancelling within a defined forward window (typically 30, 60, or 90 days) so that the business can intervene with retention offers, proactive customer success contact, or product improvements. Churn has high economic impact: acquiring a new customer costs 5–25× retaining an existing one, and CLV models depend heavily on retention rate assumptions.

The problem has two distinct variants:
1. **Voluntary churn**: Customer actively cancels or fails to renew — addressable through intervention
2. **Involuntary churn**: Payment failure (credit card expiry, insufficient funds) — addressable through payment recovery workflows

Most churn models target voluntary churn at the individual customer level.

## Users / Stakeholders

| Role | Decision driven by churn score |
|---|---|
| Customer success manager | Prioritise outreach queue; decide which at-risk accounts to call |
| Marketing manager | Design and trigger retention email/SMS campaigns |
| Product manager | Identify product friction signals correlated with churn |
| Finance | Revenue forecasting; cohort-based churn rate projections |
| Executive | Board-level NRR (Net Revenue Retention) reporting |

The primary consumer is an **operational CRM system** (Salesforce, HubSpot) that triggers automated workflows or populates a human review queue. Scores need to be refreshed weekly or daily depending on intervention latency.

## Domain Context

- **SaaS/subscription specifics**: Churn only observable at renewal date or explicit cancel event. Right-censoring: active customers who haven't yet churned are not churned yet — don't label them as non-churn until renewal window closes.
- **Class imbalance**: Monthly churn rates 2–8% in B2C, 0.5–3% in B2B SaaS. Severe class imbalance requires calibration. Precision-recall tradeoff dominates AUC as primary metric.
- **Intervention validity window**: Contacting a customer who is 5 minutes from churning is too late. The model must predict far enough in advance for intervention to be effective (typically 30–90 days).
- **GDPR / data minimisation**: Behavioural data (session logs, feature usage) may require explicit consent or legitimate interest basis. Some markets restrict automated individual-level retention scoring without human oversight.
- **B2B vs B2C**: B2B churn is account-level, influenced by champion departure (key user leaves), contract expansion/contraction, organisational changes. Multiple users per account — aggregate signals needed.

## Inputs and Outputs

**Feature categories**:
```
Recency: days_since_last_login, days_since_last_purchase, days_to_renewal
Frequency: logins_last_30d, sessions_last_90d, feature_use_count
Monetary: mrr, total_spend_12m, discount_depth, payment_method_type
Engagement: nps_score, support_tickets_open, tickets_last_30d, email_open_rate
Product usage: feature_A_activations, api_calls, reports_generated, integrations_active
Account health: seats_utilised / seats_purchased, admin_last_login, sso_enabled
Lifecycle: months_since_signup, plan_type, last_plan_change_direction, contract_end_date
```

**Output**:
```
churn_probability:   P(churn within 30/60/90 days) ∈ [0, 1]
churn_risk_tier:     HIGH / MEDIUM / LOW  (operational segmentation)
top_churn_reasons:   SHAP top-3 features driving prediction (for CSM context)
recommended_action:  RETENTION_CALL / EMAIL_CAMPAIGN / NO_ACTION
```

## Decision or Workflow Role

```
Weekly batch score refresh (or daily for high-value accounts)
  ↓
Scores pushed to CRM (Salesforce / HubSpot) via API
  ↓
HIGH risk (P > 0.4) → CS team action queue + manager alert
MEDIUM risk → automated email sequence triggered
LOW risk → no action
  ↓
Intervention outcome logged → feedback into training data
  ↓
Holdout control group → measure lift of intervention
```

Churn prediction is an **input to a tiered intervention strategy**. The economic value is measured as revenue saved by successful retention, minus intervention cost.

## Modeling / System Options

| Approach | Strength | Weakness | When to use |
|---|---|---|---|
| Logistic regression + feature engineering | Highly interpretable, fast, calibrated by default | Misses interaction effects | Early stage, limited data, high interpretability requirement |
| LightGBM / XGBoost | Handles interactions, missing values, fast at scale | Requires calibration; less interpretable without SHAP | Standard production choice; tabular data |
| Survival analysis (Cox PH) | Models time-to-event properly; handles right-censoring | Proportional hazards assumption; harder to operationalise | When timing of churn matters as much as whether it happens |
| Neural embedding model | Captures sequence of events; cold-start via embeddings | Data-hungry; complex deployment | High data volume; when event sequence is the key signal |
| Rules-based + ML hybrid | Instant wins from obvious patterns; explainable | Fragile; rule proliferation | Regulated environments where black boxes not permitted |

**Recommended**: LightGBM with SHAP explanations for score reasoning. Survival analysis (Kaplan-Meier + Cox PH) for cohort-level churn rate forecasting in finance reporting. Calibrate with Platt scaling.

See [[03_modeling/01_supervised_learning/02_tree_based_models/tree_ensembles_implementation|Tree Ensembles Implementation]] for code.

## Deployment Constraints

- **Latency**: Batch weekly/daily — not real-time. Inference time not a constraint.
- **Interpretability**: CSMs need to understand *why* a customer is flagged. SHAP top-3 features per customer are mandatory. "Model says so" is not actionable.
- **Calibration**: Scores are shown to business users as probabilities. A poorly calibrated model that outputs 0.8 for 40% of customers destroys trust. Evaluate and enforce calibration via reliability diagrams.
- **Fairness**: Scores must not systematically disadvantage customers based on demographic proxies (geography, language, company size) in ways that would constitute discriminatory service.
- **Volume**: Typically 10K–1M customer records. Sub-second batch inference. CRM integration via REST API or Salesforce batch upload.
- **Model update cadence**: Retrain monthly on new labels. Deploy when validation AUC-PR ≥ current champion model.

## Risks and Failure Modes

| Risk | Description | Mitigation |
|---|---|---|
| **Label leakage** | Using features computed after churn event | Strict point-in-time cutoff; validate with time-based split |
| **Selection bias in interventions** | Past interventions change who would have churned — biases labels | Track who received interventions; use counterfactual evaluation |
| **Right-censoring** | Active customers who will churn in future labeled as non-churn | Survival analysis; time-windowed binary labels only |
| **Survivorship bias** | High-engagement customers are over-represented in old cohorts | Cohort-stratified training data |
| **Intervention fatigue** | Over-contacting medium-risk customers reduces campaign effectiveness | Score-gated throttle in CRM; holdout experiment design |
| **Class imbalance** | Low churn rate → model optimises for majority class | Use AUC-PR not AUC-ROC; class weights; calibration post-training |
| **Proxy discrimination** | Small company size or geography as churn proxy → unfair service | Fairness audit; remove sensitive proxies; equalised odds check |

## Success Metrics

| Metric | Target | Notes |
|---|---|---|
| AUC-PR | > 0.45 (vs base rate ~0.05) | Better than AUC-ROC for imbalanced problems |
| Precision@top 10% | > 2× lift over random | Are the highest-risk customers actually churning? |
| Calibration (ECE) | < 0.05 | Scores must be usable as actual probabilities |
| Revenue retained | > 3× intervention cost | True business ROI; requires experiment with holdout |
| CSM adoption rate | > 70% act on flagged accounts | Model utility metric; low adoption means scores aren't trusted |
| Churn rate reduction | 5–20% relative reduction | Ultimate KPI; may take 6–12 months to measure reliably |

## References

- Hadden, J. et al. (2007). *Computer assisted customer churn management: State-of-the-art and future trends.* Computers & Operations Research.
- Verbeke, W. et al. (2012). *New insights into churn prediction in the telecommunication sector: A profit driven data mining approach.* European Journal of Operational Research.

## Links

**Modeling**
- [[03_modeling/01_supervised_learning/02_tree_based_models/gradient_boosting|Gradient Boosting]] — LightGBM for tabular classification
- [[03_modeling/07_evaluation_and_model_selection/index|Evaluation and Model Selection]] — AUC-PR, calibration, threshold selection

**ML Engineering**
- [[05_ml_engineering/06_deployment_and_serving/index|Deployment and Serving]] — CRM integration, batch scoring
- [[05_ml_engineering/07_monitoring_and_observability/index|Monitoring and Observability]] — prediction drift, label freshness

**Reference Implementations**
- [[03_modeling/01_supervised_learning/02_tree_based_models/tree_ensembles_implementation|Tree Ensembles Implementation]]
- [[08_implementations/02_end_to_end_examples/tabular_classification_pipeline|Tabular Classification Pipeline]]

**Adjacent Applications**
- [[demand_forecasting|Demand Forecasting]]
- [[07_applications/04_classification_and_decisioning/credit_scoring|Credit Scoring]]
- [[07_applications/08_domain_verticals/04_ecommerce/personalized_pricing|Personalized Pricing]]
