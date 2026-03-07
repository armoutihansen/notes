---
layer: 06_ai_engineering
type: engineering
tool: general
status: growing
tags: [quantization, deployment]
created: 2026-03-05
---
# Quantization for LLMs

## Purpose
Reducing model memory footprint and inference latency by representing weights and activations with lower-precision numeric formats. A 70B parameter model in FP32 requires ~280 GB of VRAM — beyond any single GPU. Quantization makes large models deployable on realistic hardware without retraining, enabling both local inference on consumer devices and cost-efficient cloud serving. The core trade-off is always precision vs. resource reduction.

## Architecture
Precision hierarchy per parameter:

| Format | Bytes/param | 70B model VRAM |
|--------|-------------|----------------|
| FP32   | 4           | ~280 GB        |
| FP16 / BF16 | 2      | ~140 GB        |
| INT8   | 1           | ~70 GB         |
| NF4 / INT4 | 0.5    | ~35 GB         |

**Quantization families:**

**(a) bitsandbytes** — post-load quantization via HuggingFace Transformers integration. `load_in_8bit=True` applies LLM.int8(), using mixed-precision decomposition to handle outlier activations. `load_in_4bit=True` with `bnb_4bit_quant_type="nf4"` uses NormalFloat4, an information-theoretically optimal 4-bit data type for normally distributed weights (as found in pre-trained LLMs). Simplest integration path; no calibration dataset required.

**(b) GPTQ** (Frantar et al., 2022) — post-training quantization minimizing layer-wise reconstruction error using a small calibration dataset (e.g., 128 samples from C4 or wikitext). Stores INT4 weights; fast GPU inference using triton or ExLlamaV2 kernels. Standard format for HuggingFace model hub quantized uploads.

**(c) AWQ** (Lin et al., 2023 — MLSys 2024 Best Paper) — Activation-Aware Weight Quantization. Identifies salient weight channels by examining activation magnitudes across calibration samples; protects those channels from aggressive quantization by scaling. Achieves better perplexity than GPTQ at the same 4-bit budget, with ~3x speedup vs FP16 via optimized GEMM kernels. Calibration dataset required.

**(d) HQQ** (Half-Quadratic Quantization) — optimization-based quantization using half-quadratic splitting; no calibration dataset needed. Supports 2/3/4-bit with fast quantization time. Useful when calibration data is unavailable or the quantization latency of GPTQ/AWQ is unacceptable.

**(e) GGUF + llama.cpp** — GGUF is the serialization format used by llama.cpp for CPU and heterogeneous hardware inference. Supports 2–8 bit quantization variants; `Q4_K_M` (4-bit, k-quant with medium quality) is the most popular balance of quality and model size. Runs on Apple Silicon via Metal backend, AMD/Intel GPUs via ROCm/Vulkan, and standard CPUs. Basis for Ollama.

## Implementation Notes
**bitsandbytes (4-bit NF4):**
```python
from transformers import BitsAndBytesConfig, AutoModelForCausalLM
import torch

bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type="nf4",
    bnb_4bit_compute_dtype=torch.bfloat16,
    bnb_4bit_use_double_quant=True,  # nested quantization for codebook
)
model = AutoModelForCausalLM.from_pretrained("meta-llama/Llama-3-8B", quantization_config=bnb_config)
```

**AWQ:**
```python
from awq import AutoAWQForCausalLM
model = AutoAWQForCausalLM.from_pretrained("meta-llama/Llama-3-8B")
quant_config = {"zero_point": True, "q_group_size": 128, "w_bit": 4, "version": "GEMM"}
model.quantize(tokenizer, quant_config=quant_config)
model.save_quantized("llama3-8b-awq")
```

**GGUF inference via llama.cpp:**
```bash
./llama-cli -m llama-3-8B-Q4_K_M.gguf -p "The key insight is" -n 200
./llama-server -m llama-3-8B-Q4_K_M.gguf --port 8080  # REST API
```

**Quality degradation benchmarks (approximate perplexity increase on wikitext-2):**
- INT8: ~0% (effectively lossless)
- INT4/NF4: ~1–3% perplexity increase
- INT3: ~5–10%, model-dependent

## Trade-offs
**INT8 vs INT4:** INT8 gives near-zero quality loss at 2x memory reduction; INT4 gives slight but measurable quality loss at 4x reduction. For most applications INT4 is acceptable; for high-stakes or long-form generation, INT8 may be preferred.

**Quality ranking at INT4:** AWQ ≥ GPTQ > bitsandbytes NF4 ≈ HQQ. The gap is small for instruction-tuned models and widens for base models.

**Calibration-based (GPTQ, AWQ) vs calibration-free (bitsandbytes, HQQ):** Calibration-based methods achieve better perplexity retention but require a representative dataset and add quantization time. For domain-specific models, use in-domain calibration data.

**GGUF/llama.cpp vs vLLM:** GGUF is CPU-first with broad hardware support; vLLM delivers higher GPU throughput but requires CUDA. For local/edge deployment GGUF dominates; for cloud serving vLLM dominates.

**Double quantization:** bitsandbytes `bnb_4bit_use_double_quant=True` quantizes the quantization constants themselves, saving an additional ~0.4 bits/param with negligible quality impact. Enable by default.

## References
- LLM.int8(): Dettmers et al. (2022). "LLM.int8(): 8-bit Matrix Multiplication for Transformers at Scale." NeurIPS 2022.
- GPTQ: Frantar et al. (2022). "GPTQ: Accurate Post-Training Quantization for Generative Pre-trained Transformers." ICLR 2023.
- AWQ: Lin et al. (2023). "AWQ: Activation-aware Weight Quantization for LLM Compression and Acceleration." MLSys 2024 Best Paper.
- QLoRA: Dettmers et al. (2023). "QLoRA: Efficient Finetuning of Quantized LLMs." NeurIPS 2023. (uses NF4)
- llama.cpp: Gerganov (2023–). https://github.com/ggerganov/llama.cpp

## Links
- [[attention_and_kv_cache|Attention Optimization and KV Cache]]
- [[serving_frameworks|LLM Serving Frameworks]]
- [[model_compression|Model Compression]]
