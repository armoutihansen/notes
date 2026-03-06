---
layer: 06_applications
type: application
status: growing
tags: [llamaguard, content-moderation, safety, guardrails, llm]
created: 2026-03-06
---

# LlamaGuard Content Moderation

## Purpose

Implementation patterns for runtime LLM content moderation using Meta's LlamaGuard. LlamaGuard is a 7–8B parameter safety classifier fine-tuned to judge whether user prompts and model responses are safe across six harm categories (violence, sexual content, weapons, substances, self-harm, criminal planning). It achieves 94–95% accuracy on standard safety benchmarks, returns interpretable category codes, supports custom harm taxonomies via fine-tuning, and integrates naturally with vLLM for production-scale throughput (~50–100 req/s on a single A10).

### Examples

- Input guard: reject harmful user prompts before they reach the primary LLM
- Output guard: intercept unsafe model responses before delivery to users
- Async background moderation: classify in parallel and audit-log violations

## Architecture

**Installation:**
```bash
pip install transformers torch accelerate
huggingface-cli login  # required for LlamaGuard gated model
```

**Basic HuggingFace inference:**
```python
from transformers import AutoTokenizer, AutoModelForCausalLM
import torch

model_id  = "meta-llama/LlamaGuard-3-8B"
tokenizer = AutoTokenizer.from_pretrained(model_id)
model     = AutoModelForCausalLM.from_pretrained(
    model_id, torch_dtype=torch.bfloat16, device_map="auto"
)

def moderate(conversation: list[dict]) -> tuple[str, str | None]:
    """Returns ('safe', None) or ('unsafe', 'S1') etc."""
    input_ids = tokenizer.apply_chat_template(
        conversation, return_tensors="pt"
    ).to(model.device)
    output = model.generate(input_ids, max_new_tokens=20, pad_token_id=0)
    text   = tokenizer.decode(output[0][input_ids.shape[-1]:], skip_special_tokens=True)
    parts  = text.strip().lower().split("\n")
    verdict  = parts[0]                                     # "safe" or "unsafe"
    category = parts[1] if verdict == "unsafe" and len(parts) > 1 else None
    return verdict, category
```

**Input guard pattern (before calling primary LLM):**
```python
def handle_user_request(user_message: str) -> str:
    verdict, category = moderate([{"role": "user", "content": user_message}])
    if verdict == "unsafe":
        log_violation(user_message, category)
        return f"I'm unable to help with that request."

    response = primary_llm.generate(user_message)

    # Output guard
    verdict, category = moderate([
        {"role": "user",      "content": user_message},
        {"role": "assistant", "content": response}
    ])
    if verdict == "unsafe":
        log_violation(response, category, source="model")
        return "I encountered an issue generating a safe response. Please rephrase."

    return response
```

**Harm categories:**

| Code | Category |
|---|---|
| S1 | Violent Crimes |
| S2 | Non-Violent Crimes (fraud, hacking) |
| S3 | Sex-Related Crimes |
| S4 | Child Sexual Exploitation (absolute block) |
| S5 | Defamation / Influence Ops |
| S6 | Weapons (CBRN — chemical, biological, radiological, nuclear) |

**vLLM deployment (high-throughput production):**
```bash
# Serve LlamaGuard on a separate port alongside the primary LLM
vllm serve meta-llama/LlamaGuard-3-8B \
    --dtype bfloat16 \
    --port 8001 \
    --gpu-memory-utilization 0.5   # share GPU with primary model
```
```python
from openai import OpenAI

guard_client = OpenAI(base_url="http://localhost:8001/v1", api_key="EMPTY")

def moderate_via_vllm(conversation: list[dict]) -> tuple[str, str | None]:
    prompt = tokenizer.apply_chat_template(conversation, tokenize=False)
    response = guard_client.completions.create(
        model="meta-llama/LlamaGuard-3-8B",
        prompt=prompt,
        max_tokens=20,
        temperature=0.0
    )
    text     = response.choices[0].text.strip().lower()
    parts    = text.split("\n")
    verdict  = parts[0]
    category = parts[1] if verdict == "unsafe" and len(parts) > 1 else None
    return verdict, category
```

**Async parallel moderation (input check while generating):**
```python
import asyncio

async def moderate_async(conversation):
    loop = asyncio.get_event_loop()
    return await loop.run_in_executor(None, moderate, conversation)

async def handle_async(user_msg):
    input_verdict_future = asyncio.create_task(
        moderate_async([{"role": "user", "content": user_msg}])
    )
    input_verdict, _ = await input_verdict_future
    if input_verdict == "unsafe":
        return "Request blocked."

    response = await generate_response_async(user_msg)

    output_verdict, cat = await moderate_async([
        {"role": "user",      "content": user_msg},
        {"role": "assistant", "content": response}
    ])
    return "Response blocked." if output_verdict == "unsafe" else response
```

**NeMo Guardrails integration (behavioral + topicality policies):**
```python
from nemoguardrails import RailsConfig, LLMRails

config = RailsConfig.from_path("./guardrails_config/")
# guardrails_config/ contains:
#   config.yml      — LLM + rails settings
#   flows.co        — Colang dialogue policies (off-topic, competitor blocks)
#   prompts.yml     — prompt templates for rails

rails    = LLMRails(config)
response = await rails.generate_async(
    messages=[{"role": "user", "content": user_message}]
)
```

**Audit logging policy:**
```python
import logging
import json

audit_log = logging.getLogger("safety.audit")

def log_violation(content: str, category: str, source: str = "user"):
    # Log sanitized input — do NOT log PII in cleartext
    audit_log.warning(json.dumps({
        "source":   source,
        "category": category,
        "content_hash": hashlib.sha256(content.encode()).hexdigest()[:16],
    }))
```

**Latency budget:**
- HuggingFace (A10 GPU, BF16): ~100–200 ms per call
- vLLM batch (A10 GPU): ~50–100 req/s
- Pattern: run input classification synchronously; output classification asynchronously

## Links
- [[safety_and_content_moderation|Safety and Content Moderation]]
- [[prompt_injection_and_guardrails|Prompt Injection and Guardrails]]
- [[ai_application_architecture|AI Application Architecture]]
- [[vllm_serving|vLLM Serving]]
