---
layer: 03_modeling
type: concept
status: seed
tags: [algorithm, nlp]
created: 2026-03-02
---

# Transformers Overview

## Definition

The Transformer is a neural network architecture based entirely on self-attention mechanisms, replacing recurrence and convolution for sequence modelling. It has become the dominant architecture across NLP, vision, audio, and multimodal AI.

## Intuition

Before Transformers, sequence models (RNNs, LSTMs) processed tokens one at a time, bottlenecking information through a sequential chain that degraded over long distances. Transformers instead allow every token to directly attend to every other token in a single layer, making long-range dependencies as easy to learn as local ones. Combined with large-scale pretraining, this enabled a paradigm shift: train once on massive data, fine-tune cheaply for specific tasks.

## Formal Description

**Core building block — self-attention:**

$$
\text{Attention}(Q, K, V) = \text{softmax}\!\left(\frac{QK^\top}{\sqrt{d_k}}\right)V
$$

All tokens are processed in parallel; queries, keys, and values are learned linear projections of the input.

**Encoder architecture (BERT-style):**
- Tokenize input → token embeddings + positional encodings
- Stack $N$ identical blocks: [LayerNorm → Multi-Head Self-Attention → Residual] → [LayerNorm → FFN → Residual]
- Output: contextualized representation for each token

**Decoder architecture (GPT-style):**
- Same as encoder but with **causal masking** on self-attention (tokens only attend to past positions)
- Output: probability distribution over vocabulary at each position for next-token prediction

**Encoder-decoder (T5/original Transformer):**
- Encoder produces representations; decoder cross-attends to encoder outputs plus autoregressively generates output tokens

**Pretraining objectives:**
- **Masked Language Modeling (BERT):** randomly mask 15% of tokens; predict the masked tokens
- **Causal Language Modeling (GPT):** predict the next token given all previous tokens
- **Span denoising (T5):** mask contiguous spans and reconstruct them

**Scaling:** performance scales predictably with model size, data, and compute (Chinchilla scaling laws); models now range from millions to trillions of parameters.

## Applications

- **NLP:** BERT (classification, NER, QA), GPT (text generation), T5 (seq2seq tasks)
- **Vision:** Vision Transformer (ViT) patches images into sequences; CLIP aligns vision and language
- **Multimodal:** GPT-4V, Gemini — unified architecture across modalities
- **Science:** AlphaFold2 (protein structure), code generation (Codex, GitHub Copilot)

## Trade-offs

| Property | Transformer | RNN/LSTM |
|----------|-------------|----------|
| Parallelism | Full (training) | Sequential |
| Long-range dependencies | O(1) path length | O(T) path |
| Memory | O(T²) for attention | O(T) |
| Locality bias | None (needs positional encoding) | Built-in |
| Pretrained models available | Yes (vast ecosystem) | Limited |

**Key limitations:**
- $O(T^2)$ attention cost limits context length; addressed by sparse attention (Longformer), linear attention (Performer), and state-space models (Mamba)
- No built-in inductive bias for images or sequences; requires more data than CNNs/RNNs for small datasets
- Large pretrained models are expensive to train; fine-tuning and PEFT methods (LoRA, adapters) make deployment feasible

## Links

- [[transformer]]
- [[attention_mechanism]]
- [[word_embeddings]]
- [[sequence_to_sequence]]
- [[03_modeling/04_deep_learning/03_sequence_models/recurrent_networks|Recurrent Networks]]
- [[03_modeling/04_deep_learning/04_transformers/index|Transformers Index]]
- [[01_foundations/06_deep_learning_theory/backpropagation|Backpropagation — gradient flow through attention layers]]
- [[01_foundations/06_deep_learning_theory/layer_normalization|Layer Normalization — normalisation in transformer blocks]]
