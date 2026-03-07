---
layer: 07_applications
type: application
status: growing
tags: [regression, tabular, classification]
created: 2026-03-11
---

# Claim Severity Prediction

## Problem

Estimate the expected cost of an insurance claim at FNOL (First Notice of Loss) or during the claims triage process. Severity models inform: reserving (how much capital to set aside), triage (route to fast-track vs specialist handling), settlement negotiation, and fraud detection. The claim severity distribution is highly skewed — most claims are small, but a small proportion of complex claims account for the majority of total cost.

Typically modelled as a two-part model:
1. **Severity | claim is open**: Predicted ultimate cost given a claim has been filed
2. **Pure premium** = Frequency × Severity (combined with frequency model for pricing)

## Users / Stakeholders

| Role | Decision |
|---|---|
| Claims adjuster | Triage priority; settlement authority level |
| Reserve actuary | Case reserve adequacy; IBNR estimation |
| Claims manager | Resource allocation; SLA management |
| Pricing actuary | Pure premium for product pricing |
| SIU (Special Investigations Unit) | Severity outliers as fraud signals |

## Domain Context

- **Actuarial tradition**: Severity modelling uses GLMs with log-link (Gamma distribution for continuous severity; Tweedie for combined frequency-severity). Regulatory capital requires actuarial certification of reserves.
- **IBNR**: Incurred But Not Reported claims are estimated statistically from development triangles. ML supplements but rarely replaces chain-ladder methods for reserving.
- **Long-tailed lines**: Workers' comp, liability, and casualty claims may take 5–10 years to fully develop. Early severity estimates have high uncertainty.
- **Covariates available at FNOL**: Only limited information is available at First Notice of Loss (cause code, reported injury type, claimant demographics, policy). More features become available as investigation proceeds.
- **Regulatory**: Insurance is regulated at state/country level. Rate filings may require actuarial approval. Model documentation standards (ASOP 56 — Modeling in the US). GDPR applies to personal data in EU.
- **Data**: Closed claims with known ultimate are training labels. Open claims are right-censored. Severity is zero-inflated (small claims settled at zero after investigation).

## Inputs and Outputs

**FNOL features**:
```
Claim: cause_code, injury_type_code, body_part_code, accident_description_nlp
Policy: coverage_type, limit, deductible, policy_age, prior_claims_count
Claimant: age, occupation_class, injury_severity_index
Location: state, urban/rural, jurisdiction_factor
Context: day_of_week_filed, time_to_report (days), represented_by_attorney_flag
Prior development: initial_reserve_set, n_medical_visits, litigation_flag
```

**Output**:
```
predicted_severity:  Expected ultimate incurred cost
severity_percentile: Position in severity distribution (e.g., P75 = complex claim)
triage_tier:         FAST_TRACK / STANDARD / COMPLEX / LARGE_LOSS
reserve_recommendation: Point estimate + uncertainty range for reserving
fraud_flag_score:    Severity outlier relative to expected (feeds SIU)
```

## Decision or Workflow Role

```
FNOL received → initial triage model scores claim
  ↓
FAST_TRACK (< P50 severity, no red flags) → automated settlement flow
STANDARD → assigned adjuster with case reserve guidance
COMPLEX (> P90 severity) → specialist team with higher authority
LARGE_LOSS → executive escalation + dedicated team
  ↓
Claim develops → feature updates → reserve updates
  ↓
Claim closes → actual = training label → model refresh
  ↓
Severity model feeds pricing actuaries via pure premium calculation
```

## Modeling / System Options

| Approach | Strength | Weakness | When to use |
|---|---|---|---|
| GLM Gamma with log-link | Actuarially validated; interpretable; regulator-familiar; handles skewed distribution | Misses interactions; manual feature engineering | Primary model for regulated reserving; pricing filings |
| Tweedie regression | Handles combined zero-inflated frequency × severity in one model | Less interpretable than two-part model | Pure premium for pricing |
| LightGBM | Captures non-linear interactions; higher predictive accuracy | Requires careful calibration; model risk overhead | Triage decisions (non-regulatory); fraud scoring |
| Quantile regression | Provides prediction intervals (e.g., P10/P50/P90) | Does not model full distribution | Reserving uncertainty quantification |
| Log-normal OLS | Simple; analytically tractable | Smearing correction required for back-transform | Quick baseline |
| Neural network | Highest raw accuracy on large datasets | Black box; actuarial acceptance barriers | Large direct carriers with 1M+ claims; challenger |

**Recommended**: Gamma GLM for actuarial/regulatory use cases. LightGBM for triage and operational routing. Quantile regression for reserving uncertainty.

## Deployment Constraints

- **Latency**: Triage decision at FNOL should complete in <5 seconds. Batch reserve updates: overnight run.
- **Interpretability**: Adjusters need to understand why a claim was scored as complex. SHAP reason codes are useful for high-severity flags.
- **Actuarial certification**: In many jurisdictions, reserve models require sign-off by a qualified actuary. ML models need documentation and validation that meets ASOP standards.
- **Update cadence**: Pricing models retrained annually (or at rate filing). Triage models can be retrained quarterly.

## Risks and Failure Modes

| Risk | Description | Mitigation |
|---|---|---|
| **Right-censoring** | Open claims not settled yet; unknown true ultimate | Develop only on settled claims; truncation correction |
| **Attorney representation bias** | Attorney involvement inflates severity; proxy for litigation jurisdiction | Explicit feature; jurisdiction-specific models |
| **Reserve setting incentive** | Adjusters under-reserve to hit targets; corrupts training labels | Independent actuarial label review |
| **Distribution shift** | Inflation, legal environment changes, pandemic effects | Periodic recalibration; economic index adjustment |
| **Small sample in specialty lines** | Niche products have few claims | Pooling with similar lines; credibility weighting |

## Success Metrics

| Metric | Target | Notes |
|---|---|---|
| Reserve adequacy ratio | 95–105% of ultimate | Actuarial KPI; ratio of predicted to actual |
| MAE (log-scale) | < 0.3 log-£ | Model accuracy on held-out settled claims |
| Triage accuracy | > 85% correct tier | Fraction of complex claims correctly routed |
| SIU referral precision | > 30% confirmed fraud | Of claims flagged, fraction with confirmed fraud |
| Adjuster satisfaction | > 4.0/5 tool rating | Model utility for frontline users |

## References

- Frees, E.W. et al. (2014). *Predictive Modeling Applications in Actuarial Science.* Cambridge University Press.
- Tweedie, M.C.K. (1984). *An index which distinguishes between some important exponential families.*

## Links

**Modeling**
- [[03_modeling/01_supervised_learning/01_linear_and_glm/linear_and_glm|Linear and GLMs]] — Gamma GLM, Tweedie regression
- [[03_modeling/01_supervised_learning/02_tree_based_models/gradient_boosting|Gradient Boosting]] — LightGBM for triage

**Reference Implementations**
- [[03_modeling/01_supervised_learning/01_linear_and_glm/linear_models_implementation|Linear Models Implementation]]
- [[03_modeling/01_supervised_learning/02_tree_based_models/tree_ensembles_implementation|Tree Ensembles Implementation]]

**Adjacent Applications**
- [[credit_scoring|Credit Scoring]]
- [[07_applications/08_domain_verticals/01_insurance/claims_automation|Claims Automation]]
- [[07_applications/08_domain_verticals/01_insurance/underwriting_support|Underwriting Support]]
- [[07_applications/03_detection_and_monitoring/fraud_detection|Fraud Detection]]
