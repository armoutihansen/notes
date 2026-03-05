---
layer: 03_software_engineering
type: engineering
tool: pytorch
status: seed
tags:
  - deep_learning
  - pytorch
  - tensorflow
  - jax
created: 2026-03-02
---

# Deep Learning Frameworks

## Purpose

Deep learning frameworks provide:
- **Automatic differentiation** — compute gradients of any computation without manual derivation
- **Hardware abstraction** — run the same code on CPU, GPU, or TPU
- **Optimized primitives** — BLAS, cuDNN, and XLA kernels for tensor operations
- **Ecosystem** — dataloaders, model zoos, deployment tooling

The three dominant frameworks are **PyTorch**, **TensorFlow/Keras**, and **JAX**.

## Architecture

All frameworks build on the same core abstraction: a **tensor** (n-dimensional array on a device) + a **computation graph** that records operations for gradient computation via reverse-mode autodiff.

| | PyTorch | TensorFlow 2 / Keras | JAX |
|-|---------|----------------------|-----|
| Graph style | Dynamic (eager by default) | Eager + `@tf.function` static | Functional, JIT via `jit` |
| Autodiff | `autograd` via `.backward()` | `GradientTape` | `jax.grad`, `jax.value_and_grad` |
| Primary abstraction | `nn.Module` | `keras.Model` | Pure functions + `pytree` |
| Parallelism | `DataParallel`, `DistributedDataParallel` | `MirroredStrategy` | `jax.pmap`, `jax.vmap` |
| Deployment | TorchScript, ONNX | SavedModel, TFLite, TF Serving | XLA-compiled HLO |

**Dynamic vs static graphs:** PyTorch builds the graph on-the-fly each forward pass (easy to debug with standard Python tools); TensorFlow's `@tf.function` and JAX's `jit` compile a static graph for performance at the cost of tracing overhead and constraints on Python control flow.

## Implementation Notes

### PyTorch — core patterns

**Define a model:**
```python
import torch
import torch.nn as nn

class MLP(nn.Module):
    def __init__(self, in_dim, hidden, out_dim):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(in_dim, hidden),
            nn.ReLU(),
            nn.Linear(hidden, out_dim),
        )

    def forward(self, x):
        return self.net(x)
```

**Standard training loop:**
```python
model = MLP(784, 256, 10).to(device)
optimizer = torch.optim.Adam(model.parameters(), lr=1e-3)
loss_fn = nn.CrossEntropyLoss()

for epoch in range(num_epochs):
    for x, y in dataloader:
        x, y = x.to(device), y.to(device)
        optimizer.zero_grad()
        logits = model(x)
        loss = loss_fn(logits, y)
        loss.backward()          # populate .grad on all parameters
        optimizer.step()         # update parameters
```

**Key APIs:**

| Task | API |
|------|-----|
| Move to GPU | `tensor.to(device)` / `model.to(device)` |
| Disable gradient tracking | `torch.no_grad()` context manager |
| Save / load checkpoint | `torch.save(model.state_dict(), path)` / `model.load_state_dict(...)` |
| Freeze layers | `param.requires_grad = False` |
| Inspect parameter count | `sum(p.numel() for p in model.parameters())` |
| Data pipeline | `Dataset` + `DataLoader` with `num_workers` |
| Compile for speed | `torch.compile(model)` (PyTorch 2+, wraps Triton/Inductor) |

**Gradient utilities:**
```python
# Gradient clipping (important for RNNs)
torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)

# Manual gradient inspection
for name, param in model.named_parameters():
    if param.grad is not None:
        print(name, param.grad.norm())
```

### TensorFlow / Keras — core patterns

```python
model = tf.keras.Sequential([
    tf.keras.layers.Dense(256, activation='relu'),
    tf.keras.layers.Dense(10),
])
model.compile(optimizer='adam', loss='sparse_categorical_crossentropy', metrics=['accuracy'])
model.fit(train_ds, epochs=10, validation_data=val_ds)
```

Use `GradientTape` for custom training loops:
```python
with tf.GradientTape() as tape:
    logits = model(x, training=True)
    loss = loss_fn(y, logits)
grads = tape.gradient(loss, model.trainable_variables)
optimizer.apply_gradients(zip(grads, model.trainable_variables))
```

### JAX — core patterns

JAX is functional: models are plain Python functions; state is explicit.
```python
import jax
import jax.numpy as jnp

def loss_fn(params, x, y):
    logits = model.apply(params, x)   # Flax / Haiku style
    return cross_entropy(logits, y)

grad_fn = jax.value_and_grad(loss_fn)
loss, grads = grad_fn(params, x, y)
params = jax.tree_util.tree_map(lambda p, g: p - lr * g, params, grads)
```

Use `jax.jit` for compilation, `jax.vmap` for batching, `jax.pmap` for multi-device.

## Trade-offs

| Criterion | PyTorch | TensorFlow 2 | JAX |
|-----------|---------|--------------|-----|
| Ease of debugging | ✅ Native Python | ⚠️ Graph tracing errors can be opaque | ⚠️ Functional style has a learning curve |
| Research flexibility | ✅ Industry standard | ✅ Good | ✅ Best for custom autodiff |
| Production deployment | ✅ TorchScript/ONNX | ✅ TF Serving, TFLite | ⚠️ Less mature tooling |
| Performance | ✅ `torch.compile` competitive | ✅ XLA strong | ✅ XLA best-in-class on TPU |
| Ecosystem / model zoos | ✅ Largest (HuggingFace, timm) | ✅ Large | ⚠️ Smaller but growing (Flax, Optax) |
| Multi-device parallelism | ✅ DDP mature | ✅ Strategy API | ✅ `pmap` elegant |

**Framework selection heuristic:**
- Default to **PyTorch** — largest research community, HuggingFace ecosystem, strong production story
- Use **TensorFlow/Keras** when deploying to mobile/edge (TFLite) or when TF Serving is already in production
- Use **JAX** for research requiring custom autodiff, TPU-heavy workloads, or functional programming style (e.g., DeepMind, Google Brain work)

## References

- [PyTorch documentation](https://pytorch.org/docs/stable/)
- [PyTorch 2 `torch.compile`](https://pytorch.org/docs/stable/torch.compiler.html)
- [TensorFlow documentation](https://www.tensorflow.org/api_docs)
- [JAX documentation](https://jax.readthedocs.io/)
- [Flax (JAX neural network library)](https://flax.readthedocs.io/)

## Links
- [[01_foundations/_legacy/deep_learning_theory 1/backpropagation]] (in 01_foundations/deep_learning_theory)
- [[01_foundations/_legacy/deep_learning_theory 1/gradient_descent]] (in 01_foundations/deep_learning_theory)
- [[02_modeling/_legacy/deep_learning/regularization]] (in 02_modeling/deep_learning)
