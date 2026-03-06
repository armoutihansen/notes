---
layer: 06_applications
type: application
status: growing
tags: [grpc, protobuf, rpc, streaming, service-definition, python]
created: 2026-03-06
---

# gRPC Service Implementation

## Purpose

Practical walkthrough of building a gRPC service in Python: proto schema design, code generation, server implementation, client usage, streaming, and error handling. Synthesized from: [[grpc_and_protobuf|gRPC and Protobuf]].

### Examples

**Step 1 — Define the proto schema (`inference.proto`):**
```protobuf
syntax = "proto3";
package inference;

// Unary RPC
service InferenceService {
  rpc Predict (PredictRequest) returns (PredictResponse);
  rpc BatchPredict (BatchPredictRequest) returns (BatchPredictResponse);
  // Server-streaming: return results as they are computed
  rpc StreamPredict (StreamRequest) returns (stream PredictResponse);
}

message PredictRequest {
  string model_name = 1;
  repeated float features = 2;   // repeated = array
  map<string, string> metadata = 3;
}

message PredictResponse {
  string label = 1;
  float score = 2;
  int64 latency_ms = 3;
}

message BatchPredictRequest {
  repeated PredictRequest requests = 1;
}
message BatchPredictResponse {
  repeated PredictResponse responses = 1;
}
message StreamRequest {
  string model_name = 1;
  int32 n_samples = 2;
}
```

**Step 2 — Generate Python code:**
```bash
pip install grpcio grpcio-tools

python -m grpc_tools.protoc \
  -I. \
  --python_out=. \
  --pyi_out=. \
  --grpc_python_out=. \
  inference.proto
# generates: inference_pb2.py, inference_pb2.pyi, inference_pb2_grpc.py
```

**Step 3 — Implement the server:**
```python
import grpc
from concurrent import futures
import time
import inference_pb2
import inference_pb2_grpc

class InferenceServicer(inference_pb2_grpc.InferenceServiceServicer):

    def __init__(self, model):
        self.model = model

    def Predict(self, request, context):
        try:
            start = time.perf_counter()
            result = self.model.predict([request.features])[0]
            latency = int((time.perf_counter() - start) * 1000)
            return inference_pb2.PredictResponse(
                label=str(result["label"]),
                score=float(result["score"]),
                latency_ms=latency,
            )
        except ValueError as e:
            context.abort(grpc.StatusCode.INVALID_ARGUMENT, str(e))

    def StreamPredict(self, request, context):
        """Server-streaming: yield one response per sample."""
        for i in range(request.n_samples):
            if context.is_active():
                result = self.model.predict_one(i)
                yield inference_pb2.PredictResponse(
                    label=result["label"], score=result["score"]
                )

def serve(model, port: int = 50051):
    server = grpc.server(
        futures.ThreadPoolExecutor(max_workers=10),
        options=[
            ("grpc.max_send_message_length",    50 * 1024 * 1024),
            ("grpc.max_receive_message_length", 50 * 1024 * 1024),
        ],
    )
    inference_pb2_grpc.add_InferenceServiceServicer_to_server(
        InferenceServicer(model), server
    )
    server.add_insecure_port(f"[::]:{port}")
    server.start()
    print(f"gRPC server listening on :{port}")
    server.wait_for_termination()
```

**Step 4 — Implement the client:**
```python
import grpc
import inference_pb2
import inference_pb2_grpc

def get_channel(host: str = "localhost", port: int = 50051) -> grpc.Channel:
    return grpc.insecure_channel(
        f"{host}:{port}",
        options=[("grpc.keepalive_time_ms", 10000)],
    )

def predict(channel, model_name: str, features: list[float]) -> dict:
    stub = inference_pb2_grpc.InferenceServiceStub(channel)
    request = inference_pb2.PredictRequest(
        model_name=model_name,
        features=features,
    )
    try:
        response = stub.Predict(request, timeout=5.0)
        return {"label": response.label, "score": response.score}
    except grpc.RpcError as e:
        raise RuntimeError(f"gRPC error {e.code()}: {e.details()}")

# Context manager for resource cleanup
with get_channel() as channel:
    result = predict(channel, "classifier", [1.0, 2.0, 3.0])
```

**TLS / mTLS (production):**
```python
# Server
server_credentials = grpc.ssl_server_credentials(
    [(open("server.key","rb").read(), open("server.crt","rb").read())]
)
server.add_secure_port("[::]:50051", server_credentials)

# Client
channel_credentials = grpc.ssl_channel_credentials(
    root_certificates=open("ca.crt","rb").read()
)
channel = grpc.secure_channel("myservice:50051", channel_credentials)
```

**Testing gRPC services:**
```python
import pytest
from grpc import experimental

@pytest.fixture
def grpc_channel(model):
    with experimental.channel_ready_future(
        grpc.insecure_channel("localhost:50052")
    ):
        yield
    # Or use grpcio-testing:
    # from grpc_testing import server_from_dictionary, strict_real_time

def test_predict_unary(model):
    servicer = InferenceServicer(model)
    context = MockContext()
    req = inference_pb2.PredictRequest(model_name="cls", features=[1.0, 2.0])
    resp = servicer.Predict(req, context)
    assert resp.score > 0.0
```

## Architecture

```
Client (Python/Go/Rust)
    │
    │  HTTP/2 + binary Protobuf frames
    ▼
gRPC Server (Python ThreadPoolExecutor)
    ├── InferenceServicer.Predict()          ← unary
    ├── InferenceServicer.BatchPredict()     ← unary, batched
    └── InferenceServicer.StreamPredict()   ← server-streaming
```

**When to prefer gRPC over REST:**
- Internal service-to-service communication where latency and throughput matter
- Strongly-typed contracts shared across multiple language clients (generate stubs for Go, Python, Rust from one `.proto`)
- Streaming: server-push, bidirectional streaming
- Large message volumes: Protobuf is ~3–10× smaller than equivalent JSON

**When to prefer REST:**
- Public-facing APIs consumed by browsers (gRPC-Web adds complexity)
- Teams that prioritise human-readable payloads for debugging
- Simple request/response with no streaming requirements

## Links
- [[grpc_and_protobuf|gRPC and Protobuf]]
- [[rest_api_design|REST API Design]]
- [[fastapi_patterns|FastAPI Patterns]]
- [[docker_patterns|Docker Patterns]]
