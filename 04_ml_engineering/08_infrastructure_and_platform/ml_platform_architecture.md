---
layer: 04_ml_engineering
type: engineering
tool: general
status: growing
tags: [ml-platform, mlops, feature-store, model-registry, orchestration]
created: 2026-03-05
---

# ML Platform Architecture

## Purpose

As ML teams scale beyond a handful of models, ad-hoc scripts and manual processes become the primary bottleneck. An ML platform provides shared infrastructure that abstracts the operational complexity of the ML lifecycle — from data ingestion to model serving — so data scientists can focus on modeling rather than plumbing. A mature platform reduces mean time to deploy a model from weeks to days and enables reproducibility, governance, and operational reliability at scale.

## Architecture

### Core Components

| Component | Responsibility | Examples |
|---|---|---|
| **Data store** | Versioned access to raw and processed data | S3 + Delta Lake, BigQuery, Snowflake |
| **Feature store** | Compute, store, and serve features with train/serve parity | Feast, Tecton, Vertex AI Feature Store |
| **Experiment tracking** | Log parameters, metrics, artifacts per run | MLflow Tracking, W&B, Neptune |
| **Training service** | Managed compute for training jobs (GPU/CPU autoscaling) | SageMaker Training, Vertex AI Training, Ray Train |
| **Model registry** | Versioned model artifacts + lifecycle state machine | MLflow Registry, SageMaker Registry |
| **Serving infrastructure** | Real-time and batch inference endpoints | TorchServe, Triton, KServe, SageMaker Endpoints |
| **Monitoring** | Data drift, prediction drift, business metric tracking | Evidently AI, Arize, WhyLogs |

### Orchestration Tools

Pipelines glue components together into reproducible, schedulable workflows:

- **Apache Airflow:** DAG-based scheduler; Python operators; ubiquitous in data engineering; verbose for ML-specific patterns.
- **Prefect:** Python-native; dynamic task graphs; better ergonomics than Airflow; supports hybrid execution.
- **Kubeflow Pipelines:** Kubernetes-native; containerized steps; strong reproducibility; steep learning curve; good for large ML-specific orgs.
- **Metaflow:** Data-scientist-focused; linear step notation; native versioning of data artifacts; developed at Netflix; lowest barrier to adoption for DS teams.
- **ZenML:** Framework-agnostic; stack-based configuration; designed specifically for ML pipelines with built-in artifact lineage.

### Model Registry Lifecycle

A model registry tracks the state machine: `Staging → Production → Archived`. Each transition should be gated by automated evaluation (e.g., shadow test passage, A/B test result) and require human approval for production promotion. MLflow's registry exposes this via UI and REST API; SageMaker Model Registry integrates with CodePipeline for CI/CD gating.

## Implementation Notes

### Build vs. Buy Decision Framework

| Criterion | Managed platform (SageMaker, Vertex AI) | Open-source / custom |
|---|---|---|
| Team size | < 10 ML engineers | > 20 ML engineers |
| Time to value | Weeks | Months |
| Customizability | Limited | Full |
| Cost at scale | High (vendor markup) | Lower (infra cost only) |
| Vendor lock-in | High | Low |

Typical recommendation: start with a managed platform, extract components to open-source alternatives as specific pain points emerge (e.g., replace SageMaker Experiments with MLflow once experiment volume grows).

### Reference Architectures

- **AWS SageMaker:** Studio (IDE) + Pipelines (orchestration) + Feature Store + Model Registry + Endpoints. Tightest integration; highest lock-in.
- **GCP Vertex AI:** Unified Data & AI platform; AutoML and custom training; Vertex Pipelines (Kubeflow-compatible); Vertex Feature Store; Model Monitoring.
- **Azure ML:** Designer (GUI) + Pipelines + Model Registry + Managed Endpoints; strong integration with Azure DevOps.
- **Open-source (Kubeflow + MLflow + Feast):** Full control; requires dedicated platform team; typical stack at mid-to-large tech companies.

### Artifact Management

Every artifact — dataset snapshot, trained model, evaluation report — should be stored with a content-addressable URI (e.g., `s3://ml-artifacts/model/abc123/model.pkl`), linked to its producing pipeline run, and never mutated in place. MLflow and DVC both implement this pattern. Immutable artifacts are a prerequisite for reproducible model re-evaluation and audit.

## Trade-offs

- **Monolithic platform vs. best-of-breed:** A single integrated platform (SageMaker) reduces integration cost but creates lock-in and may lag behind the ecosystem. Best-of-breed tools (MLflow + Feast + Ray) offer flexibility but require significant glue code.
- **Abstraction level:** High-level abstractions (e.g., Metaflow's `@step`) reduce cognitive overhead but can obscure what's happening under the hood, making debugging harder.
- **Centralized vs. federated:** A centralized platform team owning infra creates a bottleneck; federated ownership (each team runs its own stack) leads to duplication. The recommended pattern is a thin platform team owning shared primitives with self-service access.

## References

- Sculley et al., "Hidden Technical Debt in Machine Learning Systems" (NeurIPS 2015)
- Kleppmann, *Designing Data-Intensive Applications*, Chapter 10 (O'Reilly, 2017)
- Huyen, *Designing Machine Learning Systems* (O'Reilly, 2022), Chapter 10
- MLflow documentation: https://mlflow.org/docs/latest/
- Kubeflow documentation: https://www.kubeflow.org/docs/

## Links
- [[ml_environment_management|ML Environment Management]]
- [[retraining_strategies|Retraining Strategies]]
- [[testing_in_production|Testing in Production]]
