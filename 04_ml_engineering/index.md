# ML Engineering

## Purpose of This Layer

This layer captures the engineering discipline required to design, build, deploy, and operate production-grade machine learning systems.

It focuses on structured ML systems where:

- You train and own the model
- You manage feature pipelines
- You deploy trained artifacts
- You monitor and retrain models over time

Guiding question:

> How do we build reliable, scalable ML systems in production?

This layer does NOT cover:

- Foundation-model-based system design (→ 05_ai_engineering)
- Model family selection and statistical trade-offs (→ 02_modeling)
- Mathematical derivations (→ 01_foundations)

---
## Structure

## 00_principles_and_lifecycle/

High-level system thinking.

Includes:
- ML systems overview
- Reliability, scalability, maintainability, adaptability
- Iterative development cycles

Focus:
Understanding the end-to-end ML lifecycle before implementation details.

---
### 01_data_engineering/

Data infrastructure for ML systems.

Includes:
- Data sources
- Data formats and data models
- OLTP vs OLAP systems
- ETL / ELT
- Batch vs stream processing
- Dataflow between services

Focus:
Reliable movement and transformation of raw data.

---
### 02_training_data/

Construction of model-ready datasets.

Includes:
- Sampling strategies
- Labeling strategies
- Weak and natural labels
- Class imbalance handling
- Data augmentation
- Dataset versioning and lineage

Focus:
Turning raw data into structured training data.

---
### 03_feature_engineering/

Designing model inputs.

Includes:
- Learned vs engineered features
- Handling missing values
- Scaling and encoding
- Feature crosses and embeddings
- Data leakage
- Feature stores

Focus:
Bridging raw data and model representation.

---
### 04_model_development_and_offline_eval/

Controlled model development.

Includes:
- Baselines and error analysis
- Training and regularization
- Model selection trade-offs
- Evaluation methods and calibration
- Experiment tracking (e.g., MLflow)
- Model versioning and reproducibility
- Distributed training

Focus:
Offline experimentation and model iteration.

---
### 05_deployment_and_serving/

Making models accessible in production.

Includes:
- Batch vs online prediction
- Serving patterns (API, streaming)
- Model optimization and compression
- Edge and browser inference
- Rollout strategies

Focus:
Operationalizing trained models.

---
### 06_monitoring_and_observability/

Operating ML systems safely.

Includes:
- Operational metrics
- ML-specific metrics (predictions, features, inputs)
- Drift detection (covariate, label, concept)
- Logging, tracing, dashboards, alerts
- Incident response playbooks

Focus:
Detecting failures and maintaining reliability.

---
### 07_continual_learning_and_testing_in_prod/

Adapting models over time.

Includes:
- Retraining strategies (stateless vs stateful)
- Data freshness considerations
- Test-in-production
- Shadow, A/B, canary, bandit testing
- Safe rollback and backfills

Focus:
Managing model evolution in dynamic environments.

---
### 08_infrastructure_and_platform/

Platform-level concerns.

Includes:
- Development environment standardization
- Containers
- Schedulers and orchestrators
- Workflow management systems
- ML platform reference architecture
- Model stores
- Build vs buy decisions

Focus:
The infrastructure foundation that enables ML engineering.

---
## Relationship to Other Layers

02_modeling:
Defines what models and evaluation strategies to use.

04_ml_engineering:
Implements, deploys, and operates those models in production.

05_ai_engineering:
Designs systems built around pretrained foundation models.

This layer treats ML models as production artifacts that require lifecycle management, infrastructure, and observability.

---
## Explicit Structure

```
04_ml_engineering/
├── index.md
├── 00_principles_and_lifecycle/
│   ├── ml_systems_overview.md
│   ├── requirements_reliability_scalability_maintainability_adaptability.md
│   └── iterative_development_cycle.md
├── 01_data_engineering/
│   ├── data_sources.md
│   ├── data_formats_and_models.md
│   ├── databases_oltp_vs_olap.md
│   ├── etl_elt.md
│   ├── batch_vs_stream_processing.md
│   └── dataflow_between_services.md
├── 02_training_data/
│   ├── sampling_strategies.md
│   ├── labeling_strategies.md
│   ├── weak_natural_labels.md
│   ├── class_imbalance.md
│   ├── data_augmentation.md
│   └── dataset_versioning_and_lineage.md
├── 03_feature_engineering/
│   ├── learned_vs_engineered_features.md
│   ├── missing_values_scaling_encoding.md
│   ├── feature_crosses_embeddings.md
│   ├── data_leakage.md
│   └── feature_stores.md
├── 04_model_development_and_offline_eval/
│   ├── baselines_and_error_analysis.md
│   ├── training_and_regularization.md
│   ├── model_selection_tradeoffs.md
│   ├── evaluation_methods_metrics_calibration.md
│   ├── experiment_tracking_mlflow.md
│   ├── model_versioning_reproducibility.md
│   └── distributed_training.md
├── 05_deployment_and_serving/
│   ├── batch_vs_online_prediction.md
│   ├── serving_patterns_api_realtime_streaming.md
│   ├── model_optimization_compression.md
│   ├── edge_and_browser_inference.md
│   └── rollout_strategies.md
├── 06_monitoring_and_observability/
│   ├── operational_metrics.md
│   ├── ml_metrics_predictions_features_inputs.md
│   ├── drift_detection_covariate_label_concept.md
│   ├── logging_tracing_dashboards_alerts.md
│   └── incident_response_playbooks.md
├── 07_continual_learning_and_testing_in_prod/
│   ├── retraining_strategies_stateless_vs_stateful.md
│   ├── data_freshness_and_update_frequency.md
│   ├── test_in_production.md
│   ├── shadow_ab_canary_bandits.md
│   └── safe_rollback_and_backfills.md
└── 08_infrastructure_and_platform/
    ├── dev_env_standardization.md
    ├── containers.md
    ├── schedulers_orchestrators.md
    ├── workflow_management.md
    ├── ml_platform_reference_architecture.md
    ├── model_store.md
    └── build_vs_buy.md
```
