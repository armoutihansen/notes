---
layer: 04_ml_engineering
type: engineering
tool: pytorch
status: growing
tags: [distributed-training, ddp, deepspeed, fsdp, accelerate, gpu]
created: 2026-03-05
---

# Distributed Training

## Purpose

Distributed training spreads the computational work of training a machine learning model across multiple devices (GPUs, TPUs, nodes) to reduce wall-clock training time or to accommodate models and datasets that cannot fit on a single device. It is essential for training large language models, large vision models, and any workload where single-GPU training time exceeds practical limits. Choosing the right parallelism strategy depends on whether the bottleneck is compute throughput, memory, or communication bandwidth.

## Architecture

### When to Use Distributed Training

- **Dataset too large**: even if the model fits on one GPU, training on a large dataset takes too long without parallelism.
- **Model too large**: models with billions of parameters exceed the memory of any single GPU (e.g., a 70B parameter model in BF16 requires ~140 GB of GPU VRAM).
- **Iteration speed**: research workflows where fast feedback loops on large-scale experiments are required.
- Rule of thumb: single-GPU training >24 hours is a candidate for distribution; >72 hours is a strong candidate.

### Data Parallelism — PyTorch DDP

**Distributed Data Parallel (DDP)** replicates the full model on each GPU and assigns each GPU a different shard of the mini-batch. After the forward and backward passes, gradients are aggregated across all GPUs using an all-reduce operation (typically ring-all-reduce), and all replicas update their weights identically.

```python
# Launch with: torchrun --nproc_per_node=4 train.py
from torch.nn.parallel import DistributedDataParallel as DDP
import torch.distributed as dist

dist.init_process_group("nccl")
model = DDP(model.to(local_rank), device_ids=[local_rank])
```

DDP is the standard approach when the model fits on a single GPU. It scales near-linearly with the number of GPUs for compute-bound workloads. The communication overhead is proportional to the number of parameters; large models with slow inter-GPU links can saturate the interconnect.

### Model Parallelism

When the model is too large to fit on one GPU, parameters must be distributed:

**Tensor parallelism**: individual weight matrices are split across GPUs along a tensor dimension. Each GPU computes a partial matrix product and results are synchronized via all-reduce. Used in Megatron-LM for transformer attention and MLP layers. Requires high-bandwidth GPU interconnects (NVLink) because all-reduce operations occur within every forward pass.

**Pipeline parallelism**: the model is partitioned into sequential stages, each assigned to a different GPU. Micro-batches flow through the pipeline; GPUs run concurrently on different micro-batches (GPipe, PipeDream). Reduces memory per GPU but introduces pipeline bubbles (idle GPU time) at the start and end of each batch.

### ZeRO Optimization (DeepSpeed)

ZeRO (Zero Redundancy Optimizer) eliminates the redundant state maintained across data-parallel replicas. There are three stages:

| Stage | What is sharded |
|---|---|
| ZeRO-1 | Optimizer states (e.g., Adam momentum and variance) |
| ZeRO-2 | + Gradients |
| ZeRO-3 | + Model parameters |

ZeRO-3 achieves near-linear memory reduction with the number of GPUs. A 175B model that requires 1.2 TB of GPU memory with naive data parallelism can be trained with ZeRO-3 across 8×80 GB A100s (640 GB aggregate, with parameters gathered on demand). DeepSpeed is the reference implementation; configure via a JSON config file:

```json
{
  "zero_optimization": { "stage": 3 },
  "bf16": { "enabled": true }
}
```

### HuggingFace Accelerate

Accelerate provides a unified high-level API that abstracts over DDP, DeepSpeed ZeRO, and FSDP. The same training script runs on a single GPU, multi-GPU, or multi-node by changing the launch configuration:

```python
from accelerate import Accelerator
accelerator = Accelerator()
model, optimizer, dataloader = accelerator.prepare(model, optimizer, dataloader)
# training loop unchanged
loss.backward() → accelerator.backward(loss)
```

`accelerate config` generates a configuration file; `accelerate launch train.py` handles process spawning. Accelerate is the recommended starting point for new projects: it avoids lock-in to a specific backend.

### FSDP — PyTorch Native Fully Sharded Data Parallel

FSDP (introduced in PyTorch 1.11, stable in 1.12+) is PyTorch's native implementation of ZeRO-3-style sharding. Parameters, gradients, and optimizer states are sharded across GPUs; parameters are gathered for computation and immediately discarded. FSDP integrates natively with `torch.compile` and supports mixed precision via `MixedPrecision` policies. Preferred over DeepSpeed when staying within the PyTorch ecosystem.

## Implementation Notes

- **Batch size scaling**: with $N$ GPUs and local batch size $B$, the effective batch size is $N \times B$. Large batch training requires adjusting the learning rate — the **linear scaling rule** (Goyal et al.) suggests multiplying the learning rate by $N$, with a gradual warmup period.
- **Gradient accumulation**: simulates a larger effective batch size without requiring more GPUs. Accumulate gradients over $k$ steps before calling `optimizer.step()`. Effective batch = local batch × $k$ × $N$.
- **Learning rate warmup**: essential for large-batch training and large models. A linear warmup over 1–5% of total steps stabilizes early training.
- **Gradient clipping**: `torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)` prevents gradient explosions that are more likely with large models and large batches.
- **Communication backends**: use NCCL for GPU-to-GPU communication on NVIDIA hardware; Gloo for CPU or testing.

## Trade-offs

Data parallelism is simple and scales well but requires the model to fit on one device. Model parallelism reduces per-device memory but adds implementation complexity and communication overhead. ZeRO/FSDP offer a middle ground — data-parallel semantics with model-parallel memory savings — at the cost of increased all-gather/reduce-scatter communication. The right choice depends on model size, hardware topology (NVLink vs. Ethernet), and engineering capacity. For most practitioners, starting with Accelerate + ZeRO-2 or FSDP covers the majority of use cases with minimal custom code.

## References

- PyTorch DDP tutorial: pytorch.org/tutorials/intermediate/ddp_tutorial.html
- DeepSpeed: deepspeed.ai
- ZeRO paper: Rajbhandari et al., "ZeRO: Memory Optimizations Toward Training Trillion Parameter Models" (SC 2020)
- HuggingFace Accelerate: huggingface.co/docs/accelerate
- Goyal et al., "Accurate, Large Minibatch SGD: Training ImageNet in 1 Hour" (2017)

## Links
- [[experiment_tracking|Experiment Tracking]]
- [[offline_evaluation|Offline Evaluation]]
- [[ml_lifecycle|ML Lifecycle]]
- [[regularization|Regularization]]
- [[ml_platform_architecture|ML Platform Architecture]]
- [[model_compression|Model Compression for Deployment]]
