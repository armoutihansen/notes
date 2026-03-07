---
layer: 08_implementations
type: application
status: growing
tags: [workflow, deployment, safety, llm]
created: 2026-03-06
---

# Production LLM Serving with Safety Stack

## Purpose

End-to-end reference architecture for deploying an LLM as a production API with a full safety, monitoring, and reliability stack. The system integrates vLLM for high-throughput inference, LlamaGuard for input/output content moderation, LangSmith for tracing and quality evaluation, and a model gateway pattern (fallback routing, semantic caching). Spans `06_ai_engineering` (serving frameworks, safety, observability, AI application architecture) and `04_software_engineering` (Docker, CI/CD, FastAPI).

### Examples

- Customer-facing LLM API with content moderation and usage tracking
- Internal copilot service with audit logging and cost controls
- Multi-tenant LLM gateway with per-tenant rate limiting and safety policies

## Architecture

```
Client request
    │
    ▼
[1] FastAPI Gateway
    ├─ Auth / rate limiting (per-user token quotas)
    ├─ Semantic cache (Redis → skip LLM if near-duplicate query cached)
    └─ Request metadata tagging (user_id, session, feature)
    │
    ▼
[2] Input Guard (LlamaGuard)
    │  Safe? → continue    Unsafe? → 403 + audit log
    ▼
[3] vLLM LLM Server (primary)
    │  Timeout? Rate-limit? → Fallback LLM (GPT-4o-mini or smaller model)
    ▼
[4] Output Guard (LlamaGuard, async)
    │  Unsafe response? → replace + audit log
    ▼
[5] Response to client (streaming via SSE)
    │
    ▼
[6] LangSmith trace (async)
    └─ Token usage, latency, safety verdicts, user feedback
```

## Implementation Notes

**Step 1 — Start vLLM and LlamaGuard servers:**
```bash
# Primary LLM — Llama-3-8B on GPU 0
CUDA_VISIBLE_DEVICES=0 vllm serve meta-llama/Llama-3-8B-Instruct \
    --dtype bfloat16 \
    --enable-prefix-caching \
    --max-model-len 8192 \
    --port 8000

# LlamaGuard on GPU 1 (can share GPU with lower utilization)
CUDA_VISIBLE_DEVICES=1 vllm serve meta-llama/LlamaGuard-3-8B \
    --dtype bfloat16 \
    --gpu-memory-utilization 0.5 \
    --port 8001
```

**Step 2 — FastAPI gateway with LangSmith tracing:**
```python
import os
os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_API_KEY"]    = "lsv2_..."
os.environ["LANGCHAIN_PROJECT"]    = "llm-gateway-prod"

from fastapi import FastAPI, HTTPException, Depends, Header
from fastapi.responses import StreamingResponse
from pydantic import BaseModel
from openai import OpenAI
from langsmith import traceable
import asyncio

app = FastAPI()

llm_client   = OpenAI(base_url="http://localhost:8000/v1", api_key="EMPTY")
guard_client = OpenAI(base_url="http://localhost:8001/v1", api_key="EMPTY")
fallback_client = OpenAI(api_key=os.environ["OPENAI_API_KEY"])  # cloud fallback

class ChatRequest(BaseModel):
    message:    str
    session_id: str = "default"

def moderate(conversation: list[dict]) -> tuple[str, str | None]:
    from transformers import AutoTokenizer
    tokenizer = AutoTokenizer.from_pretrained("meta-llama/LlamaGuard-3-8B")
    prompt = tokenizer.apply_chat_template(conversation, tokenize=False)
    resp = guard_client.completions.create(
        model="meta-llama/LlamaGuard-3-8B",
        prompt=prompt, max_tokens=20, temperature=0.0
    )
    text = resp.choices[0].text.strip().lower().split("\n")
    return text[0], (text[1] if len(text) > 1 and text[0] == "unsafe" else None)

@traceable(name="chat-request")
def process_chat(message: str, user_id: str, session_id: str) -> str:
    # Input guard
    verdict, category = moderate([{"role": "user", "content": message}])
    if verdict == "unsafe":
        raise HTTPException(status_code=400, detail=f"Request blocked: {category}")
    
    # LLM call with fallback
    try:
        response = llm_client.chat.completions.create(
            model="meta-llama/Llama-3-8B-Instruct",
            messages=[{"role": "user", "content": message}],
            max_tokens=512, temperature=0.7
        )
        answer = response.choices[0].message.content
    except Exception:
        response = fallback_client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[{"role": "user", "content": message}],
            max_tokens=512
        )
        answer = response.choices[0].message.content
    
    # Output guard (async in production; sync here for simplicity)
    out_verdict, out_cat = moderate([
        {"role": "user",      "content": message},
        {"role": "assistant", "content": answer}
    ])
    if out_verdict == "unsafe":
        return "I'm unable to provide a response to that request."
    
    return answer

@app.post("/chat")
async def chat(req: ChatRequest, x_user_id: str = Header(default="anonymous")):
    answer = process_chat(req.message, x_user_id, req.session_id)
    return {"answer": answer}
```

**Step 3 — Semantic cache (Redis + embedding similarity):**
```python
import redis
import numpy as np
from openai import OpenAI

cache    = redis.Redis(host="localhost", port=6379)
embedder = OpenAI()

def embed(text: str) -> list[float]:
    return embedder.embeddings.create(
        model="text-embedding-3-small", input=text
    ).data[0].embedding

CACHE_THRESHOLD = 0.95  # cosine similarity threshold

def cache_lookup(query: str) -> str | None:
    q_emb = np.array(embed(query))
    for key in cache.scan_iter("cache:*"):
        stored = cache.hgetall(key)
        c_emb  = np.frombuffer(stored[b"embedding"], dtype=np.float32)
        sim    = float(np.dot(q_emb, c_emb) / (np.linalg.norm(q_emb) * np.linalg.norm(c_emb)))
        if sim >= CACHE_THRESHOLD:
            return stored[b"response"].decode()
    return None

def cache_store(query: str, response: str):
    key     = f"cache:{hash(query)}"
    q_emb   = np.array(embed(query), dtype=np.float32)
    cache.hset(key, mapping={"embedding": q_emb.tobytes(), "response": response})
    cache.expire(key, 86400)  # 24-hour TTL
```

**Step 4 — Docker Compose stack:**
```yaml
# docker-compose.yml
version: "3.9"
services:
  vllm:
    image: vllm/vllm-openai:latest
    command: --model meta-llama/Llama-3-8B-Instruct --port 8000
    deploy:
      resources:
        reservations:
          devices: [{driver: nvidia, device_ids: ["0"], capabilities: [gpu]}]

  llamaguard:
    image: vllm/vllm-openai:latest
    command: --model meta-llama/LlamaGuard-3-8B --port 8001
    deploy:
      resources:
        reservations:
          devices: [{driver: nvidia, device_ids: ["1"], capabilities: [gpu]}]

  redis:
    image: redis:7-alpine
    ports: ["6379:6379"]

  gateway:
    build: .
    ports: ["8080:8080"]
    environment:
      - LANGSMITH_API_KEY=${LANGSMITH_API_KEY}
      - LANGCHAIN_TRACING_V2=true
    depends_on: [vllm, llamaguard, redis]
```

**Step 5 — Key SLOs to monitor (LangSmith + Prometheus):**

| Metric | Target | Alert |
|---|---|---|
| Gateway TTFT p95 | < 300 ms | > 500 ms |
| Total latency p95 | < 2 s | > 5 s |
| Input block rate | < 2% | > 10% (injection attack?) |
| Output block rate | < 0.5% | > 2% (model regression?) |
| Fallback rate | < 1% | > 5% (vLLM health issue?) |
| Cost per request p99 | < $0.01 | > $0.05 (runaway loop?) |

## References

- [vLLM Documentation](https://docs.vllm.ai/)
- [LlamaGuard: LLM-based Input-Output Safeguard for Human-AI Conversations (Meta AI)](https://ai.meta.com/research/publications/llama-guard-llm-based-input-output-safeguard-for-human-ai-conversations/)

## Links
- [[serving_frameworks|LLM Serving Frameworks]]
- [[vllm_serving|vLLM Serving]]
- [[safety_and_content_moderation|Safety and Content Moderation]]
- [[llamaguard_content_moderation|LlamaGuard Content Moderation]]
- [[llm_observability|LLM Observability]]
- [[langsmith_llm_observability|LangSmith LLM Observability]]
- [[ai_application_architecture|AI Application Architecture]]
- [[model_serving_with_fastapi|Model Serving with FastAPI]]
- [[docker_ml_pipeline|Docker for ML Pipelines]]
