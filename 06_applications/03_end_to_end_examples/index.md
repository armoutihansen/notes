---
layer: 06_applications
type: index
status: growing
tags: []
created: 2026-03-06
---
  
Small complete systems that combine modeling, software engineering, ML engineering, and AI engineering into a coherent implementation.  
  
> What does a full working system look like when multiple layers come together?  
  
This section contains compact reference examples that integrate:  
- models  
- data workflows  
- APIs or interfaces  
- deployment logic  
- monitoring or feedback loops  
  
These notes are larger than single implementation notes, but smaller and more reusable than full project documentation.  
  
This section does NOT cover:  
- abstract theory without implementation (→ `01_foundations`, `02_modeling`)  
- isolated algorithm notes (→ `01_model_implementations`)  
- reusable single-pattern system designs only (→ `02_system_patterns`)  
- live project tracking and project-specific artifacts (→ `07_projects`)  
  
## Example Categories

### Predictive ML Systems
| Note | Description |
|------|-------------|
| [[tabular_classification_pipeline\|Tabular Classification Pipeline]] | Raw data → EDA → feature pipeline → XGBoost + MLflow → SHAP explanations → FastAPI serving → Evidently drift monitoring |
| [[batch_ml_prediction_pipeline\|Batch ML Prediction Pipeline]] | Full tabular ML lifecycle: data pipeline → training → evaluation → batch scoring → drift monitoring |
| [[continuous_training_pipeline\|Continuous Training Pipeline]] | Closed-loop CT system: drift detection → automated retraining → evaluation gate → canary deployment |
| [[demand_forecasting_pipeline\|Demand Forecasting Pipeline]] | SARIMA + LightGBM ensemble for monthly series: EDA, walk-forward CV, lag features, deployment |
| [[anomaly_detection_pipeline\|Anomaly Detection Pipeline]] | Isolation Forest + GMM ensemble: PCA preprocessing, unsupervised scoring, UMAP inspection, PR-AUC evaluation |

### Deep Learning Systems
| Note | Description |
|------|-------------|
| [[deep_learning_training_workflow\|Deep Learning Training Workflow]] | PyTorch model → Accelerate multi-GPU → MLflow tracking → checkpoint → evaluation → model registry |

### Software Service Systems
| Note | Description |
|------|-------------|
| [[containerised_api_service\|Containerised REST API Service]] | FastAPI + Docker + Kubernetes + GitHub Actions CI/CD: full lifecycle from code to production pods |
| [[llm_coding_assistant\|LLM Coding Assistant]] | MCP server + code RAG + agentic loop: an LLM assistant that reads, writes, and tests your codebase |

### Intelligent Data Products
*Notes to be added.*

### Foundation Model Applications
| Note | Description |
|------|-------------|
| [[nlp_text_classification\|NLP Text Classification]] | Text dataset → tokenizer → LoRA fine-tuning (SFTTrainer) → F1 evaluation → vLLM deployment → FastAPI wrapper |
| [[rag_qa_system\|RAG Question-Answering System]] | Full RAG pipeline: chunking → Chroma → reranking → LLM → FastAPI + LangSmith observability |
| [[llm_finetuning_pipeline\|LLM Fine-tuning Pipeline]] | SFT + DPO pipeline with LoRA/QLoRA, MLflow tracking, evaluation, and vLLM deployment |
| [[production_llm_serving_with_safety\|Production LLM Serving with Safety Stack]] | vLLM + LlamaGuard + semantic cache + LangSmith gateway with fallback routing |
  
## Role in the Vault  
  
This section acts as the final implementation bridge before real projects:  
  
```text  
02_modeling  
03_software_engineering  
04_ml_engineering  
05_ai_engineering  
↓  
06_applications/03_end_to_end_examples  
↓  
07_projects
```

These notes should help answer:
- how the pieces fit together
- what the architecture looks like
- what an implementation sequence might be
- which reusable patterns can later be transferred into projects

## Links
- [[06_applications/01_model_implementations/index|Model Implementations]]
- [[06_applications/02_system_patterns/index|System Patterns]]
- Projects