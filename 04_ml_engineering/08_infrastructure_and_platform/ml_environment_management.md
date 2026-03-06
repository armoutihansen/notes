---
layer: 04_ml_engineering
type: engineering
tool: docker
status: growing
tags: [docker, reproducibility, environment-management, cuda, containers]
created: 2026-03-05
---

# ML Environment Management

## Purpose

Reproducibility in ML has four dimensions: **code** (which version ran), **data** (which dataset version was used), **environment** (which libraries at which versions), and **randomness** (which seeds were set). Of these, environment is the most operationally treacherous — a silent `pip install --upgrade` can change model outputs and invalidate benchmarks. Environment management is the engineering discipline that makes ML experiments deterministic and deployable.

## Architecture

### Reproducibility Challenges

- **Transitive dependency conflicts:** `torch==2.1.0` pulls `triton==2.1.0` which may conflict with other packages. `pip` resolves greedily; `poetry` and `conda` use constraint solvers.
- **CUDA/cuDNN version coupling:** PyTorch and TensorFlow are compiled against specific CUDA versions. A system CUDA upgrade silently breaks GPU training.
- **Non-determinism in GPU operations:** CUDA operations like `atomicAdd` are non-deterministic by default. Must set `torch.use_deterministic_algorithms(True)` and `CUBLAS_WORKSPACE_CONFIG=:4096:8` (costs ~10% throughput).
- **OS-level differences:** glibc version, OpenBLAS, MKL variant affect numerical results. Docker solves this by pinning the entire OS image.

### Docker for ML

**Base image selection:**

```dockerfile
# Training — CUDA-enabled base
FROM nvidia/cuda:12.1.0-cudnn8-runtime-ubuntu22.04

# Inference — lean base (no CUDA dev tools)
FROM python:3.11-slim
```

**Multi-stage build for inference:** Build stage installs all compile-time dependencies; runtime stage copies only the final artifacts, reducing image size from ~8 GB to ~2 GB:

```dockerfile
FROM python:3.11 AS builder
RUN pip install --user --no-cache-dir -r requirements.txt

FROM python:3.11-slim
COPY --from=builder /root/.local /root/.local
COPY app/ ./app/
```

**GPU support (nvidia-docker):** Requires NVIDIA Container Toolkit on the host. Runtime flag: `docker run --gpus all ...`. In Kubernetes: `nvidia.com/gpu: 1` resource request. CUDA version in the image must be ≤ host driver version.

## Implementation Notes

### Dependency Management Tools

| Tool | Lock file | Solver | Best for |
|---|---|---|---|
| `pip` + `requirements.txt` | Manual pin | Greedy | Simple projects; widest compatibility |
| `conda` + `conda.yaml` | `conda-lock.yml` | SAT solver | Data science; handles non-Python deps (CUDA, MKL) |
| `poetry` + `pyproject.toml` | `poetry.lock` | PubGrub solver | Application development; strict reproducibility |
| `uv` | `uv.lock` | Rust-based | Drop-in pip replacement; 10–100× faster resolution |

**Recommendation:** Use `conda` for training environments (handles CUDA/MKL cleanly); use `poetry` or `uv` for inference services (lean, reproducible deployments).

### Environment Pinning Strategy

1. **Develop** in a named conda environment with loose version constraints (`torch>=2.0`).
2. **Lock** with `conda env export --no-builds > conda.yaml` (drop `--no-builds` for exact binary pinning).
3. **Package** by building a Docker image from the lockfile — the image digest becomes the canonical environment identifier.
4. **Register** the image digest in the model registry alongside model weights: `image: registry.company.com/ml/train:sha256-abc123`.

### Dev/Test/Prod Parity

The same Docker image should run in all three environments, parameterized only by config (e.g., data paths, batch sizes). The classic failure mode is: local dev uses conda + GPU, CI uses CPU-only pip install, prod uses a different CUDA image — each diverges silently. Use `docker compose` locally to mirror the production Kubernetes pod spec.

### Containerized Training Jobs

- **Kubernetes:** Define a `Job` spec with `resources.limits.nvidia.com/gpu: 1`. Use a `PersistentVolumeClaim` for data; write checkpoints to object storage (S3/GCS), not the container filesystem.
- **SageMaker Training Jobs:** Bring-your-own-container (BYOC) pattern: push Docker image to ECR, reference in `Estimator(image_uri=...)`. SageMaker handles instance provisioning, data mounting from S3, and log shipping to CloudWatch.

### Reproducibility Checklist

```
□ Random seeds set: Python random, NumPy, PyTorch/TensorFlow, CUDA
□ torch.use_deterministic_algorithms(True) (or equivalent)
□ Data version recorded (DVC tag, S3 version ID, dataset registry entry)
□ Code version recorded (git commit SHA)
□ Environment version recorded (Docker image digest or conda-lock.yml hash)
□ Hyperparameters logged to experiment tracker
□ Hardware noted (GPU model, CUDA version) — some ops differ across GPU generations
```

## Trade-offs

- **Conda vs. Docker:** Conda solves Python-level env isolation well but not OS-level. Docker solves everything but adds overhead (image build time, registry management, container startup latency for short jobs).
- **Exact pinning vs. flexibility:** Exact pins (`torch==2.1.2`) maximize reproducibility but accumulate security debt; range pins (`torch>=2.0,<3.0`) allow security updates but risk breakage.
- **Determinism vs. performance:** `torch.use_deterministic_algorithms(True)` disables some CUDA kernels (e.g., certain scatter/gather ops), causing ~5–15% throughput loss on some workloads. Not always worth the cost in production training.
- **Monolithic training image vs. per-project:** A single large base image (maintained by a platform team) reduces build times via layer caching but creates a shared dependency management problem. Per-project images are more isolated but slower to build.

## References

- NVIDIA Container Toolkit: https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/
- PyTorch reproducibility guide: https://pytorch.org/docs/stable/notes/randomness.html
- conda-lock: https://github.com/conda/conda-lock
- Huyen, *Designing Machine Learning Systems* (O'Reilly, 2022), Chapter 6

## Links
- [[ml_platform_architecture|ML Platform Architecture]]
- [[feature_store|Feature Store]]
- [[data_pipeline_patterns|Data Pipeline Patterns]]
- [[experiment_tracking|Experiment Tracking]]
- [[03_software_engineering/06_devops_and_infrastructure/docker_patterns|Docker Patterns]]
- [[03_software_engineering/06_devops_and_infrastructure/kubernetes_basics|Kubernetes Basics]]
- [[03_software_engineering/06_devops_and_infrastructure/cicd_pipelines|CI/CD Pipelines]]
