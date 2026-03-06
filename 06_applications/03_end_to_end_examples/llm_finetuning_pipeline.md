---
layer: 06_applications
type: application
status: growing
tags: [fine-tuning, lora, dpo, llm, pipeline, end-to-end]
created: 2026-03-06
---

# LLM Fine-tuning Pipeline

## Purpose

End-to-end pipeline for adapting a pre-trained LLM to a specific domain or task via supervised fine-tuning (SFT) followed by preference alignment (DPO). The pipeline covers: curating and formatting training data, LoRA/QLoRA fine-tuning with TRL's SFTTrainer, reward-free preference alignment with DPO, merging the adapter, and evaluating the fine-tuned model against a benchmark. Spans `05_ai_engineering` (finetuning, dataset engineering, evaluation) and `04_ml_engineering` (experiment tracking, distributed training).

### Examples

- Domain-specific instruction-following model (legal, medical, coding)
- Custom tool-calling model fine-tuned on company API schemas
- Chat model preference-aligned on internal quality criteria

## Architecture

```
Curate instruction dataset
    │  (Alpaca/ShareGPT format, diversity analysis, dedup)
    ▼
SFT with LoRA / QLoRA
    │  (SFTTrainer, bf16, cosine LR schedule)
    ├─ Log to MLflow (loss, perplexity, grad norm)
    ▼
DPO preference alignment
    │  (DPOTrainer, beta=0.1, preference pairs)
    ▼
Merge adapter → base model
    │  (PEFT merge_and_unload → 16-bit checkpoint)
    ▼
Evaluation
    │  (lm-evaluation-harness MMLU/HumanEval benchmarks)
    │  (held-out domain-specific test set)
    ▼
Register in MLflow Model Registry
    │
    ▼
Serve with vLLM
```

## Implementation Notes

**Step 1 — Data preparation:**
```python
from datasets import Dataset
from sentence_transformers import SentenceTransformer
from sklearn.cluster import KMeans
import numpy as np

# Load and format as ShareGPT (recommended for chat models)
raw = [
    {"conversations": [
        {"from": "human", "value": "Explain LoRA."},
        {"from": "gpt",   "value": "LoRA adds low-rank matrices..."}
    ]},
    # ... target 2k–10k examples
]

# Diversity check — cluster instruction embeddings
encoder = SentenceTransformer("all-MiniLM-L6-v2")
instructions = [ex["conversations"][0]["value"] for ex in raw]
embeddings   = encoder.encode(instructions)
labels       = KMeans(n_clusters=20).fit_predict(embeddings)

from collections import Counter
print("Cluster distribution:", Counter(labels).most_common(5))
# Small clusters = underrepresented topics → fill gaps synthetically

dataset = Dataset.from_list(raw)
```

**Step 2 — SFT with LoRA (tracked in MLflow):**
```python
import mlflow
from trl import SFTTrainer, SFTConfig
from peft import LoraConfig
from transformers import AutoModelForCausalLM, AutoTokenizer, BitsAndBytesConfig
import torch

model_id  = "meta-llama/Meta-Llama-3-8B-Instruct"
tokenizer = AutoTokenizer.from_pretrained(model_id)

bnb_config = BitsAndBytesConfig(
    load_in_4bit=True, bnb_4bit_quant_type="nf4",
    bnb_4bit_compute_dtype=torch.bfloat16, bnb_4bit_use_double_quant=True
)
model = AutoModelForCausalLM.from_pretrained(
    model_id, quantization_config=bnb_config, device_map="auto"
)

lora_config = LoraConfig(
    r=16, lora_alpha=32,
    target_modules=["q_proj", "k_proj", "v_proj", "o_proj",
                    "gate_proj", "up_proj", "down_proj"],
    lora_dropout=0.05, task_type="CAUSAL_LM"
)

mlflow.set_experiment("llm-sft")
with mlflow.start_run(run_name="llama3-8b-lora-sft"):
    mlflow.log_params({"model": model_id, "lora_r": 16, "epochs": 3})
    
    trainer = SFTTrainer(
        model=model,
        args=SFTConfig(
            output_dir="./sft-checkpoint",
            per_device_train_batch_size=1,
            gradient_accumulation_steps=16,
            learning_rate=2e-4,
            num_train_epochs=3,
            lr_scheduler_type="cosine",
            bf16=True,
            logging_steps=20,
            save_steps=200,
        ),
        train_dataset=dataset,
        tokenizer=tokenizer,
        peft_config=lora_config,
    )
    trainer.train()
    
    mlflow.log_metrics({
        "train_loss":  trainer.state.log_history[-1]["train_loss"],
        "train_epoch": trainer.state.epoch
    })
    trainer.save_model("./sft-adapter")
```

**Step 3 — DPO alignment:**
```python
from trl import DPOTrainer, DPOConfig

# Preference dataset (prompt, chosen, rejected)
pref_dataset = Dataset.from_list([
    {
        "prompt":   "Explain gradient descent.",
        "chosen":   "Gradient descent minimizes a loss by following the negative gradient...",
        "rejected": "Machine learning is a type of AI that learns patterns..."
    },
    # ... ~1k–5k preference pairs
])

base_model = AutoModelForCausalLM.from_pretrained(model_id, torch_dtype="bfloat16")
peft_model = PeftModel.from_pretrained(base_model, "./sft-adapter")

with mlflow.start_run(run_name="llama3-8b-dpo"):
    trainer = DPOTrainer(
        model=peft_model,
        args=DPOConfig(
            output_dir="./dpo-adapter",
            per_device_train_batch_size=1,
            gradient_accumulation_steps=8,
            learning_rate=5e-6,
            num_train_epochs=1,
            beta=0.1,
            bf16=True,
        ),
        train_dataset=pref_dataset,
        tokenizer=tokenizer,
    )
    trainer.train()
    trainer.save_model("./dpo-adapter")
```

**Step 4 — Merge adapter and register:**
```python
from peft import PeftModel

base   = AutoModelForCausalLM.from_pretrained(model_id, torch_dtype="bfloat16")
merged = PeftModel.from_pretrained(base, "./dpo-adapter")
merged = merged.merge_and_unload()
merged.save_pretrained("./merged-llama3-8b-domain")
tokenizer.save_pretrained("./merged-llama3-8b-domain")

# Register in MLflow
with mlflow.start_run(run_name="model-registration"):
    mlflow.transformers.log_model(
        transformers_model={"model": merged, "tokenizer": tokenizer},
        artifact_path="model",
        registered_model_name="llama3-8b-domain-v1"
    )
```

**Step 5 — Evaluation with lm-evaluation-harness:**
```bash
lm_eval --model hf \
    --model_args pretrained=./merged-llama3-8b-domain \
    --tasks mmlu,hellaswag \
    --device cuda \
    --batch_size 8 \
    --output_path ./eval_results
```

**Step 6 — Serve with vLLM:**
```bash
vllm serve ./merged-llama3-8b-domain \
    --dtype bfloat16 \
    --max-model-len 8192 \
    --port 8000
```

**Experiment tracking checklist (MLflow):**
- Log: model_id, lora_r, lr, batch_size, num_epochs, dataset_size, dataset_hash
- Track: train_loss curve, perplexity, eval_loss
- Compare runs: SFT baseline vs DPO-aligned

## Links
- [[finetuning_strategies|Fine-tuning Strategies]]
- [[peft_and_lora|PEFT and LoRA]]
- [[rl_finetuning|RL Fine-tuning]]
- [[instruction_data_design|Instruction Data Design]]
- [[synthetic_data_generation|Synthetic Data Generation]]
- [[peft_lora_finetuning|PEFT and LoRA Fine-tuning (Pattern)]]
- [[trl_preference_training|TRL Preference Training (Pattern)]]
- [[mlflow_experiment_tracking|MLflow Experiment Tracking]]
- [[distributed_training_with_accelerate|Distributed Training with Accelerate]]
- [[vllm_serving|vLLM Serving]]
