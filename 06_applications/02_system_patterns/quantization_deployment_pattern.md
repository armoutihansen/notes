---
layer: 06_applications
type: application
status: growing
tags: [pattern, quantization, deployment]
created: 2026-05-10
---

# Quantization Deployment Pattern

## Purpose

LLM quantization reduces model memory footprint and inference latency by representing weights in lower-precision formats (INT4, NF4, INT8). This note provides a decision table for choosing the right quantization format by hardware target, and shows concrete implementation for the four main approaches: AWQ, GPTQ, GGUF (llama.cpp), and bitsandbytes 4-bit.

### Examples

**Cloud GPU serving**: AWQ-quantized Llama-3-70B on 2×A100 (80GB) achieving 3× throughput vs. FP16.

**Local inference on Mac M2**: GGUF Q4_K_M on llama.cpp — 13B model at ~8 GB RAM, 25 tokens/sec.

**Quick experiment**: bitsandbytes `load_in_4bit=True` on any HuggingFace model without calibration data.

---

## Architecture

Quantization operates at the **weight level**: instead of storing FP16/BF32 weights, parameters are packed into INT4/INT8 representations. The inference pipeline remains the same (tokenize → forward pass → decode), but memory bandwidth decreases and throughput increases.

```
Model weights (FP16)
       ↓  quantize
Model weights (INT4/INT8)  ← 2–4× smaller
       ↓
Load into GPU / CPU memory
       ↓
Dequantize on-the-fly during matmul  (done by kernel)
       ↓
Output logits → decode
```

The four quantization strategies differ in: whether calibration data is needed, hardware compatibility, accuracy-compression trade-off, and supported serving runtimes.

---

## Format Decision Table

| Format | Calibration data | GPU required | Best for |
|---|---|---|---|
| **bitsandbytes NF4** | None | NVIDIA only | Quick experiments, QLoRA training |
| **GPTQ INT4** | Yes (128 samples) | NVIDIA (triton) | HF hub quants, cloud serving |
| **AWQ INT4** | Yes (~128 samples) | NVIDIA (autoawq) | Best quality at INT4, production serving |
| **GGUF** | None (post-convert) | CPU / Apple Silicon / AMD | Local inference, edge, llama.cpp ecosystem |

---

## bitsandbytes 4-bit (NF4 / QLoRA)

```bash
pip install transformers bitsandbytes accelerate
```

```python
import torch
from transformers import AutoModelForCausalLM, AutoTokenizer, BitsAndBytesConfig

bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type="nf4",            # Normal Float 4-bit (best for LLMs)
    bnb_4bit_compute_dtype=torch.bfloat16,
    bnb_4bit_use_double_quant=True,       # saves ~0.4 bits/param via double quantization
)

model = AutoModelForCausalLM.from_pretrained(
    "meta-llama/Meta-Llama-3-8B",
    quantization_config=bnb_config,
    device_map="auto",
)
tokenizer = AutoTokenizer.from_pretrained("meta-llama/Meta-Llama-3-8B")

inputs  = tokenizer("Explain gradient descent:", return_tensors="pt").to("cuda")
outputs = model.generate(**inputs, max_new_tokens=100)
print(tokenizer.decode(outputs[0], skip_special_tokens=True))
```

**Memory profile (Llama-3-8B):** FP16 ~16 GB → NF4 ~5 GB

---

## GPTQ (Post-Training Quantization with Calibration)

```bash
pip install auto-gptq optimum
```

```python
from transformers import AutoTokenizer
from auto_gptq import AutoGPTQForCausalLM, BaseQuantizeConfig

model_id = "meta-llama/Meta-Llama-3-8B"
tokenizer = AutoTokenizer.from_pretrained(model_id)

# Calibration dataset (128 samples from C4 or similar)
calibration_data = [
    tokenizer("The quick brown fox", return_tensors="pt").input_ids
    for _ in range(128)
]

quantize_config = BaseQuantizeConfig(
    bits=4,
    group_size=128,        # smaller group = better quality, more memory
    desc_act=True,         # activation reordering for better quality
)

model = AutoGPTQForCausalLM.from_pretrained(model_id, quantize_config=quantize_config)
model.quantize(calibration_data)
model.save_quantized("llama3-8b-gptq-int4", use_safetensors=True)
tokenizer.save_pretrained("llama3-8b-gptq-int4")

# Load for inference
model_q = AutoGPTQForCausalLM.from_quantized("llama3-8b-gptq-int4", device="cuda:0")
```

**Pre-quantized models**: Search HuggingFace Hub for `<model_name>-GPTQ` — TheBloke's uploads are widely used.

---

## AWQ (Activation-Aware Weight Quantization)

```bash
pip install autoawq
```

```python
from awq import AutoAWQForCausalLM
from transformers import AutoTokenizer

model_id = "meta-llama/Meta-Llama-3-8B"
output_dir = "llama3-8b-awq-int4"

tokenizer = AutoTokenizer.from_pretrained(model_id)
model = AutoAWQForCausalLM.from_pretrained(model_id, device_map="cuda")

quant_config = {
    "zero_point": True,
    "q_group_size": 128,
    "w_bit": 4,
    "version": "GEMM",    # GEMM (throughput) or GEMV (latency)
}
model.quantize(tokenizer, quant_config=quant_config)
model.save_quantized(output_dir)
tokenizer.save_pretrained(output_dir)

# Load for inference
model_awq = AutoAWQForCausalLM.from_quantized(output_dir, fuse_layers=True, device_map="cuda")
```

**Why AWQ > GPTQ?** AWQ identifies and protects salient weight channels (high-activation magnitude) from aggressive quantization, resulting in 0.5–1.0 perplexity points lower than GPTQ at the same INT4 budget.

---

## GGUF (llama.cpp — CPU / Apple Silicon)

```bash
pip install llama-cpp-python    # CPU build
# Apple Silicon: CMAKE_ARGS="-DLLAMA_METAL=on" pip install llama-cpp-python
```

**Convert a Llama model to GGUF:**
```bash
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp && pip install -r requirements.txt

# Convert HF model to GGUF float16
python convert_hf_to_gguf.py /path/to/hf-model --outfile model-f16.gguf

# Quantize to Q4_K_M (best quality/size balance for 4-bit)
./quantize model-f16.gguf model-q4_k_m.gguf Q4_K_M
```

```python
from llama_cpp import Llama

llm = Llama(
    model_path="model-q4_k_m.gguf",
    n_ctx=4096,           # context window
    n_threads=8,          # CPU threads
    n_gpu_layers=0,       # 0 = CPU only; set to -1 to offload all layers to GPU/Metal
)

output = llm("Explain gradient descent in one sentence:", max_tokens=80, echo=False)
print(output["choices"][0]["text"])
```

**GGUF quantization levels:**

| Level | Bits/weight | Quality | File size (13B) |
|---|---|---|---|
| Q2_K | ~2.6 | Poor | ~3.5 GB |
| Q4_K_M | ~4.5 | Good | ~7.9 GB |
| Q5_K_M | ~5.7 | Very good | ~9.9 GB |
| Q8_0 | ~8.0 | Near-lossless | ~13.8 GB |

---

## Serving Quantized Models with vLLM

```python
from vllm import LLM, SamplingParams

# AWQ and GPTQ models load directly via vLLM
llm = LLM(model="llama3-8b-awq-int4", quantization="awq", dtype="float16",
          max_model_len=4096)
params = SamplingParams(temperature=0.7, max_tokens=200)
outputs = llm.generate(["What is RAG?"], params)
print(outputs[0].outputs[0].text)
```

---

## Links

**AI Engineering**
- [[05_ai_engineering/06_inference_optimization/quantization_overview|Quantization Overview]] — bitsandbytes, GPTQ, AWQ, GGUF theory and trade-offs
- [[05_ai_engineering/06_inference_optimization/serving_frameworks|LLM Serving Frameworks]] — vLLM, TGI, llama.cpp performance characteristics

**System Patterns**
- [[vllm_serving|vLLM Serving]] — production vLLM deployment with quantized models
- [[peft_lora_finetuning|PEFT LoRA Fine-tuning]] — QLoRA (bitsandbytes) for fine-tuning quantized models

**End-to-End Examples**
- [[production_llm_serving_with_safety|Production LLM Serving with Safety]] — serving pipeline including quantization
