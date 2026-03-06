---
layer: 07_applications
type: application
status: growing
tags: [classification, regression, tabular]
created: 2026-03-11
---

# Underwriting Support

## Problem

Assist human underwriters in assessing risk and determining appropriate terms (accept/decline, premium, coverage conditions) for insurance applications — particularly in commercial and specialty lines where risks are complex, heterogeneous, and require expert judgement. ML models augment underwriter decision-making rather than replacing it: surfacing risk signals, generating recommended premiums, flagging inconsistencies, and automating routine renewals.

## Users / Stakeholders

| Role | Decision |
|---|---|
| Underwriter | Accept/decline risk; set premium; apply endorsements |
| Underwriting manager | Portfolio quality monitoring; underwriter performance |
| Pricing actuary | Model performance vs technical price |
| Distribution (broker) | Quote turnaround speed; competitiveness |
| Compliance / Conduct | Fair pricing; prohibited rating factors |

## Domain Context

- **Judgement-heavy lines**: Commercial property, liability, D&O, marine — each risk is unique. Models provide signals; underwriter makes final decision. Model must earn underwriter trust.
- **Technical price vs market price**: The model outputs a technically appropriate premium based on expected loss. Underwriters adjust up or down based on market conditions, relationship, and judgement.
- **Protected characteristics**: UK: EHRC guidance prohibits rating on age for some products. EU: Gender Directive prohibits gender-based pricing. Models must be audited for proxy discrimination.
- **Data richness varies**: Personal lines (motor, home) have millions of data points. Commercial specialty (D&O, trade credit) may have thousands of claims over 20 years. Model complexity must match data availability.
- **Competitive intelligence**: Underwriters need to know if the technical price is competitive in the market. Pricing model output is one input; market positioning is another.
- **Actuarial standards**: ASOP 56 (US), APS X3 (UK) — actuarial professional standards for insurance modelling. Pricing models require actuarial validation.

## Inputs and Outputs

**Personal lines (motor)**:
```
Driver: age, gender (where permitted), occupation, driving history, NCB
Vehicle: make, model, engine_size, year, value, usage_type
Location: postcode density, theft_rate, flood_zone
Policy: coverage_type, voluntary_excess, named_drivers
Telematics (optional): mileage, time_of_day, braking_behaviour
```

**Commercial lines**:
```
Insured: SIC code, revenue, employees, years_in_business, public/private
Risk: premises type, construction, occupation, geographic_spread
Claims history: prior 5-year claims by peril and amount
Financial health: credit score, CCJs, director changes
Exposure: sum_insured, turnover_insured, contractual_liability
```

**Output**:
```
technical_premium:    Model-derived expected loss cost + expense + profit margin
risk_grade:           A / B / C / D / DECLINE (risk quality tier)
premium_range:        Acceptable range for underwriter negotiation
risk_flags:           Specific concerns (high claims freq, flood zone, concentration)
comparable_risks:     Similar risks in portfolio for reference
recommended_terms:    Suggested endorsements, sublimits, exclusions
```

## Decision or Workflow Role

```
Application/renewal received (broker portal / direct)
  ↓
Automated data enrichment: postcode lookups, credit check, claims history
  ↓
Risk model: technical price + risk grade + flags
  ↓
Straight-through rules: 
  Grade A + small commercial → auto-quote (no UW intervention)
  Grade B/C → UW review with model output as reference
  Grade D / flagged → senior UW + manual referral
  ↓
Underwriter reviews, adjusts, quotes
  ↓
Broker/customer accepts → bound risk
  ↓
Claims development vs technical price → model performance feedback
```

## Modeling / System Options

| Component | Approach | Notes |
|---|---|---|
| Frequency model | GLM Poisson / Negative Binomial | Count of claims per policy year |
| Severity model | GLM Gamma / Tweedie | Average claim cost given claim occurs |
| Pure premium | Frequency × Severity or Tweedie combined | Main pricing output |
| Risk grade | Logistic regression / XGBoost classifier | Ordinal risk tier |
| Anomaly / flag | Isolation Forest or rule layer | Unusual risk characteristics |
| Telematics UBI | LSTM or gradient boosting on trip data | Usage-based insurance |

**Recommended**: GLM Gamma + Poisson (frequency-severity) for actuarial pricing. LightGBM challenger for higher accuracy. Both require actuarial validation.

## Deployment Constraints

- **Regulatory**: Rate filings (US) or actuarial sign-off (UK) required before deployment. Cannot change rating factors without regulatory process in some jurisdictions.
- **Explainability**: Underwriters and brokers must understand premium components. Waterfall charts showing factor contributions are standard in insurance pricing tools.
- **Auditability**: Every quote must log the model version, input data, and output. Regulatory audit can request individual rating decisions years later.
- **Refresh cadence**: Personal lines: annual rate review. Commercial: may be triggered by claim events or market changes.

## Risks and Failure Modes

| Risk | Description | Mitigation |
|---|---|---|
| **Proxy discrimination** | Postcode correlated with ethnicity → indirect discrimination | Fairness audit; geographic smoothing |
| **Adverse selection** | Model over-prices low-risk customers → they leave; under-prices high-risk → they stay | Portfolio monitoring; Lorenz curve analysis |
| **Model overfit** | Small data (specialty lines) → overfit to historical anomalies | Conservative regularisation; actuarial validation |
| **Underwriter override concentration** | UWs always override model for certain risks → model never improves | Track override patterns; recalibrate based on outcomes |

## Success Metrics

| Metric | Target | Notes |
|---|---|---|
| Loss ratio improvement | > 3pp vs no-model baseline | Primary actuarial KPI |
| Gini coefficient | > 0.45 | Discriminatory power |
| Combined ratio | < 95% | Profitability: loss + expense ratios |
| Technical price accuracy | ± 5% of actual development | Calibration metric |
| UW productivity | +20% quotes per UW per day | Operational efficiency |
| Adverse action compliance | 100% | No discriminatory rating factors |

## References

- Frees, E.W. & Valdez, E. (1998). *Understanding Relationships Using Copulas.* NAAJ.
- Tweedie GLMs: de Jong, P. & Heller, G.Z. (2008). *Generalized Linear Models for Insurance Data.* Cambridge.

## Links

**Modeling**
- [[03_modeling/01_supervised_learning/01_linear_and_glm/linear_and_glm|Linear and GLMs]] — GLM pricing models (Poisson, Gamma, Tweedie)
- [[03_modeling/07_evaluation_and_model_selection/index|Evaluation and Model Selection]] — Gini, Lorenz curve

**Reference Implementations**
- [[08_reference_implementations/01_model_implementations/linear_models_implementation|Linear Models Implementation]]
- [[08_reference_implementations/01_model_implementations/interpretability_implementation|Interpretability Implementation]]

**Adjacent Applications**
- [[claims_automation|Claims Automation]]
- [[07_applications/04_classification_and_decisioning/claim_severity_prediction|Claim Severity Prediction]]
- [[07_applications/04_classification_and_decisioning/credit_scoring|Credit Scoring]]
