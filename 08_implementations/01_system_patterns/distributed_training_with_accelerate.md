---
layer: 08_implementations
type: application
status: growing
tags: [pattern, distributed, training]
created: 2026-03-10
---

# Distributed Training with HuggingFace Accelerate

## Purpose

Implements multi-GPU and multi-node distributed training using HuggingFace Accelerate, which provides a unified API over PyTorch DDP, DeepSpeed ZeRO, and FSDP. The same training script runs on a single GPU or across a cluster by changing a configuration file, without code changes.

### Examples

**Fine-tuning a 7B LLM on 4×A100s**: Use Accelerate + DeepSpeed ZeRO-3 with BF16 to distribute parameters, gradients, and optimizer states across GPUs, reducing peak memory from ~56 GB to ~14 GB per GPU.

**Multi-GPU training on a single machine**: Run standard PyTorch training across 8 GPUs with DDP using 4 lines of code and `accelerate launch`.

## Architecture

### Installation and Configuration

```bash
pip install accelerate>=0.27.0

# Interactive configuration wizard
accelerate config
# Prompts: machine type, num GPUs, mixed precision, DeepSpeed/FSDP?
# Saves to ~/.cache/huggingface/accelerate/default_config.yaml
```

### Minimal 4-Line Conversion

```python
from accelerate import Accelerator

accelerator = Accelerator(mixed_precision="bf16")         # 1

model = MyModel()
optimizer = torch.optim.AdamW(model.parameters(), lr=1e-4)
dataloader = DataLoader(dataset, batch_size=8)

model, optimizer, dataloader = accelerator.prepare(        # 2
    model, optimizer, dataloader
)

for batch in dataloader:
    optimizer.zero_grad()
    loss = model(**batch).loss
    accelerator.backward(loss)                             # 3
    optimizer.step()

accelerator.wait_for_everyone()                            # 4
```

```bash
# Launch on all available GPUs
accelerate launch train.py

# Launch on N specific GPUs
accelerate launch --num_processes 4 train.py
```

### DeepSpeed ZeRO-3 Configuration

For models that don't fit on a single GPU, use ZeRO-3 to shard parameters, gradients, and optimizer states:

```yaml
# accelerate_config.yaml
compute_environment: LOCAL_MACHINE
distributed_type: DEEPSPEED
deepspeed_config:
  deepspeed_multinode_launcher: standard
  gradient_accumulation_steps: 4
  gradient_clipping: 1.0
  offload_optimizer_device: none
  offload_param_device: none
  zero3_init_flag: true
  zero3_save_16bit_model: true
  zero_stage: 3
mixed_precision: bf16
num_machines: 1
num_processes: 4
```

```python
# train.py — identical to DDP script; Accelerate handles the ZeRO sharding
accelerator = Accelerator()
model, optimizer, dataloader = accelerator.prepare(model, optimizer, dataloader)
```

### FSDP Configuration (PyTorch Native)

```yaml
# accelerate_config_fsdp.yaml
compute_environment: LOCAL_MACHINE
distributed_type: FSDP
fsdp_config:
  fsdp_auto_wrap_policy: TRANSFORMER_BASED_WRAP
  fsdp_backward_prefetch_policy: BACKWARD_PRE
  fsdp_forward_prefetch: false
  fsdp_offload_params: false
  fsdp_sharding_strategy: 1          # FULL_SHARD = ZeRO-3 equivalent
  fsdp_state_dict_type: FULL_STATE_DICT
  fsdp_transformer_layer_cls_to_wrap: LlamaDecoderLayer
mixed_precision: bf16
num_processes: 4
```

### Gradient Accumulation

```python
accelerator = Accelerator(gradient_accumulation_steps=4)

for batch in dataloader:
    with accelerator.accumulate(model):    # handles sync/no-sync automatically
        optimizer.zero_grad()
        loss = model(**batch).loss
        accelerator.backward(loss)
        optimizer.step()
# Effective batch = local_batch × grad_accum_steps × num_gpus
```

### Checkpointing

```python
# Save — only writes on main process
accelerator.save_state("checkpoint/")

# Load — all processes synchronize
accelerator.load_state("checkpoint/")

# Save unwrapped model (for inference)
unwrapped = accelerator.unwrap_model(model)
unwrapped.save_pretrained("saved_model/", save_function=accelerator.save)
```

### Learning Rate Scheduling

```python
from transformers import get_linear_schedule_with_warmup

scheduler = get_linear_schedule_with_warmup(
    optimizer,
    num_warmup_steps=100,
    num_training_steps=len(dataloader) * num_epochs,
)

# Prepare scheduler alongside model and optimizer
model, optimizer, dataloader, scheduler = accelerator.prepare(
    model, optimizer, dataloader, scheduler
)
```

## Links
- [[05_ml_engineering/05_model_development/distributed_training|Distributed Training]]
- [[05_ml_engineering/05_model_development/experiment_tracking|Experiment Tracking]]
- [[mlflow_experiment_tracking|MLflow Experiment Tracking Pattern]]
- [[large_model_training_pipeline|Large Model Training Pipeline]]
