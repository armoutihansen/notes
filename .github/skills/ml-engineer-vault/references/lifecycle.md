# ML Production Lifecycle Reference

The `04_ml_engineering` layer is structured around the ML production lifecycle. Each stage has a clear scope and flows into the next.

---

## Stage 00: Principles & Lifecycle Overview

**Purpose:** Establish reliability contract, architecture decisions, and the iterative nature of ML systems.

**Key questions:**
- What makes ML systems different from traditional software?
- What is the full lifecycle from ideation to retirement?
- How do we define success for a production ML system?

**Representative notes:** `ml_system_design.md`, `ml_lifecycle.md`

**Tools:** None specific — conceptual

---

## Stage 01: Data Engineering

**Purpose:** Build reliable pipelines that deliver data to the ML system.

**Key questions:**
- How do we ingest data from heterogeneous sources?
- Batch vs. stream processing — when to use each?
- How do we build and serve features at scale?

**Representative notes:** `data_pipeline_patterns.md`, `feature_store.md`

**Tools:** Apache Airflow, Prefect, dbt, Apache Spark, Apache Kafka, Feast, Tecton

---

## Stage 02: Training Data

**Purpose:** Curate high-quality labeled data for model training.

**Key questions:**
- How do we label data efficiently and accurately?
- How do we handle class imbalance?
- How do we version datasets and track lineage?

**Representative notes:** `data_labeling.md`, `class_imbalance_and_augmentation.md`, `dataset_versioning.md`

**Tools:** Label Studio, Scale AI, DVC, Delta Lake, nemo-curator, ray-data

---

## Stage 03: Feature Engineering

**Purpose:** Transform raw data into predictive representations without leakage.

**Key questions:**
- Which encodings suit which feature types?
- How do we prevent data leakage in time-series and CV settings?
- How do we manage feature reuse across models?

**Representative notes:** `feature_engineering_patterns.md`, `data_leakage.md`

**Tools:** pandas, scikit-learn, Feast, Tecton, Featureform

---

## Stage 04: Model Development

**Purpose:** Train, track, and offline-evaluate models systematically.

**Key questions:**
- How do we ensure reproducible experiments?
- How do we scale training to multiple GPUs/nodes?
- How do we evaluate offline before deployment?

**Representative notes:** `experiment_tracking.md`, `distributed_training.md`, `offline_evaluation.md`

**Tools:** MLflow, W&B, TensorBoard, PyTorch, DeepSpeed, Accelerate, FSDP

---

## Stage 05: Deployment & Serving

**Purpose:** Serve model predictions reliably at scale with acceptable latency.

**Key questions:**
- Batch vs. online vs. edge inference — when to use each?
- How do we reduce model size for deployment?
- How do we safely roll out new models?

**Representative notes:** `serving_patterns.md`, `model_compression.md`, `rollout_strategies.md`

**Tools:** FastAPI, vLLM, TorchServe, Triton Inference Server, ONNX, BentoML, Seldon

---

## Stage 06: Monitoring & Observability

**Purpose:** Detect when model behavior degrades in production.

**Key questions:**
- How do we detect data drift and concept drift?
- Which operational metrics signal a problem?
- How do we alert efficiently without alert fatigue?

**Representative notes:** `drift_detection.md`, `ml_observability.md`

**Tools:** Evidently AI, Arize Phoenix, WhyLabs, Grafana, Prometheus, W&B

---

## Stage 07: Continual Learning

**Purpose:** Keep models fresh as the world changes.

**Key questions:**
- When should we retrain (triggers vs. schedules)?
- How do we test model updates safely in production?
- What does a data flywheel look like?

**Representative notes:** `retraining_strategies.md`, `testing_in_production.md`

**Tools:** MLflow, W&B, Airflow, A/B testing frameworks, multi-armed bandits

---

## Stage 08: Infrastructure & Platform

**Purpose:** Provide the compute and tooling substrate that all other stages depend on.

**Key questions:**
- How do we orchestrate ML workloads at scale?
- What does an internal ML platform provide?
- How do we manage environments and dependencies?

**Representative notes:** `ml_platform_architecture.md`, `ml_environment_management.md`

**Tools:** Kubernetes, Kubeflow, MLflow, SageMaker, Vertex AI, Modal, SkyPilot, conda, uv

---

## Cross-Stage Dependencies

```
01_data_engineering
    └── feeds → 02_training_data
                    └── feeds → 03_feature_engineering
                                    └── feeds → 04_model_development
                                                    └── feeds → 05_deployment_and_serving
                                                                    └── feeds → 06_monitoring
                                                                                    └── triggers → 07_continual_learning
                                                                                                        └── loops back to 02
08_infrastructure_and_platform ← enables all stages
```
