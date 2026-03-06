---
layer: 06_applications
type: application
status: growing
tags: [pytorch, neural-networks, mlp, cnn, rnn, training-loop, early-stopping]
created: 2026-05-10
---

# Neural Network Implementation (PyTorch)

## Purpose

Practical reference for implementing and training feedforward, convolutional, and recurrent neural networks with PyTorch. Covers a complete training loop with early stopping, a validation loop, loss curve logging, and model serialization — the patterns that appear in every PyTorch project regardless of architecture.

### Examples

**Tabular classification**: MLP with BatchNorm and Dropout for structured tabular data.

**Image classification**: CNN encoder with fully-connected classification head.

**Sequence modeling**: LSTM applied to time-series or NLP token sequences.

---

## Architecture

**Standard training workflow:**

```
Dataset → DataLoader → Model (nn.Module)
         ↑                      ↓
    Transforms            Forward pass
                               ↓
                          Loss function
                               ↓
                          Backward pass
                               ↓
                         Optimizer.step()
                               ↓
                   Validation loop (no_grad)
                               ↓
                   Checkpoint best model
```

---

## MLP with Full Training Loop

```python
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader, TensorDataset, random_split

# --- Model definition ---
class MLP(nn.Module):
    def __init__(self, input_size: int, hidden_sizes: list[int], num_classes: int, dropout: float = 0.3):
        super().__init__()
        layers = []
        in_size = input_size
        for h in hidden_sizes:
            layers += [nn.Linear(in_size, h), nn.BatchNorm1d(h), nn.ReLU(), nn.Dropout(dropout)]
            in_size = h
        layers.append(nn.Linear(in_size, num_classes))
        self.net = nn.Sequential(*layers)

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        return self.net(x)

# --- Dataset setup ---
DEVICE = torch.device("cuda" if torch.cuda.is_available() else "cpu")
BATCH_SIZE, LR, N_EPOCHS, PATIENCE = 64, 1e-3, 50, 7

X = torch.randn(5000, 20)   # replace with real features
y = torch.randint(0, 3, (5000,))
dataset = TensorDataset(X, y)
n_train = int(0.8 * len(dataset))
train_ds, val_ds = random_split(dataset, [n_train, len(dataset) - n_train])
train_loader = DataLoader(train_ds, batch_size=BATCH_SIZE, shuffle=True)
val_loader   = DataLoader(val_ds,   batch_size=BATCH_SIZE)

# --- Model / optimizer / scheduler ---
model     = MLP(20, [128, 64], 3, dropout=0.3).to(DEVICE)
criterion = nn.CrossEntropyLoss()
optimizer = optim.AdamW(model.parameters(), lr=LR, weight_decay=1e-2)
scheduler = optim.lr_scheduler.CosineAnnealingLR(optimizer, T_max=N_EPOCHS)

# --- Training loop with early stopping ---
best_val_loss, no_improve, train_losses, val_losses = float("inf"), 0, [], []

for epoch in range(N_EPOCHS):
    # Train
    model.train()
    epoch_loss = 0.0
    for X_b, y_b in train_loader:
        X_b, y_b = X_b.to(DEVICE), y_b.to(DEVICE)
        optimizer.zero_grad()
        loss = criterion(model(X_b), y_b)
        loss.backward()
        nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)
        optimizer.step()
        epoch_loss += loss.item()
    train_losses.append(epoch_loss / len(train_loader))

    # Validate
    model.eval()
    val_loss = 0.0
    with torch.no_grad():
        for X_b, y_b in val_loader:
            X_b, y_b = X_b.to(DEVICE), y_b.to(DEVICE)
            val_loss += criterion(model(X_b), y_b).item()
    val_loss /= len(val_loader)
    val_losses.append(val_loss)

    scheduler.step()

    # Early stopping + best checkpoint
    if val_loss < best_val_loss:
        best_val_loss = val_loss
        no_improve = 0
        torch.save({"epoch": epoch, "model_state": model.state_dict(),
                    "optimizer_state": optimizer.state_dict(), "val_loss": val_loss},
                   "best_model.pt")
    else:
        no_improve += 1
        if no_improve >= PATIENCE:
            print(f"Early stopping at epoch {epoch}")
            break

# --- Loss curve logging (matplotlib) ---
import matplotlib.pyplot as plt
plt.plot(train_losses, label="train"); plt.plot(val_losses, label="val")
plt.xlabel("Epoch"); plt.ylabel("Loss"); plt.legend(); plt.savefig("loss_curve.png")

# --- Load best model for inference ---
ckpt = torch.load("best_model.pt", weights_only=True)
model.load_state_dict(ckpt["model_state"])
model.eval()
```

---

## CNN for Image Classification

```python
class CNN(nn.Module):
    def __init__(self, in_channels: int = 3, num_classes: int = 10):
        super().__init__()
        self.features = nn.Sequential(
            nn.Conv2d(in_channels, 32, kernel_size=3, padding=1), nn.ReLU(), nn.MaxPool2d(2),
            nn.Conv2d(32, 64, kernel_size=3, padding=1),           nn.ReLU(), nn.MaxPool2d(2),
            nn.Conv2d(64, 128, kernel_size=3, padding=1),          nn.ReLU(), nn.AdaptiveAvgPool2d((4, 4)),
        )
        self.classifier = nn.Sequential(
            nn.Flatten(),
            nn.Linear(128 * 4 * 4, 256), nn.ReLU(), nn.Dropout(0.4),
            nn.Linear(256, num_classes),
        )

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        return self.classifier(self.features(x))

# Usage with torchvision datasets:
from torchvision import datasets, transforms
transform = transforms.Compose([transforms.ToTensor(), transforms.Normalize((0.5,), (0.5,))])
train_data = datasets.CIFAR10(root="./data", train=True, download=True, transform=transform)
train_loader = DataLoader(train_data, batch_size=64, shuffle=True, num_workers=4)
```

---

## LSTM for Sequence Modeling

```python
class LSTMClassifier(nn.Module):
    def __init__(self, input_size: int, hidden_size: int, num_layers: int, num_classes: int, dropout: float = 0.3):
        super().__init__()
        self.lstm = nn.LSTM(input_size, hidden_size, num_layers,
                            batch_first=True, dropout=dropout if num_layers > 1 else 0.0)
        self.fc = nn.Linear(hidden_size, num_classes)

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        # x: (batch, seq_len, input_size)
        _, (h_n, _) = self.lstm(x)
        return self.fc(h_n[-1])  # use last layer hidden state

# Synthetic time-series example
model_lstm = LSTMClassifier(input_size=10, hidden_size=64, num_layers=2, num_classes=3).to(DEVICE)
x_seq = torch.randn(32, 20, 10).to(DEVICE)   # batch=32, seq=20, features=10
out   = model_lstm(x_seq)                     # (32, 3)
```

---

## Model Serialization

```python
# Preferred: save/load state dict only
torch.save(model.state_dict(), "model_weights.pt")
model.load_state_dict(torch.load("model_weights.pt", weights_only=True))

# Full checkpoint (for resuming training)
torch.save({
    "epoch": epoch,
    "model_state_dict": model.state_dict(),
    "optimizer_state_dict": optimizer.state_dict(),
    "val_loss": best_val_loss,
}, "checkpoint.pt")

# Resume
ckpt = torch.load("checkpoint.pt", weights_only=False)
model.load_state_dict(ckpt["model_state_dict"])
optimizer.load_state_dict(ckpt["optimizer_state_dict"])
start_epoch = ckpt["epoch"] + 1
```

---

## Key Configuration Choices

| Concern | Recommendation |
|---|---|
| Optimizer | AdamW (L2 via `weight_decay`) for most tasks; SGD+momentum for large-batch CV |
| Learning rate | 1e-3 for Adam; 0.1 for SGD; always use a scheduler |
| Scheduler | CosineAnnealingLR for fixed-budget; ReduceLROnPlateau for dynamic |
| Gradient clipping | `clip_grad_norm_(params, 1.0)` — prevents exploding gradients |
| Batch norm | Use after Linear/Conv, before activation; disable eval mode noise with `model.eval()` |
| Dropout | 0.1–0.3 in hidden layers; 0.4–0.5 before final classifier |
| Early stopping | patience=5–10 epochs on val loss; restore best checkpoint |

---

## Links

**Foundations**
- [[01_foundations/06_deep_learning_theory/backpropagation|Backpropagation]] — gradient flow through computation graph
- [[01_foundations/06_deep_learning_theory/batch_normalization|Batch Normalization]] — normalising layer activations
- [[01_foundations/06_deep_learning_theory/dropout|Dropout]] — regularisation via random unit masking
- [[01_foundations/06_deep_learning_theory/weight_initialization|Weight Initialization]] — Xavier / Kaiming init
- [[01_foundations/06_deep_learning_theory/adaptive_optimizers|Adaptive Optimizers]] — Adam, AdamW, RMSProp

**Modeling**
- [[02_modeling/03_model_families/04_neural_networks/mpl|Multi-Layer Perceptron]] — MLP architecture theory
- [[02_modeling/03_model_families/04_neural_networks/cnn_architecture|CNN Architecture]] — convolutional feature learning
- [[02_modeling/03_model_families/04_neural_networks/recurrent_networks|Recurrent Networks]] — LSTM / GRU theory

**Applications**
- [[deep_learning_training_workflow|Deep Learning Training Workflow]] — end-to-end PyTorch + Accelerate + MLflow pipeline
- [[distributed_training_with_accelerate|Distributed Training with Accelerate]] — multi-GPU training wrapper
