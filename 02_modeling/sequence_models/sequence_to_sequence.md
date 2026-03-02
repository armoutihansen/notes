---
layer: 02_modeling
type: concept
status: seed
tags: [sequence_models, seq2seq, beam_search, bleu, machine_translation]
created: 2026-03-02
---

# Sequence-to-Sequence Models

## Definition

Encoder-decoder architectures that map a variable-length input sequence to a variable-length output sequence; combined with beam search for decoding and BLEU for evaluation.

## Intuition

The encoder summarizes the input sequence into a context representation; the decoder generates the output sequence one token at a time, conditioned on the context; beam search avoids greedy decoding by maintaining multiple candidate sequences.

## Formal Description

**Seq2seq**: encoder RNN reads $x^{<1>}, \ldots, x^{<T_x>}$ producing final hidden state (context vector) $c$; decoder RNN generates $y^{<1>}, \ldots, y^{<T_y>}$, initialized with $c$; $P(y^{<t>} \mid y^{<1>}, \ldots, y^{<t-1>}, c)$ modeled by the decoder at each step; unlike language model, conditioned on input.

**Machine translation as conditional LM**: $P(y_1, \ldots, y_{T_y} \mid x_1, \ldots, x_{T_x}) = \prod_{t=1}^{T_y} P(y^{<t>} \mid y^{<1:t-1>}, x)$; training: teacher forcing (feed ground-truth previous tokens); inference: autoregressive decoding.

**Beam search**: at each decoding step, keep top $B$ partial sequences (by cumulative log-prob); at step $t$, expand each of $B$ beams by all $|V|$ vocab words, score, keep top $B$; length normalization: divide by $(T_y)^\alpha$ with $\alpha \approx 0.7$ to prevent preference for short sequences.

**BLEU score** (Bilingual Evaluation Understudy): measures n-gram precision of machine translation against reference translations; $\text{BLEU} = \text{BP} \cdot \exp(\sum_{n=1}^N w_n \log p_n)$ where $p_n$ = modified n-gram precision, $\text{BP}$ = brevity penalty (punishes short outputs), $w_n = 1/N$ typically; ranges 0–1 (higher is better); BLEU-4 (up to 4-grams) is most common.

## Applications

Machine translation, text summarization, question answering (generative), code generation, speech-to-text (with encoder over audio features).

## Trade-offs

Vanilla seq2seq bottlenecks all information through a fixed-size context vector (the "information bottleneck"); attention mechanism (see [[attention_mechanism]]) resolves this; beam search with large $B$ is more accurate but slower; BLEU is imperfect (no semantic understanding, language-dependent tokenization).

## Links
- [[attention_mechanism]]
- [[recurrent_networks]]
- [[transformer]]
- [[word_embeddings]]
