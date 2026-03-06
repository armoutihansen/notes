# Vault Conventions  
  
## 1. Purpose of This Vault  
  
This vault is a structured professional knowledge system supporting:  
  
- Mathematical and statistical foundations  
- Machine learning modeling  
- Software engineering depth  
- ML engineering (production systems)  
- AI engineering (LLM systems)  
- Insurance domain expertise  
- Active and completed projects  
  
The vault is organized by **epistemic layer and abstraction level**, not by buzzwords or temporary interests.  
  
Top-level structure:
```
00_meta  
01_foundations  
02_modeling  
03_software_engineering  
04_ml_engineering  
05_ai_engineering  
06_applications  
07_projects  
08_reading  
09_logs  
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
  
---  
## 02_modeling    
### Purpose  
Model classes and modeling strategies, framework-agnostic.  
### Contains  
- Supervised learning  
- Unsupervised learning  
- Deep learning architectures  
- Probabilistic models  
- Feature engineering  
- Evaluation metrics  
- Insurance modeling techniques (e.g., GLMs for pricing)  
### Does NOT Contain  
- Docker, CI/CD, orchestration  
- MLflow, Airflow  
- API deployment instructions  
- Pure mathematical derivations  
### Guiding Question  
**How do we model this problem?**  
  
Modeling notes describe algorithms, inductive biases, trade-offs, and modeling logic.  
  
---  
## 03_software_engineering  

### Purpose  
General software engineering knowledge supporting all technical work.    
### Contains  
- Programming languages (Python, Go, TypeScript, JavaScript)
- Principles and design patterns (SOLID, clean code)
- System design and distributed systems
- API design (REST, FastAPI, gRPC)
- Databases and storage (SQL, NoSQL, caching)
- Testing strategies (TDD, property-based testing)
- DevOps and infrastructure (Docker, Kubernetes, CI/CD)
- Version control (Git, GitHub)
- Security and sandboxing
- AI-assisted engineering (LLM code generation, MCP, agentic coding)
### Does NOT Contain  
- Statistical theory  
- Business domain reasoning  
- ML evaluation frameworks (unless purely tooling-related)  
### Guiding Question  
**How do we build robust software systems?**  
  
If the note is about implementation patterns or tooling → it belongs here.

### Sublayer Structure
```
03_software_engineering/
├── 00_principles_and_patterns/
├── 01_programming_and_runtime/
│   ├── 00_python/
│   ├── 01_go/
│   ├── 02_javascript/
│   └── 03_typescript/
├── 02_system_design/
├── 03_apis_and_services/
├── 04_databases_and_storage/
├── 05_testing_and_quality/
├── 06_devops_and_infrastructure/
│   └── 00_ml_frameworks/
├── 07_security/
├── 08_version_control/
│   └── 00_git/
└── 09_ai_assisted_engineering/
```
  
---  
## 04_ml_engineering

### Purpose
Productionization of machine learning systems (model-agnostic). Covers the full ML production lifecycle from data ingestion to model retirement.

### Contains
- Data engineering (pipelines, ETL/ELT, batch vs stream, feature stores)
- Training data (labeling, sampling, augmentation, versioning)
- Feature engineering (encoding, leakage prevention, feature stores)
- Model development (experiment tracking, distributed training, offline evaluation)
- Deployment and serving (batch/online/edge, compression, rollout strategies)
- Monitoring and observability (drift detection, operational metrics, alerting)
- Continual learning (retraining triggers, test-in-production)
- Infrastructure and platform (orchestration, ML platforms, environment management)

### Does NOT Contain
- Foundation-model-based system design (→ 05_ai_engineering)
- Model family selection and statistical trade-offs (→ 02_modeling)
- Mathematical derivations (→ 01_foundations)

### Guiding Question
**How do we build reliable, scalable ML systems in production?**

### Sublayer Structure
```
04_ml_engineering/
├── 00_principles_and_lifecycle/
├── 01_data_engineering/
├── 02_training_data/
├── 03_feature_engineering/
├── 04_model_development/
├── 05_deployment_and_serving/
├── 06_monitoring_and_observability/
├── 07_continual_learning/
└── 08_infrastructure_and_platform/
```

---
## 05_ai_engineering

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
- General ML infrastructure (→ 04_ml_engineering)
- Pure business problem framing (→ 06_applications)

### Guiding Question
**How do we integrate and operate large-scale pretrained models?**

### Sublayer Structure
```
05_ai_engineering/
├── 00_foundation_models/
├── 01_evaluation/
├── 02_prompt_engineering/
├── 03_rag_and_agents/
├── 04_finetuning/
├── 05_dataset_engineering/
├── 06_inference_optimization/
└── 07_architecture_and_feedback/
```  
  
---  
## 06_applications  

### Purpose  
Synthesis layer bridging foundations, modeling, and engineering into working implementations and end-to-end systems.

### Contains
- Model implementation notes (concrete runnable code for each model family)
- System pattern notes (how to build production patterns: feature stores, training pipelines, RAG, monitoring, agents, quantization)
- End-to-end example notes (complete ML/AI systems spanning multiple source layers)

### Does NOT Contain  
- Core mathematical derivations  
- Algorithmic definitions (the "what" and "why" → 02_modeling)
- Infrastructure-only tooling notes (→ 04_ml_engineering)

### Sublayer Structure
```
06_applications/
├── 01_model_implementations/   ← runnable code per model family
├── 02_system_patterns/         ← production pattern implementations
└── 03_end_to_end_examples/     ← complete ML/AI system walkthroughs
```

### Guiding Question  
**How do we implement and integrate this knowledge into a working system?**

Applications synthesize knowledge from all other layers into concrete implementations.  
  
---  
# 3. Projects (07_projects)  

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
07_projects/  
_active/  
_completed/  
_experimental/
```

Each project has its own folder:
```
07_projects/_active/project_name/
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
  
Move it to:  
- Foundations  
- Modeling  
- Software engineering  
- ML engineering  
- AI engineering  
- Applications  
  
Replace the project content with a link.  
  
---  
# 4. Reading (08_reading)  

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
08_reading/  
papers/  
books/  
articles/  
reports/
```
---  
  
## Reading Rules  
  
1. Reading notes summarize and interpret external content.  
2. They must link to relevant core layers.  
3. If a concept becomes central → extract to core layer.  
4. Avoid storing raw highlights without synthesis.  
  
Reading is intake.  
Core layers are synthesis.  
  
---  
# 5. Logs (09_logs)  

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
09_logs/  
daily/  
weekly/
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
  
- Why does this work mathematically? → Foundations  
- How do we model this? → Modeling  
- How do we implement this? → Software engineering  
- How do we productionize this? → ML engineering  
- How do we integrate LLM systems? → AI engineering  
- Why are we solving this problem? → Applications  
- Is this a concrete execution instance? → Projects  
- Is this intake from external material? → Reading  
- Is this temporary thought or reflection? → Logs  
  
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
  
## Conceptual / Modeling Notes

```
## Definition

## Intuition

## Formal Description

## Applications

## Trade-offs

## Links
```
---  
## Engineering Notes

Used in `03_software_engineering/`, `04_ml_engineering/`, `05_ai_engineering/`.

```
## Purpose

## Architecture

## Implementation Notes

## Trade-offs

## References

## Links
```

**Note on language-level notes**: For programming language reference notes (e.g., Go control flow, JS collections, TypeScript generics), `## Architecture` may be omitted when it is not meaningful. These notes use `## Implementation Notes` as the primary section.



Used in `06_applications/`. Three sub-types share the same template structure:

```
## Purpose

### Examples

## Architecture

## Links
```

Notes in `01_model_implementations/` and `02_system_patterns/` typically expand `## Architecture` with concrete implementation sections (setup, code examples, configuration). Notes in `03_end_to_end_examples/` use `## Architecture` for a component diagram and add a step-by-step implementation sequence.

**Legacy domain-specific format** (for business/domain problem framing, if needed):
```
## Problem Definition

## Domain Context

## Data Requirements

## Modeling Options

## Deployment Constraints

## Risks

## Links
```

---
# 9. Cross-Link Governance

To prevent silos:

1. Every application note must link to:
   - At least one modeling note
   - At least one engineering note

2. Every modeling note should link to:
   - At least one foundational note

3. Engineering notes support outward but should not contain theory.

4. Avoid duplicate conceptual notes.
   If overlap occurs → merge.

---
# 10. Knowledge Lifecycle

Information flows as follows:

Reading → Logs → Projects → Core Layers

Core layers are stable.
Everything else is transitional.

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

All tags must come from the approved list in [[00_meta/tag_vocabulary|Tag Vocabulary]]. The vocabulary is organized into five orthogonal dimensions:

| Dimension | Tags |
|-----------|------|
| **Concept type** | `algorithm`, `pattern`, `theory`, `workflow` |
| **Lifecycle stage** | `data`, `feature-engineering`, `training`, `evaluation`, `deployment`, `monitoring` |
| **Domain / modality** | `llm`, `nlp`, `vision`, `time-series`, `tabular`, `multimodal`, `recommendation` |
| **Capability / task** | `classification`, `regression`, `clustering`, `generation`, `retrieval`, `reasoning`, `forecasting`, `anomaly-detection`, `interpretability` |
| **Infra / system concern** | `mlops`, `llmops`, `distributed`, `quantization`, `fine-tuning`, `safety` |

## Rules

1. **No structural redundancy.** Never tag a note with an attribute already encoded by its folder path. Examples of forbidden tags:
   - `python` on notes inside `03_software_engineering/01_programming_and_runtime/00_python/`
   - `deep-learning` on notes inside `01_foundations/06_deep_learning_theory/`
   - `mlops` on notes inside `04_ml_engineering/`
   - `llm` on notes inside `05_ai_engineering/` (unless the note is also relevant from another layer's perspective)

2. **No tool-specific tags.** Never use tags like `mlflow`, `langchain`, `xgboost`, `shap`, `pytorch`. Use the concept/capability tags instead (e.g., `workflow`, `retrieval`, `algorithm`, `interpretability`).

3. **`llm` exception.** `llm` is the single permitted technology-name tag. Use it when a note is specifically about large language model systems and lives outside `05_ai_engineering/` (e.g., an LLM serving pattern in `06_applications/`).

4. **Kebab-case only.** All tags are lowercase with hyphens: `fine-tuning`, not `finetuning` or `fine_tuning`.

5. **Maximum 4 tags per note.** Prefer 1–3 tags. Over-tagging defeats filtering utility.

6. **Index files must have `tags: []`.** They are navigation nodes, not content.

## What Not to Tag

- Folder-level attributes (already encoded by path)
- Specific algorithm names (`svd`, `kmeans`, `adam`) — use `algorithm` instead
- Specific model names (`bert`, `gpt`, `resnet`) — use the domain/capability tag instead
- Lifecycle-stage words that restate the layer (`training` in `04_ml_engineering/04_model_development/` is redundant)

## Tag Application Guidelines

When assigning tags to a note, ask:
1. **Does this note describe a specific named algorithm?** → `algorithm`
2. **Does this note describe an architectural/design pattern?** → `pattern`
3. **Does this note describe mathematical/statistical theory?** → `theory`
4. **Does this note describe an operational process?** → `workflow`
5. **What stage of the ML/AI lifecycle does it serve?** → lifecycle stage tag (if not already encoded by folder)
6. **What data modality or domain does it target?** → domain tag (if not already encoded by folder)
7. **What task does it enable?** → capability tag