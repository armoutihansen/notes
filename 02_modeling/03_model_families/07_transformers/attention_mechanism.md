---
layer: 02_modeling
type: concept
status: seed
tags: [algorithm, nlp]
created: 2026-03-02
---

# Attention Mechanism

## Definition

A mechanism that computes a context vector as a learned, input-dependent weighted combination of encoder states, allowing the decoder to "attend" to different parts of the input at each output step.

## Intuition

The fixed-size context vector in vanilla seq2seq is an information bottleneck for long sequences; attention lets the decoder look back at all encoder states and focus on the relevant parts at each output step — analogous to how a human translator refers back to the source sentence while translating.

## Formal Description

At decoder step $t$, compute alignment score with each encoder state $a^{<t'>}$:

$$e_{t,t'} = \text{score}(s^{<t-1>}, a^{<t'>})$$

Common score functions:
- **Dot product:** $s^\top a$
- **Additive (Bahdanau):** $v^\top \tanh(W_1 s + W_2 a)$
- **Multiplicative:** $s^\top W a$

Attention weights (softmax):

$$\alpha_{t,t'} = \frac{\exp(e_{t,t'})}{\sum_{j} \exp(e_{t,j})}, \qquad \sum_{t'} \alpha_{t,t'} = 1$$

Context vector:

$$c^{<t>} = \sum_{t'} \alpha_{t,t'} a^{<t'>}$$

Fed to decoder along with previous output $y^{<t-1>}$.

Complexity: $O(T_x \cdot T_y)$ alignment computations per sequence pair.

**Relation to self-attention:** in attention the query comes from the decoder, keys/values from the encoder; in self-attention all three come from the same sequence (see [[transformer]]).

## Applications

Machine translation (Bahdanau 2015), image captioning (attend to spatial CNN features), speech recognition; the attention concept generalizes to Transformers (self-attention).

## Trade-offs

$O(T_x \cdot T_y)$ alignment computation can be expensive; attention is not a silver bullet for very long sequences; the attention weights provide human-interpretable alignment but can be noisy.

## Links

- [[sequence_to_sequence]]
- [[transformer]]
- [[02_modeling/03_model_families/04_neural_networks/recurrent_networks|Recurrent Networks]]
- [[02_modeling/03_model_families/07_transformers/index|Transformers Index]]
- [[01_foundations/01_linear_algebra/02_matrices/matrix_operations|Matrix Operations — QKV matmuls, scaled dot-product]]
- [[01_foundations/03_probability_and_statistics/probability_distributions|Probability Distributions — softmax as categorical distribution]]
