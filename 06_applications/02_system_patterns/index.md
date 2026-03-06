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
| [[feature_store_pattern\|Feature Store Pattern]] | Feast online/offline feature store, point-in-time correct joins, and low-latency online retrieval |
| [[training_pipeline_pattern\|Training Pipeline Pattern]] | End-to-end training pipeline: data ingestion → validation → features → train → evaluation gate → registry (DVC + MLflow + Airflow) |
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
| [[quantization_deployment_pattern\|Quantization Deployment Pattern]] | Decision table (AWQ/GPTQ/GGUF/bitsandbytes), concrete quantization code for each format, vLLM serving |

### Prompt Engineering and Structured Output
| Note | Description |
|------|-------------|
| [[instructor_structured_outputs\|Instructor Structured Outputs]] | Type-safe LLM output extraction with Pydantic validation and automatic retry |
| [[dspy_prompt_optimization\|DSPy Prompt Optimization]] | Declarative LM programming and data-driven prompt optimization with DSPy |

### Retrieval and Memory
| Note | Description |
|------|-------------|
| [[chroma_vector_store\|Chroma Vector Store]] | Embedding database setup, similarity search, metadata filtering, and LangChain/LlamaIndex integration |
| [[vector_database_retrieval\|Vector Database Retrieval]] | FAISS vs Chroma decision guide; IVF and HNSW index types; batch upsert; k-NN query patterns |
| [[rag_pipeline_pattern\|RAG Pipeline Pattern]] | Full RAG pipeline: document loading, chunking strategy, embedding, Chroma indexing, hybrid search, reranking |

### Monitoring and Feedback
| Note | Description |
|------|-------------|
| [[drift_monitoring_with_evidently\|Drift Monitoring with Evidently]] | Statistical drift detection reports, test suites, and Airflow-scheduled automated alerting |
| [[model_monitoring_system\|Model Monitoring System]] | End-to-end drift detection + Prometheus metrics export + alerting rules + automated retraining trigger |
| [[langsmith_llm_observability\|LangSmith LLM Observability]] | Tracing, evaluation datasets, LLM-as-judge scoring, and production cost monitoring |
| [[llm_evaluation_pipeline\|LLM Evaluation Pipeline]] | Golden dataset creation, LLM-as-judge rubrics, LangSmith evaluate(), A/B comparison, CI regression gates |

### Safety and Guardrails
| Note | Description |
|------|-------------|
| [[llamaguard_content_moderation\|LlamaGuard Content Moderation]] | Input/output safety classification, vLLM deployment, and NeMo Guardrails integration |

### Testing and Quality
| Note | Description |
|------|-------------|
| [[pytest_testing_patterns\|pytest Testing Patterns]] | Fixture composition, parametrize, test doubles (mocker), async tests, and CI coverage integration |

### Agent Architectures
| Note | Description |
|------|-------------|
| [[agentic_loop_pattern\|Agentic Loop Pattern]] | ReAct single-agent loop (LangChain), multi-agent workflow (CrewAI), and MCP tool-calling example |
| [[mcp_server_implementation\|MCP Server Implementation]] | MCP server exposing tools, resources, and prompts to AI coding assistants in Python and TypeScript |

### Service Implementation Patterns
| Note | Description |
|------|-------------|
| [[grpc_service_implementation\|gRPC Service Implementation]] | Protobuf schema design, Python server/client, server-streaming, TLS, and testing gRPC services |

### DevOps and Infrastructure
| Note | Description |
|------|-------------|
| [[docker_ml_pipeline\|Docker for ML Pipelines]] | CUDA base images, multi-stage inference builds, GPU runtime, Docker Compose for local ML stack |
| [[cicd_for_ml\|CI/CD for ML Pipelines]] | GitHub Actions workflows for data validation, training gates, model registration, and canary deploy |
| [[kubernetes_deployment\|Kubernetes Deployment Patterns]] | Deployment/Service/Ingress/HPA manifests, rolling update config, and Helm chart patterns |

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
- Projects