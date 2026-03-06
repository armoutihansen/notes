---
layer: 03_software_engineering
type: engineering
tool: Docker
status: growing
tags: [docker, containers, devops, infrastructure, ml-ops]
created: 2026-03-05
---

# Docker Patterns

## Purpose

Docker provides reproducible, isolated runtime environments packaged as images. The goal is to eliminate "works on my machine" problems, standardise deployment units, and decouple application lifecycle from host infrastructure. In ML contexts Docker is also the boundary between a researcher's experiment and a production-grade inference service.

## Architecture

### Image Layering Model

Docker images are ordered stacks of read-only layers. Each instruction in a Dockerfile (`RUN`, `COPY`, `ADD`) creates a new layer identified by a content hash. At runtime a thin writable layer is added on top. Because layers are content-addressed and shared across images on a host, the cache is the primary lever for build speed.

```
Image = base layer → dependency layer → code layer → config layer
Container = Image + writable layer (ephemeral)
```

### Docker Compose Service Graph

```
          ┌─────────────────────────────────┐
          │        Docker Compose           │
          │                                 │
          │  ┌──────────┐  ┌─────────────┐  │
          │  │  app     │  │  worker     │  │
          │  │ (web)    │─▶│ (celery)    │  │
          │  └────┬─────┘  └──────┬──────┘  │
          │       │               │         │
          │  ┌────▼───────────────▼──────┐  │
          │  │         redis             │  │
          │  └───────────────────────────┘  │
          └─────────────────────────────────┘
```

## Implementation Notes

### Dockerfile Best Practices

**Layer Caching**

Order instructions from least-changing to most-changing. Dependency installation almost never changes between code iterations; source code changes every commit.

```dockerfile
# Good — cache dependencies before copying source
FROM python:3.12-slim

WORKDIR /app

# Copy only requirement files first
COPY requirements.txt pyproject.toml ./
RUN pip install --no-cache-dir -r requirements.txt

# Source code copied last — cache busted only on code changes
COPY src/ ./src/
```

**Multi-Stage Builds**

Separate build-time tooling from the runtime image. The final stage ships nothing that was only needed to compile or install.

```dockerfile
# Stage 1: build / compile dependencies
FROM python:3.12 AS builder
WORKDIR /build
COPY requirements.txt .
RUN pip install --prefix=/install --no-cache-dir -r requirements.txt

# Stage 2: minimal runtime
FROM python:3.12-slim AS runtime
COPY --from=builder /install /usr/local
COPY src/ /app/src/
WORKDIR /app
CMD ["python", "-m", "src.server"]
```

For compiled languages (Go, Rust) the pattern is even more dramatic: the final stage can be `FROM scratch` or `FROM gcr.io/distroless/static`.

**Minimal Base Images**

| Base | Typical size | Notes |
|------|-------------|-------|
| `ubuntu:22.04` | ~80 MB | Full OS, easiest debugging |
| `python:3.12-slim` | ~45 MB | Debian slim, no dev tools |
| `python:3.12-alpine` | ~20 MB | musl libc — beware binary wheel incompatibilities |
| `gcr.io/distroless/python3` | ~15 MB | No shell, minimises attack surface |

Distroless images have no package manager, no shell, no debug tools — they are the hardest to debug but the most secure option for production inference services.

**`.dockerignore`**

Prevents the build context from bloating the image and leaking secrets:

```
.git/
.env
.env.*
__pycache__/
*.pyc
*.pyo
.pytest_cache/
.mypy_cache/
data/
models/
notebooks/
*.ipynb
.DS_Store
```

### Image Tagging Strategy

Avoid `latest` in production — it is a moving target that breaks reproducibility.

```
# Immutable: content hash
myapp@sha256:abc123...

# Semver: human-readable, still reproducible
myapp:1.4.2

# Git SHA: ties image to exact commit
myapp:git-a3f8c12

# Environment aliases (mutable pointers, avoid in K8s manifests)
myapp:staging
myapp:production
```

In a CI pipeline: build with git SHA tag, promote to semver on release, update environment alias only after smoke tests pass.

### Docker Compose

```yaml
version: "3.9"

services:
  api:
    build:
      context: .
      target: runtime          # multi-stage target
    image: myapp:${GIT_SHA:-dev}
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/mydb
    env_file:
      - .env.local             # secrets not in image
    depends_on:
      db:
        condition: service_healthy
    networks:
      - backend
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 10s

  db:
    image: postgres:16-alpine
    volumes:
      - pg_data:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: pass
      POSTGRES_USER: user
      POSTGRES_DB: mydb
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  pg_data:

networks:
  backend:
    driver: bridge
```

### ML-Specific Patterns

**GPU Access — NVIDIA Container Toolkit**

Install the [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html) on the host, then:

```yaml
# docker-compose.yml
services:
  trainer:
    image: pytorch/pytorch:2.2.0-cuda12.1-cudnn8-runtime
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all         # or specific count: 2
              capabilities: [gpu]
```

```bash
# CLI equivalent
docker run --gpus all pytorch/pytorch:2.2.0-cuda12.1-cudnn8-runtime python train.py

# Specific GPU by index
docker run --gpus '"device=0,1"' ...
```

**Mounting Datasets and Model Weights**

```yaml
services:
  trainer:
    volumes:
      - /mnt/data/datasets:/datasets:ro    # read-only dataset mount
      - /mnt/models:/models                # writable checkpoint output
      - ./configs:/app/configs:ro
```

Keep large data outside the image — never `COPY` datasets into an image layer.

**Jupyter in Docker**

```dockerfile
FROM jupyter/pytorch-notebook:latest
# or build from pytorch base:
FROM pytorch/pytorch:2.2.0-cuda12.1-cudnn8-devel
RUN pip install jupyterlab ipywidgets
EXPOSE 8888
CMD ["jupyter", "lab", "--ip=0.0.0.0", "--no-browser", "--allow-root"]
```

```bash
docker run --gpus all -p 8888:8888 -v $(pwd):/workspace my-jupyter
```

### Security Hardening

**Non-Root User**

```dockerfile
# Create a non-root user and group
RUN groupadd --gid 1001 appgroup \
 && useradd --uid 1001 --gid appgroup --no-create-home appuser

# Change ownership of application files
COPY --chown=appuser:appgroup src/ /app/src/

# Drop to non-root before CMD
USER appuser
```

**Read-Only Filesystem**

```bash
docker run --read-only --tmpfs /tmp --tmpfs /run myapp
```

In Compose:
```yaml
services:
  api:
    read_only: true
    tmpfs:
      - /tmp
      - /run
```

**Secrets Management**

Never bake secrets into images. Options in order of preference:
1. **Docker Secrets** (Swarm / Compose secrets) — mounted at `/run/secrets/<name>`
2. **Environment variables from `.env`** — never commit `.env` to git
3. **Vault / AWS Secrets Manager** — inject at container start via entrypoint script
4. **BuildKit `--secret` flag** — for build-time secrets (e.g. pip private registry tokens)

```dockerfile
# Build-time secret — NOT stored in any layer
RUN --mount=type=secret,id=pip_token \
    PIP_INDEX_URL=$(cat /run/secrets/pip_token) \
    pip install --no-cache-dir private-package
```

## Trade-offs

| Approach | Pro | Con |
|----------|-----|-----|
| Alpine base | Smallest image | musl libc breaks many Python wheels |
| Distroless | Most secure | No shell — hard to debug |
| Multi-stage | Lean runtime | More complex Dockerfile |
| Bind mounts for data | Fast I/O, no copy | Host path coupling |
| COPY data into image | Fully reproducible | Huge image, slow push/pull |
| `--gpus all` | Simple | No isolation between containers sharing GPU memory |

## References

- [Docker Best Practices (official)](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/)
- [Distroless images — GoogleContainerTools](https://github.com/GoogleContainerTools/distroless)
- [Docker Compose spec](https://compose-spec.io/)
- [BuildKit secrets](https://docs.docker.com/build/buildkit/secrets/)

## Links
- [[kubernetes_basics|Kubernetes]]
- [[cicd_pipelines|CI/CD]]
- [[04_ml_engineering/08_infrastructure_and_platform/ml_environment_management|ML Environment Management]]
