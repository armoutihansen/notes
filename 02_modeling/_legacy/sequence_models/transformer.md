---
layer: 02_modeling
type: concept
status: seed
tags: [transformers, sequence_models, self_attention, multi_head_attention]
created: 2026-03-02
---

# Transformer

## Definition

A sequence model built entirely from self-attention and feed-forward blocks, with no recurrence; processes all tokens in parallel and connects any two positions in O(1) operations.

## Intuition

RNNs process tokens sequentially — information from early tokens must "pass through" all subsequent hidden states, degrading over long distances; self-attention connects every token to every other token directly, making long-range dependencies equally accessible; multi-head attention learns multiple attention patterns simultaneously (syntactic, semantic, positional).

## Formal Description

**Scaled dot-product attention**: $\text{Attention}(Q, K, V) = \text{softmax}(QK^\top / \sqrt{d_k})V$ where $Q \in \mathbb{R}^{T \times d_k}$, $K \in \mathbb{R}^{T \times d_k}$, $V \in \mathbb{R}^{T \times d_v}$; scaling by $\sqrt{d_k}$ prevents softmax saturation for large $d_k$.

**Self-attention**: queries, keys, and values are all linear projections of the same input sequence; $Q = XW^Q$, $K = XW^K$, $V = XW^V$; captures dependencies within the sequence.

**Multi-head attention**: $h$ parallel attention heads with separate projection matrices $W^Q_i, W^K_i, W^V_i \in \mathbb{R}^{d_\text{model} \times d_k}$; $\text{MultiHead}(Q,K,V) = \text{Concat}(\text{head}_1, \ldots, \text{head}_h)W^O$; different heads can specialize in different relation types.

**Transformer block**: LayerNorm → Multi-Head Self-Attention → residual add → LayerNorm → Feed-Forward (two linear + ReLU) → residual add; encoder: stack of $N$ such blocks; decoder: same but adds cross-attention over encoder output.

**Positional encoding**: since self-attention is permutation-equivariant, inject position information; sinusoidal: $\text{PE}(pos, 2i) = \sin(pos / 10000^{2i/d})$, $\text{PE}(pos, 2i+1) = \cos(\ldots)$; or learned positional embeddings (BERT/GPT style).

Complexity: $O(T^2 d)$ per layer for self-attention (vs. $O(Td^2)$ for RNN); for $T < d$, Transformers are faster than RNNs.

## Applications

All modern NLP (BERT, GPT, T5); vision (ViT, DINO); multimodal models; audio processing; protein structure (AlphaFold2).

## Trade-offs

$O(T^2)$ attention scaling limits context length (addressed by sparse attention, linear attention, state-space models); no inductive bias for locality (CNNs have this for free); requires large datasets to train from scratch; fine-tuning pretrained Transformers is the standard approach.

## Links
- [[attention_mechanism]]
- [[recurrent_networks]]
- [[word_embeddings]]
- [[sequence_to_sequence]]
- [[01_foundations/_legacy/deep_learning_theory 1/backpropagation]] (in 01_foundations/deep_learning_theory)
