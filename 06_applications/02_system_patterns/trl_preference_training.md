---
layer: 06_applications
type: application
status: growing
tags: [pattern, fine-tuning, llm]
created: 2026-03-06
---

# TRL: Preference and RL Fine-tuning

## Purpose

Implementation patterns for preference learning and reinforcement learning fine-tuning using the TRL (Transformer Reinforcement Learning) library. TRL provides production-quality implementations of DPO, GRPO, and PPO trainers. DPO is the standard starting point — it directly optimizes preferences from (prompt, chosen, rejected) triples without a reward model. GRPO (DeepSeek-R1) trains a reasoning model by comparing within-group completions scored by custom reward functions; it requires verifiable rewards (math answers, code execution, format adherence) but no preference dataset.

### Examples

- DPO alignment of an instruction-tuned Llama-3-8B model
- GRPO training for structured output format compliance
- Reward model training from human-annotated preference pairs

## Architecture

**Installation:**
```bash
pip install trl>=0.14.0 transformers peft datasets accelerate
```

**DPO fine-tuning (most common starting point):**
```python
from transformers import AutoModelForCausalLM, AutoTokenizer
from trl import DPOTrainer, DPOConfig
from datasets import Dataset
from peft import LoraConfig

model_id  = "meta-llama/Meta-Llama-3-8B-Instruct"
tokenizer = AutoTokenizer.from_pretrained(model_id)
model     = AutoModelForCausalLM.from_pretrained(model_id, torch_dtype="bfloat16")

# Dataset: each row has prompt, chosen, rejected (plain strings)
# Use trl.apply_chat_template to format if needed
dataset = Dataset.from_list([
    {
        "prompt":   "Explain gradient descent.",
        "chosen":   "Gradient descent minimizes a loss function by...",
        "rejected": "I'll tell you about AI. Neural networks are..."
    },
    # ... minimum ~500 examples; 2k–10k typical
])

# QLoRA adapter to fit on a single GPU
peft_config = LoraConfig(r=16, lora_alpha=32, target_modules="all-linear",
                         bias="none", task_type="CAUSAL_LM")

config = DPOConfig(
    output_dir      = "./dpo-llama3",
    per_device_train_batch_size = 1,
    gradient_accumulation_steps = 8,
    learning_rate   = 5e-6,
    num_train_epochs= 3,
    beta            = 0.1,      # KL penalty — lower = more divergence allowed
    bf16            = True,
    logging_steps   = 10,
    save_steps      = 200,
)

trainer = DPOTrainer(
    model         = model,
    args          = config,
    train_dataset = dataset,
    tokenizer     = tokenizer,
    peft_config   = peft_config,
)
trainer.train()
```

**GRPO for verifiable task training:**
```python
from trl import GRPOTrainer, GRPOConfig
import re

SYSTEM = """
Respond with:
<reasoning>your step-by-step reasoning</reasoning>
<answer>final answer</answer>
"""

def reward_format(completions, **kwargs) -> list[float]:
    """Reward correct XML format — 0.5 for having tags, 1.0 for correct nesting."""
    scores = []
    for c in completions:
        text = c[0]["content"]
        has_reasoning = bool(re.search(r"<reasoning>.*?</reasoning>", text, re.DOTALL))
        has_answer    = bool(re.search(r"<answer>.*?</answer>",    text, re.DOTALL))
        scores.append(0.5 * has_reasoning + 0.5 * has_answer)
    return scores

def reward_correctness(completions, answer, **kwargs) -> list[float]:
    """+2.0 if extracted answer matches ground truth."""
    scores = []
    for c, gt in zip(completions, answer):
        text  = c[0]["content"]
        match = re.search(r"<answer>(.*?)</answer>", text, re.DOTALL)
        pred  = match.group(1).strip() if match else ""
        scores.append(2.0 if pred == gt.strip() else 0.0)
    return scores

# Dataset must have a "prompt" column (list of chat messages)
dataset = Dataset.from_list([
    {
        "prompt": [
            {"role": "system",  "content": SYSTEM},
            {"role": "user",    "content": "What is 17 × 23?"}
        ],
        "answer": "391"
    }
])

config = GRPOConfig(
    output_dir                  = "./grpo-reasoning",
    per_device_train_batch_size = 2,
    num_generations             = 8,   # completions per prompt (the "group")
    max_new_tokens              = 512,
    learning_rate               = 5e-6,
    num_train_epochs            = 2,
    gradient_checkpointing      = True,
    bf16                        = True,
)

trainer = GRPOTrainer(
    model       = model_id,
    args        = config,
    train_dataset = dataset,
    reward_funcs= [reward_format, reward_correctness],
)
trainer.train()
```

**Reward model training (for PPO):**
```python
from trl import RewardTrainer, RewardConfig

# Dataset: "input_ids_chosen" / "input_ids_rejected" (tokenized)
# or "chosen" / "rejected" (strings) — RewardTrainer handles both

config = RewardConfig(
    output_dir="./reward-model",
    per_device_train_batch_size=4,
    learning_rate=1e-5,
    num_train_epochs=1,
    bf16=True,
)

trainer = RewardTrainer(
    model         = model,
    args          = config,
    tokenizer     = tokenizer,
    train_dataset = preference_dataset,
)
trainer.train()
```

**Key hyperparameter guide:**

| Param | DPO | GRPO | Notes |
|---|---|---|---|
| `beta` | 0.05–0.3 | N/A | Higher = stay closer to reference |
| `num_generations` | N/A | 4–16 | More = better gradient signal; more VRAM |
| `learning_rate` | 5e-7 – 5e-6 | 5e-7 – 2e-6 | Lower than SFT |
| `per_device_batch` | 1–2 | 1 | Large batches via gradient_accumulation |

**Common pitfalls:**
- DPO: not applying the correct chat template to `prompt/chosen/rejected` before training — use `trl.apply_chat_template`
- GRPO: reward functions must return a `list[float]` of length equal to `len(completions)` — mismatch causes silent errors
- Both: forgetting `pad_token = eos_token` for models without a native pad token (Llama family)

## Links
- [[rl_finetuning|RL Fine-tuning]]
- [[alignment_and_rlhf|Alignment and RLHF]]
- [[peft_lora_finetuning|PEFT and LoRA Fine-tuning]]
- [[instruction_data_design|Instruction Data Design]]
