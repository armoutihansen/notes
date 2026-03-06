---
layer: 06_applications
type: application
status: growing
tags: [pattern, fine-tuning, llm]
created: 2026-03-06
---

# PEFT and LoRA Fine-tuning

## Purpose

Implementation patterns for parameter-efficient fine-tuning (PEFT) of large language models using LoRA and QLoRA. LoRA adds trainable rank-decomposition matrices to frozen weight matrices, reducing trainable parameters by 100–1000× vs full fine-tuning while achieving near-equivalent downstream quality. QLoRA combines NF4 quantization with LoRA to fine-tune 13B–70B models on a single consumer GPU. The HuggingFace PEFT library is the standard implementation; Axolotl provides a YAML-driven training harness for common configurations.

### Examples

- Domain-adaptive fine-tuning of Llama-3-8B on a medical QA corpus
- DPO preference alignment with QLoRA on a 13B model (single A100)
- Function-calling fine-tune of a base model on custom tool schemas

## Architecture

**Installation:**
```bash
pip install peft transformers bitsandbytes accelerate datasets
```

**Full LoRA fine-tuning (16-bit, single or multi-GPU):**
```python
from transformers import AutoModelForCausalLM, AutoTokenizer, TrainingArguments
from peft import LoraConfig, get_peft_model, TaskType
from trl import SFTTrainer
from datasets import load_dataset

model_id = "meta-llama/Meta-Llama-3-8B"
tokenizer = AutoTokenizer.from_pretrained(model_id)
model = AutoModelForCausalLM.from_pretrained(model_id, torch_dtype="bfloat16", device_map="auto")

# LoRA config — target the attention projection matrices
lora_config = LoraConfig(
    r=16,                          # rank — 8/16 for instruction, 32/64 for heavy tasks
    lora_alpha=32,                 # scaling factor (usually 2× rank)
    target_modules=["q_proj", "k_proj", "v_proj", "o_proj",
                    "gate_proj", "up_proj", "down_proj"],
    lora_dropout=0.05,
    bias="none",
    task_type=TaskType.CAUSAL_LM,
)

model = get_peft_model(model, lora_config)
model.print_trainable_parameters()
# trainable params: 20,971,520 || all params: 8,051,232,768 || trainable%: 0.2604

dataset = load_dataset("json", data_files="train.jsonl", split="train")

training_args = TrainingArguments(
    output_dir="./lora-llama3-8b",
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
)

trainer = SFTTrainer(
    model=model,
    args=training_args,
    train_dataset=dataset,
    tokenizer=tokenizer,
    dataset_text_field="text",
    max_seq_length=2048,
)
trainer.train()
```

**QLoRA (4-bit NF4 quantization — enables 70B on 2×A100):**
```python
from transformers import BitsAndBytesConfig
import torch

bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type="nf4",
    bnb_4bit_compute_dtype=torch.bfloat16,
    bnb_4bit_use_double_quant=True,   # quantize the quantization constants
)

model = AutoModelForCausalLM.from_pretrained(
    "meta-llama/Meta-Llama-3-70B",
    quantization_config=bnb_config,
    device_map="auto"
)

# Rest is identical to LoRA — apply LoraConfig and train
```

**Saving and merging the adapter:**
```python
# Save adapter only (~100 MB for r=16)
model.save_pretrained("./lora-adapter")

# Merge adapter into base model weights (for serving)
from peft import PeftModel

base = AutoModelForCausalLM.from_pretrained(model_id, torch_dtype="bfloat16")
merged = PeftModel.from_pretrained(base, "./lora-adapter")
merged = merged.merge_and_unload()          # fuses LoRA weights into base
merged.save_pretrained("./merged-model")    # full 16-bit model, no adapter overhead
```

**Inference with adapter (no merge required):**
```python
from peft import PeftModel

base   = AutoModelForCausalLM.from_pretrained(model_id, torch_dtype="bfloat16")
model  = PeftModel.from_pretrained(base, "./lora-adapter")
model  = model.eval()

inputs = tokenizer("Summarize: The paper proposes...", return_tensors="pt").to("cuda")
with torch.no_grad():
    output = model.generate(**inputs, max_new_tokens=200)
print(tokenizer.decode(output[0], skip_special_tokens=True))
```

**Axolotl YAML config (alternative to writing training code):**
```yaml
# axolotl_config.yml
base_model: meta-llama/Meta-Llama-3-8B
model_type: LlamaForCausalLM

datasets:
  - path: train.jsonl
    type: sharegpt

adapter: lora
lora_r: 16
lora_alpha: 32
lora_dropout: 0.05
lora_target_modules:
  - q_proj
  - v_proj

sequence_len: 2048
train_on_inputs: false
flash_attention: true
bf16: true

micro_batch_size: 2
gradient_accumulation_steps: 8
num_epochs: 3
learning_rate: 0.0002
optimizer: adamw_bnb_8bit
lr_scheduler: cosine
warmup_ratio: 0.05
```
```bash
axolotl train axolotl_config.yml
```

**Rank selection guide:**

| Task | r | Notes |
|---|---|---|
| Style / format | 4–8 | Minimal adapter, quick |
| Domain knowledge | 16 | Standard starting point |
| Complex reasoning | 32–64 | More capacity, more VRAM |
| Full task learning | 128 | Rare; usually full FT better |

**Common pitfalls:**
- Not including all MLP layers in `target_modules` → slow convergence for knowledge tasks; include `gate/up/down_proj` for Llama
- Not applying chat template when fine-tuning chat models → training distribution mismatch
- `train_on_inputs=True` trains on the prompt tokens as well — usually incorrect for instruction tuning

## Links
- [[peft_and_lora|PEFT and LoRA]]
- [[finetuning_strategies|Fine-tuning Strategies]]
- [[rl_finetuning|RL Fine-tuning]]
- [[instruction_data_design|Instruction Data Design]]
- [[distributed_training_with_accelerate|Distributed Training with Accelerate]]
