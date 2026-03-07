---
layer: 07_applications
type: application
status: growing
tags: [recommendation, ranking, tabular]
created: 2026-03-11
---

# Content Ranking

## Problem

Order a dynamic stream of content items (articles, posts, videos, stories, advertisements) to maximise a user's engagement, satisfaction, or time spent — subject to integrity, diversity, and safety constraints. Unlike product recommendation, content is produced continuously, has a short relevance half-life, and is consumed in a sequential feed rather than a catalogue browse. The optimisation target is deeply contested: pure engagement maximisation leads to outrage content amplification and filter bubbles.

## Users / Stakeholders

| Role | Interest |
|---|---|
| End user | Relevant, high-quality content; not overwhelmed or offended |
| Content creator | Fair exposure; predictable reach |
| Editorial/Trust & Safety | Integrity constraints; downrank harmful content |
| Advertising team | Ad slot placement and fill rate |
| Product/growth team | Session depth, DAU, subscriber retention |

## Domain Context

- **Content velocity**: Millions of new items daily. Freshness is a signal. Stale content is penalised.
- **Multi-objective**: Engagement (clicks, shares, watch time) vs quality (fact-checked, source credibility) vs safety (no misinformation, hate speech) vs creator fairness (not just top accounts).
- **Cold start**: New creators and new content items have no engagement history. Content features (text, image, topic) must provide signal.
- **Regulatory landscape**: EU Digital Services Act (DSA) mandates transparency in recommendation systems for Very Large Online Platforms. Users must have a non-personalised option. Algorithmic accountability audits required.
- **Position bias and feedback loops**: The feed itself generates training data — showing content in position 1 guarantees more engagement. Feedback loops amplify existing biases.

## Inputs and Outputs

**Signals**:
```
Content features: text embeddings, topic category, author credibility score, 
                  image/video embeddings, fact-check status, publication_age
User features:    topic affinity scores, social graph, historical engagement,
                  session length, device, time of day
Interaction history: clicks, shares, comments, dwell_time, scrolled_past,
                     negative signals (hide, report, unfollow)
Context: scroll position, session depth, time since last session
```

**Output**:
```
Ranked feed: ordered list of content items for the next N slots
Ad insertion positions: where ads are placed
Diversity flags: variety in topic, source, format
Safety scores: probability of harmful content per item
```

## Decision or Workflow Role

```
Candidate generation (~1K items from subscriptions + interest graph)
  ↓
Multi-objective scoring: P(click), P(share), P(long-read), P(harm)
  ↓
Scalarisation: weighted combination with editorial constraints applied
  ↓
Integrity filter: suppress items with safety score > threshold
  ↓
Diversity post-filter: ensure topic, source, format variety
  ↓
Ad insertion: auction-determined ad slots interleaved
  ↓
Serve to user → log interactions → online feedback to ranking model
```

## Modeling / System Options

| Approach | Strength | Weakness |
|---|---|---|
| Logistic regression on handcrafted features | Fast, interpretable, debuggable | Misses complex interactions; manual feature work |
| GBDT (LightGBM) with cross-features | High accuracy with feature engineering | Static; no sequential modelling |
| Transformer-based sequence model (BERT4Rec) | Captures sequential user interest evolution | High compute; harder to serve |
| Multi-task learning (MTL) | Optimises multiple signals simultaneously | Complex loss weighting; harder to debug |
| Bandits / online learning | Adaptive; handles exploration-exploitation | Variance in production; requires exploration budget |

**Standard**: Multi-task GBDT with logistic regression for safety scoring. MTL neural ranking (DIN, BST) at larger scale.

## Deployment Constraints

- **Latency**: Feed render < 200ms. Ranking model must score 1K candidates in <50ms.
- **Throughput**: Major platforms: hundreds of thousands to millions of feed requests per second.
- **Explainability**: EU DSA requires user-facing explanation for recommendation (e.g., "Because you follow X"). Systems need an explanation generation layer.
- **Auditability**: Ranking logic must be auditable for regulatory reporting. Version all ranking models and configuration.

## Risks and Failure Modes

| Risk | Description | Mitigation |
|---|---|---|
| **Engagement-quality divergence** | Outrage, clickbait, misinformation drives clicks but harms users | Multi-objective with quality signals; safety classifier |
| **Filter bubble** | Users only see content confirming existing beliefs | Diversity constraints; serendipity injection |
| **Creator inequality** | Large accounts dominate; newcomers get no exposure | Creator fairness constraints; new creator boost |
| **Adversarial actors** | Coordinated inauthentic behaviour inflates signals | Spam/bot detection; signal robustness |
| **Data poisoning** | Adversarially generated content manipulates embeddings | Adversarial training; input validation |

## Success Metrics

| Metric | Target | Notes |
|---|---|---|
| Session depth (items viewed) | +X% vs baseline | Primary engagement metric |
| Satisfaction survey score | > 4.0/5 | Quality beyond clicks |
| News diversity index | Defined minimum | Regulatory / editorial commitment |
| Harmful content prevalence | < 0.01% impressions | Safety KPI; tracked via adversarial audit |
| Creator reach Gini coefficient | < defined threshold | Fairness metric |

## References

- Covington, P. et al. (2016). *Deep Neural Networks for YouTube Recommendations.* RecSys.
- Ribeiro, M. et al. (2020). *Auditing Radicalization Pathways on YouTube.* FAT*.

## Links

**Modeling**
- [[03_modeling/01_supervised_learning/02_tree_based_models/gradient_boosting|Gradient Boosting]] — GBDT ranking models
- [[03_modeling/04_deep_learning/index|Deep Learning]] — sequential attention models (DIN, BST)

**AI Engineering**
- [[06_ai_engineering/03_prompt_engineering/index|Prompt Engineering]] — LLM-based content quality scoring

**Reference Implementations**
- [[03_modeling/01_supervised_learning/02_tree_based_models/tree_ensembles_implementation|Tree Ensembles Implementation]]

**Adjacent Applications**
- [[product_recommendation|Product Recommendation]]
- [[07_applications/06_search_and_retrieval/semantic_search|Semantic Search]]
