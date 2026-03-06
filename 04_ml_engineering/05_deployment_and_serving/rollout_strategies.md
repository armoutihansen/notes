---
layer: 04_ml_engineering
type: engineering
tool: general
status: growing
tags: [deployment, rollout, canary, blue-green, shadow, ab-testing, mlops]
created: 2026-03-05
---

# Model Rollout Strategies

## Purpose

Deploying a new model version carries risk: the new model may behave poorly on the live distribution, degrade business metrics, or fail under production load in ways not visible in offline evaluation. Rollout strategies manage this risk by controlling how and when traffic is shifted to the new model, and by defining clear rollback paths. The strategy chosen must balance speed of deployment against safety guarantees.

## Architecture

### Blue/Green Deployment
Two identical production environments (blue = current, green = new) are maintained in parallel. Traffic is shifted from blue to green as an atomic switchover — typically by updating a load balancer routing rule or DNS record.

- **Rollback**: Instantaneous — repoint the load balancer to blue. No state migration required.
- **Infrastructure cost**: Requires 2× the compute for the duration of the transition (minutes to hours).
- **Risk profile**: All-or-nothing. If green has a subtle quality regression, 100% of users are affected before it is detected.
- **Best for**: Infrastructure or runtime changes (framework upgrades, container image changes) where correctness is verified offline. Less suited to model changes where production behaviour cannot be fully anticipated.

### Canary Releases
Traffic is shifted incrementally: a small slice (e.g., 1–5%) is routed to the new model while the remainder continues to use the current model. The canary slice is expanded in stages (e.g., 5% → 20% → 50% → 100%) as confidence in the new model grows.

- **Signal**: Online metrics (business KPIs, ML metrics, error rates) are compared between canary and baseline populations in near-real time.
- **Traffic splitting**: Implemented at the load balancer (NGINX, Envoy weighted clusters), API gateway (AWS API Gateway stage variables, Istio VirtualService weights), or feature flag service.
- **Duration per stage**: Determined by the time required to accumulate sufficient statistical power for the key metric (see A/B testing notes below).
- **Risk profile**: Limits blast radius — only the canary slice is affected by a regression.
- **Best for**: Model quality changes where production behaviour is uncertain.

### Shadow Deployment (Dark Launch)
The new model receives a copy of every live request but its predictions are **not** returned to users. Responses from both models are captured and compared offline.

- **User impact**: Zero — the shadow model's outputs are discarded.
- **Purpose**: Validates correctness, latency, and resource consumption of the new model under real production traffic without any risk.
- **Implementation**: The serving layer fans out each request to both models asynchronously; shadow responses are logged to a store (S3, BigQuery, Kafka topic) for analysis.
- **Latency**: Shadow calls should not be on the critical path. Use fire-and-forget async invocations.
- **Best for**: Major model changes (architecture replacement, new feature schema) where there is high uncertainty about production behaviour.

### A/B Testing for Model Comparison
Random assignment of users (or requests) to control (model A) and treatment (model B) groups. Measures whether the new model causes a statistically significant improvement in a target metric.

**Statistical requirements:**
- Define the primary metric and minimum detectable effect (MDE) before running the experiment.
- Compute required sample size: `n ≈ 2(z_α/2 + z_β)² σ² / δ²` where δ is the MDE and σ² is the metric variance.
- Common settings: α = 0.05 (Type I error), power 1-β = 0.80, two-tailed test.
- Run for at least 1–2 full business cycles (typically 1–2 weeks) to account for day-of-week effects and novelty effects.

**Runtime**: Assignment must be deterministic and persistent per user (hash on user_id mod N) to avoid switching users between groups mid-experiment.

### Champion-Challenger Pattern
The current production model (champion) continuously competes against one or more challenger models. A fixed fraction of traffic (e.g., 10%) is permanently allocated to challengers. If a challenger consistently outperforms the champion on defined KPIs over a rolling window, it is promoted to champion.

- **Operationalisation**: Requires a model registry with metadata, automated metric collection, and a promotion gate (manual approval or automated threshold).
- **Useful in**: High-stakes domains (credit risk, bidding algorithms) where continuous improvement is required but each promotion must be auditable.

## Implementation Notes

### Rollback Triggers and Automation
Define rollback triggers before deployment begins:
- **Hard triggers** (automated, immediate rollback): Error rate spike > threshold (e.g., 5× baseline), p99 latency > SLA, null/NaN prediction rate > 1%.
- **Soft triggers** (human review): Business metric regression beyond MDE, prediction distribution shift, anomalous feature importance.

Implement rollback as a one-command or one-click operation. The canary weight should be configurable via a control plane (feature flag, config service) without requiring a code deploy.

Automated rollback pipeline example:
1. Prometheus alert fires on error rate.
2. PagerDuty page triggers on-call.
3. On-call runs `./rollback.sh <model_version>` which updates the load balancer weight to 0% canary.
4. Post-incident review captures the root cause.

## Trade-offs

| Strategy | Risk | Rollback Speed | Infrastructure Cost | Observability |
|---|---|---|---|---|
| Blue/Green | High (all-at-once) | Instant | 2× during transition | Coarse |
| Canary | Low (limited blast) | Fast (reweight) | 1× + overhead | Good |
| Shadow | Zero | N/A | ~2× (dual inference) | Excellent |
| A/B test | Low | Fast | 1× | High (statistically rigorous) |

## References

- Sculley et al., "Hidden Technical Debt in Machine Learning Systems", *NeurIPS* 2015.
- Kleppmann, *Designing Data-Intensive Applications*, Ch. 1 (reliability).
- Google SRE Book, Ch. 8 (Release Engineering).
- Fowler, M., "BlueGreenDeployment", martinfowler.com.

## Links
- [[serving_patterns|Model Serving Patterns]]
- [[model_compression|Model Compression for Deployment]]
- [[drift_detection|Drift Detection]]
- [[ml_observability|ML Observability]]
