---
layer: 03_software_engineering
type: engineering
tool: HuggingFace
status: growing
tags: [huggingface, transformers, nlp, models, tokenizers, hub, datasets]
created: 2026-03-05
---

# HuggingFace Usage

## Purpose

HuggingFace provides the `transformers`, `datasets`, `tokenizers`, and `PEFT` libraries that together form the dominant Python ecosystem for working with pretrained language models. This note covers the core usage patterns: loading models from the Hub, inference pipelines, custom training, and dataset manipulation. For fine-tuning strategies see [[05_ai_engineering/04_finetuning/peft_and_lora|PEFT and LoRA]] and [[05_ai_engineering/04_finetuning/finetuning_strategies|Finetuning Strategies]].

## Architecture

```
HuggingFace Hub (model/dataset registry)
        │
        ▼
transformers (AutoTokenizer, AutoModel, Pipeline, Trainer)
datasets    (Dataset, DatasetDict, streaming)
tokenizers  (fast Rust tokenizers backing transformers)
peft        (LoRA, QLoRA, adapter injection into any model)
trl         (SFTTrainer, DPOTrainer, reward model training)
accelerate  (distributed training wrapper)
```

Models are identified by **model IDs** (`"meta-llama/Llama-3-8B"`, `"bert-base-uncased"`) and loaded via `from_pretrained`. Weights are cached under `~/.cache/huggingface/hub/`.

## Implementation Notes

### Hub and Model Loading

```python
from transformers import AutoTokenizer, AutoModelForCausalLM
import torch

model_id = "meta-llama/Llama-3-8B-Instruct"

tokenizer = AutoTokenizer.from_pretrained(model_id)
model = AutoModelForCausalLM.from_pretrained(
    model_id,
    torch_dtype=torch.bfloat16,   # use bf16 on Ampere+
    device_map="auto",             # automatically split across available GPUs
    attn_implementation="flash_attention_2",  # optional speed-up
)
```

**Key `from_pretrained` kwargs:**

| Kwarg | Purpose |
|-------|---------|
| `torch_dtype=torch.bfloat16` | Half-precision weights (~2× memory reduction) |
| `device_map="auto"` | Multi-GPU / CPU offload via `accelerate` device map |
| `load_in_8bit=True` | INT8 quantisation via `bitsandbytes` |
| `load_in_4bit=True` | NF4 quantisation for QLoRA |
| `trust_remote_code=True` | Allow custom model code from Hub (use carefully) |

### Pipeline API — Quick Inference

```python
from transformers import pipeline

# Text generation
gen = pipeline("text-generation", model="meta-llama/Llama-3-8B-Instruct",
               device_map="auto", torch_dtype=torch.bfloat16)
out = gen("Summarise this: ...", max_new_tokens=256, do_sample=True, temperature=0.7)

# Classification
clf = pipeline("text-classification", model="distilbert-base-uncased-finetuned-sst-2-english")
print(clf("I really loved the movie!"))
# → [{'label': 'POSITIVE', 'score': 0.9998}]

# NER
ner = pipeline("ner", model="Jean-Baptiste/roberta-large-ner-english", aggregation_strategy="simple")
```

### Chat / Instruction Models

```python
messages = [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user",   "content": "What is the capital of France?"},
]

# Apply the model's chat template
input_ids = tokenizer.apply_chat_template(messages, tokenize=True,
                                          add_generation_prompt=True,
                                          return_tensors="pt").to(device)

outputs = model.generate(input_ids, max_new_tokens=200, do_sample=False)
reply = tokenizer.decode(outputs[0][input_ids.shape[-1]:], skip_special_tokens=True)
```

### Datasets Library

```python
from datasets import load_dataset, Dataset

# Load from Hub
ds = load_dataset("HuggingFaceH4/ultrachat_200k")
train = ds["train_sft"]

# Load from local files
ds = Dataset.from_dict({"text": texts, "label": labels})
ds = load_dataset("json", data_files="data.jsonl", split="train")

# Preprocessing
def tokenize(batch):
    return tokenizer(batch["text"], truncation=True, max_length=512)

tokenized = train.map(tokenize, batched=True, remove_columns=["text"])
tokenized = tokenized.with_format("torch")  # returns torch tensors

# Streaming (avoids downloading full dataset)
streaming_ds = load_dataset("c4", "en", split="train", streaming=True)
for sample in streaming_ds.take(10):
    print(sample["text"][:100])
```

### Trainer API

```python
from transformers import TrainingArguments, Trainer

args = TrainingArguments(
    output_dir="./output",
    num_train_epochs=3,
    per_device_train_batch_size=8,
    gradient_accumulation_steps=4,
    learning_rate=2e-5,
    fp16=True,                        # or bf16=True on Ampere
    evaluation_strategy="epoch",
    save_strategy="epoch",
    load_best_model_at_end=True,
    logging_steps=50,
    report_to="wandb",                # or "mlflow", "tensorboard"
)

trainer = Trainer(
    model=model,
    args=args,
    train_dataset=tokenized_train,
    eval_dataset=tokenized_eval,
    tokenizer=tokenizer,
    data_collator=DataCollatorWithPadding(tokenizer),
)
trainer.train()
trainer.save_model("./best_model")
```

### Pushing to the Hub

```python
from huggingface_hub import HfApi

# After training
model.push_to_hub("myorg/my-finetuned-model")
tokenizer.push_to_hub("myorg/my-finetuned-model")

# Or via Trainer
trainer.push_to_hub()
```

## Trade-offs

| Pattern | Pro | Con |
|---------|-----|-----|
| `device_map="auto"` | Trivial multi-GPU/CPU offload | Inference only; training needs `accelerate` |
| Pipeline API | Simplest interface | Less control; no easy batching customisation |
| Trainer | Batteries included | Opaque; hard to debug custom training logic |
| Streaming datasets | No disk space required | Slower; no random access |
| `trust_remote_code=True` | Required for some models | Arbitrary code execution risk |

## References

- [Transformers documentation](https://huggingface.co/docs/transformers/)
- [Datasets documentation](https://huggingface.co/docs/datasets/)
- [HuggingFace Hub](https://huggingface.co/docs/hub/)
- [Trainer documentation](https://huggingface.co/docs/transformers/main_classes/trainer)

## Links
- [[deep_learning_frameworks|Deep Learning Frameworks]]
- [[pytorch_patterns|PyTorch Patterns]]
- [[framework_comparison|Framework Comparison]]
- [[05_ai_engineering/04_finetuning/peft_and_lora|PEFT and LoRA]]
- [[05_ai_engineering/00_foundation_models/tokenization|Tokenization]]
- [[04_ml_engineering/04_model_development/experiment_tracking|Experiment Tracking]]
