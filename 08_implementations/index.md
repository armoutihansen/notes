---
layer: 08_implementations
type: index
status: growing
tags: []
created: 2026-03-05
---

# 08 — Implementations

> Guiding question: "How is this cross-cutting executable pattern or end-to-end system implemented in code?"

This layer contains **cross-layer, reusable executable reference architectures**: system patterns that integrate multiple components across layers, and complete end-to-end examples that span data ingestion through deployment.

**What belongs here:**
- Production system patterns that integrate ≥2 layers (e.g. MLflow + feature store + serving)
- End-to-end reference architectures adaptable to real projects
- Reusable orchestration and deployment patterns

**What does NOT belong here:**
- Concept-specific implementation notes — those live with their parent concept. For example:
  - XGBoost / tree ensemble code → [[03_modeling/01_supervised_learning/02_tree_based_models/index|03 — Tree-Based Models]]
  - SHAP / interpretability workflow → [[03_modeling/07_evaluation_and_model_selection/index|03 — Evaluation and Model Selection]]
  - Neural network PyTorch implementations → [[03_modeling/04_deep_learning/01_mlp_and_representation_learning/index|03 — MLP and Representation Learning]]
  - Transformer fine-tuning pipeline → [[06_ai_engineering/05_finetuning/index|06 — Fine-tuning]]
- Business problem framing → [[07_applications/index|07 — Applications]]
- Theoretical derivations → [[01_foundations/index|01 — Foundations]]
- Project-specific work → [[09_projects/index|09 — Projects]]

**Placement rule:** *Implementation notes live with the concept unless they are cross-cutting reference architectures that integrate multiple layers.*

---

## [[08_implementations/01_system_patterns/index|01 — System Patterns]]

Reusable production patterns for ML/AI systems: experiment tracking, feature stores, model serving, monitoring, RAG pipelines, agents, quantization, CI/CD, and more. Each pattern integrates components across `05_ml_engineering/` and `06_ai_engineering/`.

---

## [[08_implementations/02_end_to_end_examples/index|02 — End-to-End Examples]]

Complete ML/AI system walkthroughs combining multiple components across multiple layers. Each is a reference architecture spanning data → model → deployment → monitoring, adaptable to real projects.

---

## Links

- [[03_modeling/index|03 — Modeling]]
- [[05_ml_engineering/index|05 — ML Engineering]]
- [[06_ai_engineering/index|06 — AI Engineering]]
- [[07_applications/index|07 — Applications]]
