---
layer: 05_ai_engineering
type: engineering
tool: general
status: growing
tags: [dataset, instruction-tuning, data-quality, sft, alignment]
created: 2026-03-05
---

# Instruction Data Design

## Purpose

Creating and curating training data for instruction-following fine-tuning is as important as model architecture choices — possibly more so. The quality, diversity, and format of instruction data determines what behaviors the fine-tuned model will exhibit, what it can and cannot do, and what failure modes it will express. "Garbage in, garbage out" is nowhere more literal than in SFT data design.

Instruction data design sits at the intersection of ML engineering and data engineering. It involves decisions about collection strategy (human annotation vs. synthetic generation), format, diversity, quality filtering, and distribution balance — all of which compound. See [[synthetic_data_generation|Synthetic Data Generation]] for methods of generating this data programmatically.

## Architecture

**Instruction Data Formats**

Three standard formats dominate:

1. **Alpaca format** (single-turn):
```json
{
  "instruction": "Summarize the following article in one sentence.",
  "input": "Article text...",
  "output": "The article discusses..."
}
```
`input` is optional; when absent, `instruction` contains the full prompt.

2. **ShareGPT / ChatML format** (multi-turn):
```json
{
  "conversations": [
    {"from": "human", "value": "Explain gradient descent."},
    {"from": "gpt", "value": "Gradient descent is..."},
    {"from": "human", "value": "What's the learning rate?"},
    {"from": "gpt", "value": "The learning rate controls..."}
  ]
}
```
Preferred for chat models; models trained on ChatML generalize better to multi-turn inference.

3. **OpenHermes / instruction-response pairs with system prompts**: extends Alpaca with a `system` field for persona or policy specification.

**Data Quality Dimensions**

Quality is multidimensional — a dataset can excel on one dimension while failing on others:

- **Instruction diversity**: coverage across task types (QA, summarization, code, math, creative writing, safety refusal), domains, and styles. Cluster instruction embeddings to measure and fill gaps.
- **Response quality**: factual accuracy, helpfulness, appropriate length, safety. The most common failure mode in synthetic data.
- **Length balance**: mix short (1–3 sentence) and long (multi-paragraph) responses. Models trained on uniform-length data generalize poorly.
- **Difficulty distribution**: include a range from trivial to challenging; models trained only on easy examples fail on hard ones.
- **Safety coverage**: include examples of appropriate refusal, boundary-setting, and harm avoidance — not just capability demonstrations.

**Collection Strategies**

| Strategy | Quality | Cost | Scale |
|---|---|---|---|
| Human annotation | Gold standard | $10–50/example | Limited |
| Model distillation (e.g., GPT-4) | High | $0.01–0.10/example | Large |
| Self-Instruct / Evol-Instruct | Medium | Very low | Very large |
| Curated public datasets | Varies | Free | Fixed |

Public sources: FLAN (Google, 1,800+ tasks), OpenAssistant, Dolly 15K, OpenHermes, UltraChat. See [[synthetic_data_generation|Synthetic Data Generation]] for generative methods.

## Implementation Notes

**Diversity analysis workflow:**
```python
from sentence_transformers import SentenceTransformer
from sklearn.cluster import KMeans

model = SentenceTransformer("all-MiniLM-L6-v2")
embeddings = model.encode([ex["instruction"] for ex in dataset])

kmeans = KMeans(n_clusters=50)
labels = kmeans.fit_predict(embeddings)

# inspect cluster sizes — small clusters = underrepresented task types
from collections import Counter
print(Counter(labels).most_common(10))
```

**Deduplication with MinHash LSH:**
```python
from datasketch import MinHash, MinHashLSH

lsh = MinHashLSH(threshold=0.7, num_perm=128)
for i, text in enumerate(instructions):
    m = MinHash(num_perm=128)
    for word in text.split():
        m.update(word.encode("utf8"))
    lsh.insert(f"doc_{i}", m)
# query for near-duplicates before adding new examples
```

**Quality filtering pipeline:**
1. Length filter: remove instructions under 5 words or responses under 10 words
2. Language detection: `langdetect` or `fasttext` — remove non-target-language examples
3. Deduplication: MinHash LSH at 0.7 Jaccard similarity threshold
4. Reward model scoring: use an open reward model (e.g., `OpenAssistant/reward-model-deberta-v3-large-v2`) to filter bottom 20% of responses
5. Perplexity filtering: high perplexity from a reference model may indicate gibberish or format errors

**Task taxonomy for diversity auditing:**
- Question answering (factual, reasoning, multi-hop)
- Summarization (extractive, abstractive, length-constrained)
- Code (generation, debugging, explanation, translation)
- Mathematics (arithmetic, algebra, word problems, proof)
- Creative writing (story, dialogue, poetry)
- Instruction following (format constraints, multi-step)
- Safety (refusal, harm avoidance, privacy)

**Dataset cards:** Document source, collection process, filtering steps, known biases, and license for every dataset — this is often skipped and always regretted.

## Trade-offs

**Quality vs. quantity:**
- LIMA (Zhou et al., 2023) showed 1,000 carefully selected examples can match models trained on 52,000 examples
- Noisy data at scale teaches the model to be uncertain and hedging — high-quality curated data produces more confident, direct responses
- However, diversity at scale still matters: 1,000 examples from one task type will not generalize

**Human vs. synthetic:**
- Human data reflects actual user needs and naturalistic language; synthetic data may be stilted or formulaic
- Human annotation at scale is expensive and slow; synthetic generation is instant and cheap
- Mix is almost always better than either extreme: seed with human data, expand with synthetic, filter aggressively

**Format choices:**
- Alpaca format is simpler to collect but limits multi-turn reasoning training
- ChatML/ShareGPT trains better chatbots but requires more complex data collection
- System prompt diversity improves robustness to different deployment contexts

**Distribution balance:**
- Under-represented task types create capability gaps; over-represented types can cause the model to default to those patterns even when inappropriate
- Safety data should be ~5–15% of total — too little creates unsafe models; too much creates over-refusal

## References

- Zhou et al. (2023) — *LIMA: Less Is More for Alignment* (arXiv:2305.11206)
- Wang et al. (2022) — *Self-Instruct: Aligning Language Models with Self-Generated Instructions*
- Köpf et al. (2023) — *OpenAssistant Conversations — Democratizing Large Language Model Alignment*
- Wei et al. (2021) — *Finetuned Language Models Are Zero-Shot Learners* (FLAN)
- Databricks (2023) — *Free Dolly: Introducing the World's First Truly Open Instruction-Tuned LLM*

## Links
- [[synthetic_data_generation|Synthetic Data Generation]]
- [[finetuning_strategies|Fine-tuning Strategies]]
- [[rl_finetuning|Reinforcement Learning Fine-tuning]]
- [[data_labeling|Data Labeling]]
