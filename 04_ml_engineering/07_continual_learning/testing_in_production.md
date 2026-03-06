---
layer: 04_ml_engineering
type: engineering
tool: general
status: growing
tags: [a-b-testing, canary, shadow-mode, bandits, production-testing]
created: 2026-03-05
---

# Testing in Production

## Purpose

Offline evaluation — held-out test sets, cross-validation, benchmark metrics — is a necessary but insufficient signal for production readiness. Three structural problems make offline metrics poor proxies for live performance:

1. **Distribution shift:** The test set is a static snapshot; production data evolves. A model with 92% offline accuracy can underperform the incumbent live.
2. **Feedback loops:** Model predictions influence future data (e.g., a recommendation model shapes what users click, poisoning the next training set). Offline evaluation cannot capture this.
3. **Real user behavior:** Users respond differently to a model than annotators do. Task completion, session length, and revenue per session are often more important than cross-entropy loss.

Production testing strategies form a risk/speed trade-off spectrum.

## Architecture

### Shadow Mode Testing

The challenger model receives a copy of live traffic but its outputs are **never shown to users**. Predictions are logged alongside the champion's, and metrics are compared offline.

- **Risk:** Zero — no user exposure.
- **Signal:** Limited to metrics computable without user feedback (e.g., latency, prediction distribution similarity, coverage). Cannot measure business outcomes.
- **Use case:** Validating infrastructure integration, latency budgets, and gross prediction sanity before any live exposure.

### A/B Testing

Users are randomly assigned to control (champion) or treatment (challenger) buckets. Business metrics are measured per group.

**Key statistical requirements:**

- **Minimum Detectable Effect (MDE):** The smallest practical improvement worth detecting. Drives sample size.
- **Sample size:** `n ≈ (z_α/2 + z_β)² · 2σ² / δ²` where `δ` is the MDE and `σ²` is outcome variance. For typical product metrics, this often requires millions of users over days to weeks.
- **p-value threshold:** Conventionally α = 0.05, but should be adjusted for multiple comparisons (Bonferroni, Benjamini-Hochberg).
- **Risks:** Winner's curse (small studies overestimate effects); novelty effects; network interference in social graphs (SUTVA violation).

### Canary Testing

The challenger serves **5–10% of traffic** (or a single region/datacenter). Key metrics — error rate, latency p99, business KPIs — are monitored against automated thresholds. Rollback triggers fire automatically if guardrail metrics degrade.

- **Faster** than full A/B; limits blast radius.
- **Less statistical power** than full A/B; suitable for catching gross regressions, not measuring small lifts.

### Multi-Armed Bandits

Instead of a fixed allocation, bandit algorithms adaptively shift traffic toward the better-performing variant in real time.

- **Thompson Sampling:** Maintain a Beta distribution over each variant's conversion rate `p`. At each request, sample `p_i ~ Beta(α_i, β_i)` for each arm and route to the argmax. Posterior updates with each observed outcome.
- **UCB (Upper Confidence Bound):** Select arm `i = argmax [μ_i + c · √(ln t / n_i)]`. Balances exploitation of known-good arms with exploration of uncertain ones.
- **Trade-offs vs. A/B:** Faster to identify winners; reduces regret; but harder to control false positive rate, and requires stationary reward distributions.

### Interleaving Experiments

Used in **search and recommendation** systems: instead of showing users results from model A or model B, interleave both result sets and track which model's items receive more engagement. Significantly more sensitive than A/B testing for ranking quality; widely used at Netflix, Google, Airbnb.

## Implementation Notes

### Guardrails and Rollback Triggers

Every production experiment should define guardrail metrics — metrics that must not degrade regardless of the primary metric outcome:
- Latency p99 < X ms
- Error rate < Y%
- Core business metric (revenue, session length) within Z% of baseline

Automated rollback: if any guardrail breaches its threshold within the canary window (e.g., 30 minutes after deployment), traffic immediately reverts to champion. Platforms like Argo Rollouts and Flagger implement this pattern on Kubernetes.

### Logging and Observability

Prediction logging must be **request-level**: model version, input features (or a hash), prediction, timestamp, and user/session ID. This enables joining predictions to downstream outcomes (conversions, returns, complaints) for delayed label computation and post-hoc analysis.

## Trade-offs

| Strategy | Risk | Speed to decision | Statistical rigor | Complexity |
|---|---|---|---|---|
| Shadow mode | None | Slow (no user signal) | Low | Low |
| A/B test | Medium | Slow (days–weeks) | High | Medium |
| Canary | Low | Fast (minutes–hours) | Low | Low |
| Bandit | Low–Medium | Fast | Medium | High |
| Interleaving | Low | Very fast | High | High |

## References

- Kohavi, Tang & Xu, *Trustworthy Online Controlled Experiments* (Cambridge, 2020)
- Chapelle & Li, "An Empirical Evaluation of Thompson Sampling" (NeurIPS 2011)
- Joachims et al., "Unbiased Learning-to-Rank with Biased Feedback" (WSDM 2017)
- Flagger: https://docs.flagger.app

## Links
- [[retraining_strategies|Retraining Strategies]]
- [[ml_platform_architecture|ML Platform Architecture]]
- [[drift_detection|Drift Detection]]
- [[ml_observability|ML Observability]]
- [[rollout_strategies|Model Rollout Strategies]]
- [[offline_evaluation|Offline Evaluation]]
