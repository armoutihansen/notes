---
layer: 06_applications
type: application
status: seed
tags: [vllm, serving, inference, llm, production, pagedattention]
created: 2026-03-06
---

# vLLM: Production LLM Serving

## Purpose

Implementation patterns for deploying LLMs in production using vLLM. vLLM's PagedAttention virtualizes KV cache memory into fixed-size pages (eliminating fragmentation) and combines it with continuous batching to achieve 24× higher throughput than naive HuggingFace generation. It exposes an OpenAI-compatible REST API, supports GPTQ/AWQ/FP8 quantization, and scales to multi-GPU tensor-parallel deployments. It is the standard serving framework for GPU-based LLM inference in 2025.

### Examples

- Single-GPU serving of Llama-3-8B-Instruct with OpenAI-compatible API
- 70B model serving across 4 GPUs with tensor parallelism + AWQ quantization
- Batch offline inference over large datasets

## Architecture

**Installation:**
```bash
pip install vllm
```

**OpenAI-compatible server (most common deployment):**
```bash
# Single GPU — 8B model
vllm serve meta-llama/Llama-3-8B-Instruct \
    --dtype bfloat16 \
    --max-model-len 8192 \
    --gpu-memory-utilization 0.9 \
    --enable-prefix-caching \
    --port 8000

# Multi-GPU tensor parallelism — 70B model
vllm serve meta-llama/Llama-3-70B-Instruct \
    --tensor-parallel-size 4 \
    --dtype bfloat16 \
    --quantization awq \
    --gpu-memory-utilization 0.9 \
    --port 8000
```

**Query with OpenAI SDK (zero code changes from OpenAI API):**
```python
from openai import OpenAI

client = OpenAI(base_url="http://localhost:8000/v1", api_key="EMPTY")

response = client.chat.completions.create(
    model="meta-llama/Llama-3-8B-Instruct",
    messages=[{"role": "user", "content": "Explain PagedAttention"}],
    temperature=0.7,
    max_tokens=512,
    stream=True
)
for chunk in response:
    print(chunk.choices[0].delta.content or "", end="")
```

**Offline batch inference (no server needed):**
```python
from vllm import LLM, SamplingParams

llm = LLM(
    model="meta-llama/Llama-3-8B-Instruct",
    dtype="bfloat16",
    tensor_parallel_size=1,
    gpu_memory_utilization=0.9,
    max_model_len=4096,
)

sampling = SamplingParams(temperature=0.7, max_tokens=256, stop=["</s>"])

# vLLM batches all prompts automatically — far more efficient than looping
prompts = ["Summarize: ...", "Translate to French: ...", "Explain: ..."]
outputs = llm.generate(prompts, sampling)

for output in outputs:
    print(output.outputs[0].text)
```

**AWQ quantized serving (4× lower VRAM):**
```bash
# First quantize (one-time, ~10 min for 8B)
pip install autoawq
python -c "
from awq import AutoAWQForCausalLM
from transformers import AutoTokenizer

model = AutoAWQForCausalLM.from_pretrained('meta-llama/Llama-3-8B-Instruct')
tokenizer = AutoTokenizer.from_pretrained('meta-llama/Llama-3-8B-Instruct')
quant_config = {'zero_point': True, 'q_group_size': 128, 'w_bit': 4, 'version': 'GEMM'}
model.quantize(tokenizer, quant_config=quant_config)
model.save_quantized('llama3-8b-awq')
"

# Serve quantized model (~5 GB VRAM instead of ~16 GB)
vllm serve ./llama3-8b-awq --quantization awq --dtype float16
```

**Docker deployment:**
```bash
docker run --gpus all \
  -p 8000:8000 \
  -v ~/.cache/huggingface:/root/.cache/huggingface \
  vllm/vllm-openai:latest \
  --model meta-llama/Llama-3-8B-Instruct \
  --gpu-memory-utilization 0.9 \
  --enable-prefix-caching
```

**Structured JSON output (compatible with Instructor):**
```bash
# Enable guided decoding
vllm serve meta-llama/Llama-3-8B-Instruct --guided-decoding-backend outlines
```
```python
from pydantic import BaseModel
import json

class Person(BaseModel):
    name: str
    age: int

schema = Person.model_json_schema()

response = client.chat.completions.create(
    model="meta-llama/Llama-3-8B-Instruct",
    messages=[{"role": "user", "content": "Extract: John Doe, 30 years old"}],
    extra_body={"guided_json": json.dumps(schema)}
)
person = Person.model_validate_json(response.choices[0].message.content)
```

**Prometheus metrics + Grafana monitoring:**
```bash
vllm serve ... --enable-metrics --metrics-port 9090
# Key metrics:
# vllm:time_to_first_token_seconds_bucket — TTFT latency histogram
# vllm:request_success_total              — throughput
# vllm:gpu_cache_usage_perc              — KV cache pressure
# vllm:num_requests_running              — active requests
```

**Speculative decoding (2–3× lower latency for constrained output):**
```bash
vllm serve meta-llama/Llama-3-70B-Instruct \
    --speculative-model meta-llama/Llama-3-8B-Instruct \
    --num-speculative-tokens 5 \
    --tensor-parallel-size 4
```

**Performance targets (single A100 80 GB):**

| Model | Quantization | VRAM | TTFT p95 | Throughput |
|---|---|---|---|---|
| Llama-3-8B-Instruct | BF16 | ~16 GB | <150 ms | ~2500 tok/s |
| Llama-3-8B-Instruct | AWQ-4bit | ~5 GB | <200 ms | ~2000 tok/s |
| Llama-3-70B-Instruct | BF16 (4×) | ~140 GB | <300 ms | ~800 tok/s |
| Llama-3-70B-Instruct | AWQ-4bit (2×) | ~40 GB | <350 ms | ~700 tok/s |

## Links
- [[serving_frameworks|LLM Serving Frameworks]]
- [[quantization_overview|Quantization for LLMs]]
- [[attention_and_kv_cache|Attention Optimization and KV Cache]]
- [[llm_observability|LLM Observability]]
- [[docker_ml_pipeline|Docker for ML Pipelines]]
