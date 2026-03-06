---
layer: 06_applications
type: application
status: seed
tags: []
created: 2026-03-06
---
# System Patterns  
  
Reusable implementation patterns for building machine learning and AI systems.  
  
> How are engineering concepts from software, ML, and AI systems implemented in practice?  
  
This section focuses on **system-level implementation patterns** rather than isolated models.  
  
It shows how concepts from:  
- `03_software_engineering`  
- `04_ml_engineering`  
- `05_ai_engineering`  
  
are translated into concrete architectures, services, pipelines, and operational workflows.  
  
This section does NOT cover:  
- model theory (→ `02_modeling`)  
- isolated algorithm implementations (→ `01_model_implementations`)  
- project-specific execution and business context (→ `07_projects`)  

## Notes

### Data and Training Pipelines
| Note | Description |
|------|-------------|
| [[mlflow_experiment_tracking\|MLflow Experiment Tracking]] | Full MLflow workflow: tracking runs, comparing experiments, model registry with named aliases (3.x) |
| [[dvc_dataset_versioning\|DVC Dataset Versioning]] | Version datasets with DVC + Git; define reproducible data pipelines with `dvc.yaml` |
| [[distributed_training_with_accelerate\|Distributed Training with Accelerate]] | Multi-GPU training with HuggingFace Accelerate, DeepSpeed ZeRO-3, and FSDP |
| [[peft_lora_finetuning\|PEFT and LoRA Fine-tuning]] | LoRA/QLoRA adapter training with PEFT and Axolotl; rank selection and adapter merging |
| [[trl_preference_training\|TRL Preference Training]] | DPO alignment, GRPO reasoning training, and reward modelling with TRL |

### Serving and Integration
| Note | Description |
|------|-------------|
| [[model_serving_with_fastapi\|Model Serving with FastAPI]] | Production REST inference API: model loading, Pydantic schemas, batching, health checks, Docker |
| [[vllm_serving\|vLLM Serving]] | High-throughput LLM serving: PagedAttention, continuous batching, tensor parallelism, AWQ quantization |

### Prompt Engineering and Structured Output
| Note | Description |
|------|-------------|
| [[instructor_structured_outputs\|Instructor Structured Outputs]] | Type-safe LLM output extraction with Pydantic validation and automatic retry |
| [[dspy_prompt_optimization\|DSPy Prompt Optimization]] | Declarative LM programming and data-driven prompt optimization with DSPy |

### Retrieval and Memory
| Note | Description |
|------|-------------|
| [[chroma_vector_store\|Chroma Vector Store]] | Embedding database setup, similarity search, metadata filtering, and LangChain/LlamaIndex integration |

### Monitoring and Feedback
| Note | Description |
|------|-------------|
| [[drift_monitoring_with_evidently\|Drift Monitoring with Evidently]] | Statistical drift detection reports, test suites, and Airflow-scheduled automated alerting |
| [[langsmith_llm_observability\|LangSmith LLM Observability]] | Tracing, evaluation datasets, LLM-as-judge scoring, and production cost monitoring |

### Safety and Guardrails
| Note | Description |
|------|-------------|
| [[llamaguard_content_moderation\|LlamaGuard Content Moderation]] | Input/output safety classification, vLLM deployment, and NeMo Guardrails integration |

### DevOps and Infrastructure
| Note | Description |
|------|-------------|
| [[docker_ml_pipeline\|Docker for ML Pipelines]] | CUDA base images, multi-stage inference builds, GPU runtime, Docker Compose for local ML stack |
| [[cicd_for_ml\|CI/CD for ML Pipelines]] | GitHub Actions workflows for data validation, training gates, model registration, and canary deploy |

---

## Role in the Vault  
  
This section connects **engineering principles** to **working system designs**:  
  
```text  
03_software_engineering  
04_ml_engineering  
05_ai_engineering  
↓  
06_applications/02_system_patterns  
↓  
07_projects
```

## Links
- [[03_software_engineering/index|Software Engineering]]
- [[04_ml_engineering/index|ML Engineering]]
- [[05_ai_engineering/index|AI Engineering]]
- [[07_projects/index|Projects]]