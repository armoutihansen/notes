---
layer: 07_applications
type: application
status: growing
tags: [classification, tabular, regression]
created: 2026-03-11
---

# Credit Scoring

## Problem

Estimate the probability that a loan applicant or existing borrower will default on a credit obligation within a specified time horizon (typically 12 or 24 months). The score is used to make binary approval/rejection decisions, set interest rates, determine credit limits, and manage portfolio risk. Credit scoring is one of the oldest and most regulated uses of statistical modelling — the Equal Credit Opportunity Act (ECOA) and Fair Credit Reporting Act (FCRA) impose strict constraints on model design, data use, and explanation obligations.

## Users / Stakeholders

| Role | Decision |
|---|---|
| Underwriter / credit analyst | Approve, decline, or counter-offer loan applications |
| Risk officer | Set scorecards, approval thresholds, risk appetite |
| Collections team | Prioritise collection efforts on delinquent accounts |
| Capital markets | Price securitised loan portfolios |
| Compliance officer | Ensure fair lending; adverse action notice obligations |
| Borrower | Understands why application was declined |

## Domain Context

- **Regulatory framework**: 
  - US: ECOA (no discrimination on protected classes), FCRA (adverse action notice requirements), model risk management guidance (SR 11-7)
  - EU: GDPR Article 22 (right to explanation for automated decisions), CRR/Basel III (regulatory capital implications)
  - UK: FCA Consumer Duty
- **Scorecard tradition**: Classic credit scoring uses logistic regression with Weight-of-Evidence binning — not because it's optimal, but because it's interpretable, auditable, and understood by regulators.
- **Adverse action notices**: If a loan is declined using an automated model, the applicant has a legal right to know the top reasons (typically top 4 reason codes). These must be human-readable.
- **Protected class proxies**: Zip code can be a proxy for race. Geographic variables require careful fairness analysis (disparate impact testing).
- **Through-the-cycle vs point-in-time**: Regulatory capital models use through-the-cycle PDs (stable). Risk-based pricing uses point-in-time PDs (current economic conditions). Different model calibration objectives.
- **Data access**: In traditional credit, data comes from bureau (Experian, Equifax, TransUnion) — structured, reliable, regulated. Alternative credit scoring (thin-file borrowers) uses non-traditional signals (rent, utility payments, telco, banking behaviour) — less regulated, higher fairness risk.

## Inputs and Outputs

**Traditional bureau-based features**:
```
Payment history: delinquency_count_12m, worst_delinquency_ever, times_90d_late
Utilisation: revolving_balance / revolving_limit (keep < 30%)
Length of credit history: oldest_account_age_months, avg_account_age
New credit: hard_inquiries_12m, new_accounts_6m
Credit mix: n_revolving_accounts, n_installment_accounts, n_mortgage_accounts
Negative marks: bankruptcies, tax_liens, civil_judgements
Income / DTI: income_reported, debt_to_income_ratio (if application data)
```

**Output**:
```
pd_score:         Probability of Default ∈ [0, 1]
scorecard_score:  Scaled score (e.g., 300–850 FICO-style) for interpretability
decision:         APPROVE / DECLINE / MANUAL_REVIEW / COUNTER_OFFER
adverse_action_codes: Top 4 reasons for decline (human-readable)
risk_tier:        PRIME / NEAR_PRIME / SUBPRIME
```

## Decision or Workflow Role

```
Application submitted → bureau data pulled (hard inquiry)
  ↓
Identity verification → KYC / AML check
  ↓
Score model → PD estimate
  ↓
Decision rules:
  pd_score < 0.02  → AUTO-APPROVE at standard rate
  0.02–0.08        → APPROVE at risk-adjusted rate
  0.08–0.15        → MANUAL_REVIEW (underwriter judgement)
  pd_score > 0.15  → AUTO-DECLINE + adverse action notice
  ↓
Loan originated → performance monitoring → vintage analysis → model refresh
```

## Modeling / System Options

| Approach | Strength | Weakness | When to use |
|---|---|---|---|
| Logistic regression (WoE scorecard) | Fully interpretable; adverse action codes; regulator-familiar | Misses non-linearity; feature engineering labour | Regulated consumer credit; SR 11-7 compliance |
| XGBoost / LightGBM + SHAP | Higher accuracy; handles non-linearity; SHAP reason codes | Requires validation model risk framework | Challenger model; internal risk management |
| Survival analysis (Cox PH) | Models time-to-default; handles right-censoring properly | Less familiar to regulators; harder to explain | Portfolio-level default timing; stress testing |
| Neural network | Highest raw accuracy on large datasets | Black box by default; heavy model risk burden | Innovation programmes; challenger models |
| Rule-based expert scorecard | Auditable; no ML risk; regulator-approved | Manual, slow to update; misses complex patterns | Thin-file borrowers; regulatory fallback |

**Recommended for production**: WoE logistic regression as champion (regulatory compliance). LightGBM + SHAP as challenger (higher accuracy, SHAP satisfies adverse action code requirement).

## Deployment Constraints

- **Latency**: Instant online credit decisions require <500ms total pipeline (bureau pull + scoring + decisioning).
- **Interpretability**: Non-negotiable. Adverse action reason codes are a legal requirement in the US.
- **Model risk governance**: SR 11-7 (US) requires model documentation, independent validation, ongoing monitoring, and board-level oversight.
- **Fairness**: Disparate impact analysis on race, gender, age, national origin required. 4/5ths rule as a statistical test for disparate impact.
- **Model stability**: Scorecard PSI (Population Stability Index) > 0.25 triggers mandatory review.
- **Vintage analysis**: Loans take time to season. Model validation requires sufficient performance window (minimum 12 months).

## Risks and Failure Modes

| Risk | Description | Mitigation |
|---|---|---|
| **Disparate impact** | Model discriminates against protected classes indirectly | Regular disparate impact testing; proxy variable removal |
| **Model decay** | Economic cycle changes default behaviour | Vintage monitoring; through-the-cycle recalibration |
| **Population shift** | Origination strategy changes who applies | PSI monitoring; covariate shift testing |
| **Label contamination** | Loans approved by model → biased training data (rejects unknown) | Reject inference; credit bureau industry data pooling |
| **Champion model over-rejection** | Overly conservative threshold → revenue loss | Portfolio-level test with challenger; threshold optimisation |
| **Gaming** | Consumers manipulate credit signals for approval | Model hardening; lag features; bureau-verified data only |

## Success Metrics

| Metric | Target | Notes |
|---|---|---|
| Gini / AUC | > 0.70 (consumer credit) | Industry benchmark; higher is better |
| KS statistic | > 0.40 | Discriminatory power between goods/bads |
| PSI (stability) | < 0.10 | Population Stability Index over time |
| Default rate at approval | Within ±15% of forecast | Model calibration |
| Approval rate | Business KPI vs risk appetite | Tradeoff metric |
| Disparate impact ratio | > 0.80 (80% rule) | Fairness metric; legal threshold |

## References

- Anderson, R. (2007). *The Credit Scoring Toolkit.* Oxford University Press.
- Siddiqi, N. (2017). *Intelligent Credit Scoring.* Wiley.
- Board of Governors of the Federal Reserve (2011). *SR 11-7: Guidance on Model Risk Management.*

## Links

**Modeling**
- [[03_modeling/01_supervised_learning/01_linear_and_glm/linear_and_glm|Linear and GLMs]] — logistic regression, WoE scorecard
- [[03_modeling/01_supervised_learning/02_tree_based_models/gradient_boosting|Gradient Boosting]] — challenger model
- [[03_modeling/07_evaluation_and_model_selection/index|Evaluation and Model Selection]] — Gini, KS, calibration

**Reference Implementations**
- [[03_modeling/01_supervised_learning/01_linear_and_glm/linear_models_implementation|Linear Models Implementation]]
- [[03_modeling/07_evaluation_and_model_selection/interpretability_implementation|Interpretability Implementation]]
- [[03_modeling/01_supervised_learning/02_tree_based_models/tree_ensembles_implementation|Tree Ensembles Implementation]]

**Adjacent Applications**
- [[claim_severity_prediction|Claim Severity Prediction]]
- [[07_applications/03_detection_and_monitoring/fraud_detection|Fraud Detection]]
- [[07_applications/08_domain_verticals/01_insurance/underwriting_support|Underwriting Support]]
- [[07_applications/08_domain_verticals/02_finance/risk_portfolio_monitoring|Portfolio Risk Monitoring]]
