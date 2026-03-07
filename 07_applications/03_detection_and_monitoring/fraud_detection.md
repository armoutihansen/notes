---
layer: 07_applications
type: application
status: growing
tags: [classification, tabular, anomaly-detection]
created: 2026-03-11
---

# Fraud Detection

## Problem

Identify fraudulent transactions — payments, account takeovers, application fraud, insurance claims — in real time to decline, flag, or challenge them before loss is incurred. Fraud detection operates under extreme class imbalance (0.1–2% fraud rate), adversarial conditions (fraudsters adapt to detection logic), and strict latency constraints (credit card authorisation: <100ms).

Three distinct fraud types with different modelling approaches:
1. **Transaction fraud**: Stolen card use or account compromise — real-time pattern detection
2. **Application fraud**: Synthetic identity or document forgery at account opening — multi-signal verification
3. **Account takeover (ATO)**: Login credential theft — behavioural biometrics + device intelligence

## Users / Stakeholders

| Role | Decision |
|---|---|
| Fraud operations analyst | Review flagged transactions; rule tuning |
| Risk officer | Set risk appetite; fraud loss vs customer friction tradeoff |
| Cardholder | Challenged or declined transaction experience |
| Compliance / BSA officer | SAR filing; AML programme |
| Product team | False positive rate vs fraud rate tradeoff; feature design |

## Domain Context

- **Extreme class imbalance**: 0.1–1% fraud rate. SMOTE/oversampling, class weighting, and calibrated scoring are essential.
- **Ground truth delay**: Chargebacks (confirmed fraud) may arrive 30–90 days after transaction. Training on recent data requires label propagation.
- **Adversarial dynamics**: Fraudsters observe decline reasons and adapt. Model performance degrades faster than typical ML — retraining cadence critical (weekly or daily).
- **Cost asymmetry**: False negative (missed fraud) costs 100% of transaction value. False positive (blocked legitimate transaction) costs ~$20 in customer service + cart abandonment rate of 30–40%.
- **Regulatory**: PCI-DSS requires data minimisation and audit trails. GDPR Article 22 restricts fully automated adverse decisions without human review pathway. US: ECOA, FCRA requirements for dispute handling.
- **Network effects**: Fraud rings share cards/accounts. Graph features (shared device, IP, address) are extremely high-value signals.

## Inputs and Outputs

**Features**:
```
Transaction: amount, merchant_category, country, channel (online/POS), currency
Card: card_age, card_type, issuer_country, previous_decline_count
Velocity: txn_count_1h, txn_count_24h, amount_sum_1h, unique_merchants_7d
Behavioural: time_since_last_txn, typical_merchant_category_match, amount_deviation_from_mean
Device: device_fingerprint, ip_address, is_new_device, vpn_flag, geolocation
Graph: is_known_fraud_network, shared_ip_flag, shared_device_other_accounts
History: chargeback_history, dispute_count, address_change_recency
```

**Output**:
```
fraud_score:       P(fraud) ∈ [0, 1]
decision:          APPROVE / STEP_UP (3DS challenge) / DECLINE
challenge_type:    OTP / BIOMETRIC / MANUAL_REVIEW
reason_codes:      Top fraud signals for analyst review (SHAP-based)
```

## Decision or Workflow Role

```
Transaction arrives → real-time feature computation (<10ms)
  ↓
Rules engine: hard blocks (known bad IP, stolen card list)
  ↓
ML scoring: fraud probability (<30ms)
  ↓
Decision matrix:
  score < 0.1   → APPROVE
  0.1–0.5       → STEP_UP (3DS challenge)  
  score > 0.5   → DECLINE or MANUAL_REVIEW
  ↓
Outcome logged → chargeback matching → label pipeline → retraining
```

The ML model operates **alongside a rules engine**, not instead of it. Rules provide hard constraints (block known bad actors); ML handles pattern generalisation.

## Modeling / System Options

| Approach | Strength | Weakness | When to use |
|---|---|---|---|
| Logistic regression + engineered features | Interpretable; fast; solid baseline | Misses non-linear interactions | Baseline; interpretability-first |
| XGBoost / LightGBM | High accuracy; handles class imbalance well; SHAP interpretable | Batch features only; static model | Standard production choice |
| Graph Neural Network | Captures fraud ring structure; high-value network signals | Requires graph infrastructure; complex | Large-scale with graph data |
| Isolation Forest (unsupervised) | Catches novel fraud patterns without labels | High false positive rate; needs tuning | Cold start; novel fraud type detection |
| Real-time streaming model | Low-latency; captures velocity patterns | Complex infrastructure (Kafka + Flink) | Card-present transactions at bank scale |

**Recommended**: XGBoost/LightGBM with engineered velocity and graph features. Real-time feature computation via Redis or a feature store. Weekly retraining. Rules engine for hard blocks.

## Deployment Constraints

- **Latency**: <100ms total decision latency for card authorisation. ML scoring budget ~30ms.
- **Throughput**: Large card networks: 10K–65K transactions/second peak.
- **Interpretability**: Regulatory audit trail for declined transactions. Reason codes required for customer dispute process.
- **Model versioning**: Shadow mode deployment for new models. Champion-challenger A/B with holdout fraud measurement.
- **Feature freshness**: Velocity features (txn_count_1h) must be computed in real time. Stale features = missed fraud.

## Risks and Failure Modes

| Risk | Description | Mitigation |
|---|---|---|
| **Model staleness** | Fraud patterns shift; old model misses new schemes | Weekly retraining; concept drift monitoring |
| **Label noise** | Chargebacks don't map 1:1 to fraud events | Tiered label confidence; semi-supervised approach |
| **False positive harm** | Declining legitimate transactions causes churn | Customer friction metrics; segment-specific thresholds |
| **Adversarial adaptation** | Fraudsters probe system to find low-score paths | Regular pattern audits; ensemble models; dark web intelligence |
| **Cascade failure** | ML service down → default to rule-only → higher fraud | Fallback strategy; circuit breaker; default-to-challenge policy |
| **Demographic disparity** | Model declines higher rates for certain groups | Fairness audit; equalised FPR across demographics |

## Success Metrics

| Metric | Target | Notes |
|---|---|---|
| Fraud detection rate (recall) | > 90% at 1% FPR | Business-critical; missed fraud = direct loss |
| False positive rate | < 1% of legitimate transactions | Customer friction KPI |
| Fraud loss rate (bps) | Industry benchmark: 5–15bps of GMV | Ultimate financial metric |
| Model latency p99 | < 30ms | Operational SLA |
| Analyst review queue clearance | > 95% within 4h | Operational efficiency |
| Chargeback rate | < 1% of transactions | Card network threshold |

## References

- Dal Pozzolo, A. et al. (2015). *Calibrating Probability with Undersampling for Unbalanced Classification.* CIDM.
- van Belle, V. et al. (2021). *GOTCHA! Network-based Fraud Detection for Social Security Fraud.* Management Science.

## Links

**Modeling**
- [[03_modeling/01_supervised_learning/02_tree_based_models/gradient_boosting|Gradient Boosting]] — XGBoost/LightGBM for fraud classification
- [[03_modeling/02_unsupervised_learning/index|Unsupervised Learning]] — anomaly detection baselines

**ML Engineering**
- [[05_ml_engineering/06_deployment_and_serving/index|Deployment and Serving]] — real-time low-latency inference
- [[05_ml_engineering/07_monitoring_and_observability/index|Monitoring and Observability]] — fraud rate and model drift

**Reference Implementations**
- [[03_modeling/01_supervised_learning/02_tree_based_models/tree_ensembles_implementation|Tree Ensembles Implementation]]
- [[08_implementations/01_system_patterns/drift_monitoring_with_evidently|Drift Monitoring with Evidently]]

**Adjacent Applications**
- [[anomaly_detection_operations|Anomaly Detection — Operations]]
- [[07_applications/08_domain_verticals/02_finance/risk_portfolio_monitoring|Portfolio Risk Monitoring]]
- [[07_applications/04_classification_and_decisioning/credit_scoring|Credit Scoring]]
