# Layer Placement Decision Guide

Use this guide to decide which vault layer a note belongs in. Ask the questions in order — the first matching layer wins.

---

## Decision Tree

### Does it contain mathematical derivations, proofs, or formal definitions?
→ **`01_foundations`**

Examples: backpropagation derivation, Bayes' theorem proof, Cramér-Rao bound, PAC learning framework

---

### Does it describe a model family, algorithm, or modeling strategy — framework-agnostic?
→ **`02_modeling`**

Examples: how random forests work, transformer architecture, VAE theory, bias-variance tradeoff, evaluation metrics (AUC, RMSE)

---

### Does it describe a programming language, general SE pattern, testing approach, or infrastructure tool used across multiple domains?
→ **`03_software_engineering`**

Examples: Python async patterns, REST API design, Docker multi-stage builds, Git rebase, SQL window functions, property-based testing

---

### Does it describe taking ML models to production — data, features, training systems, deployment, monitoring, retraining, or ML platforms?
→ **`04_ml_engineering`**

Examples: feature store design, MLflow experiment tracking, drift detection, canary deployments, model compression, K8s GPU workloads for ML

**Key test:** Would this note be useful to an ML engineer who is NOT working with LLMs?

---

### Does it describe working with large pretrained models / foundation models, LLM-based systems, or AI-specific tooling?
→ **`05_ai_engineering`**

Examples: RAG architecture, LoRA fine-tuning, prompt engineering, vLLM serving, LangSmith observability, multi-agent systems

**Key test:** Does the note specifically require a foundation model / LLM context?

---

### Does it describe a business domain, industry context, regulatory environment, or stakeholder constraints?
→ **`06_applications`**

Examples: insurance pricing models, GDPR constraints on ML, fraud detection problem framing

---

### Is this a time-bound execution instance with specific deliverables?
→ **`07_projects`**

---

## Edge Cases

### PyTorch training loop — which layer?
- General PyTorch API patterns → `03_software_engineering/06_devops_and_infrastructure/00_ml_frameworks/`
- PyTorch for distributed training in production → `04_ml_engineering/04_model_development/`
- PyTorch for LoRA fine-tuning → `05_ai_engineering/04_finetuning/`

### FastAPI model serving — which layer?
- FastAPI patterns in general → `03_software_engineering/03_apis_and_services/`
- FastAPI specifically as an ML model serving pattern → `04_ml_engineering/05_deployment_and_serving/`

### vLLM serving — which layer?
- vLLM is specifically for LLM inference → `05_ai_engineering/06_inference_optimization/`

### Transformer architecture — which layer?
- Mathematical/theoretical understanding (attention mechanism derivation) → `01_foundations/06_deep_learning_theory/`
- Architecture description for practitioners → `02_modeling/03_model_families/07_transformers/`
- Using transformers in production (tokenization, serving) → `05_ai_engineering/`

### Kubernetes — which layer?
- General K8s basics → `03_software_engineering/06_devops_and_infrastructure/`
- K8s specifically for ML workloads (GPU node pools, training jobs) → `04_ml_engineering/08_infrastructure_and_platform/`

---

## Note Splitting Rules

Split a note if it covers more than ONE of:
- A distinct algorithm or concept
- A distinct tool or library
- A distinct production lifecycle stage

Merge notes if they are so closely related they cannot stand alone (e.g., precision and recall → one note on classification metrics).
