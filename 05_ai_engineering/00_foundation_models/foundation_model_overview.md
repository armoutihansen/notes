---
layer: 05_ai_engineering
type: engineering
tool: general
status: growing
tags: [foundation-models, transformers, llm, architecture]
created: 2026-03-05
---

# Foundation Model Overview

## Purpose

Foundation models are large neural networks pre-trained on massive, diverse datasets — typically hundreds of billions to trillions of tokens drawn from web text, books, code, and scientific literature. The central engineering insight is the **pre-training + fine-tuning paradigm**: one expensive pre-training run produces a general-purpose model that can be adapted cheaply to many downstream tasks.

These models exhibit **emergent capabilities** — abilities that appear at scale but are absent in smaller models: multi-step reasoning, in-context learning (few-shot prompting), instruction following, and rudimentary tool use. This makes them qualitatively different from task-specific models.

From an engineering standpoint, foundation models are **infrastructure components**: they provide general-purpose language understanding and generation that you layer application logic on top of. The decision of which model to use, how to serve it, and how to constrain its behaviour are first-class engineering decisions.

## Architecture

### Transformer Core

All dominant foundation models are built on the **Transformer** (Vaswani et al., 2017). Key components:

- **Multi-head self-attention**: each token attends to all other tokens in the context window; `O(n²)` in sequence length. Captures long-range dependencies.
- **Feed-forward network (FFN)**: two linear layers with a non-linearity (GELU/SwiGLU). Expanded dimension typically 4× model dimension.
- **Layer normalisation (LayerNorm / RMSNorm)**: applied pre- or post-attention; stabilises training.
- **Residual connections**: wrap both attention and FFN sub-layers; enable gradient flow through depth.
- **Positional encoding**: learned absolute (GPT-2), sinusoidal (original Transformer), or **RoPE** (Rotary Position Embedding, most modern LLMs) / **ALiBi** for length generalisation.

### Model Families by Architecture

| Family | Architecture | Examples |
|---|---|---|
| Decoder-only (autoregressive) | Causal self-attention | GPT-4, Llama 3, Mistral, Qwen, Gemma |
| Encoder-only (masked LM) | Bidirectional attention | BERT, RoBERTa, DeBERTa |
| Encoder-decoder (seq2seq) | Cross-attention between encoder and decoder | T5, BART, Flan-T5 |

Decoder-only models dominate generative AI workloads. Encoder-only models remain useful for classification, NER, and retrieval (bi-encoder embeddings). Encoder-decoder models suit structured generation (summarisation, translation) where the input and output are clearly distinct.

### Multimodal Models

- **CLIP**: contrastive vision-language alignment; image and text encoders trained on 400M image-text pairs. Enables zero-shot image classification and cross-modal retrieval.
- **LLaVA / BLIP-2**: vision-language models that project visual features into the LLM token space, enabling image-conditioned generation.
- **Whisper**: encoder-decoder model for speech recognition; trained on 680K hours of multilingual audio.

### State-Space Alternatives

**Mamba** (Gu & Dao, 2023) is a selective state-space model with `O(n)` complexity in sequence length, no KV cache, and competitive performance with transformers on language tasks. Hardware-aware selective scan enables fast training and inference on long sequences. An emerging alternative for long-context workloads where quadratic attention becomes a bottleneck.

## Implementation Notes

### Model Families and Sizes

Common parameter counts and typical use cases:

| Size | Examples | Use Case |
|---|---|---|
| 1B–3B | Phi-3-mini, Llama 3.2 3B | Edge deployment, low-latency classification |
| 7B–8B | Llama 3.1 8B, Mistral 7B, Gemma 7B | Capable general-purpose; fits single consumer GPU |
| 13B–14B | Llama 2 13B, Qwen2 14B | Better reasoning; 2× GPU |
| 70B | Llama 3.1 70B | Near-frontier open quality |
| Frontier | GPT-4o, Claude 3.5, Gemini 1.5 | Best capability; API only |

### Context Lengths and Engineering Implications

Longer context windows (`32K–1M tokens`) enable retrieval-free document QA, long conversation memory, and whole-codebase context. Engineering implications:
- **KV cache** grows linearly with context length; at `128K` tokens with a 70B model, KV cache alone can exceed 80 GB.
- **Attention cost** is `O(n²)` unless using Flash Attention or sliding-window attention.
- **Needle-in-a-haystack** degradation: many models perform worse at the middle of long contexts.

### VRAM Requirements (approximate, FP16)

| Parameters | FP16 | INT8 | INT4 |
|---|---|---|---|
| 7B | ~14 GB | ~7 GB | ~4 GB |
| 13B | ~26 GB | ~13 GB | ~7 GB |
| 70B | ~140 GB | ~70 GB | ~35 GB |

### Distribution and Selection

**HuggingFace Hub** is the canonical distribution point for open-weight models: model cards, tokenizer configs, quantised variants (GGUF, AWQ, GPTQ). Key selection criteria:

1. **Capability**: benchmark scores (MMLU, HumanEval, MT-Bench) relative to task requirements.
2. **License**: permissive (Apache 2.0: Mistral, Falcon) vs restricted (Llama community licence) vs proprietary API.
3. **Latency and throughput**: smaller models at higher batch sizes often win on tokens/sec/dollar.
4. **Domain fit**: code models (DeepSeek Coder, CodeLlama), multilingual models (Qwen, Aya), long-context models (Yarn-Mistral, Claude).

## Trade-offs

| Dimension | Open-weight | Closed / API |
|---|---|---|
| Cost at scale | Infra cost only | Per-token pricing |
| Privacy | Data stays in-house | Data leaves to provider |
| Capability | Up to 70B open rivals frontier | GPT-4-class leads on hard reasoning |
| Ops burden | Full MLOps stack | Zero infra |

**Capability vs cost vs latency**: frontier API models maximise quality but cost 10–100× open-weight alternatives at equivalent throughput. Smaller quantised open-weight models running locally minimise latency (no network round-trip) at the cost of capability.

**General vs domain-specific**: general models trained on diverse data generalise better zero-shot; domain-specific fine-tunes (code, biomedical, legal) outperform on narrow tasks with limited prompting.

## References

- Vaswani et al. (2017). *Attention Is All You Need*. NeurIPS.
- Touvron et al. (2023). *Llama 2: Open Foundation and Fine-Tuned Chat Models*. Meta AI.
- Hoffmann et al. (2022). *Training Compute-Optimal Large Language Models* (Chinchilla). DeepMind.
- Gu & Dao (2023). *Mamba: Linear-Time Sequence Modelling with Selective State Spaces*.
- Radford et al. (2021). *Learning Transferable Visual Models from Natural Language Supervision* (CLIP). OpenAI.

## Links

- [[alignment_and_rlhf|Alignment and RLHF]]
- [[tokenization|Tokenization]]
- [[llm_evaluation_overview|LLM Evaluation Overview]]
- [[distributed_training|Distributed Training]]
- [[serving_patterns|Serving Patterns]]
