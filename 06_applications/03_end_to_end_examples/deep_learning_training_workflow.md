---
layer: 06_applications
type: application
status: growing
tags: [workflow, training, distributed]
created: 2026-05-10
---

# Deep Learning Training Workflow

## Purpose

An end-to-end PyTorch training workflow at scale: local single-GPU development → multi-GPU distributed training with HuggingFace Accelerate → MLflow experiment tracking → checkpoint → evaluation → model registry. This pattern applies to any PyTorch model (MLP, CNN, transformer) and scales from a laptop to a cluster without code changes.

### Examples

**Image classifier**: Train a ResNet-50 on custom product images; scale to 4×A100 with Accelerate; track experiments with MLflow; register the best checkpoint.

**Custom transformer**: Pre-train a small language model on domain-specific text using multi-node distributed training; push to HuggingFace Hub.

---

## Architecture

```
PyTorch model (nn.Module)
    │
    ├──[1] Local dev: single GPU, small dataset, debug loop
    │
    ├──[2] Accelerate: wrap model + optimizer + dataloader → multi-GPU
    │       ├── DDP (default)
    │       ├── DeepSpeed ZeRO-2/3 (large models)
    │       └── FSDP (very large models)
    │
    ├──[3] MLflow: log params, metrics, artifacts per epoch
    │       └── Checkpoint best model (val loss)
    │
    ├──[4] Evaluation: held-out test set metrics (accuracy, F1, perplexity)
    │
    └──[5] Model registry: promote champion; load for downstream serving
```

---

## Step 1: Model Definition (nn.Module)

```python
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader, TensorDataset

class ResidualMLP(nn.Module):
    """MLP with residual connections — suitable for tabular/sequence tasks."""
    def __init__(self, input_dim: int, hidden_dim: int, num_classes: int, depth: int = 4, dropout: float = 0.2):
        super().__init__()
        self.input_proj = nn.Linear(input_dim, hidden_dim)
        self.layers = nn.ModuleList([
            nn.Sequential(
                nn.Linear(hidden_dim, hidden_dim),
                nn.LayerNorm(hidden_dim),
                nn.GELU(),
                nn.Dropout(dropout),
            )
            for _ in range(depth)
        ])
        self.head = nn.Linear(hidden_dim, num_classes)

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        h = self.input_proj(x)
        for layer in self.layers:
            h = h + layer(h)    # residual: output = input + f(input)
        return self.head(h)
```

---

## Step 2: Accelerate — Multi-GPU with 4 Lines

```python
from accelerate import Accelerator, set_seed

set_seed(42)

# initialise — auto-detects available hardware (CPU, single GPU, multi-GPU, DDP, FSDP)
accelerator = Accelerator(
    mixed_precision="bf16",            # BF16 for modern GPUs; "fp16" for older
    gradient_accumulation_steps=4,     # effective batch = batch_size × n_gpus × 4
)

# Data
X = torch.randn(10000, 128)
y = torch.randint(0, 10, (10000,))
dataset  = TensorDataset(X, y)
loader   = DataLoader(dataset, batch_size=64, shuffle=True)

# Model + optimizer
model     = ResidualMLP(input_dim=128, hidden_dim=256, num_classes=10)
optimizer = optim.AdamW(model.parameters(), lr=3e-4, weight_decay=1e-2)
scheduler = optim.lr_scheduler.CosineAnnealingLR(optimizer, T_max=20)
criterion = nn.CrossEntropyLoss()

# Wrap with Accelerate — handles device placement, DDP wrapping, mixed precision
model, optimizer, loader = accelerator.prepare(model, optimizer, loader)
```

---

## Step 3: Training Loop + MLflow Tracking

```python
import mlflow

mlflow.set_experiment("residual-mlp-v1")

best_val_loss, N_EPOCHS, PATIENCE = float("inf"), 20, 5
no_improve = 0

with mlflow.start_run(run_name="bf16-accelerate-4gpu"):
    mlflow.log_params({
        "hidden_dim": 256, "depth": 4, "dropout": 0.2,
        "lr": 3e-4, "batch_size": 64, "epochs": N_EPOCHS,
        "mixed_precision": "bf16", "gradient_accumulation": 4,
    })

    for epoch in range(N_EPOCHS):
        # --- Train ---
        model.train()
        total_loss, n_correct, n_total = 0.0, 0, 0
        for X_b, y_b in loader:
            with accelerator.accumulate(model):
                logits = model(X_b)
                loss   = criterion(logits, y_b)
                accelerator.backward(loss)
                if accelerator.sync_gradients:
                    accelerator.clip_grad_norm_(model.parameters(), 1.0)
                optimizer.step()
                optimizer.zero_grad()
            total_loss += loss.item()
            n_correct  += (logits.argmax(1) == y_b).sum().item()
            n_total    += y_b.size(0)

        train_acc  = n_correct / n_total
        train_loss = total_loss / len(loader)

        # --- Validate (simplified: use same loader for demo) ---
        model.eval()
        with torch.no_grad():
            val_logits = model(X_b)
            val_loss   = criterion(val_logits, y_b).item()

        scheduler.step()

        # Log metrics only from the main process
        if accelerator.is_main_process:
            mlflow.log_metrics({"train_loss": train_loss, "train_acc": train_acc,
                                "val_loss": val_loss}, step=epoch)

        # Checkpoint best model
        if val_loss < best_val_loss:
            best_val_loss = val_loss
            no_improve    = 0
            accelerator.wait_for_everyone()
            unwrapped = accelerator.unwrap_model(model)
            accelerator.save(unwrapped.state_dict(), "best_model.pt")
        else:
            no_improve += 1
            if no_improve >= PATIENCE:
                if accelerator.is_main_process:
                    print(f"Early stopping at epoch {epoch}")
                break

    # Register best model in MLflow
    if accelerator.is_main_process:
        mlflow.log_artifact("best_model.pt")
        mlflow.pytorch.log_model(unwrapped, "model",
                                  registered_model_name="residual_mlp")
```

---

## Step 4: Evaluation

```python
# Load best checkpoint for final evaluation
from sklearn.metrics import classification_report
import numpy as np

unwrapped.load_state_dict(torch.load("best_model.pt", weights_only=True))
unwrapped.eval()

all_preds, all_labels = [], []
eval_loader = DataLoader(dataset, batch_size=512, shuffle=False)
with torch.no_grad():
    for X_b, y_b in eval_loader:
        preds = unwrapped(X_b.to(accelerator.device)).argmax(1).cpu().numpy()
        all_preds.extend(preds)
        all_labels.extend(y_b.numpy())

print(classification_report(all_labels, all_preds))
```

---

## Step 5: Launch Commands

```bash
# Configure Accelerate (interactive, one-time)
accelerate config

# Single GPU
accelerate launch train.py

# Multi-GPU (4 GPUs on one machine)
accelerate launch --num_processes 4 train.py

# DeepSpeed ZeRO-2 (for large models)
accelerate launch --config_file deepspeed_z2.yaml train.py

# Multi-node (2 machines × 4 GPUs)
accelerate launch --num_machines 2 --num_processes 8 \
  --machine_rank 0 --main_process_ip $MASTER_ADDR train.py
```

---

## Key Configuration Choices

| Concern | Recommendation |
|---|---|
| Mixed precision | BF16 for A100/H100; FP16 for older GPUs |
| Gradient accumulation | 4–8 steps to maintain effective large-batch size on limited memory |
| Gradient clipping | `clip_grad_norm_(params, 1.0)` — use `accelerator.clip_grad_norm_` |
| Checkpointing | `accelerator.unwrap_model()` before saving to get clean state dict |
| Scheduler | Step `scheduler.step()` AFTER the optimizer, once per epoch |
| DeepSpeed ZeRO | ZeRO-2 for 7B+; ZeRO-3 for 13B+ (offloads params across GPUs) |

---

## Links

**Foundations**
- [[01_foundations/06_deep_learning_theory/residual_connections|Residual Connections]] — skip connections and gradient highway theory
- [[01_foundations/06_deep_learning_theory/batch_normalization|Batch Normalization]] — stabilising activations during training
- [[01_foundations/06_deep_learning_theory/adaptive_optimizers|Adaptive Optimizers]] — AdamW, learning rate schedules

**Modeling**
- [[02_modeling/03_model_families/04_neural_networks/mpl|Multi-Layer Perceptron]] — MLP architecture
- [[02_modeling/03_model_families/04_neural_networks/recurrent_networks|Recurrent Networks]] — LSTM/GRU for sequence tasks

**Model Implementations**
- [[neural_network_implementation|Neural Network Implementation]] — standalone PyTorch training loop (without Accelerate)

**System Patterns**
- [[distributed_training_with_accelerate|Distributed Training with Accelerate]] — Accelerate config, ZeRO, FSDP setup
- [[mlflow_experiment_tracking|MLflow Experiment Tracking]] — MLflow registry and artifact tracking
