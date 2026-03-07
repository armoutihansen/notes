---
layer: 07_applications
type: application
status: growing
tags: [recommendation, tabular, retrieval]
created: 2026-03-11
---

# Product Recommendation

## Problem

Surface the products a user is most likely to purchase from a catalogue of thousands to millions of items. Recommendations appear on: homepage, product detail page ("you may also like"), cart page ("frequently bought together"), email campaigns, and search results. The system must balance relevance, diversity, novelty, and business objectives (margin, inventory clearance, promoted products).

## Users / Stakeholders

| Role | Decision / Action |
|---|---|
| Shopper | Discovers products they would not have searched for |
| Merchandiser | Allocates promotional slots; monitors recommendation diversity |
| Category manager | Ensures recommendations support category strategy |
| Revenue team | Tracks incremental revenue attributed to recommendations |

## Domain Context

- **Catalogue scale**: 10K–100M items. Full pairwise scoring is infeasible. Two-stage architecture (retrieval → ranking) is standard.
- **Data sparsity**: Most users interact with <0.01% of the catalogue. Collaborative filtering suffers from cold start for new users and items.
- **Position bias**: Items shown in top positions receive more clicks regardless of quality. Training on raw click data encodes this bias — debiasing (inverse propensity scoring) is required.
- **Seasonality**: Holiday periods, fashion seasons, promotional events shift preferences dramatically.
- **Regulatory**: In some markets (EU Digital Markets Act), algorithmic ranking of third-party products must be disclosed. Personalisation based on sensitive attributes (health, religion) is restricted.
- **Business constraints**: Contracts with brands may require minimum impressions for sponsored products. Margin rules may suppress low-margin recommendations.

## Inputs and Outputs

**Input signals**:
```
User signals: purchase history, browse history, wishlist, ratings, returns
Item signals: category, price, brand, ratings, inventory level, launch date
Context: device, location, time of day, session context, referral source
Interaction signals: clicks, add-to-cart, purchases, dwell time, scroll depth
Business rules: promoted products, inventory thresholds, brand partnerships
```

**Output**:
```
Ranked item list: [item_id_1, item_id_2, ..., item_id_N] with relevance scores
Explanation (optional): "Because you bought X" or "Trending in your region"
Metadata: source (collaborative / content / trending / sponsored)
```

## Decision or Workflow Role

```
[Retrieval stage] 
  → Candidate generation: ANN search over item embeddings (top 200–1000)
  → Collaborative filtering recall (user-item similarity)
  → Content-based recall (item attribute similarity)

[Ranking stage]
  → Feature assembly (user + item + context features)  
  → Learning-to-Rank model (LambdaMART / neural ranking)
  → Business rule overlay (suppress OOS, boost promoted)

[Presentation]
  → A/B test slot allocation
  → Diversity post-filter (MMR or intent-aware diversification)
  → Logging for feedback loop
```

## Modeling / System Options

| Approach | Strength | Weakness | When to use |
|---|---|---|---|
| Matrix Factorisation (SVD, ALS) | Simple, well-understood, handles sparsity | No context features; cold start; static | Baseline collaborative filtering |
| Neural CF (NCF, LightFM) | Learns non-linear interactions; can incorporate features | Training cost; harder to explain | Medium-scale, some feature richness |
| Two-tower (DSSM) | Fast ANN retrieval; user + item embedding spaces | Context not captured in tower | Large-scale retrieval stage |
| LambdaMART (LTR) | Directly optimises ranking metrics (NDCG); feature-rich | Not end-to-end; requires good candidate set | Ranking stage after retrieval |
| Session-based (BERT4Rec, SASRec) | Captures in-session intent; no long-term profile needed | Cold start if no session | Short-session or anonymous users |
| GNN (PinSage) | Graph structure (bought together) improves retrieval | Complex to build and maintain | Pinterest/Amazon scale, explicit graph data |

**Recommended**: Two-tower retrieval → LambdaMART re-ranker for most e-commerce contexts. Session-based model for anonymous/new user fallback.

## Deployment Constraints

- **Latency**: 50–150ms p99 for ranked results. Retrieval stage (ANN) must complete in <20ms.
- **Throughput**: Homepage impressions for large retailers: 10K–100K QPS during peak.
- **Freshness**: Item embeddings need nightly refresh for new items. User embeddings updated daily or real-time for session signals.
- **Cold start fallback**: New users → trending + editorial curated lists. New items → content-based embedding from product attributes.
- **Explainability**: "Customers who viewed X also viewed Y" — simple rule-based explanation overlaid on model results. Not legally mandated but improves user trust.

## Risks and Failure Modes

| Risk | Description | Mitigation |
|---|---|---|
| **Filter bubble** | Recommending only items similar to past — reduces discovery | Diversity constraints; novelty penalty in ranking objective |
| **Popularity bias** | Popular items recommended everywhere — rich get richer | Debiasing; long-tail exposure metrics |
| **Position bias in training** | Model learns to recommend what was shown, not what was preferred | IPS weighting; counterfactual evaluation |
| **Cold start** | New users / items have no signal | Fallback rules; content-based embeddings; onboarding flow |
| **Out-of-stock recommendations** | Recommending unavailable items damages UX | Real-time inventory filter in ranking stage |
| **Feedback loop collapse** | Model trains on biased outputs → amplifies initial errors | Offline evaluation + holdout group; exploration policy |

## Success Metrics

| Metric | Target | Notes |
|---|---|---|
| Click-through rate (CTR) | +10–30% vs random baseline | Primary engagement metric |
| Add-to-cart rate | +5–15% lift in experiment | Stronger purchase intent signal |
| Revenue per session | +2–8% incremental | True business outcome; requires A/B test |
| Coverage (catalogue %) | > 20% of items recommended per day | Prevents long-tail neglect |
| NDCG@10 | Offline ranking quality | Proxy for online performance |
| Diversity (ILD) | > 0.6 intra-list diversity | User satisfaction; reduces abandonment |

## References

- Koren, Y. et al. (2009). *Matrix Factorization Techniques for Recommender Systems.* IEEE Computer.
- Yi, X. et al. (2019). *Sampling-Bias-Corrected Neural Modeling for Large Corpus Item Recommendations.* RecSys.

## Links

**Modeling**
- [[03_modeling/02_unsupervised_learning/index|Unsupervised Learning]] — matrix factorisation, embedding methods
- [[03_modeling/01_supervised_learning/02_tree_based_models/gradient_boosting|Gradient Boosting]] — LambdaMART ranking

**AI Engineering**
- [[06_ai_engineering/04_rag_and_agents/vector_stores|Vector Stores]] — ANN search for candidate retrieval

**ML Engineering**
- [[05_ml_engineering/04_feature_engineering/index|Feature Engineering]] — user and item feature computation
- [[05_ml_engineering/06_deployment_and_serving/index|Deployment and Serving]] — real-time inference at QPS scale

**Reference Implementations**
- [[08_implementations/01_system_patterns/vector_database_retrieval|Vector Database Retrieval]]
- [[08_implementations/01_system_patterns/feature_store_pattern|Feature Store Pattern]]

**Adjacent Applications**
- [[content_ranking|Content Ranking]]
- [[07_applications/06_search_and_retrieval/semantic_search|Semantic Search]]
- [[07_applications/08_domain_verticals/04_ecommerce/personalized_pricing|Personalized Pricing]]
