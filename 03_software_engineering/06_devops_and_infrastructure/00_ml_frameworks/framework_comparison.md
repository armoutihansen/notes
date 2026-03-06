---
layer: 03_software_engineering
type: engineering
tool: PyTorch/TF/JAX
status: growing
tags: [pytorch, tensorflow, jax, deep-learning, framework-selection]
created: 2026-03-05
---

# ML Framework Comparison

## Purpose

Choosing the right deep learning framework affects research velocity, deployment options, and ecosystem access. The three dominant frameworks — **PyTorch**, **TensorFlow 2 / Keras**, and **JAX** — share the same core abstraction (tensors + automatic differentiation) but differ in programming model, ecosystem, and deployment story. See [[deep_learning_frameworks|Deep Learning Frameworks]] for detailed API usage patterns.

## Architecture

All frameworks implement reverse-mode automatic differentiation over a computation graph:

| Dimension | PyTorch | TensorFlow 2 / Keras | JAX |
|-----------|---------|----------------------|-----|
| **Graph model** | Dynamic (define-by-run) | Eager + `@tf.function` static graph | Functional; JIT via `jax.jit` |
| **Primary abstraction** | `nn.Module` (stateful) | `keras.Model` (stateful) | Pure functions + pytrees (stateless) |
| **Autodiff engine** | `torch.autograd` | `GradientTape` | `jax.grad` / `jax.value_and_grad` |
| **Parallelism** | DDP / FSDP (via Accelerate) | `MirroredStrategy` / `tf.distribute` | `jax.pmap` / `jax.vmap` |
| **Compilation** | `torch.compile` (Triton/Inductor) | `@tf.function` → XLA | `jax.jit` → XLA |
| **Deployment** | TorchScript / ONNX / TorchServe | SavedModel / TFLite / TF Serving | XLA-compiled HLO |
| **Ecosystem** | Largest (HuggingFace, timm, Lightning) | Large (TFHub, TFX) | Growing (Flax, Optax, Equinox) |

## Implementation Notes

### Choosing a Framework

**Default to PyTorch when:**
- Starting a new research or production project
- Consuming HuggingFace models, datasets, or tokenizers
- Building anything in the LLM ecosystem (PEFT, TRL, vLLM, Axolotl all assume PyTorch)
- Needing the widest range of third-party libraries (timm, detectron2, diffusers)

**Choose TensorFlow 2 / Keras when:**
- Deploying to mobile/edge via TFLite
- TF Serving is already in production infrastructure
- Using Cloud TPUs via Google Cloud's TPU runtime
- Inheriting a large existing TF codebase

**Choose JAX when:**
- Custom autodiff is required (higher-order gradients, custom VJPs)
- Running TPU-heavy research workloads (XLA shines on TPU)
- Preferring functional programming style — JAX forces explicit state management
- Working at a research lab with heavy JAX investment (Google DeepMind)

### Side-by-Side: Training Loop

```python
# --- PyTorch ---
model = MLP().to(device)
opt = torch.optim.Adam(model.parameters(), lr=1e-3)
for x, y in loader:
    opt.zero_grad()
    loss = criterion(model(x.to(device)), y.to(device))
    loss.backward()
    opt.step()

# --- Keras ---
model.compile(optimizer='adam', loss='sparse_categorical_crossentropy')
model.fit(train_ds, epochs=10)

# --- JAX + Optax ---
params = model.init(rng, dummy_input)
tx = optax.adam(1e-3)
opt_state = tx.init(params)

@jax.jit
def train_step(params, opt_state, x, y):
    loss, grads = jax.value_and_grad(loss_fn)(params, x, y)
    updates, opt_state = tx.update(grads, opt_state)
    params = optax.apply_updates(params, updates)
    return params, opt_state, loss
```

### Interoperability

- **ONNX** is the lingua franca for cross-framework export: `torch.onnx.export` → deploy with ONNX Runtime on CPU/GPU/edge
- **HuggingFace Transformers** supports both `pt` and `tf` backends; JAX/Flax via `from_pretrained(..., from_flax=True)`
- **torchvision / timm** models can be wrapped and exported to ONNX for TF or other runtimes

## Trade-offs

| Criterion | PyTorch | TensorFlow 2 | JAX |
|-----------|---------|--------------|-----|
| Debugging ease | ✅ Native Python debugger | ⚠️ `@tf.function` traces obscure errors | ⚠️ `jax.debug.print` needed |
| Research flexibility | ✅ Industry standard | ✅ Good | ✅ Best for custom autodiff |
| Production story | ✅ TorchScript, TorchServe | ✅ TF Serving, TFLite | ⚠️ Less mature |
| TPU performance | ⚠️ Limited XLA | ✅ Strong | ✅ Best-in-class |
| LLM ecosystem | ✅ Dominant | ⚠️ Marginal | ⚠️ Marginal |

## References

- [PyTorch documentation](https://pytorch.org/docs/stable/)
- [TensorFlow documentation](https://www.tensorflow.org/api_docs)
- [JAX documentation](https://jax.readthedocs.io/)
- [Flax neural network library](https://flax.readthedocs.io/)
- [Optax optimizer library for JAX](https://optax.readthedocs.io/)

## Links
- [[deep_learning_frameworks|Deep Learning Frameworks]]
- [[pytorch_patterns|PyTorch Patterns]]
- [[huggingface_usage|HuggingFace Usage]]
- [[04_ml_engineering/04_model_development/distributed_training|Distributed Training]]
- [[05_ai_engineering/04_finetuning/peft_and_lora|PEFT and LoRA]]
