---
layer: 08_implementations
type: index
status: growing
tags: []
created: 2026-03-05
---

# System Patterns

Reusable production patterns for ML/AI systems in code.

> How do I build this type of ML/AI system component?

Each note covers the design, code structure, key configuration, and integration points for a specific production pattern. Synthesises `05_ml_engineering/` and `06_ai_engineering/` engineering knowledge into runnable implementations.

## Notes

**Experiment & Data Management**
- [[mlflow_experiment_tracking|MLflow Experiment Tracking]] — run logging, artifact storage, model registry with named aliases
- [[dvc_dataset_versioning|DVC Dataset Versioning]] — data versioning, pipeline caching, remote storage
- [[feature_store_pattern|Feature Store Pattern]] — offline/online feature computation, point-in-time correctness
- [[training_pipeline_pattern|Training Pipeline Pattern]] — orchestrated data → features → training → registry workflow

**Model Serving**
- [[model_serving_with_fastapi|Model Serving with FastAPI]] — REST inference API with Pydantic, background workers, health checks
- [[vllm_serving|vLLM Serving]] — high-throughput LLM serving with PagedAttention and OpenAI-compatible API

**Training & Fine-tuning**
- [[distributed_training_with_accelerate|Distributed Training with Accelerate]] — multi-GPU/TPU training with Hugging Face Accelerate
- [[deep_learning_training_patterns|Deep Learning Training Patterns (PyTorch)]] — MLP, CNN, LSTM training loops with early stopping, validation, and model serialisation
- [[peft_lora_finetuning|PEFT LoRA Fine-tuning]] — parameter-efficient fine-tuning with LoRA/QLoRA
- [[trl_preference_training|TRL Preference Training]] — DPO/RLHF/GRPO preference optimisation
- [[quantization_deployment_pattern|Quantization Deployment Pattern]] — AWQ, GPTQ, GGUF quantization for inference

**RAG & Agents**
- [[rag_pipeline_pattern|RAG Pipeline Pattern]] — indexing pipeline, query pipeline, hybrid retrieval
- [[vector_database_retrieval|Vector Database Retrieval]] — FAISS, pgvector, Qdrant comparison and implementation
- [[chroma_vector_store|Chroma Vector Store]] — Chroma setup, embedding, CRUD, metadata filtering
- [[agentic_loop_pattern|Agentic Loop Pattern]] — ReAct agent, tool use, multi-agent with LangGraph/CrewAI

**LLM Tooling**
- [[instructor_structured_outputs|Instructor Structured Outputs]] — validated JSON extraction from LLMs with Pydantic
- [[dspy_prompt_optimization|DSPy Prompt Optimization]] — automatic prompt optimisation with DSPy signatures
- [[llm_evaluation_pipeline|LLM Evaluation Pipeline]] — LM Eval Harness, LLM-as-judge, custom benchmarks
- [[langsmith_llm_observability|LangSmith LLM Observability]] — tracing, dataset management, prompt versioning
- [[llamaguard_content_moderation|LlamaGuard Content Moderation]] — safety classification for LLM inputs and outputs

**Monitoring & Quality**
- [[drift_monitoring_with_evidently|Drift Monitoring with Evidently]] — feature/prediction drift detection, data quality reports
- [[model_monitoring_system|Model Monitoring System]] — operational metrics, alerting, retraining triggers

**Infrastructure**
- [[docker_ml_pipeline|Docker ML Pipeline]] — containerized training and serving images
- [[cicd_for_ml|CI/CD for ML]] — automated testing, model evaluation, and deployment pipelines
- [[mcp_server_implementation|MCP Server Implementation]] — Model Context Protocol server for LLM tool integration

## Links
- [[08_implementations/index|08 — Implementations]]
- [[08_implementations/02_end_to_end_examples/index|End-to-End Examples]]
- [[05_ml_engineering/index|05 — ML Engineering]]
- [[06_ai_engineering/index|06 — AI Engineering]]
