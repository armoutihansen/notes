---
layer: 00_meta
type: index
status: evergreen
tags: []
created: 2026-01-01
---

# Vault Conventions

## 1. Purpose of This Vault

This vault is a structured professional knowledge system supporting:

- Mathematical and statistical foundations
- Data science analytical practice
- Machine learning modeling
- Software engineering depth
- ML engineering (production systems)
- AI engineering (LLM systems)
- Domain application patterns across industries
- Active and completed projects

The vault is organized by **epistemic layer and abstraction level**, not by buzzwords or temporary interests.

Top-level structure:
```
00_meta
01_foundations
02_data_science
03_modeling
04_software_engineering
05_ml_engineering
06_ai_engineering
07_applications
08_implementations
09_projects
10_reading
11_logs
99_archive
```

The top-level structure is stable. Changes require explicit revision of this document.

---

# 2. Layer Definitions

## 01_foundations

### Purpose
Timeless theoretical foundations.

### Contains
- Mathematics (calculus, linear algebra, differential equations, real analysis)
- Optimization theory
- Probability theory
- Statistical inference
- Time series theory
- Actuarial mathematics
- Theoretical deep learning

### Does NOT Contain
- Framework-specific code
- Deployment instructions
- Business context
- Tool configuration

### Guiding Question
**Why does this work mathematically?**

If a note contains derivations, formal definitions, or proofs → it belongs here.

### Sublayer Structure
```
01_foundations/
├── 01_linear_algebra/
├── 02_calculus_and_analysis/
├── 03_probability_and_statistics/
├── 04_optimization/
├── 05_statistical_learning_theory/
└── 06_deep_learning_theory/
```

---

## 02_data_science

### Purpose
Practical analytical reasoning: how to turn raw data into insight, decisions, and well-scoped ML problems.

### Contains
- Problem framing and hypothesis generation
- Data representation and data types
- Exploratory data analysis and visualization
- Feature engineering (conceptual approach)
- Experimentation design and validation
- Interpretability and communication of results
- Decision analysis and business metrics

### Does NOT Contain
- Specific model family descriptions (→ 03_modeling)
- Production ML pipelines (→ 05_ml_engineering)
- Mathematical derivations (→ 01_foundations)
- Cross-layer executable reference architectures (→ 08_implementations)

### Guiding Question
**How do we reason from data to decisions?**

If the note is about understanding data and translating it into a well-posed problem or insight → it belongs here.

### Sublayer Structure
```
02_data_science/
├── 01_problem_framing/
├── 02_data_representation/
├── 03_exploratory_data_analysis/
├── 04_feature_engineering/
├── 05_experimentation_and_validation/
├── 06_interpretability_and_communication/
└── 07_decision_analysis_and_business_metrics/
```

---

## 03_modeling

### Purpose
Model families, training objectives, and model selection — framework-agnostic conceptual understanding.

### Contains
- Supervised learning (linear models, tree ensembles, kernel methods, instance-based)
- Unsupervised learning (clustering, dimensionality reduction, density estimation, representation)
- Probabilistic models (graphical models, latent variable models, Bayesian modeling)
- Deep learning (MLPs, CNNs, RNNs, Transformers, multimodal)
- Time series (classical, state-space, ML/DL forecasting)
- Training and regularization (loss functions, optimizers, regularization, hyperparameter tuning)
- Evaluation and model selection (metrics, validation, model risk, post-hoc interpretability)

### Does NOT Contain
- Production ML infrastructure (→ 05_ml_engineering)
- Business problem framing (→ 02_data_science)
- Mathematical derivations (→ 01_foundations)
- Cross-layer executable reference architectures (→ 08_implementations)

### Guiding Question
**Which model should we use and why?**

### Sublayer Structure
```
03_modeling/
├── 01_supervised_learning/
│   ├── 01_linear_and_glm/
│   ├── 02_tree_based_models/
│   ├── 03_kernel_methods/
│   └── 04_instance_based_methods/
├── 02_unsupervised_learning/
│   ├── 01_clustering/
│   ├── 02_dimensionality_reduction/
│   ├── 03_density_estimation/
│   └── 04_representation_learning/
├── 03_probabilistic_models/
│   ├── 01_graphical_models/
│   ├── 02_latent_variable_models/
│   └── 03_bayesian_modeling/
├── 04_deep_learning/
│   ├── 01_mlp_and_representation_learning/
│   ├── 02_convolutional_networks/
│   ├── 03_sequence_models/
│   ├── 04_transformers/
│   └── 05_multimodal_models/
├── 05_time_series/
│   ├── 01_classical_forecasting/
│   ├── 02_state_space_and_probabilistic/
│   └── 03_ml_and_dl_forecasting/
├── 06_training_and_regularization/
└── 07_evaluation_and_model_selection/
```

---

## 04_software_engineering

### Purpose
General software engineering knowledge supporting all technical work.

### Contains
- Principles and design patterns (SOLID, clean code, DDD)
- Programming languages (Python, Go, TypeScript, JavaScript)
- System design and distributed systems
- API design (REST, FastAPI, gRPC)
- Databases and storage (SQL, NoSQL, caching)
- Testing strategies (TDD, property-based testing)
- DevOps and infrastructure (Docker, Kubernetes, CI/CD, Git/GitHub)
- Security and sandboxing
- AI-assisted engineering (LLM code generation, MCP, agentic coding)

### Does NOT Contain
- Statistical theory
- Business domain reasoning
- ML evaluation frameworks (unless purely tooling-related)

### Guiding Question
**How do we build robust software systems?**

### Sublayer Structure
```
04_software_engineering/
├── 01_principles_and_patterns/
├── 02_programming_languages/
│   ├── 00_python/
│   ├── 01_go/
│   ├── 02_javascript/
│   └── 03_typescript/
├── 03_system_design/
├── 04_apis_and_services/
├── 05_databases_and_storage/
├── 06_testing_and_quality/
├── 07_devops_and_infrastructure/
│   └── 01_git/
├── 08_security/
└── 09_ai_assisted_software_engineering/
```

---

## 05_ml_engineering

### Purpose
Productionization of machine learning systems (model-agnostic). Covers the full ML production lifecycle from data ingestion to model retirement.

### Contains
- Data engineering (pipelines, ETL/ELT, batch vs stream, feature stores)
- Training data (labeling, sampling, augmentation, versioning)
- Feature engineering (production pipelines and feature stores)
- Model development (experiment tracking, distributed training, offline evaluation)
- Deployment and serving (batch/online/edge, compression, rollout strategies)
- Monitoring and observability (drift detection, operational metrics, alerting)
- Continual learning (retraining triggers, test-in-production)
- Infrastructure and platform (orchestration, ML platforms, environment management)

### Does NOT Contain
- Foundation-model-based system design (→ 06_ai_engineering)
- Model family selection and statistical trade-offs (→ 03_modeling)
- Mathematical derivations (→ 01_foundations)

### Guiding Question
**How do we build reliable, scalable ML systems in production?**

### Sublayer Structure
```
05_ml_engineering/
├── 01_principles_and_lifecycle/
├── 02_data_engineering/
├── 03_training_data/
├── 04_feature_engineering/
├── 05_model_development/
├── 06_deployment_and_serving/
├── 07_monitoring_and_observability/
├── 08_continual_learning/
└── 09_infrastructure_and_platform/
```

---

## 06_ai_engineering

### Purpose
Productionization and integration of foundation models and LLM-based systems. Covers the full AI engineering lifecycle from model selection to production feedback loops.

### Contains
- Foundation models (architecture, scaling, alignment, tokenization)
- Evaluation (benchmarks, AI-as-judge, code eval, LM Eval Harness)
- Prompt engineering (CoT, few-shot, structured outputs, injection defense)
- RAG and agents (retrieval architectures, vector stores, agentic loop, multi-agent, DSPy)
- Fine-tuning (LoRA/QLoRA, RLHF/DPO/GRPO, Axolotl, LLaMA-Factory)
- Dataset engineering (instruction data, synthetic data, data flywheel)
- Inference optimization (quantization: AWQ/GPTQ/GGUF, Flash Attention, KV cache, vLLM)
- Architecture and feedback (AI app architecture, LLMOps, safety, model gateway)

### Does NOT Contain
- Transformer mathematical derivations (→ 01_foundations/06_deep_learning_theory)
- General ML infrastructure (→ 05_ml_engineering)
- Domain-specific application patterns (→ 07_applications)

### Guiding Question
**How do we integrate and operate large-scale pretrained models?**

### Sublayer Structure
```
06_ai_engineering/
├── 01_foundation_models/
├── 02_evaluation/
├── 03_prompt_engineering/
├── 04_rag_and_agents/
├── 05_finetuning/
├── 06_dataset_engineering/
├── 07_inference_optimization/
└── 08_architecture_and_feedback/
```

---

## 07_applications

### Purpose
Real-world use cases, business functions, and domain vertical patterns. This layer captures *why* we build ML/AI systems — the problem, the stakeholders, the decision context, and the domain-specific constraints.

### Contains
- Prediction and forecasting use cases (demand forecasting, churn, risk scoring)
- Recommendation and ranking systems
- Detection and monitoring applications (fraud, anomaly detection)
- Classification and decisioning workflows (credit scoring, triage)
- Generation and assistance systems (summarization, code assistants)
- Search and retrieval applications (semantic search, RAG assistants)
- Multimodal systems (visual inspection, document intelligence)
- Domain verticals (insurance, finance, health, e-commerce, mobility, operations)

### Does NOT Contain
- Cross-layer executable reference architectures (→ 08_implementations)
- General model family descriptions (→ 03_modeling)
- Infrastructure/MLOps tooling (→ 05_ml_engineering)

### Guiding Question
**Why are we solving this problem, and what does the full decision context look like?**

### Sublayer Structure
```
07_applications/
├── 01_prediction_and_forecasting/
├── 02_recommendation_and_ranking/
├── 03_detection_and_monitoring/
├── 04_classification_and_decisioning/
├── 05_generation_and_assistance/
├── 06_search_and_retrieval/
├── 07_multimodal_systems/
└── 08_domain_verticals/
    ├── 01_insurance/
    ├── 02_finance/
    ├── 03_health/
    ├── 04_ecommerce/
    ├── 05_mobility/
    └── 06_operations/
```

---

## 08_implementations

### Purpose
Cross-layer, reusable executable reference architectures: system patterns that integrate multiple layers, and end-to-end examples that span data ingestion through deployment.

**Key placement rule:** *Implementation notes live with the concept unless they are cross-cutting reference architectures that integrate multiple layers.*

- If an implementation note is primarily the code expression of **one concept**, it lives with that concept in its home layer (e.g., XGBoost code → `03_modeling/01_supervised_learning/02_tree_based_models/`; SHAP workflow → `03_modeling/07_evaluation_and_model_selection/`; LoRA fine-tuning → `06_ai_engineering/05_finetuning/`).
- If an implementation note is **reusable across layers** and represents a broader executable architecture or production pattern, it belongs here.

### Contains
- System pattern notes (production patterns integrating ≥2 layers: feature stores, training pipelines, RAG, monitoring, agents, quantization)
- End-to-end example notes (complete ML/AI systems spanning multiple source layers)

### Does NOT Contain
- Concept-specific model implementation notes (→ home layer alongside the concept)
- Core mathematical derivations (→ 01_foundations)
- Algorithmic definitions and model selection rationale (→ 03_modeling)
- Business domain problem framing (→ 07_applications)
- Cross-layer executable reference architectures that are project-specific (→ 09_projects)

### Guiding Question
**How is this cross-cutting executable pattern or end-to-end system implemented in code?**

### Sublayer Structure
```
08_implementations/
├── 01_system_patterns/         ← production pattern implementations spanning ≥2 layers
└── 02_end_to_end_examples/     ← complete ML/AI system walkthroughs
```

---

# 3. Projects (09_projects)

## Purpose

Projects are time-bound execution instances.

They integrate:
- Foundations
- Modeling
- Engineering
- Applications

Projects are not abstraction layers.

---

## Structure

```
09_projects/
├── _active/
├── _completed/
└── _experimental/
```

Each project has its own folder:
```
09_projects/_active/project_name/
```

Recommended internal structure:
```
- overview.md
- problem_definition.md
- data_pipeline.md
- modeling.md
- evaluation.md
- deployment.md
- lessons_learned.md
```

---

## Project Rules

1. Projects may reference any layer.
2. Projects must not duplicate core knowledge.
3. Reusable insights must be extracted to the appropriate layer.
4. Temporary experimentation is allowed inside projects.
5. Projects are integration hubs, not theory repositories.

---

## Extraction Rule

When content becomes:
- Generalizable
- Reusable across projects
- Conceptually stable

Move it to the appropriate core layer and replace the project content with a link.

---

# 4. Reading (10_reading)

## Purpose

Structured intake of external material.

Contains notes derived from:
- Papers
- Books
- Articles
- Documentation
- Industry reports

---

## Structure

```
10_reading/
├── 01_books/
├── 02_papers/
└── 03_articles_and_posts/
```

---

## Reading Rules

1. Reading notes summarize and interpret external content.
2. They must link to relevant core layers.
3. If a concept becomes central → extract to core layer.
4. Avoid storing raw highlights without synthesis.

Reading is intake. Core layers are synthesis.

---

# 5. Logs (11_logs)

## Purpose

Operational and temporary memory.

Contains:
- Daily notes
- Weekly reviews
- Meeting summaries
- Brain dumps
- Reflections

---

## Structure

```
11_logs/
├── daily/
└── weekly/
```

---

## Log Rules

1. Logs are not permanent knowledge.
2. Valuable insights must be extracted to core layers.
3. Logs may contain unstructured content.
4. Logs are reviewed weekly for extraction.

---

# 6. Abstraction Placement Rule

When unsure where a note belongs, ask:

| Question | Layer |
|----------|-------|
| Why does this work mathematically? | 01_foundations |
| How do we reason from data to decisions? | 02_data_science |
| Which model should we use and why? | 03_modeling |
| How do we build robust software systems? | 04_software_engineering |
| How do we productionize ML systems? | 05_ml_engineering |
| How do we integrate and operate LLM systems? | 06_ai_engineering |
| Why are we solving this problem in this domain? | 07_applications |
| How is this cross-cutting executable pattern or end-to-end system implemented? | 08_implementations |
| Is this a concrete execution instance? | 09_projects |
| Is this intake from external material? | 10_reading |
| Is this temporary thought or reflection? | 11_logs |

**Boundary cases:**
- Feature engineering conceptual approach → `02_data_science/04_feature_engineering/`
- Feature store as production system → `05_ml_engineering/04_feature_engineering/`
- SHAP for communicating to stakeholders → `02_data_science/06_interpretability_and_communication/`
- SHAP post-hoc model analysis → `03_modeling/07_evaluation_and_model_selection/`
- SHAP implementation code → `03_modeling/07_evaluation_and_model_selection/interpretability_implementation` (concept-specific, lives with the concept)
- MLflow training pipeline pattern (cross-layer) → `08_implementations/01_system_patterns/`

---

# 7. Naming Conventions

## File Names

- Lowercase only
- Underscores instead of spaces
- Singular nouns preferred

Examples:
- logistic_regression.md
- gradient_boosting.md
- docker_networking.md
- insurance_pricing.md

Avoid:
- CamelCase
- Spaces
- Generic names (e.g., "notes", "stuff")

---

## Folder Names

- Lowercase
- Descriptive
- No vague categories (e.g., misc, advanced_topics)

---

# 8. Note Structure Standards

## Template Mapping

Each layer has a canonical template in `00_meta/templates/`:

| Layer | Template |
|-------|----------|
| 01_foundations | `tpl_foundation.md` or `tpl_proof.md` (for derivations) |
| 02_data_science | `tpl_data_science.md` |
| 03_modeling | `tpl_model.md` |
| 04_software_engineering | `tpl_software_note.md` |
| 05_ml_engineering | `tpl_ml_system.md` |
| 06_ai_engineering | `tpl_ai_system.md` |
| 07_applications | `tpl_application.md` |
| 08_implementations | `tpl_reference_implementation.md` (implementations) or `tpl_end_to_end_example.md` |
| 09_projects | `tpl_project_overview.md` |
| 10_reading | `tpl_reading_note.md` |

## Frontmatter

Every note must include YAML frontmatter:

```yaml
---
title: "Note Title"
tags: [tag1, tag2]
type: concept
layer: 02_data_science
status: seed | growing | evergreen
---
```

- `type`: `concept`, `algorithm`, `model`, `pattern`, `workflow`, `proof`, `engineering`, `ai_system`, `application`, `project`, `index`
- `layer`: the layer folder (e.g., `03_modeling`)
- `status`: `seed` (stub), `growing` (substantive but incomplete), `evergreen` (complete, reviewed)

## Required Sections (all notes)

All notes end with:
```
## References

## Links
```

The `## Links` section organizes wikilinks by source layer.

---

# 9. Cross-Link Governance

To prevent silos:

1. Every `08_implementations` note must link to:
   - At least one `05_ml_engineering` or `06_ai_engineering` note (system context)
   - At least one `03_modeling` note where applicable (if the pattern has a model component)

2. Every `03_modeling` note should link to:
   - At least one `01_foundations` note

3. Every `07_applications` note should link to:
   - At least one `03_modeling` note (which model approaches apply)
   - At least one `05_ml_engineering` or `06_ai_engineering` note (production context)

4. Engineering notes support outward but should not contain theory.

5. Avoid duplicate conceptual notes. If overlap occurs → merge.

---

# 10. Knowledge Lifecycle

Information flows as follows:

Reading → Logs → Projects → Core Layers

Core layers are stable. Everything else is transitional.

---

# 11. Stability Principle

The top-level folder structure is stable.

Any structural change requires:
- Clear justification
- Update to this document

This vault is a professional knowledge architecture, not a casual note collection.

---

# 12. Tag Governance

Tags are lightweight, cross-cutting metadata. They must **not** duplicate the folder hierarchy; they capture attributes that cut across multiple layers.

## Purpose of Tags

Folders encode *where a note belongs epistemically*. Tags encode *what kind of thing it is* and *what cross-cutting attribute it has*. Use tags to enable discovery across layers — e.g., "show me all `retrieval` notes regardless of layer."

## Controlled Vocabulary

All tags must come from the approved list in [[00_meta/tag_vocabulary|Tag Vocabulary]]. See that document for the full controlled vocabulary organized by dimension. The key dimensions are: concept type, lifecycle stage, domain/modality, capability/task, and infra/system concern.

## Rules

1. **No structural redundancy.** Never tag a note with an attribute already encoded by its folder path. Examples of forbidden tags:
   - `python` on notes inside `04_software_engineering/02_programming_languages/01_python/`
   - `deep-learning` on notes inside `01_foundations/06_deep_learning_theory/`
   - `mlops` on notes inside `05_ml_engineering/`
   - `llm` on notes inside `06_ai_engineering/` (unless the note is also relevant from another layer's perspective)

2. **No tool-specific tags.** Never use tags like `mlflow`, `langchain`, `xgboost`, `shap`, `pytorch`. Use the concept/capability tags instead (e.g., `workflow`, `retrieval`, `algorithm`, `interpretability`).

3. **`llm` exception.** `llm` is the single permitted technology-name tag. Use it when a note is specifically about large language model systems and lives outside `06_ai_engineering/`.

4. **Kebab-case only.** All tags are lowercase with hyphens: `fine-tuning`, not `finetuning` or `fine_tuning`.

5. **Maximum 4 tags per note.** Prefer 1–3 tags. Over-tagging defeats filtering utility.

6. **Index files must have `tags: []`.** They are navigation nodes, not content.

## What Not to Tag

- Folder-level attributes (already encoded by path)
- Specific algorithm names (`svd`, `kmeans`, `adam`) — use `algorithm` instead
- Specific model names (`bert`, `gpt`, `resnet`) — use the domain/capability tag instead

## Tag Application Guidelines

When assigning tags to a note, ask:
1. **Does this note describe a specific named algorithm?** → `algorithm`
2. **Does this note describe an architectural/design pattern?** → `pattern`
3. **Does this note describe mathematical/statistical theory?** → `theory`
4. **Does this note describe an operational process?** → `workflow`
5. **What stage of the ML/AI lifecycle does it serve?** → lifecycle stage tag (if not already encoded by folder)
6. **What data modality or domain does it target?** → domain tag (if not already encoded by folder)
7. **What task does it enable?** → capability tag
