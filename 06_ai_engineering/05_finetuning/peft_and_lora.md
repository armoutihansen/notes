---
layer: 06_ai_engineering
type: engineering
tool: peft
status: evergreen
tags: [fine-tuning]
created: 2026-03-05
---

# PEFT and LoRA

## Purpose

Parameter-Efficient Fine-Tuning (PEFT) addresses the central tension in LLM adaptation: full fine-tuning achieves the best task performance, but updating billions of parameters requires GPU memory and compute that is inaccessible for most practitioners. PEFT methods achieve near-full-fine-tune quality while training fewer than 1% of model parameters — often far less.

LoRA (Low-Rank Adaptation) is the dominant PEFT technique in practice, with QLoRA (quantized LoRA) enabling fine-tuning of 65B+ parameter models on consumer-grade hardware. The PEFT library from HuggingFace provides standardized implementations of LoRA, QLoRA, prefix tuning, IA³, and adapter layers. Frameworks like Axolotl and LLaMA-Factory wrap PEFT into high-level training pipelines.

## Architecture

**LoRA — Core Mechanism**

Freeze the pre-trained weight matrix `W ∈ ℝ^{d×k}`. Inject two trainable low-rank matrices `A ∈ ℝ^{d×r}` and `B ∈ ℝ^{r×k}` where `r << min(d, k)`:

```
h = Wx + (α/r) · BAx
```

- `A` is initialized with random Gaussian; `B` is initialized to zero → ΔW = 0 at training start (no perturbation at initialization)
- `α` is a scaling hyperparameter (often set to `2r` as a starting point)
- The rank `r` controls expressiveness: r=8 (light), r=16 (standard), r=64 (heavy adaptation)

At inference, adapters can be **merged** back into the base weights: `W' = W + (α/r)·BA`, adding zero inference latency. Alternatively, adapters can be kept separate for hot-swapping between tasks.

**Target modules:** Typically applied to attention projection matrices `q_proj`, `v_proj`. Extending to `k_proj`, `o_proj`, and MLP layers (`gate_proj`, `up_proj`, `down_proj`) increases expressiveness at modest cost.

**QLoRA — Quantized LoRA**

Dettmers et al. (2023) extends LoRA with three key innovations:
1. **NF4 (Normal Float 4-bit)** quantization of the frozen base model weights — information-theoretically optimal for normally-distributed weights
2. **Double quantization** — quantize the quantization constants themselves to save additional memory
3. **Paged optimizers** — use CPU memory as overflow for GPU optimizer states (prevents OOM during gradient spikes)

LoRA adapters are trained in BF16 on top of the 4-bit base. This enables fine-tuning a 65B model on a single 48GB GPU — previously requiring 780GB of GPU memory for full fine-tuning in BF16.

**Other PEFT methods:**
- **Prefix tuning**: prepend learned soft tokens to each transformer layer's key/value sequence; no weight modification
- **Adapter layers**: small feed-forward modules inserted between transformer sub-layers; higher parameter count than LoRA for similar performance
- **IA³**: scales attention keys/values and MLP activations with learned vectors; extremely parameter-efficient (~0.01% of params)

## Implementation Notes

**HuggingFace PEFT library:**
```python
from peft import get_peft_model, LoraConfig, TaskType

config = LoraConfig(
    r=16,
    lora_alpha=32,
    target_modules=["q_proj", "v_proj"],
    lora_dropout=0.05,
    bias="none",
    task_type=TaskType.CAUSAL_LM,
)
model = get_peft_model(base_model, config)
model.print_trainable_parameters()
# trainable params: 4,194,304 || all params: 6,742,609,920 || trainable%: 0.06
```

**Adapter merging:**
```python
model = model.merge_and_unload()  # merges LoRA weights into base, returns standard model
model.save_pretrained("merged_model/")
```

**Axolotl YAML config (LoRA):**
```yaml
adapter: lora
lora_r: 16
lora_alpha: 32
lora_dropout: 0.05
lora_target_modules:
  - q_proj
  - v_proj
```

**Axolotl YAML config (QLoRA):**
```yaml
adapter: qlora
load_in_4bit: true
lora_r: 64
lora_alpha: 128
lora_target_modules:
  - q_proj
  - k_proj
  - v_proj
  - o_proj
  - gate_proj
  - up_proj
  - down_proj
```

**Rank selection heuristics:**
- r=8: minimal compute, good for style/format adaptation
- r=16: standard default for task-specific fine-tuning
- r=64: heavy domain adaptation, complex task; comparable to full fine-tuning in some benchmarks
- Higher rank = more trainable parameters (2×r×d per target module) = more GPU memory for optimizer states

**LLaMA-Factory** provides a WebUI for LoRA/QLoRA configuration without writing code, supporting Llama, Qwen, Gemma, Mistral, and 100+ other architectures. Includes DPO and GRPO trainers; supports multimodal fine-tuning.

**SFTTrainer key settings (TRL):**
- `model.config.use_cache = False` — disable KV-cache during training (required; cache is incompatible with gradient checkpointing and wastes memory)
- `packing=True` — concatenate short examples into fixed-length sequences for GPU utilisation; set `False` when training on long documents or when sequences are already near `max_seq_length`
- `report_to` — pass `"mlflow"`, `"wandb"`, or `"tensorboard"` to route training metrics to the appropriate experiment tracker

## Trade-offs

**LoRA vs full fine-tuning:**
- LoRA trains ~0.1–1% of parameters → 5–10× less GPU memory for optimizer states
- Adapter files are tiny (typically 50–500MB vs 14GB+ for a 7B full model)
- Performance gap vs full fine-tuning narrows with higher rank and broader target module coverage
- Full fine-tuning still wins on tasks requiring deep structural changes to representations

**QLoRA vs LoRA (full precision):**
- QLoRA enables fine-tuning on consumer GPUs (24GB VRAM for 7B, 48GB for 70B)
- ~1–3% quality degradation vs full-precision LoRA due to quantization noise
- Inference with a QLoRA-trained adapter requires either dequantizing the base or running inference at 4-bit (which can be slower depending on hardware)

**Adapter composability:**
- Multiple LoRA adapters can be merged additively: useful for combining a domain adapter with a task adapter
- Hot-swapping adapters (without merge) is supported by PEFT's `set_adapter()` — enables a single base model to serve multiple tasks
- Merging is irreversible; keep separate copies for flexibility in production

**Prefix tuning vs LoRA:**
- Prefix tuning modifies attention patterns via soft prompts rather than weight updates
- More sensitive to hyperparameters; LoRA is generally more robust and has displaced it in practice

## References

- Hu et al. (2021) — *LoRA: Low-Rank Adaptation of Large Language Models* (arXiv:2106.09685)
- Dettmers et al. (2023) — *QLoRA: Efficient Finetuning of Quantized LLMs* (arXiv:2305.14314)
- Liu et al. (2022) — *Few-Shot Parameter-Efficient Fine-Tuning is Better and Cheaper than In-Context Learning*
- HuggingFace PEFT: https://github.com/huggingface/peft
- Axolotl: https://github.com/OpenAccess-AI-Collective/axolotl
- LLaMA-Factory: https://github.com/hiyouga/LLaMA-Factory

## Links
- [[finetuning_strategies|Fine-tuning Strategies]]
- [[rl_finetuning|Reinforcement Learning Fine-tuning]]
- [[instruction_data_design|Instruction Data Design]]
- [[distributed_training|Distributed Training]]
- [[model_compression|Model Compression]]
