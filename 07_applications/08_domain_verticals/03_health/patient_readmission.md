---
layer: 07_applications
type: application
status: growing
tags: [classification, tabular, forecasting]
created: 2026-03-11
---

# Patient Readmission Risk

## Problem

Predict which patients are at elevated risk of being readmitted to hospital within 30 days of discharge, enabling targeted interventions (community support, pharmacy follow-up, GP communication) that prevent avoidable readmissions. In the NHS, 30-day readmission rates are ~15% and are used as a quality indicator. In the US, CMS penalises hospitals for excess readmissions under the HRRP (Hospital Readmissions Reduction Program).

## Users / Stakeholders

| Role | Decision |
|---|---|
| Discharge coordinator | Identify high-risk patients needing enhanced discharge planning |
| Community care team | Prioritise post-discharge calls and home visits |
| Hospital administrator | Track readmission KPIs; CMS penalty avoidance |
| GP / primary care | Follow-up appointment scheduling |
| Patient / carer | Understand discharge instructions; support needs |

## Domain Context

- **Timing sensitivity**: The intervention window is at discharge. Score must be available before the patient leaves the ward. Real-time score at discharge event required.
- **Social determinants**: Readmission is strongly predicted by social factors (social isolation, housing instability, health literacy) that are often not in the EHR. Missing features limit model accuracy.
- **Condition specificity**: Readmission predictors differ by diagnosis. A single global model underperforms condition-specific models for high-prevalence conditions (heart failure, COPD, pneumonia).
- **Intervention effectiveness**: If no intervention is available or the patient declines support, the score is not actionable. Model should only flag patients where intervention is feasible.
- **GDPR / DSPT**: NHS Data Security and Protection Toolkit. Patient data must not leave NHS systems without IG approval.

## Inputs and Outputs

**Features**:
```
Clinical: primary_diagnosis (ICD code), Charlson comorbidity index, length_of_stay,
          procedure_codes, lab_values_at_discharge, medication_changes
Prior utilisation: n_admissions_12m, n_ED_visits_6m, n_GP_contacts_3m
Demographics: age, sex, IMD_deprivation_decile
Social: lives_alone, carer_present, care_package
Discharge: discharge_destination (home/care home/other), time_of_day_discharged
```

**Output**:
```
readmission_score:    P(readmission within 30 days) ∈ [0, 1]
risk_tier:            LOW / MEDIUM / HIGH
intervention_flag:    Recommended intervention type
top_risk_factors:     Clinician-interpretable explanation (SHAP top 3)
```

## Decision or Workflow Role

```
Discharge planning meeting (24–48h before discharge)
  ↓
EHR triggers readmission score computation
  ↓
HIGH risk → Enhanced discharge: community referral, pharmacy review, 
             GP alert, 48h follow-up call scheduled
MEDIUM → Standard: GP letter, patient discharge summary
LOW → Routine discharge
  ↓
30-day outcome tracked → confirmed readmission or not
  ↓
Monthly model performance review → recalibrate if AUC drifts
```

## Modeling / System Options

| Approach | Strength | Weakness |
|---|---|---|
| LACE+ Index (rule-based) | Validated; regulatory accepted; simple | Limited accuracy; no personalisation |
| Logistic regression | Interpretable; fast; calibrated | Misses complex comorbidity interactions |
| LightGBM | Higher AUC; handles ICD code sparsity | Less interpretable; more validation effort |
| Condition-specific ensemble | Best performance; handles heterogeneity | Complex to maintain (one model per condition) |

**Recommended**: LACE+ as baseline. LightGBM for high-volume conditions. SHAP explanations mandatory.

## Deployment Constraints

- **Real-time scoring**: Score must be available at discharge. EHR trigger-based computation.
- **NHS IG**: Data must stay within NHS environment. On-premises deployment only.
- **Equity**: Score must not disadvantage protected groups. Deprivation is a predictor — monitoring required to ensure interventions are equitably distributed.

## Risks and Failure Modes

| Risk | Description | Mitigation |
|---|---|---|
| **Demographic bias** | Model higher FPR for ethnic minorities | Subgroup AUC monitoring; representative training data |
| **Label definition** | Readmission vs planned readmission — planned should be excluded | Careful label curation; exclude planned admissions |
| **Social factor blindspot** | High-risk social factors not in EHR → missed predictions | Structured social data collection at admission |
| **Intervention capacity** | High-risk list longer than team can handle | Tiered intervention; capacity-constrained prioritisation |

## Success Metrics

| Metric | Target | Notes |
|---|---|---|
| AUROC | > 0.75 | vs LACE+ benchmark ~0.65 |
| 30-day readmission rate reduction | > 10% relative | Intervention group vs control |
| High-risk recall | > 80% | Fraction of readmitted patients flagged |
| Alert actionability | > 60% result in documented intervention | Not just flagged; acted upon |
| Equity: AUC gap | < 0.03 across demographic groups | Fairness metric |

## References

- van Walraven, C. et al. (2010). *Derivation and Validation of an Index to Predict Early Death or Unplanned Readmission.*
- Rajkomar, A. et al. (2018). *Scalable and accurate deep learning with electronic health records.* npj Digital Medicine.

## Links

**Modeling**
- [[03_modeling/01_supervised_learning/02_tree_based_models/gradient_boosting|Gradient Boosting]] — LightGBM classification
- [[03_modeling/07_evaluation_and_model_selection/index|Evaluation and Model Selection]] — calibration, fairness

**Reference Implementations**
- [[08_reference_implementations/01_model_implementations/tree_ensembles_implementation|Tree Ensembles Implementation]]
- [[08_reference_implementations/01_model_implementations/interpretability_implementation|Interpretability Implementation]]

**Adjacent Applications**
- [[clinical_decision_support|Clinical Decision Support]]
- [[07_applications/01_prediction_and_forecasting/churn_prediction|Churn Prediction]]
