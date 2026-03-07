---
layer: 06_ai_engineering
type: engineering
tool: lm-evaluation-harness
status: growing
tags: [evaluation]
created: 2026-03-05
---

# Benchmarks and Evaluation Harness

## Purpose

Academic benchmarks provide **standardised, reproducible measurement** of model capabilities across well-defined task distributions. They enable: (1) comparison of models across organisations on a common scale; (2) tracking capability improvements across training runs; (3) informing model selection for downstream tasks.

Without shared benchmarks, every team would evaluate on proprietary datasets, making cross-model comparisons impossible. The HuggingFace Open LLM Leaderboard and Stanford HELM have operationalised this into public ranking systems that drive open-model development.

However, benchmarks are imperfect proxies: they measure narrow capabilities, can be contaminated during training, and may not predict real-world usefulness. An engineering-grade evaluation strategy uses benchmarks for **baseline capability assessment** alongside task-specific and production evaluations.

## Architecture

### EleutherAI LM Evaluation Harness

The canonical open-source evaluation framework (Gao et al., 2021):

- **60+ tasks** (benchmarks) with standardised prompting, few-shot setups, and scoring.
- **Unified task interface**: each task defines a dataset loader, prompt template, output format, and metric.
- **Backend support**: HuggingFace `transformers`, `vLLM` (for high-throughput batch eval), OpenAI/Anthropic APIs, GGUF models via `llama.cpp`.
- Used by: HuggingFace Open LLM Leaderboard, Mistral, Meta, and most open-model evaluations.

### Key Benchmarks

| Benchmark | Task | Metric | Notes |
|---|---|---|---|
| **MMLU** | 57 academic subjects (STEM, humanities, law, medicine) | 5-shot accuracy | Standard knowledge breadth measure; 14K questions |
| **HellaSwag** | Commonsense sentence completion | 10-shot accuracy | Easy for humans (95%); hard for LLMs historically |
| **HumanEval** | 164 Python function completions | pass@1 (greedy) | Code generation; execution-based correctness |
| **GSM8K** | 8.5K grade school math word problems | 8-shot chain-of-thought accuracy | Multi-step arithmetic reasoning |
| **TruthfulQA** | 817 questions designed to elicit misconceptions | MC1/MC2 accuracy | Factuality; adversarially constructed |
| **ARC (Challenge)** | Grade 3–9 science questions | 25-shot accuracy | Elementary reasoning; Challenge subset is hard |
| **BIG-Bench Hard** | 23 tasks that resist chain-of-thought improvement | 3-shot CoT accuracy | Tests frontier reasoning capabilities |
| **MATH** | 12.5K competition math problems | 4-shot accuracy | Advanced symbolic reasoning; hard even for GPT-4 |
| **WinoGrande** | Commonsense pronoun resolution | 5-shot accuracy | Winograd schema; tests world knowledge |

### HuggingFace Open LLM Leaderboard

Standardised evaluation runs on the same hardware for all submitted models. Uses 6 benchmarks: ARC, HellaSwag, MMLU, TruthfulQA, Winograde, GSM8K. Normalised to 0–100 for ranking. Key reference point for open-weight model comparison.

## Implementation Notes

### Running the Harness

Install and run a basic evaluation:

```bash
pip install lm-eval

# Evaluate a HuggingFace model on MMLU (5-shot)
lm_eval \
  --model hf \
  --model_args pretrained=meta-llama/Meta-Llama-3-8B-Instruct \
  --tasks mmlu \
  --num_fewshot 5 \
  --batch_size auto \
  --output_path results/llama3-8b-mmlu/

# Evaluate with vLLM backend (much faster for large batches)
lm_eval \
  --model vllm \
  --model_args pretrained=meta-llama/Meta-Llama-3-8B-Instruct,gpu_memory_utilization=0.9 \
  --tasks mmlu,gsm8k,hellaswag \
  --num_fewshot 5,8,10 \
  --batch_size 32 \
  --output_path results/

# Evaluate an OpenAI API model
lm_eval \
  --model openai-chat-completions \
  --model_args model=gpt-4o \
  --tasks mmlu \
  --num_fewshot 5
```

Output: a JSON results file with per-task accuracy, standard error, and per-sample details.

### Pass@k for Code (HumanEval)

HumanEval uses **pass@k** to measure code generation quality:

- For each of 164 problems, generate `n` completions (typically `n=20` or `n=200`).
- `pass@k = 1 - C(n-c, k) / C(n, k)` where `c` is the number of passing completions.
- `pass@1`: standard greedy/beam search; a single attempt per problem.
- `pass@10`, `pass@100`: higher k rewards models that can generate diverse correct solutions.

Temperature matters: for `pass@1`, use greedy (T=0) or low temperature (T=0.2); for `pass@k` with k>1, use higher temperature (T=0.8) to ensure diversity.

```bash
lm_eval --model hf \
  --model_args pretrained=deepseek-ai/deepseek-coder-7b-instruct-v1.5 \
  --tasks humaneval \
  --num_fewshot 0 \
  --gen_kwargs temperature=0.0,max_gen_toks=512
```

### MMLU Few-Shot Setup

MMLU uses **5-shot**: 5 example questions from the same subject are prepended to the prompt. The model scores each of the 4 answer choices (A/B/C/D) and picks the highest log-probability. No generation required; this is a scoring task.

```
# Example MMLU prompt (5-shot, Anatomy subject)
Question: Which of the following is the body cavity that contains the pituitary gland?
A. Abdominal
B. Cranial
C. Pleural
D. Spinal
Answer: B

[... 4 more examples ...]

Question: [target question]
Answer:
```

### Benchmark Contamination Risks

If benchmark questions appear in pre-training data, reported scores are inflated and non-representative. Mitigation:
- Use contamination detection tools (`lm_eval` has a `--decontamination_ngrams_path` flag).
- Prefer **held-out** benchmarks (LiveBench, BIG-Bench Hard) and **dynamic/rolling** benchmarks (LiveCodeBench, HELM Lite).
- Be sceptical of scores near ceiling (>90% MMLU); apply newer harder benchmarks.

## Trade-offs

**Benchmarks measure narrow capabilities**: MMLU measures knowledge recall, not reasoning in context. GSM8K measures arithmetic, not real-world problem solving. Models can be optimised to score well on specific benchmarks without general capability improvement ("Goodhart's Law for benchmarks").

**Contamination risk**: models trained on larger internet scrapes are increasingly likely to have seen benchmark questions verbatim. Benchmark scores from unknown training datasets should be treated as upper bounds, not reliable comparisons.

**Academic vs real-world performance gap**: a model that outperforms another on MMLU may underperform on customer service tasks, SQL generation, or creative writing. Task-specific evaluation is necessary to validate benchmark-based model selection decisions.

**Task-specific vs general benchmarks**: MMLU/HellaSwag are general-purpose; HumanEval/GSM8K are domain-specific. For a coding product, HumanEval and SWE-bench are far more predictive of value than MMLU.

## References

- Gao et al. (2021). *A Framework for Few-Shot Language Model Evaluation* (LM Evaluation Harness). EleutherAI.
- Hendrycks et al. (2021). *Measuring Massive Multitask Language Understanding* (MMLU). ICLR 2021.
- Chen et al. (2021). *Evaluating Large Language Models Trained on Code* (HumanEval). OpenAI.
- Cobbe et al. (2021). *Training Verifiers to Solve Math Word Problems* (GSM8K). OpenAI.
- Zellers et al. (2019). *HellaSwag: Can a Machine Really Finish Your Sentence?* ACL 2019.
- Lin et al. (2021). *TruthfulQA: Measuring How Models Mimic Human Falsehoods*. ACL 2022.

## Links

- [[llm_evaluation_overview|LLM Evaluation Overview]]
- [[evaluating_code_models|Evaluating Code Generation Models]]
- [[foundation_model_overview|Foundation Model Overview]]
