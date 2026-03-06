---
layer: 08_reference_implementations
type: application
status: growing
tags: [algorithm, nlp, fine-tuning]
created: 2026-05-10
---

# Transformer Fine-tuning Implementation (LoRA / PEFT)

## Goal

Fine-tune a pretrained transformer (BERT, LLaMA) using LoRA/QLoRA via HuggingFace PEFT and TRL SFTTrainer.

## Conceptual Counterpart

- [[06_ai_engineering/05_finetuning/lora_qlora|LoRA and QLoRA Fine-tuning]] — low-rank adaptation theory, rank selection, target modules
- [[06_ai_engineering/04_rag_and_agents/rag_architecture|RAG Architecture]] — alternative to fine-tuning for knowledge-grounded generation
- [[07_applications/05_generation_and_assistance/code_generation_assistant|Code Generation Assistant]] — downstream application of transformer fine-tuning

## Purpose

Practical guide for fine-tuning a pretrained causal or sequence-classification transformer using LoRA and the HuggingFace PEFT + TRL stack.

### Examples

**Instruction tuning**: Fine-tune Llama-3-8B on a custom Q&A dataset for domain-specific assistant behaviour.

**Sentiment / classification**: Attach a classification head via `AutoModelForSequenceClassification` and fine-tune with LoRA adapters on a sentiment dataset.

---

## Architecture

```
Pretrained model (frozen base weights W)
        ↓
   LoRA: inject A ∈ ℝ^{d×r} and B ∈ ℝ^{r×k} per target layer
   Output: h = Wx + (α/r) · BAx      (r ≪ min(d,k))
        ↓
   SFTTrainer (TRL) wraps HF Trainer with text-field handling
        ↓
   Save adapter weights (separate from base)
        ↓
   Merge adapters into base for inference (optional)
```

---

## Setup

```bash
pip install transformers peft trl bitsandbytes accelerate datasets
```

---

## LoRA Fine-tuning (BF16, Single or Multi-GPU)

```python
from transformers import AutoModelForCausalLM, AutoTokenizer, TrainingArguments
from peft import LoraConfig, get_peft_model, TaskType
from trl import SFTTrainer
from datasets import load_dataset

MODEL_ID = "meta-llama/Meta-Llama-3-8B"
OUTPUT_DIR = "./lora-llama3-8b"

# --- Tokenizer ---
tokenizer = AutoTokenizer.from_pretrained(MODEL_ID)
tokenizer.pad_token = tokenizer.eos_token    # required for causal LM training
tokenizer.padding_side = "right"

# --- Model ---
model = AutoModelForCausalLM.from_pretrained(
    MODEL_ID,
    torch_dtype="bfloat16",
    device_map="auto",           # distributes layers across available GPUs
)
model.config.use_cache = False   # disable KV cache during training

# --- LoRA config ---
lora_config = LoraConfig(
    r=16,                          # rank — 8/16 for instruction, 32–64 for heavy tasks
    lora_alpha=32,                 # scaling factor (typically 2× rank)
    target_modules=["q_proj", "k_proj", "v_proj", "o_proj",
                    "gate_proj", "up_proj", "down_proj"],
    lora_dropout=0.05,
    bias="none",
    task_type=TaskType.CAUSAL_LM,
)
model = get_peft_model(model, lora_config)
model.print_trainable_parameters()
# Output: trainable params: ~20M || all params: ~8B || trainable%: 0.26

# --- Dataset ---
dataset = load_dataset("json", data_files={"train": "train.jsonl"}, split="train")
# Expected format: {"text": "### Instruction:\n...\n### Response:\n..."}

# --- Training args ---
training_args = TrainingArguments(
    output_dir=OUTPUT_DIR,
    per_device_train_batch_size=2,
    gradient_accumulation_steps=8,   # effective batch = 16
    learning_rate=2e-4,
    num_train_epochs=3,
    lr_scheduler_type="cosine",
    warmup_ratio=0.05,
    bf16=True,
    logging_steps=10,
    save_steps=200,
    evaluation_strategy="steps",
    eval_steps=200,
    save_total_limit=3,
    report_to="mlflow",             # or "wandb", "tensorboard"
)

# --- SFTTrainer ---
trainer = SFTTrainer(
    model=model,
    args=training_args,
    train_dataset=dataset,
    tokenizer=tokenizer,
    dataset_text_field="text",
    max_seq_length=2048,
    packing=True,                   # pack short examples for efficiency
)
trainer.train()
```

---

## QLoRA (4-bit NF4 — Fine-tune 70B on 2×A100)

```python
from transformers import BitsAndBytesConfig
import torch

bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type="nf4",              # Normal Float 4-bit
    bnb_4bit_compute_dtype=torch.bfloat16,
    bnb_4bit_use_double_quant=True,         # double quantization saves ~0.4 bits/param
)

model = AutoModelForCausalLM.from_pretrained(
    MODEL_ID,
    quantization_config=bnb_config,
    device_map="auto",
)

# Use the same LoRA config and SFTTrainer as above
# QLoRA automatically trains adapters in BF16 on top of 4-bit base
```

---

## Saving and Loading Adapter Weights

```python
# Save adapter weights only (lightweight ~20–200 MB)
trainer.model.save_pretrained(f"{OUTPUT_DIR}/adapter")
tokenizer.save_pretrained(f"{OUTPUT_DIR}/adapter")

# Load adapter for inference
from peft import PeftModel

base_model = AutoModelForCausalLM.from_pretrained(MODEL_ID, torch_dtype="bfloat16", device_map="auto")
model = PeftModel.from_pretrained(base_model, f"{OUTPUT_DIR}/adapter")
model.eval()
```

---

## Merging Adapters into Base Weights

```python
# Merge LoRA weights into the base model for zero-latency inference
merged_model = model.merge_and_unload()

# Save merged model
merged_model.save_pretrained(f"{OUTPUT_DIR}/merged")
tokenizer.save_pretrained(f"{OUTPUT_DIR}/merged")

# Load merged model (standard HuggingFace load, no PEFT dependency)
from transformers import pipeline
pipe = pipeline("text-generation", model=f"{OUTPUT_DIR}/merged", torch_dtype="bfloat16", device_map="auto")
print(pipe("Explain gradient descent in one sentence:", max_new_tokens=80)[0]["generated_text"])
```

---

## Rank Selection Guide

| Task | Recommended rank `r` | Notes |
|---|---|---|
| Instruction following | 8–16 | Style/format changes |
| Domain adaptation | 16–32 | Knowledge injection |
| Task fine-tuning (classification) | 32–64 | Heavier adaptation |
| Full task re-training | 64–128 | Rarely better than full FT |

**Pitfalls:**
- Not applying chat template → training distribution mismatch for chat models
- `train_on_inputs=True` trains on prompt tokens — incorrect for instruction tuning
- Missing MLP target modules (`gate/up/down_proj`) → slow convergence for knowledge tasks
- Not setting `tokenizer.padding_side = "right"` for causal LMs → shape errors

---

## Links

**AI Engineering**
- [[06_ai_engineering/05_finetuning/peft_and_lora|PEFT and LoRA]] — LoRA theory, QLoRA, target modules
- [[06_ai_engineering/05_finetuning/finetuning_strategies|Fine-tuning Strategies]] — when to use LoRA vs full FT vs frozen head
- [[06_ai_engineering/05_finetuning/rl_finetuning|RL Fine-tuning]] — DPO / GRPO after SFT

**Modeling**
- [[03_modeling/04_deep_learning/04_transformers/transformer|Transformer Architecture]] — attention mechanism underpinning the base model

**System Patterns**
- [[peft_lora_finetuning|PEFT LoRA Fine-tuning Pattern]] — deployment-ready pattern with QLoRA + evaluation gate
- [[vllm_serving|vLLM Serving]] — serving the merged model for high-throughput inference

**End-to-End Examples**
- [[nlp_text_classification|NLP Text Classification]] — fine-tune → evaluate → deploy pipeline
