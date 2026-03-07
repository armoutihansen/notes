---
layer: 08_implementations
type: application
status: growing
tags: [pattern, deployment, mlops]
created: 2026-03-10
---

# Model Serving with FastAPI

## Purpose

Implements a production-ready ML model inference API using FastAPI. Covers model loading, request/response schemas with Pydantic, async serving, health checks, batching, and Docker containerization. This pattern applies to any scikit-learn, PyTorch, XGBoost, or ONNX model that needs a REST or gRPC-compatible HTTP/2 interface.

### Examples

**Online fraud detection**: Expose a gradient boosting model as a `POST /predict` endpoint; return probability scores in < 50 ms p99 latency with input validation.

**Batch scoring API**: Accept arrays of records, vectorize inference, return scores in bulk — 10–100× more efficient than one-request-per-record.

## Architecture

### Project Structure

```
serving/
├── app/
│   ├── main.py           # FastAPI app, lifespan hooks, routers
│   ├── model.py          # model loading + inference logic
│   ├── schemas.py        # Pydantic request/response models
│   └── config.py         # settings (model path, thresholds)
├── Dockerfile
└── requirements.txt
```

### Model Loading with Lifespan

```python
# app/model.py
import mlflow.pyfunc
from functools import lru_cache

@lru_cache(maxsize=1)
def load_model(model_uri: str):
    """Load model once and cache in memory."""
    return mlflow.pyfunc.load_model(model_uri)
```

```python
# app/main.py
from contextlib import asynccontextmanager
from fastapi import FastAPI
from .model import load_model
from .config import settings

ml_models = {}

@asynccontextmanager
async def lifespan(app: FastAPI):
    # Load model on startup
    ml_models["fraud_detector"] = load_model(settings.model_uri)
    yield
    # Clean up on shutdown
    ml_models.clear()

app = FastAPI(title="Fraud Detection API", lifespan=lifespan)
```

### Request and Response Schemas

```python
# app/schemas.py
from pydantic import BaseModel, Field
from typing import Literal

class PredictRequest(BaseModel):
    transaction_amount: float = Field(gt=0, description="Amount in USD")
    merchant_category: str
    card_present: bool
    hour_of_day: int = Field(ge=0, le=23)
    days_since_last_transaction: float = Field(ge=0)

class PredictResponse(BaseModel):
    fraud_probability: float = Field(ge=0.0, le=1.0)
    decision: Literal["approve", "review", "decline"]
    model_version: str

class BatchPredictRequest(BaseModel):
    records: list[PredictRequest] = Field(max_length=1000)

class BatchPredictResponse(BaseModel):
    results: list[PredictResponse]
    latency_ms: float
```

### Inference Endpoints

```python
# app/main.py (continued)
import time
import pandas as pd
from fastapi import HTTPException
from .schemas import PredictRequest, PredictResponse, BatchPredictRequest, BatchPredictResponse
from .config import settings

@app.post("/predict", response_model=PredictResponse)
async def predict(request: PredictRequest):
    model = ml_models.get("fraud_detector")
    if model is None:
        raise HTTPException(status_code=503, detail="Model not loaded")

    features = pd.DataFrame([request.model_dump()])
    prob = float(model.predict(features)[0])

    if prob < settings.approve_threshold:
        decision = "approve"
    elif prob < settings.review_threshold:
        decision = "review"
    else:
        decision = "decline"

    return PredictResponse(
        fraud_probability=prob,
        decision=decision,
        model_version=settings.model_version,
    )

@app.post("/predict/batch", response_model=BatchPredictResponse)
async def batch_predict(request: BatchPredictRequest):
    model = ml_models["fraud_detector"]
    t0 = time.perf_counter()

    features = pd.DataFrame([r.model_dump() for r in request.records])
    probs = model.predict(features).tolist()

    results = []
    for prob in probs:
        if prob < settings.approve_threshold:
            decision = "approve"
        elif prob < settings.review_threshold:
            decision = "review"
        else:
            decision = "decline"
        results.append(PredictResponse(
            fraud_probability=prob,
            decision=decision,
            model_version=settings.model_version,
        ))

    return BatchPredictResponse(
        results=results,
        latency_ms=(time.perf_counter() - t0) * 1000,
    )
```

### Health and Readiness Endpoints

```python
from fastapi.responses import JSONResponse

@app.get("/health")
async def health():
    """Liveness probe — server is running."""
    return {"status": "ok"}

@app.get("/ready")
async def readiness():
    """Readiness probe — model is loaded."""
    if "fraud_detector" not in ml_models:
        return JSONResponse(status_code=503, content={"status": "not ready"})
    return {"status": "ready", "model_version": settings.model_version}
```

### Dockerfile

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app/ ./app/

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8080", "--workers", "4"]
```

```bash
# Build and run
docker build -t fraud-api:latest .
docker run -p 8080:8080 \
  -e MODEL_URI="models:/fraud-detector@champion" \
  -e MLFLOW_TRACKING_URI="http://mlflow:5000" \
  fraud-api:latest

# Test
curl -X POST http://localhost:8080/predict \
  -H "Content-Type: application/json" \
  -d '{"transaction_amount": 250.0, "merchant_category": "online", "card_present": false, "hour_of_day": 2, "days_since_last_transaction": 1.5}'
```

### Production Scaling

- **Horizontal scaling**: Deploy multiple replicas behind a load balancer (Kubernetes `Deployment` + `Service`).
- **CPU/GPU affinity**: For GPU models, request `nvidia.com/gpu: 1` per pod; for CPU-only models, request `requests.cpu: "2"` and `limits.cpu: "4"`.
- **Latency SLA**: Set Kubernetes readiness probe to the `/ready` endpoint; configure HPA on `requests per second` or custom metric.

## Links
- [[05_ml_engineering/06_deployment_and_serving/serving_patterns|Model Serving Patterns]]
- [[05_ml_engineering/06_deployment_and_serving/rollout_strategies|Model Rollout Strategies]]
- [[04_software_engineering/04_apis_and_services/fastapi_patterns|FastAPI Patterns]]
- [[mlflow_experiment_tracking|MLflow Experiment Tracking Pattern]]
- [[docker_ml_pipeline|Docker for ML Pipelines]]
- [[batch_ml_prediction_pipeline|Batch ML Prediction Pipeline]]
