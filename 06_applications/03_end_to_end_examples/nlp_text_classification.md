---
layer: 06_applications
type: application
status: growing
tags: [workflow, nlp, fine-tuning, classification]
created: 2026-05-10
---

# NLP Text Classification

## Purpose

A complete reference pipeline for fine-tuning a pretrained transformer as a text classifier: from raw text dataset to production API. This end-to-end example covers: tokenization → LoRA fine-tuning with SFTTrainer → evaluation (F1, classification report) → vLLM deployment → FastAPI wrapper.

### Examples

**Customer support ticket routing**: Classify incoming support tickets into 10 categories; fine-tune a Llama-3-8B with LoRA on 5,000 labelled tickets; serve at 300 req/s with vLLM.

**Legal document classification**: Fine-tune a BERT-based model with sequence classification head on contract clauses; deploy as a low-latency microservice.

---

## Architecture

```
Labelled text dataset (JSONL or HuggingFace Dataset)
    │
    ├──[1] Tokenization (AutoTokenizer, chat template)
    │
    ├──[2] LoRA fine-tuning (PEFT + SFTTrainer)
    │       ├── LoRA config (r=16, target_modules)
    │       └── Training args (BF16, gradient accumulation)
    │
    ├──[3] Evaluation
    │       ├── F1 score (macro / weighted)
    │       └── Per-class classification report
    │
    ├──[4] Adapter saving + optional merging
    │
    ├──[5] vLLM deployment (offline batch or online serving)
    │
    └──[6] FastAPI wrapper with /classify endpoint
```

---

## Step 1: Dataset Preparation

```python
from datasets import Dataset
import pandas as pd

# Expected format: {"text": "...", "label": "category_name"}
# Convert to instruction format for generative fine-tuning
LABEL2ID = {"billing": 0, "technical": 1, "shipping": 2, "returns": 3, "general": 4}
ID2LABEL = {v: k for k, v in LABEL2ID.items()}
LABEL_STR = ", ".join(LABEL2ID.keys())

def format_example(row: dict) -> dict:
    instruction = (
        f"Classify the following customer support ticket into exactly one category: {LABEL_STR}.\n\n"
        f"Ticket: {row['text']}\n\nCategory:"
    )
    return {"text": instruction + " " + row["label"]}

df = pd.read_json("data/tickets.jsonl", lines=True)
dataset = Dataset.from_pandas(df).map(format_example)
splits  = dataset.train_test_split(test_size=0.1, seed=42)
train_ds, test_ds = splits["train"], splits["test"]
```

---

## Step 2: Tokenizer and LoRA Fine-tuning

```python
from transformers import AutoModelForCausalLM, AutoTokenizer, TrainingArguments
from peft import LoraConfig, get_peft_model, TaskType
from trl import SFTTrainer

MODEL_ID = "meta-llama/Meta-Llama-3-8B"
OUTPUT_DIR = "./ticket-classifier-lora"

tokenizer = AutoTokenizer.from_pretrained(MODEL_ID)
tokenizer.pad_token = tokenizer.eos_token
tokenizer.padding_side = "right"

model = AutoModelForCausalLM.from_pretrained(
    MODEL_ID, torch_dtype="bfloat16", device_map="auto",
)
model.config.use_cache = False

lora_config = LoraConfig(
    r=16,
    lora_alpha=32,
    target_modules=["q_proj", "k_proj", "v_proj", "o_proj",
                    "gate_proj", "up_proj", "down_proj"],
    lora_dropout=0.05,
    bias="none",
    task_type=TaskType.CAUSAL_LM,
)
model = get_peft_model(model, lora_config)
model.print_trainable_parameters()

training_args = TrainingArguments(
    output_dir=OUTPUT_DIR,
    per_device_train_batch_size=4,
    gradient_accumulation_steps=4,
    learning_rate=2e-4,
    num_train_epochs=3,
    lr_scheduler_type="cosine",
    warmup_ratio=0.05,
    bf16=True,
    logging_steps=20,
    save_steps=100,
    evaluation_strategy="steps",
    eval_steps=100,
    report_to="mlflow",
)

trainer = SFTTrainer(
    model=model,
    args=training_args,
    train_dataset=train_ds,
    eval_dataset=test_ds,
    tokenizer=tokenizer,
    dataset_text_field="text",
    max_seq_length=512,
    packing=False,    # keep False for short classification examples
)
trainer.train()
```

---

## Step 3: Evaluation (F1 + Classification Report)

```python
from sklearn.metrics import f1_score, classification_report
import torch, re

model.eval()
predictions, labels = [], []

for row in test_ds:
    inputs   = tokenizer(row["text"].split("Category:")[0] + "Category:", return_tensors="pt").to("cuda")
    with torch.no_grad():
        output = model.generate(**inputs, max_new_tokens=10, temperature=0.1, do_sample=False)
    decoded = tokenizer.decode(output[0][inputs["input_ids"].shape[1]:], skip_special_tokens=True).strip().lower()

    # Extract first matching label
    pred_label = next((lbl for lbl in LABEL2ID if lbl in decoded), "general")
    predictions.append(pred_label)
    labels.append(row["label"])

f1_macro    = f1_score(labels, predictions, average="macro",    labels=list(LABEL2ID.keys()))
f1_weighted = f1_score(labels, predictions, average="weighted", labels=list(LABEL2ID.keys()))

print(f"Macro F1: {f1_macro:.3f}  |  Weighted F1: {f1_weighted:.3f}")
print(classification_report(labels, predictions, labels=list(LABEL2ID.keys())))
```

---

## Step 4: Save and Merge Adapter

```python
# Save adapter weights
trainer.model.save_pretrained(f"{OUTPUT_DIR}/adapter")
tokenizer.save_pretrained(f"{OUTPUT_DIR}/adapter")

# Optionally merge into base for zero-latency inference
from peft import PeftModel
from transformers import AutoModelForCausalLM

base = AutoModelForCausalLM.from_pretrained(MODEL_ID, torch_dtype="bfloat16", device_map="auto")
peft_model = PeftModel.from_pretrained(base, f"{OUTPUT_DIR}/adapter")
merged = peft_model.merge_and_unload()
merged.save_pretrained(f"{OUTPUT_DIR}/merged")
tokenizer.save_pretrained(f"{OUTPUT_DIR}/merged")
```

---

## Step 5: vLLM Deployment

```bash
pip install vllm
python -m vllm.entrypoints.openai.api_server \
  --model ./ticket-classifier-lora/merged \
  --dtype bfloat16 \
  --max-model-len 1024 \
  --served-model-name ticket-classifier \
  --port 8001
```

```python
# Test the vLLM endpoint
from openai import OpenAI
client = OpenAI(base_url="http://localhost:8001/v1", api_key="dummy")

response = client.completions.create(
    model="ticket-classifier",
    prompt=f"Classify the following customer support ticket into exactly one category: {LABEL_STR}.\n\nTicket: My invoice shows the wrong amount.\n\nCategory:",
    max_tokens=5, temperature=0.0,
)
print(response.choices[0].text.strip())  # Expected: "billing"
```

---

## Step 6: FastAPI Wrapper

```python
# api.py
from fastapi import FastAPI
from pydantic import BaseModel
from openai import OpenAI

app    = FastAPI(title="Ticket Classifier")
vllm   = OpenAI(base_url="http://localhost:8001/v1", api_key="dummy")
LABELS = list(LABEL2ID.keys())

class TicketRequest(BaseModel):
    text: str

@app.post("/classify")
def classify(req: TicketRequest):
    prompt = (
        f"Classify the following customer support ticket into exactly one category: {', '.join(LABELS)}.\n\n"
        f"Ticket: {req.text}\n\nCategory:"
    )
    resp  = vllm.completions.create(model="ticket-classifier", prompt=prompt, max_tokens=5, temperature=0.0)
    label = resp.choices[0].text.strip().lower()
    pred  = next((l for l in LABELS if l in label), "general")
    return {"category": pred, "input_text": req.text}

@app.get("/health")
def health(): return {"status": "ok"}
```

```bash
uvicorn api:app --host 0.0.0.0 --port 8080 --workers 2
```

---

## Key Trade-offs

| Decision | Choice | Alternative |
|---|---|---|
| Architecture | Generative LLM + LoRA | BERT with classification head (faster, smaller, more accurate for short text) |
| Evaluation | Label extraction from generated text | Logits over label tokens (faster, more reliable) |
| Deployment | vLLM (OpenAI-compatible API) | Text Generation Inference (HuggingFace) |
| Fine-tuning scale | LoRA (7B) | Full fine-tuning of small encoder (BERT-base, 110M params) |

**When to choose BERT over generative LLM**: For text classification with ≤512 tokens, a fine-tuned BERT-base achieves comparable F1 at 10× less compute and 100× faster inference. Use generative LLMs for multi-label, complex instruction following, or zero-shot classification.

---

## Links

**AI Engineering**
- [[05_ai_engineering/04_finetuning/peft_and_lora|PEFT and LoRA]] — LoRA/QLoRA theory and target modules
- [[05_ai_engineering/04_finetuning/finetuning_strategies|Fine-tuning Strategies]] — SFT vs. RLHF vs. frozen head
- [[05_ai_engineering/06_inference_optimization/serving_frameworks|LLM Serving Frameworks]] — vLLM, TGI performance characteristics

**Modeling**
- [[02_modeling/03_model_families/07_transformers/transformer_architecture|Transformer Architecture]] — attention mechanism of the base model

**Model Implementations**
- [[transformer_finetuning_implementation|Transformer Fine-tuning Implementation]] — LoRA/QLoRA standalone fine-tuning reference

**System Patterns**
- [[peft_lora_finetuning|PEFT LoRA Fine-tuning]] — LoRA + SFTTrainer production pattern
- [[vllm_serving|vLLM Serving]] — vLLM deployment with PagedAttention
- [[model_serving_with_fastapi|Model Serving with FastAPI]] — FastAPI production serving patterns
