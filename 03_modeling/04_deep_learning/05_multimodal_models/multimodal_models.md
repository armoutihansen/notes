---
layer: 03_modeling
type: concept
status: seed
tags: [algorithm, vision, nlp, multimodal]
created: 2026-05-31
---

# Multimodal Models

## Core Idea

Multimodal models jointly encode and align information from two or more data modalities — most commonly images and text. The central challenge is building a shared representation space where semantically related content from different modalities is nearby, enabling cross-modal retrieval, visual question answering, and image captioning.

## Mathematical Formulation

### Contrastive Vision-Language Pre-training (CLIP)

CLIP (Radford et al., 2021) trains an image encoder $f_I$ and a text encoder $f_T$ on $N$ (image, text) pairs using InfoNCE loss:

$$
\mathcal{L} = -\frac{1}{N} \sum_{i=1}^N \left[\log \frac{\exp(\text{sim}(f_I(\mathbf{v}_i), f_T(\mathbf{t}_i)) / \tau)}{\sum_{j=1}^N \exp(\text{sim}(f_I(\mathbf{v}_i), f_T(\mathbf{t}_j)) / \tau)}\right]
$$

where $\text{sim}(\mathbf{u}, \mathbf{v}) = \mathbf{u}^\top \mathbf{v} / (\|\mathbf{u}\|\|\mathbf{v}\|)$ and $\tau$ is a learnable temperature.

After pre-training, zero-shot classification is performed by computing cosine similarity between the image embedding and text embeddings for each class label.

### Fusion Strategies

**Early fusion:** concatenate raw inputs or low-level features before processing.

**Late fusion:** process each modality independently and combine final representations:
$$
\hat{y} = g(f_I(\mathbf{v}), f_T(\mathbf{t}))
$$
where $g$ is a learned combiner (e.g., MLP, cross-attention).

**Cross-attention fusion:** one modality attends to the other:
$$
\text{Attn}(Q, K, V) = \text{softmax}\!\left(\frac{QK^\top}{\sqrt{d_k}}\right)V
$$
with $Q$ from one modality and $K, V$ from another. Used in Visual Question Answering (VQA) models.

### Visual Question Answering (VQA)

Given image $I$ and question $q$ (text), predict answer $a$:
1. Extract visual features: $\mathbf{v} = f_I(I) \in \mathbb{R}^{K \times d}$ (feature map grid or object detections).
2. Encode question: $\mathbf{q} = f_T(q) \in \mathbb{R}^d$.
3. Cross-attend $\mathbf{q}$ over $\mathbf{v}$, then classify over answer vocabulary.

## Inductive Bias

- **Contrastive alignment**: assumes semantically matching image-text pairs should have similar embeddings; relies on large datasets of image-caption pairs for pre-training.
- **Cross-attention fusion**: assumes relevant visual regions can be identified from the query modality; spatially explicit.

## Training Objective

Pre-training typically uses one or more of:
- **Contrastive (InfoNCE)**: aligns matching pairs, pushes away non-matching.
- **Masked language modelling (MLM)** conditioned on image features.
- **Image-text matching (ITM)**: binary classification of whether image and text correspond.

## Strengths

- Zero-shot transfer: CLIP can classify images into novel categories by describing them in text.
- Generalises across vision tasks by leveraging language supervision.
- Enables multimodal retrieval (image → text, text → image).

## Weaknesses

- Requires very large-scale data (400M+ image-text pairs for CLIP).
- Contrastive objectives may align surface-level patterns rather than deep semantic content.
- Fusion is non-trivial: naively combining modalities often underperforms single-modality baselines.
- Compositional understanding is a known weakness (e.g., "a blue circle above a red square" vs "a red circle above a blue square").

## Variants

- **BLIP-2**: adds a lightweight Querying Transformer (Q-Former) bridging frozen image encoder and LLM.
- **Flamingo**: interleaves cross-attention layers into a frozen LLM to inject visual conditioning.
- **LLaVA**: instruction-tuned vision-language model using CLIP image encoder + LLaMA/Vicuna LLM.
- **ImageBind**: aligns six modalities (image, text, audio, depth, thermal, IMU) in a single embedding space.

## References

- Radford, A. et al. (2021). "Learning Transferable Visual Models From Natural Language Supervision." ICML (CLIP).
- Li, J. et al. (2023). "BLIP-2: Bootstrapping Language-Image Pre-training with Frozen Image Encoders and Large Language Models." ICML.
- Alayrac, J-B. et al. (2022). "Flamingo: a Visual Language Model for Few-Shot Learning." NeurIPS.

## Links

- [[03_modeling/04_deep_learning/04_transformers/attention_mechanism|Attention Mechanism — cross-modal attention]]
- [[03_modeling/04_deep_learning/02_convolutional_networks/cnn_architecture|CNN Architectures — visual encoder backbone]]
- [[03_modeling/02_unsupervised_learning/04_representation_learning/representation_learning|Representation Learning — contrastive pre-training]]
- [[01_foundations/06_deep_learning_theory/index|Deep Learning Theory — training objectives]]
