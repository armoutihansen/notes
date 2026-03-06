---
layer: 06_applications
type: application
status: growing
tags: [fastapi, docker, kubernetes, github-actions, ci-cd, rest-api, testing, production]
created: 2026-03-06
---

# Containerised REST API Service

## Purpose

End-to-end walkthrough of building, testing, containerising, and deploying a production FastAPI service with full CI/CD. Covers the complete lifecycle from app code to running pods. Synthesized from: [[fastapi_patterns|FastAPI Patterns]], [[docker_patterns|Docker Patterns]], [[kubernetes_basics|Kubernetes Basics]], [[cicd_pipelines|CI/CD Pipelines]], [[testing_strategies|Testing Strategies]], [[github_actions|GitHub Actions]].

## Architecture

```
Developer push
    │
    ▼
GitHub → GitHub Actions CI pipeline
    ├── 1. Unit tests (pytest)
    ├── 2. Integration tests (TestClient + SQLite)
    ├── 3. Build Docker image (multi-stage)
    ├── 4. Push to GHCR (tagged with SHA + semver)
    └── 5. Deploy to K8s via helm upgrade

Docker Registry (GHCR)
    │
    ▼
Kubernetes Cluster
    ├── Ingress (nginx) ← HTTPS + TLS cert
    ├── Service (ClusterIP)
    ├── Deployment (3 replicas, rolling update)
    │   └── inference-api containers
    │         ├── /health        readiness probe
    │         └── /predict       application endpoint
    └── HPA (min 2, max 20, 70% CPU)
```

**Component stack:**
| Layer | Technology | Role |
|-------|-----------|------|
| Application | FastAPI + Pydantic v2 | REST API, validation, async |
| Testing | pytest + pytest-asyncio + httpx | Unit, integration, async E2E |
| Containerisation | Docker multi-stage | Reproducible, small images |
| Orchestration | Kubernetes + Helm | Scaling, rolling updates |
| CI/CD | GitHub Actions | Automated build/test/deploy |
| Registry | GitHub Container Registry | Image storage |

### Examples

**FastAPI application (`src/app/main.py`):**
```python
from fastapi import FastAPI, Depends, HTTPException
from pydantic import BaseModel
from contextlib import asynccontextmanager

class PredictRequest(BaseModel):
    features: list[float]
    model_version: str = "latest"

class PredictResponse(BaseModel):
    label: str
    score: float

model_cache: dict = {}

@asynccontextmanager
async def lifespan(app: FastAPI):
    model_cache["clf"] = load_model()   # load once at startup
    yield
    model_cache.clear()

app = FastAPI(title="Inference API", lifespan=lifespan)

@app.get("/health")
def health():
    return {"status": "ok"}

@app.post("/predict", response_model=PredictResponse)
async def predict(req: PredictRequest):
    clf = model_cache.get("clf")
    if clf is None:
        raise HTTPException(503, "Model not loaded")
    result = clf.predict([req.features])[0]
    return PredictResponse(label=str(result["label"]), score=float(result["score"]))
```

**Multi-stage Dockerfile:**
```dockerfile
# Stage 1: builder — installs deps only
FROM python:3.12-slim AS builder
WORKDIR /build
COPY requirements.txt .
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt

# Stage 2: runtime — minimal image
FROM python:3.12-slim AS runtime
WORKDIR /app
COPY --from=builder /install /usr/local
COPY src/ ./src/
ENV PYTHONPATH=/app/src
EXPOSE 8000
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

**GitHub Actions workflow (`.github/workflows/deploy.yml`):**
```yaml
name: CI/CD

on:
  push:
    branches: [main]
    tags: ["v*"]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - run: pip install -r requirements-dev.txt
      - run: pytest --cov=src --cov-fail-under=80

  build-push:
    needs: test
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v4
      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - id: meta
        uses: docker/metadata-action@v5
        with:
          images: ghcr.io/${{ github.repository }}/inference-api
          tags: |
            type=sha
            type=semver,pattern={{version}}
      - uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy:
    needs: build-push
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4
      - run: |
          helm upgrade --install inference-api ./charts/inference-api \
            --namespace prod --create-namespace \
            --set image.tag=${{ github.sha }} \
            --wait
```

**Integration test (`tests/integration/test_api.py`):**
```python
import pytest
from httpx import AsyncClient, ASGITransport
from app.main import app

@pytest.fixture
async def client():
    async with AsyncClient(
        transport=ASGITransport(app=app), base_url="http://test"
    ) as c:
        yield c

@pytest.mark.asyncio
async def test_predict(client):
    resp = await client.post("/predict", json={"features": [1.0, 2.0, 3.0]})
    assert resp.status_code == 200
    body = resp.json()
    assert "label" in body and "score" in body

@pytest.mark.asyncio
async def test_health(client):
    assert (await client.get("/health")).status_code == 200
```

**Step-by-step deployment sequence:**
```bash
# 1. Local dev
uvicorn src.app.main:app --reload

# 2. Build + test image locally
docker build -t inference-api:local .
docker run -p 8000:8000 inference-api:local

# 3. Push to K8s (Helm)
helm install inference-api ./charts/inference-api \
  --namespace staging --create-namespace \
  -f charts/inference-api/values-staging.yaml

# 4. Verify rollout
kubectl rollout status deployment/inference-api -n staging
kubectl logs -l app=inference-api -n staging --tail=50
```

## Links
- [[fastapi_patterns|FastAPI Patterns]]
- [[docker_patterns|Docker Patterns]]
- [[kubernetes_basics|Kubernetes Basics]]
- [[cicd_pipelines|CI/CD Pipelines]]
- [[github_actions|GitHub Actions]]
- [[testing_strategies|Testing Strategies]]
- [[pytest_testing_patterns|pytest Testing Patterns]]
- [[kubernetes_deployment|Kubernetes Deployment Patterns]]
