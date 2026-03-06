---
layer: 02_modeling
type: concept
status: seed
tags: [nlp, embeddings, word2vec, glove, fairness]
created: 2026-03-02
---

# Word Embeddings

## Definition

Dense, low-dimensional real-valued vector representations of tokens where geometric proximity encodes semantic and syntactic similarity; learned from large corpora without explicit supervision.

## Intuition

One-hot vectors have no notion of similarity; word embeddings capture that "king" and "queen" are more similar than "king" and "table"; the vector arithmetic $\text{king} - \text{man} + \text{woman} \approx \text{queen}$ reveals that linear structure encodes semantic relationships.

## Formal Description

**Word2Vec:** two tasks for training embeddings:
- **(a) Skip-gram:** predict context words given center word
- **(b) CBOW:** predict center word from context

Objective: maximize $\sum \log P(\text{context} \mid \text{center})$; embedding matrix $E \in \mathbb{R}^{d \times |V|}$; $e_w = E \cdot o_w$ where $o_w$ is one-hot.

**Negative sampling:** approximate the full softmax over $|V|$ words by sampling $k$ negative words for each positive context pair:

$$\mathcal{L} = \log\sigma(u_{c_0}^\top v_w) + \sum_{j=1}^k \log\sigma(-u_{c_j}^\top v_w)$$

$k = 5$–$20$ for small datasets; reduces computation from $O(|V|)$ to $O(k)$.

**GloVe:** factorize the global co-occurrence matrix $X$:

$$\sum_{i,j} f(X_{ij})(w_i^\top \tilde w_j + b_i + \tilde b_j - \log X_{ij})^2$$

where $f$ is a weighting function; captures global statistics directly; typically faster to train on large corpora.

**Debiasing:** embeddings trained on real text encode human biases (gender, racial); mitigation: (1) identify bias direction via SVD on gender-definitional pairs; (2) neutralize: project non-definitional words onto the complement of bias direction; (3) equalize: make definitional pairs equidistant from bias direction; limitations: debiasing is incomplete and can introduce other artifacts.

## Applications

Feature representations for NLP models, semantic search, recommendation systems (item embeddings), drug discovery (molecular embeddings); static embeddings (Word2Vec, GloVe) largely replaced by contextual embeddings (BERT, GPT) but still useful as lightweight baselines.

## Trade-offs

Static embeddings assign one vector per word regardless of context ("bank" as in river vs. finance); out-of-vocabulary words have no representation; dimension $d$ is a hyperparameter (typically 100–300); GloVe and Word2Vec often give similar downstream performance.

## Links

- [[transformer]]
- [[sequence_to_sequence]]
- [[02_modeling/03_model_families/04_neural_networks/recurrent_networks|Recurrent Networks]]
- [[02_modeling/03_model_families/07_transformers/index|Transformers Index]]
