---
layer: 03_software_engineering
type: engineering
tool: PyTorch
status: growing
tags: [pytorch, deep-learning, training, custom-layers, optimization]
created: 2026-03-05
---

# PyTorch Patterns

## Purpose

Practical PyTorch patterns for building and training neural networks efficiently. Covers custom datasets and data loaders, model checkpointing, mixed-precision training, gradient accumulation, and model compilation. Complements [[deep_learning_frameworks|Deep Learning Frameworks]] which covers the core API and framework comparison.

## Architecture

PyTorch is built around three layers:
1. **Tensor + autograd** — `torch.Tensor` with gradient tracking; `.backward()` populates `.grad` on leaf tensors
2. **`nn.Module`** — composable stateful containers for parameters and submodules; forward pass defined in `.forward()`
3. **Optimizers + schedulers** — `torch.optim.*` decouple parameter updates from the model; `torch.optim.lr_scheduler.*` for learning-rate policies

```
Data → Dataset → DataLoader → Model (nn.Module) → Loss → backward() → Optimizer.step()
```

## Implementation Notes

### Custom Dataset and DataLoader

```python
from torch.utils.data import Dataset, DataLoader

class TextDataset(Dataset):
    def __init__(self, texts, labels, tokenizer, max_len=512):
        self.encodings = tokenizer(texts, truncation=True, padding='max_length',
                                   max_length=max_len, return_tensors='pt')
        self.labels = torch.tensor(labels)

    def __len__(self):
        return len(self.labels)

    def __getitem__(self, idx):
        item = {k: v[idx] for k, v in self.encodings.items()}
        item['labels'] = self.labels[idx]
        return item

ds = TextDataset(texts, labels, tokenizer)
loader = DataLoader(ds, batch_size=32, shuffle=True, num_workers=4,
                    pin_memory=True,          # faster GPU transfer
                    persistent_workers=True)  # keep workers alive between epochs
```

### Checkpointing

```python
# Save
torch.save({
    'epoch': epoch,
    'model_state_dict': model.state_dict(),
    'optimizer_state_dict': optimizer.state_dict(),
    'loss': loss,
}, 'checkpoint.pt')

# Load
checkpoint = torch.load('checkpoint.pt', map_location=device)
model.load_state_dict(checkpoint['model_state_dict'])
optimizer.load_state_dict(checkpoint['optimizer_state_dict'])
start_epoch = checkpoint['epoch'] + 1
```

### Mixed-Precision Training (AMP)

Cuts GPU memory roughly in half and speeds up training ~2× on Ampere+ GPUs. Use `torch.amp` (PyTorch ≥ 1.6).

```python
from torch.amp import autocast, GradScaler

scaler = GradScaler('cuda')

for x, y in loader:
    x, y = x.to(device), y.to(device)
    optimizer.zero_grad()
    with autocast('cuda'):                   # fp16 forward pass
        logits = model(x)
        loss = criterion(logits, y)
    scaler.scale(loss).backward()            # scale loss to avoid underflow
    scaler.unscale_(optimizer)
    torch.nn.utils.clip_grad_norm_(model.parameters(), 1.0)
    scaler.step(optimizer)
    scaler.update()
```

### Gradient Accumulation

Simulate larger effective batch sizes on memory-constrained hardware.

```python
ACCUM_STEPS = 4   # effective_batch = batch_size * ACCUM_STEPS

optimizer.zero_grad()
for step, (x, y) in enumerate(loader):
    x, y = x.to(device), y.to(device)
    with autocast('cuda'):
        loss = criterion(model(x), y) / ACCUM_STEPS  # normalise
    scaler.scale(loss).backward()
    if (step + 1) % ACCUM_STEPS == 0:
        scaler.unscale_(optimizer)
        torch.nn.utils.clip_grad_norm_(model.parameters(), 1.0)
        scaler.step(optimizer)
        scaler.update()
        optimizer.zero_grad()
```

### `torch.compile` (PyTorch 2+)

One-line speedup via Triton kernel fusion; 10–40% faster on modern GPUs.

```python
model = torch.compile(model)                  # default: "default" backend
model = torch.compile(model, mode='reduce-overhead')  # trades compile time for speed
model = torch.compile(model, backend='inductor', dynamic=True)  # dynamic shapes
```

**Gotchas:** first forward pass incurs compilation overhead; not all models compile cleanly (check for graph breaks with `torch._dynamo.explain(model)(inputs)`).

### Gradient Checkpointing

Trade compute for memory: recompute activations during backward instead of storing them.

```python
from torch.utils.checkpoint import checkpoint

class BigModel(nn.Module):
    def forward(self, x):
        # recomputes block1 during backward — saves activation memory
        x = checkpoint(self.block1, x, use_reentrant=False)
        x = self.block2(x)
        return x
```

### Learning-Rate Schedulers

```python
# Cosine annealing with warm restarts
scheduler = torch.optim.lr_scheduler.CosineAnnealingWarmRestarts(optimizer, T_0=10)

# Linear warmup then cosine — common for transformers
from transformers import get_cosine_schedule_with_warmup
scheduler = get_cosine_schedule_with_warmup(optimizer,
    num_warmup_steps=100, num_training_steps=total_steps)

# Step scheduler at end of epoch
scheduler.step()
```

### Debugging Utilities

```python
# Count parameters
total = sum(p.numel() for p in model.parameters())
trainable = sum(p.numel() for p in model.parameters() if p.requires_grad)

# Gradient norms — watch for vanishing/exploding gradients
total_norm = sum(p.grad.data.norm(2).item() ** 2
                 for p in model.parameters() if p.grad is not None) ** 0.5

# Profile a forward pass
with torch.profiler.profile(activities=[torch.profiler.ProfilerActivity.CPU,
                                        torch.profiler.ProfilerActivity.CUDA]) as prof:
    model(x)
print(prof.key_averages().table(sort_by='cuda_time_total', row_limit=10))
```

## Trade-offs

| Pattern | Pro | Con |
|---------|-----|-----|
| `pin_memory=True` in DataLoader | ~2× faster CPU→GPU transfer | Uses more host RAM |
| AMP (fp16) | 2× memory reduction, faster matmuls | Numerical instability on some models |
| `torch.compile` | 10–40% speedup, no code change | First-step latency; dynamic shapes tricky |
| Gradient checkpointing | Large model fits in GPU | ~30% extra forward pass compute |
| Gradient accumulation | Large effective batch | More steps per epoch; sync overhead |

## References

- [PyTorch documentation](https://pytorch.org/docs/stable/)
- [torch.amp (Automatic Mixed Precision)](https://pytorch.org/docs/stable/amp.html)
- [torch.compile](https://pytorch.org/docs/stable/torch.compiler.html)
- [torch.utils.checkpoint](https://pytorch.org/docs/stable/checkpoint.html)

## Links
- [[deep_learning_frameworks|Deep Learning Frameworks]]
- [[framework_comparison|Framework Comparison]]
- [[huggingface_usage|HuggingFace Usage]]
- [[04_ml_engineering/04_model_development/distributed_training|Distributed Training]]
- [[05_ai_engineering/04_finetuning/peft_and_lora|PEFT and LoRA]]
