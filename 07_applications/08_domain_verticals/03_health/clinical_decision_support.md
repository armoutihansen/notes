---
layer: 07_applications
type: application
status: growing
tags: [classification, tabular, workflow]
created: 2026-03-11
---

# Clinical Decision Support

## Problem

Provide clinicians with ML-driven alerts, recommendations, and risk scores at the point of care to improve diagnostic accuracy, medication safety, and treatment adherence — without replacing clinical judgement. Examples: early warning scores (deterioration), drug-drug interaction alerts, diagnostic suggestion from lab results, radiology AI for preliminary reads.

## Users / Stakeholders

| Role | Decision |
|---|---|
| Clinician (doctor, nurse) | Treatment decision; diagnostic hypothesis |
| Clinical informatics team | System integration; alert calibration |
| Patient safety officer | Alert fatigue management; safety incident review |
| Hospital administrator | ROI on readmission reduction, length-of-stay |
| Regulatory / clinical governance | SaMD classification; evidence requirements |

## Domain Context

- **Alert fatigue**: Over-alerting is a major problem. Clinicians ignore systems with >50% false positive rate. Specificity matters enormously — even more than sensitivity for non-critical alerts.
- **GDPR Article 9**: Health data is special-category data requiring explicit consent or substantial public interest basis. Data minimisation mandatory.
- **SaMD regulation**: If the CDS system influences treatment decisions, it may be classified as Software as a Medical Device (SaMD) requiring regulatory approval (UKCA/CE marking, FDA 510(k) or PMA). Class II/III devices have extensive clinical evidence requirements.
- **Bias and equity**: Models trained on historical data often underperform for minority ethnic groups, women (historically underrepresented in clinical trials), and rare conditions. Subgroup performance reporting is essential.
- **Human-in-the-loop**: CDS is advisory, not autonomous. The clinician retains legal and professional responsibility. The system augments; it does not decide.
- **EHR integration**: Systems must integrate into existing clinical workflows (Epic, Cerner, MEDITECH). FHIR/HL7 interoperability standards are mandatory.

## Inputs and Outputs

**Early Warning Score (deterioration)**:
```
Vital signs: heart_rate, respiratory_rate, blood_pressure, SpO2, temperature
Lab values: lactate, WBC, creatinine, eGFR, troponin
Clinical: consciousness_level (AVPU/GCS), urine_output, age, diagnosis
Trajectory: trend over last 4/8/12 hours
```

**Output**:
```
deterioration_score:  National Early Warning Score (NEWS2) or ML-derived equivalent
risk_tier:            LOW / MEDIUM / HIGH / CRITICAL
recommended_action:   "Escalate to SpR", "Consider ITU review", "Routine monitoring"
contributing_factors: Top 3 clinical signals driving the score
```

## Decision or Workflow Role

```
EHR data update (new observation, lab result, medication)
  ↓
Real-time inference engine: score computation
  ↓
Score < threshold → no alert
Score > threshold → alert pushed to nurse call board / pager / EHR inbox
  ↓
Clinician reviews → acknowledges, escalates, or dismisses
  ↓
Outcome logged: did patient deteriorate? → validation dataset
  ↓
Monthly clinical review: alert precision/recall vs patient outcomes
```

## Modeling / System Options

| Approach | Strength | Weakness | When to use |
|---|---|---|---|
| NEWS2 (rule-based) | Clinically validated; regulatory accepted; interpretable | Not personalised; misses complex patterns | Standard baseline; regulatory pathway |
| Logistic regression on NEWS features | Improves on NEWS2; explainable | Limited feature set | Step-up from rules with minimal model risk |
| XGBoost + full EHR features | Higher AUC; captures comorbidities | Requires rigorous clinical validation | Secondary analysis; research; high-evidence context |
| LSTM on time series of vitals | Captures temporal trajectory | Training data requirements; explainability | ICU monitoring with rich streaming data |
| Federated learning | Privacy-preserving multi-site training | Complexity; communication overhead | Multi-hospital deployment without data sharing |

**Recommended**: NEWS2 as baseline (regulatory anchor). XGBoost with SHAP as enhancement layer where regulatory pathway is clear.

## Deployment Constraints

- **Regulatory pathway**: Determine SaMD classification early. Class IIa or higher requires clinical investigation. Plan 12–24 months for regulatory approval.
- **Explainability**: Clinicians must understand why an alert fired. "Black box" systems are not accepted in clinical governance.
- **Fail-safe**: If model is unavailable, fallback to NEWS2 rule-based calculation. Patient safety cannot depend on ML system uptime.
- **FHIR integration**: Use HL7 FHIR R4 for EHR data access. CDS Hooks standard for alert delivery.

## Risks and Failure Modes

| Risk | Description | Mitigation |
|---|---|---|
| **Alert fatigue** | Too many false positives → ignored | Precision tuning; alert bundling; suppress low-acuity wards |
| **Demographic bias** | Lower sensitivity for women, minorities | Subgroup evaluation; bias audit; representative training data |
| **Distribution shift** | Different patient population at deployment site | Site-specific calibration; local validation study |
| **EHR data quality** | Missing vitals, transcription errors → wrong score | Missing value handling; data quality alert |
| **Regulatory non-compliance** | Deployed without SaMD approval → legal liability | MHRA/FDA pre-submission consultation |

## Success Metrics

| Metric | Target | Notes |
|---|---|---|
| AUROC for deterioration | > 0.82 | Vs NEWS2 benchmark |
| Alert precision at high sensitivity | > 40% | Positive predictive value at 85% recall |
| Clinician acceptance rate | > 70% | Fraction of alerts acknowledged (not dismissed) |
| Length-of-stay reduction | Measurable decrease | Hospital operational outcome |
| Subgroup AUC parity | < 0.03 gap across demographics | Equity metric |

## References

- Redfern, O. et al. (2018). *Collecting data on early warning scores at scale.* BMJ Open.
- Rajpurkar, P. et al. (2022). *AI in health and medicine.* Nature Medicine.

## Links

**Modeling**
- [[03_modeling/01_supervised_learning/02_tree_based_models/gradient_boosting|Gradient Boosting]] — XGBoost for clinical scoring
- [[03_modeling/07_evaluation_and_model_selection/index|Evaluation and Model Selection]] — calibration, subgroup analysis

**Reference Implementations**
- [[03_modeling/01_supervised_learning/02_tree_based_models/tree_ensembles_implementation|Tree Ensembles Implementation]]
- [[03_modeling/07_evaluation_and_model_selection/interpretability_implementation|Interpretability Implementation]]

**Adjacent Applications**
- [[patient_readmission|Patient Readmission Risk]]
- [[07_applications/04_classification_and_decisioning/credit_scoring|Credit Scoring]]
