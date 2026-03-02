---
layer: 02_modeling
type: concept
status: seed
tags: [sequence_models, rnn, gru, lstm]
created: 2026-03-02
---

# Recurrent Networks

## Definition

Neural architectures that process sequences by maintaining a hidden state that is updated at each time step; GRUs and LSTMs are gated variants that address the vanishing gradient problem in plain RNNs.

## Intuition

A plain RNN passes a single state vector forward through time, but gradients vanish over long sequences; gates in GRU/LSTM selectively remember or forget information, allowing gradients to flow across hundreds of time steps.

## Formal Description

**RNN**: $a^{<t>} = g(W_{aa}a^{<t-1>} + W_{ax}x^{<t>} + b_a)$, $\hat y^{<t>} = g_y(W_{ya}a^{<t>} + b_y)$; same weights $W_{aa}, W_{ax}$ shared across all time steps; types: many-to-many (machine translation via seq2seq), many-to-one (sentiment), one-to-many (text generation).

**GRU**: introduces update gate $\Gamma_u$ and relevance gate $\Gamma_r$; candidate: $\tilde c^{<t>} = \tanh(W_c[\Gamma_r \odot c^{<t-1>}, x^{<t>}] + b_c)$; update: $c^{<t>} = \Gamma_u \odot \tilde c^{<t>} + (1-\Gamma_u) \odot c^{<t-1>}$; gate values near 0/1 allow exact copying of state across many steps (bypasses vanishing gradient).

**LSTM**: three gates — forget $\Gamma_f$, update $\Gamma_u$, output $\Gamma_o$; separate cell state $c^{<t>}$ and hidden state $a^{<t>}$; $c^{<t>} = \Gamma_f \odot c^{<t-1>} + \Gamma_u \odot \tilde c^{<t>}$; $a^{<t>} = \Gamma_o \odot \tanh(c^{<t>})$; LSTM has more parameters than GRU but historically stronger on many tasks.

## Applications

Time series forecasting, language modeling (token-level), speech recognition (before Transformers), NER, part-of-speech tagging; largely superseded by Transformers for NLP but still used in streaming/low-latency applications.

## Trade-offs

Sequential processing limits parallelization (Transformers are fully parallel); LSTMs are harder to train than GRUs; both still suffer from vanishing gradients at very long contexts (hundreds to thousands of steps); Transformers have O(T²) attention but O(1) path length vs. O(T) for RNNs.

## Links
- [[transformer]]
- [[sequence_to_sequence]]
- [[backpropagation_through_time]] (in 01_foundations/deep_learning_theory)
