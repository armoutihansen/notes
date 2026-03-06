---
layer: 07_applications
type: application
status: growing
tags: [workflow, classification, tabular]
created: 2026-03-11
---

# Claims Automation

## Problem

Automate the end-to-end processing of insurance claims — from First Notice of Loss (FNOL) through investigation, reserve-setting, triage, and settlement — reducing cycle time, operational cost, and inconsistency while maintaining fairness and regulatory compliance. Manual claims handling is slow (average cycle time 14–30 days), expensive (40–60% of claims cost is handling cost in some lines), and inconsistent (adjuster judgement variance). Automation addresses high-volume, low-complexity claims; human adjusters focus on complex and disputed cases.

## Users / Stakeholders

| Role | Impact |
|---|---|
| Claims adjuster | Handle fewer routine claims; focus on complex cases |
| Customer (claimant) | Faster settlement; digital-first experience |
| Claims operations manager | Reduced handling cost; SLA compliance |
| Reserve actuary | Consistent, model-driven case reserves |
| Compliance / Conduct team | Fair treatment of customers; regulatory reporting |

## Domain Context

- **Straight-through processing (STP)**: The goal is automated FNOL → settlement for eligible claims without human touch. UK motor: 40–70% STP achievable for low-severity physical damage claims.
- **FCA Consumer Duty (UK)**: Requires fair outcomes for customers. Automated decisions must be explainable. Discrimination on protected characteristics is illegal. Model audits expected.
- **Fraud in the loop**: 10–20% of claims have a fraud indicator. Automation must incorporate fraud scoring. Fraud checks must not slow legitimate claims.
- **Document intelligence**: FNOL documentation (photos, police reports, medical records) needs automated extraction. See [[07_applications/07_multimodal_systems/document_intelligence|Document Intelligence]].
- **Reserve accuracy**: Actuarial regulators require adequate reserves. Automated reserves must be validated against historical development patterns.
- **Lines of business**: Motor, property, liability, and health have very different claim types, data availability, and automation potential. Motor physical damage has highest automation rates; liability is lowest.

## Inputs and Outputs

**FNOL inputs**:
```
Policy: policy_id, coverage_type, limits, deductibles, policy_age, prior_claims
Event: event_type, event_date, location, reported_loss_amount, cause_code
Claimant: age, claim_history, contact_preference
Documents: photos, police_report, damage_estimate (extracted)
Third-party: third_party_insurer, repair_network_status
```

**Process outputs at each stage**:
```
FNOL:     coverage_confirmed, claim_id_assigned, fraud_score, triage_tier
Triage:   predicted_severity, recommended_track (STP / adjuster / specialist)
Reserve:  case_reserve_amount, reserve_confidence
Settlement: settlement_recommendation, authority_level, payment_amount
```

## Decision or Workflow Role

```
FNOL received (digital portal / phone / email)
  ↓
Coverage verification: policy lookup + rules check
  ↓
Fraud scoring: P(fraud) from transaction + network signals
  ↓
Triage model: severity prediction → route assignment
  │
  ├── LOW severity + LOW fraud risk → STP track:
  │     automated settlement offer → customer accepts → payment
  │
  ├── MEDIUM → standard adjuster track:
  │     reserve set → adjuster assigned → investigation → settlement
  │
  └── HIGH severity / HIGH fraud risk → specialist track:
        SIU referral and/or senior adjuster + EL legal team
  ↓
Settlement outcome → training data for models
```

## Modeling / System Options

| Component | Approach | Notes |
|---|---|---|
| Fraud scoring | XGBoost + network features | High recall requirement |
| Severity prediction | Gamma GLM (actuarial) + LightGBM (operational triage) | Dual model |
| Document extraction | Azure Document Intelligence / GPT-4V | Form type drives choice |
| Reserve setting | GLM-based chain ladder + ML case reserve | Regulatory: actuarial sign-off |
| Settlement recommendation | Rules + regression | Must be explainable |
| Conversational FNOL | LLM chatbot with structured extraction | NLP to structured claim record |

## Deployment Constraints

- **Regulatory**: All automated decisions subject to FCA review. Document model assumptions, validation, and outcomes monitoring. Annual model review.
- **Explainability**: Customer has right to understand why claim was routed or settled at a given amount. SHAP reason codes required.
- **Escalation**: Any automated decision must have a clear human escalation path. "Computer says no" without appeal route violates Consumer Duty.
- **Integration**: Must integrate with core claims management system (Guidewire, Duck Creek, or legacy). APIs and data contracts are critical.

## Risks and Failure Modes

| Risk | Description | Mitigation |
|---|---|---|
| **Customer detriment** | Automated settlement undervalues legitimate claim | Conservative settlement thresholds; sampling audit |
| **Fraud miss** | STP approves fraudulent claim | Fraud model recall target; post-payment audit sample |
| **Regulatory non-compliance** | Model discriminates on protected characteristic | Annual disparate impact audit |
| **System failure** | Automation down → massive backlog | Manual override mode; SLA monitoring |
| **Reserve inadequacy** | Automated reserves systematically low → capital shortfall | Monthly reserve development analysis |

## Success Metrics

| Metric | Target | Notes |
|---|---|---|
| STP rate | 40–70% (motor physical damage) | Varies significantly by line |
| Average handling cost reduction | > 30% | vs manual baseline |
| Average cycle time | < 5 days (STP) | Customer experience KPI |
| Customer satisfaction (NPS) | > 60 | Claimant experience |
| Fraud detection rate | > 85% | Fraud model KPI |
| Reserve adequacy ratio | 95–105% | Actuarial KPI |

## References

- Insurance Europe (2023). *AI in Insurance: Use Cases and Regulatory Landscape.*
- FCA (2022). *Consumer Duty: Final Rules and Guidance.*

## Links

**Modeling**
- [[03_modeling/01_supervised_learning/01_linear_and_glm/linear_and_glm|Linear and GLMs]] — Gamma GLM for reserves
- [[03_modeling/01_supervised_learning/02_tree_based_models/gradient_boosting|Gradient Boosting]] — fraud scoring, severity prediction

**Application Cross-links**
- [[07_applications/04_classification_and_decisioning/claim_severity_prediction|Claim Severity Prediction]]
- [[07_applications/03_detection_and_monitoring/fraud_detection|Fraud Detection]]
- [[07_applications/07_multimodal_systems/document_intelligence|Document Intelligence]]

**Reference Implementations**
- [[08_reference_implementations/01_model_implementations/linear_models_implementation|Linear Models Implementation]]
- [[08_reference_implementations/01_model_implementations/tree_ensembles_implementation|Tree Ensembles Implementation]]

**Adjacent Applications**
- [[underwriting_support|Underwriting Support]]
