---
layer: 05_ai_engineering
type: engineering
tool: general
status: evergreen
tags: [attention, flash-attention, kv-cache, inference, transformer, long-context]
created: 2026-03-05
---
# Attention Optimization and KV Cache

## Purpose
Reducing attention computation complexity and memory requirements for long-context inference. Standard scaled dot-product attention is O(n²) in both time and memory with respect to sequence length n, creating a hard wall for long-context applications. Flash Attention rewrites the algorithm for modern GPU memory hierarchies to maintain exact results while dramatically reducing HBM traffic; KV caching eliminates redundant computation during autoregressive generation. Together they define the baseline for any production transformer inference stack.

## Architecture
**Standard attention bottleneck:**
Attention requires materializing the full n×n score matrix in GPU HBM (high-bandwidth memory). For n=32k tokens and d=128 head dimension, this is ~4 GB per attention head in FP32. The compute is O(n²·d) FLOPs and memory is O(n²), both intractable at long contexts.

**Flash Attention (Dao et al., 2022):**
IO-aware exact attention rewrite. Partitions Q, K, V into tiles that fit in SRAM (on-chip), computes attention incrementally using the online softmax trick (maintaining running max and denominator), and writes only the final output to HBM. Never materializes the full attention matrix. Mathematical output is identical to standard attention — not an approximation.
- 2–4x wall-clock speedup vs standard PyTorch attention
- 10–20x reduction in attention memory usage
- Enables training/inference at significantly longer contexts on the same hardware

**Flash Attention 2 (Dao, 2023):**
Reduced non-matrix-multiply FLOPs, improved parallelism across sequence dimension, better utilization on A100/H100. Standard in most modern training frameworks.

**Flash Attention 3 (Shah et al., 2024):**
H100-specific optimizations exploiting warp specialization and WGMMA instructions. Further speedup on Hopper architecture.

**KV Cache:**
During autoregressive generation, each new token attends to all prior tokens. Without caching, K and V projections for past tokens are recomputed at every step — O(n²) total computation for n tokens. KV cache stores these projections after first computation, reducing per-step cost to O(n·d). Memory cost: `2 × n × d_model × n_layers × sizeof(dtype)` bytes. For Llama-3-8B (32 layers, d=4096, FP16): ~1 GB per 1k tokens.

**KV cache compression via attention variants:**
- **MHA (Multi-Head Attention):** One K, V per head — highest quality, highest KV cache cost.
- **MQA (Multi-Query Attention):** Single shared K, V across all heads (Shazeer, 2019) — minimal KV cache, some quality degradation.
- **GQA (Grouped-Query Attention):** Groups of heads share K, V (Ainslie et al., 2023). Used in Llama 2/3, Mistral, Gemma. Good quality/memory trade-off; Llama-3-8B uses 8 groups for 32 heads.

**Sliding window attention:**
Each token attends only to a window of w prior tokens. O(n·w) instead of O(n²). Used in Mistral (window=4096). Suitable when long-range dependencies are weak; fails when global context is needed.

**Long-context positional encoding:**
- **RoPE** (Su et al., 2021): Rotary position embeddings; relative positions encoded in rotation matrices. Used in Llama, Mistral, Gemma.
- **YaRN** (Peng et al., 2023): Dynamic NTK-aware RoPE scaling — extends context window 2–4x beyond training length via frequency interpolation with minimal fine-tuning (1000 steps).
- **LongRoPE:** Position interpolation with non-uniform scaling across RoPE dimensions.
- **ALiBi** (Press et al., 2021): Additive bias based on token distance rather than positional embeddings; extrapolates to longer sequences without position embedding changes.

**PagedAttention (vLLM):**
Virtual memory management for KV cache. Divides KV cache into fixed-size blocks (pages), allocates pages on demand, enables efficient memory sharing across beam search hypotheses. Eliminates fragmentation inherent in preallocating maximum-length KV tensors. Enables continuous batching across requests with different sequence lengths.

## Implementation Notes
**Flash Attention (direct library):**
```python
from flash_attn import flash_attn_qkvpacked_func, flash_attn_func
# Packed: qkv shape (batch, seqlen, 3, nheads, headdim)
out = flash_attn_qkvpacked_func(qkv, dropout_p=0.0, causal=True)
# Separate Q, K, V:
out = flash_attn_func(q, k, v, causal=True)
```

**PyTorch SDPA (recommended for portability):**
```python
import torch.nn.functional as F
# Automatically dispatches to Flash Attention if available in PyTorch >= 2.0
out = F.scaled_dot_product_attention(q, k, v, is_causal=True)
# Force Flash Attention backend:
with torch.backends.cuda.sdp_kernel(enable_flash=True, enable_math=False):
    out = F.scaled_dot_product_attention(q, k, v, is_causal=True)
```

**KV cache quantization:**
INT8 KV cache (vLLM: `--kv-cache-dtype fp8`) reduces KV memory by 2x with minimal quality impact. Useful when batch size or context length is memory-constrained.

**Enabling GQA in HuggingFace:**
Set in model config: `num_key_value_heads=8` with `num_attention_heads=32`. Supported natively in Transformers for Llama/Mistral family models.

## Trade-offs
**Flash Attention vs standard:** Strictly better on supported hardware — identical output, faster, less memory. The only cost is a non-trivial CUDA kernel dependency (`flash-attn` package). PyTorch SDPA provides a portable fallback.

**MHA vs GQA vs MQA:** GQA at 4–8 groups recovers >95% of MHA quality while reducing KV cache cost proportionally to group count. MQA is aggressive but works well for inference-optimized models (PaLM, Falcon). For most new architectures, GQA with 4–8 groups is the standard choice.

**YaRN context extension:** Enables 2–4x context window extension with minimal fine-tuning (1000 gradient steps on long-context data). Quality degrades for tokens near the extension boundary — budget additional fine-tuning for production-grade long-context capability.

**Sliding window vs full attention:** Sliding window is effective for tasks where local context dominates (code completion, chat). Fails for tasks requiring global document understanding (long-document QA, summarization of very long texts).

**KV cache budget vs throughput:** Larger KV cache allows longer sequences and more concurrent users (continuous batching). INT8/FP8 KV compression is a nearly free quality/capacity trade-off and should be enabled by default in serving.

## References
- Flash Attention: Dao et al. (2022). "FlashAttention: Fast and Memory-Efficient Exact Attention with IO-Awareness." NeurIPS 2022.
- Flash Attention 2: Dao (2023). "FlashAttention-2: Faster Attention with Better Parallelism and Work Partitioning." ICLR 2024.
- Flash Attention 3: Shah et al. (2024). "FlashAttention-3: Fast and Accurate Attention for Hopper GPUs."
- GQA: Ainslie et al. (2023). "GQA: Training Generalized Multi-Query Transformer Models from Multi-Head Checkpoints." EMNLP 2023.
- YaRN: Peng et al. (2023). "YaRN: Efficient Context Window Extension of Large Language Models."
- PagedAttention: Kwon et al. (2023). "Efficient Memory Management for Large Language Model Serving with PagedAttention." SOSP 2023.
- RoPE: Su et al. (2021). "RoFormer: Enhanced Transformer with Rotary Position Embedding."
- ALiBi: Press et al. (2021). "Train Short, Test Long: Attention with Linear Biases Enables Input Length Extrapolation." ICLR 2022.

## Links
- [[quantization_overview|Quantization for LLMs]]
- [[serving_frameworks|LLM Serving Frameworks]]
