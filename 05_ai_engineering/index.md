---
layer: 05_ai_engineering
type: index
status: evergreen
tags: []
created: 2026-03-05
---

# AI Engineering

Engineering discipline for building, evaluating, and operating applications built on foundation models.

> How do we design, evaluate, optimize, and operate systems built around foundation models?

This layer does NOT cover: general ML pipelines (→ 04_ml_engineering), classical model selection (→ 02_modeling), or mathematical derivations (→ 01_foundations).

## AI System Lifecycle

### [[05_ai_engineering/00_foundation_models/index|00 — Foundation Models]] 
Model families, scaling laws, alignment (RLHF/DPO/GRPO), tokenization.

### [[01_evaluation/index|01 — Evaluation]]
LLM evaluation taxonomy, benchmarks (MMLU/HumanEval), AI-as-judge, LM Eval Harness.

### [[02_prompt_engineering/index|02 — Prompt Engineering]]
CoT, few-shot, structured outputs (Instructor/Guidance), prompt injection defense.

### [[03_rag_and_agents/index|03 — RAG & Agents]]
RAG architectures, vector stores (Chroma/FAISS), agentic loop, function calling, multi-agent (CrewAI), DSPy.

### [[04_finetuning/index|04 — Fine-tuning]]
LoRA/QLoRA (Axolotl/LLaMA-Factory), RLHF/DPO/GRPO (TRL), fine-tuning strategy.

### [[05_dataset_engineering/index|05 — Dataset Engineering]]
Instruction data design, synthetic data generation (Self-Instruct, Constitutional AI).

### [[06_inference_optimization/index|06 — Inference Optimization]]
Quantization (AWQ/GPTQ/GGUF/bitsandbytes), Flash Attention, KV cache, vLLM/llama.cpp.

### [[07_architecture_and_feedback/index|07 — Architecture & Feedback]]
AI application architecture, LLM observability (LangSmith), safety (LlamaGuard), data flywheel.

## Links
- [[04_ml_engineering/index|ML Engineering]]
- [[03_software_engineering/index|Software Engineering]]
