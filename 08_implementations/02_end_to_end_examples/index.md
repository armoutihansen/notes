---
layer: 08_implementations
type: index
status: growing
tags: []
created: 2026-03-05
---

# End-to-End Examples

Complete ML/AI system reference architectures.

> What does a full working system look like for this use case?

Each note is a self-contained system walkthrough combining ≥3 components from ≥2 source layers. Notes include component diagrams, implementation sequences, integration code, and links to all constituent patterns and concepts.

## Notes

**ML Pipelines**
- [[tabular_classification_pipeline|Tabular Classification Pipeline]] — feature engineering + tree ensembles + SHAP + FastAPI + MLflow
- [[batch_ml_prediction_pipeline|Batch ML Prediction Pipeline]] — scheduled batch scoring with DVC, MLflow, and Airflow
- [[continuous_training_pipeline|Continuous Training Pipeline]] — trigger-based retraining with drift detection and registry promotion
- [[demand_forecasting_pipeline|Demand Forecasting Pipeline]] — hierarchical time series forecasting with SARIMAX/LightGBM + monitoring
- [[anomaly_detection_pipeline|Anomaly Detection Pipeline]] — unsupervised + rule-based anomaly detection with alerting

**Deep Learning & LLM Systems**
- [[deep_learning_training_workflow|Deep Learning Training Workflow]] — PyTorch + Accelerate + MLflow + distributed training
- [[nlp_text_classification|NLP Text Classification]] — transformer fine-tuning + PEFT + evaluation + vLLM serving
- [[llm_finetuning_pipeline|LLM Fine-tuning Pipeline]] — data curation + SFT + DPO + vLLM with safety guardrails
- [[rag_qa_system|RAG Q&A System]] — document ingestion + vector store + LLM generation + observability
- [[llm_coding_assistant|LLM Coding Assistant]] — code-specialised RAG + agent + structured output
- [[production_llm_serving_with_safety|Production LLM Serving with Safety]] — vLLM + LlamaGuard + LangSmith + rate limiting

**APIs & Infrastructure**
- [[containerised_api_service|Containerised API Service]] — FastAPI + Docker + Kubernetes + CI/CD

## Links
- [[08_implementations/index|08 — Reference Implementations]]
- [[08_implementations/01_system_patterns/index|System Patterns]]
- [[07_applications/index|07 — Applications]]
