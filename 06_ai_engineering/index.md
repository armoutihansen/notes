---
layer: 06_ai_engineering
type: index
status: evergreen
tags: []
created: 2026-03-05
---

# 06 — AI Engineering

Engineering discipline for building, evaluating, and operating applications built on foundation models — from model selection and evaluation through fine-tuning, inference optimization, and production feedback loops.

> Guiding question: "How do we build reliable, useful systems with foundation models?"

This layer does NOT cover: general ML infrastructure and pipelines (→ 05_ml_engineering), classical model selection and statistical trade-offs (→ 03_modeling), or Transformer mathematical derivations (→ 01_foundations/06_deep_learning_theory).

## Sublayers

### [[06_ai_engineering/01_foundation_models/index|01 — Foundation Models]]
Model families, scaling laws, alignment (RLHF/DPO/GRPO), tokenization.

### [[06_ai_engineering/02_evaluation/index|02 — Evaluation]]
LLM evaluation taxonomy, benchmarks (MMLU/HumanEval), AI-as-judge, LM Eval Harness.

### [[06_ai_engineering/03_prompt_engineering/index|03 — Prompt Engineering]]
CoT, few-shot, structured outputs (Instructor/Guidance), prompt injection defense.

### [[06_ai_engineering/04_rag_and_agents/index|04 — RAG & Agents]]
RAG architectures, vector stores (Chroma/FAISS), agentic loop, function calling, multi-agent (CrewAI), DSPy.

### [[06_ai_engineering/05_finetuning/index|05 — Fine-tuning]]
LoRA/QLoRA (Axolotl/LLaMA-Factory), RLHF/DPO/GRPO (TRL), fine-tuning strategy.

### [[06_ai_engineering/06_dataset_engineering/index|06 — Dataset Engineering]]
Instruction data design, synthetic data generation (Self-Instruct, Constitutional AI).

### [[06_ai_engineering/07_inference_optimization/index|07 — Inference Optimization]]
Quantization (AWQ/GPTQ/GGUF/bitsandbytes), Flash Attention, KV cache, vLLM/llama.cpp.

### [[06_ai_engineering/08_architecture_and_feedback/index|08 — Architecture & Feedback]]
AI application architecture, LLM observability (LangSmith), safety (LlamaGuard), data flywheel.

## Relationship to Other Layers

- **05_ml_engineering** — production ML infrastructure that AI engineering builds on for foundation-model-specific concerns
- **04_software_engineering** — general software patterns applied to LLM system design
- **08_implementations** — concrete implementations synthesizing this layer's concepts into working code

## Links
- [[05_ml_engineering/index|ML Engineering]]
- [[04_software_engineering/index|Software Engineering]]
- [[08_implementations/index|Reference Implementations]]
