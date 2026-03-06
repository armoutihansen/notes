---
layer: 06_applications
type: application
status: growing
tags: [pattern, deployment, mlops]
created: 2026-03-10
---

# Docker for ML Pipelines

## Purpose

Implements reproducible ML training and inference containers using Docker. Covers ML-specific patterns: CUDA base images, multi-stage builds for lean inference images, GPU runtime configuration, and ML environment reproducibility. This pattern closes the gap between research environment and production deployment.

### Examples

**Training container**: Reproducible GPU-enabled training environment that pins CUDA, cuDNN, PyTorch, and all transitive dependencies; runs identically on a workstation and a cloud VM.

**Inference container**: Lean multi-stage image that ships only inference dependencies (no CUDA dev tools, no training libraries), reducing image size from ~8 GB to ~1.5 GB.

## Architecture

### Base Image Selection

```dockerfile
# Training — CUDA 12.1 with cuDNN 8 (matches PyTorch 2.1 compiled binaries)
FROM nvidia/cuda:12.1.0-cudnn8-runtime-ubuntu22.04

# Inference — minimal Python base (CPU inference or if model is loaded as ONNX)
FROM python:3.11-slim

# HuggingFace deep learning image (includes PyTorch, transformers, CUDA)
FROM huggingface/transformers-pytorch-gpu:latest
```

### Training Image

```dockerfile
# Dockerfile.train
FROM nvidia/cuda:12.1.0-cudnn8-runtime-ubuntu22.04

# System deps
RUN apt-get update && apt-get install -y \
    python3.11 python3.11-dev python3-pip git \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /workspace

# Pin Python dependencies (use lock file for exact reproducibility)
COPY requirements-train.lock ./requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# Copy training code
COPY src/ ./src/
COPY configs/ ./configs/

ENTRYPOINT ["python", "-m", "src.train"]
```

```
# requirements-train.lock (generated with pip-compile or poetry export)
torch==2.1.2+cu121
torchvision==0.16.2+cu121
accelerate==0.27.0
transformers==4.38.0
mlflow==2.11.0
...
```

### Multi-Stage Inference Image

```dockerfile
# Dockerfile.inference
# Stage 1: install all dependencies (build stage)
FROM python:3.11 AS builder

WORKDIR /build
COPY requirements-inference.txt .
RUN pip install --no-cache-dir --prefix=/install -r requirements-inference.txt

# Stage 2: lean runtime (no build tools, no CUDA dev headers)
FROM python:3.11-slim AS runtime

WORKDIR /app

# Copy only installed packages from builder
COPY --from=builder /install /usr/local

# Copy application code
COPY app/ ./app/

# Non-root user for security
RUN useradd -m appuser && chown -R appuser /app
USER appuser

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8080"]
```

### GPU Runtime

```bash
# Requires: NVIDIA Container Toolkit installed on host
# Install: https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/

# Run with GPU access
docker run --gpus all \
  -v $(pwd)/data:/workspace/data \
  -v $(pwd)/models:/workspace/models \
  -e MLFLOW_TRACKING_URI=http://mlflow:5000 \
  training-image:latest

# Run with specific GPU(s)
docker run --gpus '"device=0,1"' training-image:latest

# Verify CUDA inside container
docker run --gpus all nvidia/cuda:12.1.0-base-ubuntu22.04 nvidia-smi
```

### Docker Compose for Local ML Dev Stack

```yaml
# docker-compose.yml
version: "3.9"

services:
  mlflow:
    image: ghcr.io/mlflow/mlflow:v2.11.0
    ports: ["5000:5000"]
    volumes:
      - mlflow-data:/mlflow
    command: >
      mlflow server
      --backend-store-uri sqlite:////mlflow/mlflow.db
      --default-artifact-root /mlflow/artifacts
      --host 0.0.0.0

  trainer:
    build:
      context: .
      dockerfile: Dockerfile.train
    runtime: nvidia
    volumes:
      - ./data:/workspace/data
      - ./configs:/workspace/configs
    environment:
      - MLFLOW_TRACKING_URI=http://mlflow:5000
    depends_on: [mlflow]
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]

  api:
    build:
      context: .
      dockerfile: Dockerfile.inference
    ports: ["8080:8080"]
    environment:
      - MODEL_URI=models:/my-model@champion
      - MLFLOW_TRACKING_URI=http://mlflow:5000
    depends_on: [mlflow]

volumes:
  mlflow-data:
```

### Environment Reproducibility

```bash
# Deterministic builds: record the exact image digest (not just tag)
docker inspect --format='{{index .RepoDigests 0}}' training-image:latest
# → training-image@sha256:abc123...

# Store digest in model metadata
mlflow.log_param("training_image", "training-image@sha256:abc123...")

# Rebuild from same digest later
docker pull training-image@sha256:abc123...
```

### Non-Determinism Controls

```python
# Inside the container: ensure deterministic GPU ops
import os, torch, random, numpy as np

SEED = 42
random.seed(SEED)
np.random.seed(SEED)
torch.manual_seed(SEED)
torch.cuda.manual_seed_all(SEED)
torch.use_deterministic_algorithms(True)
os.environ["CUBLAS_WORKSPACE_CONFIG"] = ":4096:8"
```

## Links
- [[04_ml_engineering/08_infrastructure_and_platform/ml_environment_management|ML Environment Management]]
- [[03_software_engineering/06_devops_and_infrastructure/docker_patterns|Docker Patterns]]
- [[03_software_engineering/06_devops_and_infrastructure/kubernetes_basics|Kubernetes Basics]]
- [[model_serving_with_fastapi|Model Serving with FastAPI]]
- [[cicd_for_ml|CI/CD for ML Pipelines]]
