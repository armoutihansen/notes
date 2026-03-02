---
layer: 05_ai_engineering
type: index
status: evergreen
created: 2026-03-02
---
# AI Engineering

## Purpose of This Layer

This layer captures the engineering discipline required to build applications on top of pretrained foundation models.

It is aligned with the conceptual framework of modern AI engineering as structured in:
- Foundation models
- Evaluation
- Prompting
- RAG and agents
- Finetuning
- Dataset engineering
- Inference optimization
- System architecture and user feedback

Guiding question:

> How do we design, evaluate, optimize, and operate systems built around foundation models?

This layer does NOT cover:
- General ML pipelines (→ 04_ml_engineering)
- Classical model selection and structured ML (→ 02_modeling)
- Mathematical derivations (→ 01_foundations)

---

## Structure

### 00_foundation_models/

Understanding the underlying models.

Includes:
- Transformer architecture overview
- Scaling laws
- Sampling and decoding
- SFT / RLHF alignment
- Model selection trade-offs

Focus:
Capabilities and constraints of foundation models.

---

### 01_evaluation/

Evaluation methodology for open-ended AI systems.

Includes:
- Language modeling metrics
- Exact vs subjective evaluation
- AI-as-a-judge
- Comparative evaluation
- Evaluation pipeline design
- Benchmark limitations

Focus:
How to systematically evaluate generative systems.

---

### 02_prompt_engineering/

Controlling models via inputs.

Includes:
- Prompt anatomy
- In-context learning
- Structured outputs
- Reasoning prompts
- Prompt injection risks
- Guardrails

Focus:
Human–AI interaction design.

---

### 03_rag_and_agents/

Architectures that extend foundation models.

Includes:
- RAG systems
- Retriever design
- Hybrid search
- Agent planning
- Tool use
- Memory systems

Focus:
System-level augmentation of foundation models.

---

### 04_finetuning/

Modifying model weights.

Includes:
- When to finetune vs RAG
- Full finetuning
- PEFT / LoRA
- Model merging
- Hyperparameter considerations

Focus:
Parameter adaptation strategies.

---

### 05_dataset_engineering/

Designing datasets for adaptation.

Includes:
- Dataset design principles
- Instruction data
- Synthetic data
- Data quality, coverage, quantity
- Dataset evaluation

Focus:
Data as the core adaptation lever.

---

### 06_inference_optimization/

Efficiency and cost optimization.

Includes:
- Latency vs throughput trade-offs
- KV caching
- Quantization
- Distillation
- Batching and parallelism
- Serving trade-offs

Focus:
Making foundation models usable at scale.

---

### 07_architecture_and_feedback/

System-level integration.

Includes:
- AI application architectures
- Model gateways
- Observability for LLM systems
- User feedback loops
- Data flywheel
- Build vs buy decisions

Focus:
Holistic AI system design.

---

## Relationship to Other Layers

02_modeling:
Model choice and evaluation for structured ML systems.

04_ml_engineering:
Production lifecycle for classical ML systems.

05_ai_engineering:
System design around pretrained foundation models.

This layer treats the foundation model as a core system component, not just a predictive model.

---
## Explicit Structure

```
05_ai_engineering/
├── index.md
├── 00_foundation_models/
│   ├── transformer_architecture_overview.md
│   ├── scaling_laws.md
│   ├── sampling_and_decoding.md
│   ├── alignment_sft_rlhf.md
│   └── model_selection_tradeoffs.md
├── 01_evaluation/
│   ├── language_model_metrics.md
│   ├── exact_vs_subjective_eval.md
│   ├── ai_as_a_judge.md
│   ├── comparative_eval.md
│   ├── eval_pipeline_design.md
│   └── benchmark_limitations.md
├── 02_prompt_engineering/
│   ├── prompt_anatomy.md
│   ├── in_context_learning.md
│   ├── reasoning_prompts.md
│   ├── structured_outputs.md
│   ├── prompt_injection_attacks.md
│   └── guardrails.md
├── 03_rag_and_agents/
│   ├── rag_basics.md
│   ├── retriever_design.md
│   ├── term_vs_embedding_retrieval.md
│   ├── hybrid_search.md
│   ├── agent_planning.md
│   ├── tool_use.md
│   └── memory_systems.md
├── 04_finetuning/
│   ├── when_to_finetune_vs_rag.md
│   ├── full_finetuning.md
│   ├── peft_lora.md
│   ├── model_merging.md
│   └── hyperparameters_for_finetuning.md
├── 05_dataset_engineering/
│   ├── dataset_design_principles.md
│   ├── instruction_data.md
│   ├── synthetic_data.md
│   ├── data_quality_coverage_quantity.md
│   └── dataset_evaluation.md
├── 06_inference_optimization/
│   ├── latency_vs_throughput.md
│   ├── kv_cache.md
│   ├── quantization.md
│   ├── distillation.md
│   ├── batching_and_parallelism.md
│   └── serving_tradeoffs.md
├── 07_architecture_and_feedback/
│   ├── ai_application_architecture.md
│   ├── model_gateway.md
│   ├── observability_for_llm_systems.md
│   ├── user_feedback_loops.md
│   ├── data_flywheel.md
│   └── build_vs_buy_models.md
```
