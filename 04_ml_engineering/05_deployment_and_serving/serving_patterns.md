---
layer: 04_ml_engineering
type: engineering
tool: general
status: growing
tags: [serving, deployment, batch, streaming, edge, inference]
created: 2026-03-05
---

# Model Serving Patterns

## Purpose

Defines how a trained model is exposed to consumers at inference time. The right serving pattern is determined by latency requirements, update frequency, throughput, infrastructure constraints, and the nature of the use case. Choosing poorly leads to either over-engineered pipelines or production systems that cannot meet SLAs.

## Architecture

### Batch Prediction (Offline Scoring)
Predictions are generated on a schedule rather than on demand. Input data is read in bulk, scored, and results are written to a storage sink (database, data warehouse, file store). Common runtime: Apache Spark, BigQuery ML, AWS Batch, or a simple cron-triggered container.

- **Latency**: Minutes to hours (acceptable for non-real-time use cases).
- **Throughput**: Very high — optimized for bulk I/O, vectorised operations, partitioned execution.
- **Use cases**: Churn propensity scores refreshed nightly, recommendation pre-computation, risk scoring for loan portfolios.
- **Key trade-off**: Predictions are stale by definition. If the world changes between scoring runs, predictions may degrade silently.

### Online Prediction (Real-Time REST/gRPC API)
A model is wrapped in a stateless inference server (TensorFlow Serving, TorchServe, Triton Inference Server, FastAPI + model loader) and exposed via HTTP or gRPC. Requests arrive synchronously, predictions are returned in the same connection.

- **Latency target**: Typically p99 < 100 ms for user-facing systems; < 500 ms for internal APIs.
- **Infrastructure**: Auto-scaling container fleet behind a load balancer. Horizontal scaling decouples compute from request volume.
- **Stateless requirement**: Each request must carry all required context (feature values or raw inputs). No session state in the server.
- **Use cases**: Fraud detection at payment time, search ranking, content moderation.

### Streaming Prediction (Kafka + Flink Pipelines)
Events flow through a message broker (Apache Kafka, Pulsar) into a stream processing engine (Apache Flink, Spark Structured Streaming) where feature computation and model inference occur in a continuous, low-latency pipeline. Results are published to downstream topics or sinks.

- **Latency**: Sub-second to a few seconds, bounded by stream processing window and broker lag.
- **Use cases**: Real-time anomaly detection on clickstreams, live session quality scoring, dynamic pricing.
- **Complexity**: Requires managing Kafka consumer groups, exactly-once semantics, and stateful window aggregations.

### Edge Inference (Mobile/Browser)
The model runs directly on the end-user device or at a network edge node, eliminating round-trip latency and enabling offline operation.

- **Runtimes**: ONNX Runtime (cross-platform C++/Python/WASM), TensorFlow Lite (Android/iOS), Core ML (Apple), MediaPipe.
- **Model preparation**: Requires export, quantisation (INT8 typical), and often architecture downsizing (MobileNet, EfficientNet-Lite, DistilBERT).
- **Use cases**: On-device speech recognition, camera ML (object detection in AR apps), privacy-preserving inference.

## Implementation Notes

### Unifying Batch and Streaming
**Lambda architecture**: Maintains a batch layer (high-throughput, accurate, slow) and a speed layer (low-latency, approximate). A serving layer merges outputs. Operationally costly — two separate codebases for the same logic.

**Kappa architecture**: A single streaming pipeline handles both real-time and historical reprocessing. Simplifies maintenance. Reprocessing is achieved by replaying events from the beginning of the log with an updated consumer. Preferred when stream processing is mature and throughput is manageable.

### Choosing Between Patterns

| Factor | Batch | Online | Streaming | Edge |
|---|---|---|---|---|
| Latency budget | Hours | < 1s | < 5s | < 50 ms |
| Update frequency | Daily / hourly | Per request | Per event | Infrequent |
| Infrastructure cost | Low (idle) | Medium | High (always-on) | None (device) |
| Feature freshness | Stale | Fresh | Near-real-time | Static |

## Trade-offs

- **Batch** simplicity comes at the cost of prediction staleness; acceptable for slowly changing signals.
- **Online** serving requires disciplined feature pipelines to avoid training–serving skew; stateless design limits contextual inference.
- **Streaming** brings freshness but significantly increases operational complexity and dependency on stream processing expertise.
- **Edge** removes latency and privacy concerns but constrains model size, updates require app releases, and debugging is harder.

## References

- Kleppmann, *Designing Data-Intensive Applications*, Ch. 11 (stream processing).
- Google Cloud, *Vertex AI Serving Patterns* documentation.
- NVIDIA Triton Inference Server documentation.
- Kreps, J., "Questioning the Lambda Architecture" (Kappa origin article).

## Links
- [[model_compression|Model Compression for Deployment]]
- [[rollout_strategies|Model Rollout Strategies]]
- [[drift_detection|Drift Detection]]
- [[ml_observability|ML Observability]]
- [[feature_store|Feature Store]]
- [[experiment_tracking|Experiment Tracking]]
