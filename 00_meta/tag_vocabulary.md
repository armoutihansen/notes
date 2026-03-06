---
layer: 00_meta
type: index
status: stable
tags: []
created: 2025-01-01
---

# Tag Vocabulary

The complete controlled vocabulary for Vault_2026. See [[00_meta/conventions|Conventions §12]] for rules and application guidelines.

**Key rules:** kebab-case only · ≤4 tags per note · no structural redundancy · no tool names · index files always `tags: []`

---

## Dimension 1 — Concept Type

Tags that classify *what kind of knowledge* a note contains.

| Tag | Definition | Use when | Not for |
|-----|-----------|----------|---------|
| `algorithm` | A named, step-by-step computational procedure | A note defines or analyzes a specific algorithm (backpropagation, k-means, ARIMA) | Generic topic notes; notes named after the algorithm already convey this |
| `pattern` | An architectural or design pattern | A note describes a reusable solution structure (RAG pipeline, feature store, training pipeline pattern) | One-off implementations; notes that just happen to follow a pattern |
| `theory` | Mathematical or statistical theory | A note proves or formally defines a theoretical concept (PAC learning, VC dimension, convexity) | Intuitive overviews; applied engineering notes |
| `workflow` | An operational process or practice | A note describes a repeatable process (experiment tracking, CI/CD pipeline, data labeling workflow) | Single-tool reference notes; conceptual introductions |

---

## Dimension 2 — ML Lifecycle Stage

Tags that mark *where in the ML/AI production lifecycle* a note is relevant. **Do not apply** if the note's folder already encodes the stage (e.g., don't add `training` to notes already inside `04_ml_engineering/04_model_development/`).

| Tag | Definition | Use when | Not for |
|-----|-----------|----------|---------|
| `data` | Data collection, ingestion, preprocessing, validation, augmentation | A note outside the data engineering sublayer that is primarily about data concerns | Notes inside `04_ml_engineering/01_data_engineering/` |
| `feature-engineering` | Feature creation, encoding, selection, feature stores | A note outside the feature engineering sublayer that covers feature transformation | Notes inside `04_ml_engineering/03_feature_engineering/` |
| `training` | Model fitting, optimization, regularization, hyperparameter tuning | A foundations or modeling note that is specifically about the training process (not just theory) | Notes inside `04_ml_engineering/04_model_development/` |
| `evaluation` | Metrics, benchmarking, cross-validation, calibration, model validation | Any note that is primarily about measuring model quality | Generic overview notes |
| `deployment` | Serving, inference, containerization, rollout strategies, model compression | Notes about putting models into production | Notes inside `04_ml_engineering/05_deployment_and_serving/` |
| `monitoring` | Drift detection, alerting, observability, retraining triggers | Notes about tracking model health post-deployment | Notes inside `04_ml_engineering/06_monitoring_and_observability/` |

---

## Dimension 3 — Domain / Modality

Tags that mark *what kind of data or problem domain* a note addresses. **Do not apply** if the folder already encodes the domain (e.g., don't add `nlp` to notes inside a transformer or NLP sublayer).

| Tag | Definition | Use when | Not for |
|-----|-----------|----------|---------|
| `llm` | Large language model systems | A note is about LLM-specific concerns (prompting, fine-tuning, inference, evaluation of LLMs) and is not already in `05_ai_engineering/` — or is in `05_ai_engineering/` but also highly relevant from an MLOps/engineering angle | Generic NLP notes; notes already inside `05_ai_engineering/` where `llm` is the default domain |
| `nlp` | Natural language processing tasks not specifically LLM-centric | Classical NLP, tokenization theory, text preprocessing, sentiment analysis, named entity recognition | LLM-specific notes (use `llm`) |
| `vision` | Computer vision, image understanding | Notes about CNNs, object detection, image segmentation, visual models | Notes inside a CV sublayer already |
| `time-series` | Temporal data, forecasting, sequential modeling of time | ARIMA, Prophet, state-space models, walk-forward validation | General sequence models (use `nlp` or `algorithm`) |
| `tabular` | Structured/relational data, classical ML | Tree ensembles, logistic regression, feature engineering for tabular data | Deep learning notes on tabular data (use `algorithm` + `tabular`) |
| `multimodal` | Cross-modal systems (vision+language, audio+text, etc.) | CLIP, BLIP, vision-language models, audio captioning | Single-modality notes |
| `recommendation` | Collaborative filtering, ranking, retrieval for recommender systems | Matrix factorization, two-tower models, CTR prediction | General retrieval notes (use `retrieval`) |

---

## Dimension 4 — Capability / Task Type

Tags that mark *what the system or model does*.

| Tag | Definition | Use when | Not for |
|-----|-----------|----------|---------|
| `classification` | Supervised classification | Notes about classifiers, classification loss, ROC/AUC, multiclass strategies | Regression notes |
| `regression` | Supervised regression | Notes about regression models, MSE/MAE, prediction intervals | Classification notes |
| `clustering` | Unsupervised grouping | k-means, DBSCAN, GMM, hierarchical clustering | Dimensionality reduction (unless combined) |
| `generation` | Generative models and text/image/audio generation | LLMs, diffusion models, VAEs, GANs, seq2seq generation | Discriminative models |
| `retrieval` | Search, nearest-neighbor, RAG, vector databases | Embedding search, FAISS, Chroma, BM25, hybrid search, reranking | Pure generation without retrieval |
| `reasoning` | Chain-of-thought, planning, agents, step-by-step inference | ReAct agents, DSPy, tool-calling, multi-step reasoning | Straightforward inference |
| `forecasting` | Time-series prediction | ARIMA, Prophet, temporal models predicting future values | General regression (use `regression`) |
| `anomaly-detection` | Outlier and novelty detection | Isolation Forest, one-class SVM, statistical anomaly tests | Classification of rare events (use `classification`) |
| `interpretability` | Explainability, feature attribution, fairness, model transparency | SHAP, LIME, PDP, fairness metrics, model cards | Evaluation notes (use `evaluation`) |

---

## Dimension 5 — Infrastructure / System Concern

Tags that mark *cross-cutting production system concerns*.

| Tag | Definition | Use when | Not for |
|-----|-----------|----------|---------|
| `mlops` | ML production lifecycle operations | Notes about the operational concerns of running ML systems (pipelines, registries, orchestration) that appear outside `04_ml_engineering/` | Notes inside `04_ml_engineering/` (structural redundancy) |
| `llmops` | LLM-specific production operations | Notes about operating LLM systems in production (model gateways, prompt management, LLM observability) that appear outside `05_ai_engineering/` | Notes inside `05_ai_engineering/07_architecture_and_feedback/` |
| `distributed` | Multi-GPU, multi-node, parallelism, sharding | Notes about distributed training or inference | Single-GPU notes |
| `quantization` | Model compression via reduced precision (INT8, INT4, GGUF, AWQ, GPTQ) | Notes about quantization methods and trade-offs | General compression/pruning (use `deployment`) |
| `fine-tuning` | Adapting pretrained models: LoRA, PEFT, RLHF, SFT, DPO, GRPO | Notes about fine-tuning methods and workflows | Pretraining from scratch; general transfer learning |
| `safety` | Alignment, guardrails, content moderation, red-teaming, constitutional AI | Notes about making AI systems safe and aligned | General evaluation (use `evaluation`) |

---

## Synonym → Canonical Mapping

Use this when migrating old tags:

| Old tag(s) | Canonical tag | Reason |
|-----------|--------------|--------|
| `finetuning`, `sft`, `lora`, `qlora`, `peft`, `rlhf`, `dpo`, `grpo`, `adapter`, `adapters` | `fine-tuning` | All refer to the same lifecycle concern |
| `serving`, `model-serving`, `ml-serving`, `inference-api`, `production-ml` | `deployment` | All refer to production serving |
| `metrics`, `benchmarks`, `benchmarking`, `offline-evaluation`, `llm-evaluation`, `model-validation`, `llm-eval` | `evaluation` | All about measuring quality |
| `concept-drift`, `data-drift`, `distribution-shift`, `drift`, `drift-detection` | `monitoring` | All about production health |
| `vector-store`, `vector-database`, `vector-db`, `vector-search`, `similarity-search` | `retrieval` | All vector search patterns |
| `deep_learning`, `deep-learning`, `neural-networks`, `neural_networks` | remove (structural) | Redundant with `01_foundations/06_deep_learning_theory/` |
| `mlops`, `ml-ops`, `ml-pipelines`, `ml-systems`, `ml-lifecycle` | `mlops` | When outside `04_ml_engineering/`; otherwise remove |
| `computer_vision` | `vision` | Canonical form |
| `sequence_models`, `rnn`, `lstm`, `gru`, `BPTT` | remove or `nlp` | Structural (inside transformer/RNN sublayers) |
| `transformers`, `transformer`, `attention`, `bert`, `gpt` | remove (structural) | Inside transformer sublayers already |
| `python`, `go`, `typescript`, `javascript` | remove | Inside language-specific sublayers |
| `end-to-end` | remove | Folder `06_applications/03_end_to_end_examples/` encodes this |
| `production`, `production-ml` | remove or `deployment` | Usually structural redundancy |
