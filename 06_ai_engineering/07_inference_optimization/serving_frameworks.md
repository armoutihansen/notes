---
layer: 06_ai_engineering
type: engineering
tool: vllm
status: growing
tags: [deployment]
created: 2026-03-05
---
# LLM Serving Frameworks

## Purpose
High-throughput, low-latency serving of LLM inference for production systems. The challenge is fundamentally different from training: requests arrive continuously at variable rates, sequence lengths vary widely, and the KV cache creates memory pressure that scales with both request count and length. Naive serving (one request at a time, preallocate maximum KV cache) wastes GPU utilization and fails at production load. Modern serving frameworks solve this with continuous batching, virtual KV cache management, and hardware-aware kernel dispatch.

## Architecture
**Key concepts:**

*Continuous batching* (Orca, Yu et al. 2022): Process tokens from multiple requests simultaneously without waiting for full batch completion. When any request in the batch finishes, a new request is immediately inserted. Achieves near-100% GPU utilization vs ~60% for static batching.

*Prefill vs decode phases:* Prefill processes all input tokens in parallel (compute-bound, high throughput). Decode generates output tokens one at a time (memory-bandwidth-bound, low throughput). Chunked prefill interleaves these to improve resource utilization.

*Tensor parallelism:* Split weight matrices across multiple GPUs using Megatron-style column/row parallelism. Enables serving models too large for a single GPU. Communication overhead via all-reduce on each layer; efficient on NVLink-connected GPUs.

---

**vLLM** (Kwon et al., 2023):
PagedAttention for virtual KV cache memory management — divides KV cache into fixed-size pages (blocks), allocates on demand, eliminates fragmentation. Continuous batching, speculative decoding, chunked prefill. OpenAI-compatible REST API. Supports most HuggingFace models, AWQ/GPTQ/FP8 quantization. CUDA-only (ROCm experimental). 24x throughput improvement over naive HuggingFace generation.

**llama.cpp** (Gerganov, 2023):
C++ inference engine for GGUF-quantized models. Cross-platform backends: CUDA, Metal (Apple Silicon), ROCm, Vulkan, SYCL, CPU. `llama-server` provides OpenAI-compatible HTTP API. Optimized for low-resource environments; CPU inference viable for smaller quantized models (Q4_K_M 8B ≈ 5 GB RAM).

**Ollama:**
User-friendly wrapper around llama.cpp. Handles model download, storage, and lifecycle management via a simple CLI. Provides OpenAI-compatible API locally. Runs on macOS, Linux, Windows. Best for local development, prototyping, and personal use — not tuned for multi-user production load.

**TGI (Text Generation Inference, HuggingFace):**
Production-grade inference server with tensor parallelism, quantization support (AWQ, GPTQ, EETQ, bitsandbytes), token streaming via SSE, continuous batching, and Prometheus metrics. Deeper HuggingFace ecosystem integration (model hub, tokenizers). Used by HuggingFace Inference Endpoints. More opinionated than vLLM.

**Triton Inference Server (NVIDIA):**
Enterprise model serving platform. Dynamic batching, model ensemble (chain multiple models), A/B testing, multi-framework support (TensorRT, ONNX, TF, PyTorch). High operational complexity; appropriate for large organizations with dedicated ML infrastructure teams.

**SGLang** (Zheng et al., 2024):
Structured generation language runtime. Outperforms vLLM on multi-call workloads (RAG, agents, structured output) via RadixAttention — KV cache reuse across requests sharing common prefixes. Important for agent frameworks where system prompts are reused across many requests.

## Implementation Notes
**vLLM quickstart:**
```bash
pip install vllm
vllm serve meta-llama/Llama-3-8B-Instruct \
    --dtype bfloat16 \
    --api-key token-abc \
    --max-model-len 8192
# Tensor parallel across 4 GPUs:
vllm serve meta-llama/Llama-3-70B-Instruct \
    --tensor-parallel-size 4 \
    --dtype bfloat16
```

**vLLM benchmark:**
```bash
python -m vllm.entrypoints.benchmark_throughput \
    --model meta-llama/Llama-3-8B-Instruct \
    --input-len 512 --output-len 128 --num-prompts 1000
```

**llama-server (llama.cpp):**
```bash
./llama-server -m llama-3-8B-Q4_K_M.gguf \
    --port 8080 --n-gpu-layers 35 --ctx-size 8192
# --n-gpu-layers: offload this many layers to GPU (0 = CPU only)
```

**Ollama:**
```bash
ollama pull llama3:8b
ollama run llama3:8b "Explain attention mechanisms"
ollama serve  # starts API server on :11434
```

**TGI (Docker):**
```bash
docker run --gpus all -p 8080:80 \
  ghcr.io/huggingface/text-generation-inference:latest \
  --model-id meta-llama/Llama-3-8B-Instruct \
  --quantize awq
```

**Performance targets (7–8B model, single A100 80GB):**
- Throughput: 1000–3000 tokens/s (output), depending on batch size
- TTFT (Time To First Token): <200ms at p95 for batch size ≤32
- ITL (Inter-Token Latency): <20ms per output token
- GPU utilization: >85% with continuous batching

**Speculative decoding** (vLLM `--speculative-model`): Small draft model proposes k tokens, large model verifies in parallel. 2–3x latency reduction at same quality for low-entropy outputs (code, structured data).

## Trade-offs
**vLLM vs TGI:** vLLM has superior throughput and KV cache management via PagedAttention; TGI has tighter HuggingFace integration and longer production track record. For greenfield GPU serving, vLLM is the default choice in 2025.

**llama.cpp/Ollama vs vLLM:** llama.cpp/Ollama supports far broader hardware (any machine with 8+ GB RAM can run a quantized 7B model) but throughput is 5–20x lower than vLLM on equivalent CUDA hardware. Use llama.cpp for local development and edge deployment; vLLM for cloud GPU serving.

**SGLang vs vLLM for agent workloads:** SGLang's RadixAttention provides significant speedup when many requests share long common prefixes (e.g., same system prompt + few-shot examples). Benchmark on your specific workload before choosing.

**Triton:** Enterprise-grade but carries significant operational complexity. Justified when serving heterogeneous model ensembles (e.g., embedding model + reranker + LLM in a single serving pipeline) or when existing NVIDIA MLOps infrastructure is in place.

**Speculative decoding trade-off:** Requires a compatible draft model and adds memory cost. Most effective for constrained-output tasks (JSON, code); less effective for open-ended generation where entropy is high.

## References
- PagedAttention/vLLM: Kwon et al. (2023). "Efficient Memory Management for Large Language Model Serving with PagedAttention." SOSP 2023.
- Continuous batching (Orca): Yu et al. (2022). "Orca: A Distributed Serving System for Transformer-Based Generative Models." OSDI 2022.
- SGLang: Zheng et al. (2024). "SGLang: Efficient Execution of Structured Language Model Programs."
- TGI: https://github.com/huggingface/text-generation-inference
- vLLM: https://github.com/vllm-project/vllm

## Links
- [[quantization_overview|Quantization for LLMs]]
- [[attention_and_kv_cache|Attention Optimization and KV Cache]]
- [[llm_observability|LLM Observability]]
- [[serving_patterns|Serving Patterns]]
- [[docker_patterns|Docker Patterns]]
