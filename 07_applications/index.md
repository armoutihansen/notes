---
layer: 07_applications
type: index
status: growing
tags: []
created: 2026-03-11
---

# 07 — Applications

> Guiding question: "Why are we solving this problem, and what does success look like in the real world?"

This layer captures **real-world use cases, business functions, and domain verticals** — the problems that ML and AI systems are built to solve. Notes frame the problem, describe the users and stakeholders, outline the data reality, and specify what success means in business terms.

This layer does **not** contain code (→ `08_implementations`), model theory (→ `03_modeling`), or engineering patterns (→ `05_ml_engineering`, `06_ai_engineering`).

---

## Sublayers

### [[07_applications/01_prediction_and_forecasting/index|01 — Prediction and Forecasting]]
Predicting future quantities: demand, churn, risk scores, customer lifetime value.

### [[07_applications/02_recommendation_and_ranking/index|02 — Recommendation and Ranking]]
Personalised content and product discovery, feed ranking, search result ordering.

### [[07_applications/03_detection_and_monitoring/index|03 — Detection and Monitoring]]
Fraud detection, anomaly detection, quality monitoring, alerting systems.

### [[07_applications/04_classification_and_decisioning/index|04 — Classification and Decisioning]]
Binary and multi-class decisions with regulatory and operational constraints: credit, triage, routing.

### [[07_applications/05_generation_and_assistance/index|05 — Generation and Assistance]]
LLM-powered generation: document summarization, code generation, writing assistance.

### [[07_applications/06_search_and_retrieval/index|06 — Search and Retrieval]]
Semantic search, enterprise knowledge retrieval, RAG-based assistants.

### [[07_applications/07_multimodal_systems/index|07 — Multimodal Systems]]
Computer vision, document intelligence, audio-visual systems.

### [[07_applications/08_domain_verticals/index|08 — Domain Verticals]]
Industry-specific applications: insurance, finance, health, e-commerce, mobility, operations.

---

## Cross-layer Role

```
03_modeling/          → what algorithms apply
05_ml_engineering/    → how the system is built and operated
06_ai_engineering/    → if LLMs or foundation models are involved
07_applications/      → WHY we build it, WHO uses it, WHAT success looks like
08_implementations/ → HOW it is implemented in code
09_projects/          → actual execution instances
```

Application notes are the business specification layer. They inform engineering decisions in all implementation layers.

---

## Links
- [[08_implementations/index|08 — Reference Implementations]]
- [[03_modeling/index|03 — Modeling]]
- [[05_ml_engineering/index|05 — ML Engineering]]
- [[06_ai_engineering/index|06 — AI Engineering]]
