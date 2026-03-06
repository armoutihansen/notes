---
layer: 04_ml_engineering
type: engineering
tool: mlflow
status: growing
tags: [experiment-tracking, mlflow, reproducibility, model-registry, dvc]
created: 2026-03-05
---

# Experiment Tracking

## Purpose

Experiment tracking is the systematic recording of all factors that define a machine learning experiment — hyperparameters, metrics, artifacts, code, data, and environment — so that results are reproducible, comparable, and auditable. Without tracking, the history of model development exists only in researchers' memories and undocumented notebooks, making it impossible to reliably reproduce a result, understand why one configuration outperformed another, or safely promote a model to production. Tracking also enables teams to collaborate without overwriting each other's results and supports regulatory requirements for model documentation.

## Architecture

### MLflow Components

MLflow is the most widely adopted open-source experiment tracking framework. It has four major components:

**1. MLflow Tracking**: the core logging API. Each run is associated with an experiment and records:
- **Parameters**: hyperparameters passed to the run (learning rate, batch size, model architecture).
- **Metrics**: scalar values logged over time (train loss, validation AUC per epoch).
- **Artifacts**: arbitrary files — model weights, plots, confusion matrices, serialized feature pipelines, SHAP explanations.
- **Tags**: free-form metadata (run name, team, ticket ID).

```python
import mlflow
with mlflow.start_run():
    mlflow.log_param("learning_rate", 1e-3)
    mlflow.log_metric("val_auc", 0.87)
    mlflow.sklearn.log_model(model, "model")
```

**2. MLflow Projects**: packages code in a reproducible format using a `MLproject` file that declares entry points, parameters, and the execution environment (conda or Docker). Enables `mlflow run <git_uri>` to reproduce any experiment.

**3. MLflow Models**: a standard format for packaging models with a `MLmodel` manifest that declares supported flavors (sklearn, pytorch, pyfunc). Enables framework-agnostic serving and loading.

**4. MLflow Model Registry**: a centralized model catalog. In MLflow 2.x, models progress through lifecycle stages: `Staging` → `Production` → `Archived`. In **MLflow 3.x**, named **aliases** replace fixed stages (e.g., `client.set_registered_model_alias("my_model", "champion", version=3)`), allowing arbitrary promotion labels and multiple concurrent aliases per model. Prefer aliases for new projects.

```python
client = mlflow.tracking.MlflowClient()
# MLflow 3.x: assign alias
client.set_registered_model_alias("fraud_detector", "champion", version="5")
# Load by alias
model = mlflow.pyfunc.load_model("models:/fraud_detector@champion")
```

Registry integrates with CI/CD pipelines: promoting an alias can trigger automated redeployment.

### What to Track

Comprehensive tracking records enough context to exactly reproduce any run:

| Category | Examples |
|---|---|
| Hyperparameters | learning rate, batch size, regularization coefficients, architecture choices |
| Metrics | train/val loss per epoch, final test metrics, inference latency |
| Artifacts | model weights, preprocessing pipeline, evaluation plots, feature importance |
| Code version | Git commit SHA |
| Data version | dataset hash, DVC data version, feature store snapshot ID |
| Environment | Python version, library versions (requirements.txt / conda.yml), CUDA version |

### Experiment Design

- **Baselines first**: always establish a simple baseline (majority class, linear model, constant predictor) before running complex experiments. The baseline anchors interpretation of subsequent results.
- **Ablation studies**: isolate the contribution of individual components by removing them one at a time. This distinguishes genuine improvements from noise.
- **Random seeds**: fix seeds for all sources of randomness (data shuffling, weight initialization, dropout) and log them. Run key experiments with multiple seeds to estimate variance.

### Data Versioning with DVC

DVC (Data Version Control) complements MLflow by versioning large data files and pipelines in Git-compatible workflows. A `dvc.yaml` defines pipeline stages; `dvc repro` reruns only invalidated stages. DVC stores data remotely (S3, GCS, Azure) and tracks references in `.dvc` files committed to Git, so code and data versions are always linked. Combined with MLflow, the full provenance chain — code commit + data version + hyperparameters → metrics — is captured.

### Weights & Biases (W&B)

W&B is a cloud-hosted alternative to self-hosted MLflow. Key differentiators: richer interactive dashboards, built-in hyperparameter sweep management (`wandb.sweep`), artifact lineage graphs, and team collaboration features. The API is nearly identical to MLflow Tracking. W&B is preferred in research settings; MLflow is more common in enterprise environments with on-premises infrastructure requirements.

## Implementation Notes

- Use `mlflow.autolog()` for scikit-learn, XGBoost, PyTorch Lightning, and HuggingFace Trainer to automatically log parameters, metrics, and models without manual instrumentation.
- Set `MLFLOW_TRACKING_URI` to a shared remote server (backed by PostgreSQL + S3) for team-wide experiment sharing.
- Log the Git SHA explicitly: `mlflow.set_tag("git_commit", subprocess.check_output(["git", "rev-parse", "HEAD"]).decode().strip())`.
- Use experiment naming conventions (`<team>/<project>/<model_type>`) to keep the tracking server organized.

## Trade-offs

Tracking adds logging overhead and requires infrastructure (tracking server, artifact store). The overhead is minimal (milliseconds per log call) compared to training time, but the infrastructure cost is real. Self-hosted MLflow requires operational maintenance; W&B offloads this but introduces a SaaS dependency and potential data privacy concerns. Teams must also guard against tracking theater — logging every possible metric without a clear comparison protocol — which produces noise rather than insight. Structured experiment design (baselines, ablations, fixed seeds) is necessary to extract value from the logged data.

## References

- MLflow documentation: mlflow.org
- DVC documentation: dvc.org
- Weights & Biases documentation: wandb.ai/docs
- Sculley et al., "Hidden Technical Debt in Machine Learning Systems" (NeurIPS 2015)

## Links
- [[distributed_training|Distributed Training]]
- [[offline_evaluation|Offline Evaluation]]
- [[ml_lifecycle|ML Lifecycle]]
- [[data_leakage|Data Leakage]]
- [[ml_platform_architecture|ML Platform Architecture]]
