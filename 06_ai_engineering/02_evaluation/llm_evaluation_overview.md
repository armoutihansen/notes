---
layer: 06_ai_engineering
type: engineering
tool: general
status: growing
tags: [evaluation]
created: 2026-03-05
---

# LLM Evaluation Overview

## Purpose

Evaluation is the engineering discipline of measuring whether a generative AI system is doing what you want it to do. Without systematic evaluation, model changes — prompt updates, fine-tuning, retrieval changes, model swaps — cannot be validated, and regressions are invisible.

Evaluation for LLMs is fundamentally harder than for classical ML:
- Outputs are **open-ended**: no single ground truth exists for most generation tasks.
- **Correctness is multi-dimensional**: a response can be factually accurate, incoherent, verbose, and subtly harmful simultaneously.
- **Distribution shift is fast**: model behaviour changes with prompt wording, temperature, or underlying model version.

An evaluation strategy should answer: is this system meeting user needs? Is it safe? Is it getting better or worse across versions? This requires layering multiple evaluation methods.

## Architecture

### Evaluation Taxonomy

**Automated Metrics (reference-based)**

Compare model output to a reference output:
- **Exact match**: output must equal reference string exactly; appropriate for constrained generation (SQL, structured JSON, classification labels).
- **BLEU**: n-gram precision with brevity penalty; standard for machine translation; known to correlate poorly with human judgement for long-form generation.
- **ROUGE** (ROUGE-1, ROUGE-L): recall-oriented n-gram overlap; used for summarisation; same correlation limitations.
- **BERTScore**: uses contextual BERT embeddings to compute precision/recall/F1 between generated and reference tokens; better semantic alignment than surface-level n-gram metrics.
- **Perplexity**: measures how well the model predicts a held-out text corpus; useful for comparing base model quality but not directly meaningful for task performance.

**Human Evaluation**

- **Crowdworker evaluation**: scalable; cost ~$0.10–$1.00 per rating; high variance; requires careful rubric design and annotator calibration.
- **Expert evaluation**: slow and expensive; necessary for high-stakes domains (medical, legal).
- **Preference judgements (pairwise)**: show annotators two responses (A/B) for the same prompt; ask which is better overall. More reliable than absolute Likert scales; used to produce Elo-ranked leaderboards (Chatbot Arena / LMSYS).
- Win rate vs a baseline model is the most reliable aggregate human eval signal.

**AI-as-a-Judge**

Use a strong LLM (GPT-4, Claude 3.5, Llama 3 70B) to evaluate model outputs:
- **Absolute scoring (G-Eval)**: prompt judge with a rubric; score on 1–5 scale per criterion.
- **Pairwise comparison (MT-Bench / Chatbot Arena style)**: show judge two responses A and B; ask which is better; aggregate win rates. Reduces position bias by swapping order and averaging.
- **LLM-as-critic with reference**: provide the judge with the correct answer or context; ask it to evaluate factuality.

**Task-Specific Automated Evaluation**

- **Code execution**: generate code; run it against unit tests in a sandbox; pass/fail signal (pass@k metric).
- **Tool call success rate**: in agentic systems, measure whether the correct tool was called with correct parameters.
- **Factuality checking**: extract claims from the output; verify each claim against a knowledge source.
- **Retrieval metrics (RAG)**: RAGAS framework — context precision, context recall, faithfulness, answer relevancy.

### Evaluation Levels

**Component-level evaluation** assesses individual system parts in isolation:
- Retriever: recall@k, MRR, NDCG on held-out query-document pairs.
- Generator: faithfulness and relevance given retrieved context.
- Reranker: NDCG vs gold relevance labels.

**End-to-end evaluation** measures the full pipeline on user-representative queries. Critical for identifying cross-component failure modes: a perfect retriever can feed irrelevant context to a perfect generator and still produce bad answers.

### Evaluation Dataset Design

- **Golden dataset**: curated (prompt, ideal response or label) pairs, human-verified; 100–1000 examples per capability area.
- **Adversarial set**: jailbreak attempts, edge cases, ambiguous inputs; verifies safety and robustness.
- **Regression set**: cases where previous model versions failed; ensures fixes don't regress.
- **Distribution-representative set**: sampled from actual production traffic (with PII redacted); reflects real user behaviour.

## Implementation Notes

### AI-as-Judge Methodology

Building a reliable AI-as-judge pipeline:

1. **Choose a strong judge**: GPT-4o or Claude 3.5 Sonnet for general tasks; Llama 3 70B-Instruct if you need a self-hosted judge.
2. **Define a structured rubric**: avoid vague instructions. Break down evaluation into specific criteria:
   - *Faithfulness*: does the response contain only claims supported by the provided context?
   - *Relevance*: does the response directly address the user's question?
   - *Completeness*: are all required aspects of the question addressed?
   - *Coherence*: is the response well-structured and logically consistent?
3. **Pairwise over absolute scoring**: absolute scores (1–10) have poor inter-rater reliability. Pairwise (A vs B) is more reliable and produces win-rate statistics.
4. **Control for position bias**: the judge tends to prefer whichever response appears first. Mitigate by evaluating both orderings (A,B) and (B,A); average results.
5. **Calibrate against human judgements**: measure Pearson/Spearman correlation between judge scores and human scores on a calibration set; target r > 0.7.

```python
# Example judge prompt structure (G-Eval style)
JUDGE_PROMPT = """
You are an expert evaluator. Given the following question and response, 
score the response on Faithfulness (1-5) and Relevance (1-5).

Question: {question}
Context: {context}
Response: {response}

Output JSON: {{"faithfulness": <int>, "relevance": <int>, "reasoning": "<str>"}}
"""
```

**LangSmith** provides a managed eval pipeline: dataset management, experiment tracking, LLM-as-judge integration, and A/B comparison dashboards. For self-hosted pipelines, combine HuggingFace `datasets` for golden sets with custom evaluation scripts tracked in MLflow or W&B.

### Evaluation Cadence

- Run lightweight automated eval (fast metrics, AI judge on small sample) on every PR/deployment.
- Run full human eval before major model version changes.
- Monitor production metrics continuously: user ratings, task completion rates, safety filter activations.

## Trade-offs

**Human evaluation**: highest signal quality; captures nuance and user preference; required for ground-truth calibration. Expensive ($500–$5,000 per study), slow (days to weeks), and hard to reproduce.

**Automated reference metrics (BLEU/ROUGE)**: fast and reproducible; well understood. Correlate poorly with human preference for open-ended generation; mislead on rephrased-but-correct outputs.

**AI-as-judge**: scales to thousands of examples cheaply; often correlates well with human judgement (r ≈ 0.7–0.9 with GPT-4 judge). Has systematic biases: prefers longer responses (verbosity bias), prefers its own outputs (self-preference bias), sensitive to prompt wording. Do not use as a sole evaluation signal.

**Execution-based (code, tool calls)**: objective, reproducible, no human or judge required. Only applicable to constrained output spaces; requires maintained test suites.

## References

- Liu et al. (2023). *G-Eval: NLG Evaluation using GPT-4 with Better Human Alignment*. ACL 2023.
- Zheng et al. (2023). *Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena*. LMSYS.
- Es et al. (2023). *RAGAS: Automated Evaluation of Retrieval Augmented Generation*. arXiv.
- Zhang et al. (2019). *BERTScore: Evaluating Text Generation with BERT*. ICLR 2020.
- Chiang et al. (2024). *Chatbot Arena: An Open Platform for Evaluating LLMs by Human Preference*.

## Links

- [[benchmarks_and_harness|Benchmarks and Evaluation Harness]]
- [[evaluating_code_models|Evaluating Code Generation Models]]
- [[foundation_model_overview|Foundation Model Overview]]
