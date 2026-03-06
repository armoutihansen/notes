---
layer: 06_applications
type: application
status: growing
tags: [llm-evaluation, langsmith, ai-as-judge, evaluation-dataset, regression-testing, ci]
created: 2026-05-10
---

# LLM Evaluation Pipeline

## Purpose

An LLM evaluation pipeline measures whether a generative AI system is meeting quality, safety, and correctness requirements — across versions, prompt changes, and model swaps. Without systematic evaluation, regressions are invisible. This note covers: building a golden dataset, defining an LLM-as-judge rubric, running evaluations with LangSmith, comparing runs, and setting up automated regression checks in CI.

### Examples

**RAG Q&A evaluation**: Measure faithfulness (does the answer follow from the context?) and answer relevance (does the answer address the question?) on 200 golden QA pairs after each prompt update.

**Code generation evaluation**: Run pass@1 tests on 50 coding tasks before and after a model swap; fail CI if pass@1 drops more than 3 points.

---

## Architecture

```
Golden dataset (prompt, [context], expected_output)
        ↓
   LLM / pipeline generates actual outputs
        ↓
   Evaluation: automated metrics + LLM-as-judge
        ├── Exact match / ROUGE / BLEU (reference-based)
        └── LLM judge: rubric scoring (faithfulness, relevance, etc.)
        ↓
   Results stored in LangSmith experiment
        ↓
   Compare across runs (A/B, before/after)
        ↓
   CI gate: fail if metric drops below threshold
```

---

## Setup

```bash
pip install langsmith langchain-openai
export LANGCHAIN_API_KEY="ls__..."
export LANGCHAIN_TRACING_V2="true"
export LANGCHAIN_PROJECT="my-eval-project"
```

---

## Create a Golden Evaluation Dataset

```python
from langsmith import Client

client = Client()

# Create dataset
dataset = client.create_dataset(
    dataset_name="rag-qa-golden-v1",
    description="200 curated QA pairs with grounding context",
)

# Upload examples: (inputs, expected outputs)
examples = [
    {
        "inputs":  {"question": "What is the refund window for enterprise accounts?",
                    "context":  "Enterprise customers may request refunds within 45 days..."},
        "outputs": {"answer":   "Enterprise accounts have a 45-day refund window."},
    },
    {
        "inputs":  {"question": "Is multi-factor authentication required?",
                    "context":  "MFA is mandatory for all admin accounts as of Q1 2024."},
        "outputs": {"answer":   "Yes, MFA is required for all admin accounts."},
    },
    # ... add 198 more examples
]

client.create_examples(
    inputs=[e["inputs"] for e in examples],
    outputs=[e["outputs"] for e in examples],
    dataset_id=dataset.id,
)
print(f"Dataset created: {dataset.id}")
```

---

## LLM-as-Judge Rubric

```python
from langchain_openai import ChatOpenAI
from langsmith.evaluation import LangChainStringEvaluator

# Built-in evaluators: "qa" (correctness), "context_qa", "cot_qa", "criteria"

# Custom rubric evaluator using "criteria"
faithfulness_evaluator = LangChainStringEvaluator(
    "criteria",
    config={
        "criteria": {
            "faithfulness": (
                "Does the answer accurately reflect the information in the provided context? "
                "Score 1 (faithful) or 0 (hallucinated or contradicts context)."
            )
        },
        "llm": ChatOpenAI(model="gpt-4o-mini", temperature=0),
    },
    prepare_data=lambda run, example: {
        "prediction": run.outputs["answer"],
        "input":      example.inputs["context"],
        "reference":  example.outputs["answer"],
    },
)

relevance_evaluator = LangChainStringEvaluator("qa", config={
    "llm": ChatOpenAI(model="gpt-4o-mini", temperature=0),
})
```

---

## Running Evaluations

```python
from langsmith.evaluation import evaluate
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4o-mini", temperature=0)

def answer_question(inputs: dict) -> dict:
    """Pipeline under evaluation."""
    prompt = f"Context: {inputs['context']}\n\nQuestion: {inputs['question']}\nAnswer:"
    response = llm.invoke(prompt)
    return {"answer": response.content}

# Run evaluation on the golden dataset
results = evaluate(
    answer_question,
    data="rag-qa-golden-v1",
    evaluators=[faithfulness_evaluator, relevance_evaluator],
    experiment_prefix="gpt-4o-mini-baseline",
    num_repetitions=1,
)
print(results)
```

---

## Comparing Runs (A/B Testing)

```python
# After running two experiments (e.g., baseline vs. prompt-v2)
from langsmith import Client
import pandas as pd

client = Client()

# Fetch experiment results for comparison
baseline = client.read_project(project_name="gpt-4o-mini-baseline")
improved = client.read_project(project_name="gpt-4o-mini-prompt-v2")

# Programmatic comparison (via LangSmith dashboard or API)
# Dashboard: https://smith.langchain.com → Datasets → rag-qa-golden-v1 → Compare runs
```

---

## CI Regression Check (GitHub Actions)

```yaml
# .github/workflows/llm_eval.yml
name: LLM Evaluation CI

on:
  pull_request:
    paths: ["src/prompts/**", "src/pipeline/**"]

jobs:
  evaluate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"
      - run: pip install langsmith langchain-openai
      - name: Run evaluation
        env:
          LANGCHAIN_API_KEY: ${{ secrets.LANGCHAIN_API_KEY }}
          OPENAI_API_KEY:    ${{ secrets.OPENAI_API_KEY }}
        run: python scripts/run_eval.py --fail-threshold 0.85
```

```python
# scripts/run_eval.py
import argparse, sys
from langsmith.evaluation import evaluate
# ... (same as above) ...

parser = argparse.ArgumentParser()
parser.add_argument("--fail-threshold", type=float, default=0.85)
args = parser.parse_args()

results = evaluate(answer_question, data="rag-qa-golden-v1", ...)
mean_faithfulness = results.to_pandas()["feedback.faithfulness"].mean()

print(f"Faithfulness: {mean_faithfulness:.3f} (threshold: {args.fail_threshold})")
if mean_faithfulness < args.fail_threshold:
    print("FAIL: evaluation below threshold")
    sys.exit(1)
```

---

## Evaluation Metric Summary

| Metric | Method | When to use |
|---|---|---|
| Exact match | String equality | SQL, structured outputs, classification |
| ROUGE-L | N-gram overlap | Summarisation baseline |
| Faithfulness | LLM judge | RAG — does answer follow from context? |
| Answer relevance | LLM judge | RAG — does answer address the question? |
| Correctness | LLM judge vs. reference | Open-ended Q&A |
| Pass@k | Code execution | Code generation tasks |
| Refusal rate | String match / classifier | Safety evaluation |

---

## Links

**AI Engineering**
- [[05_ai_engineering/01_evaluation/llm_evaluation_overview|LLM Evaluation Overview]] — evaluation taxonomy: automated metrics, LLM-as-judge, human eval
- [[05_ai_engineering/01_evaluation/benchmarks_and_harness|Benchmarks and Harness]] — lm-evaluation-harness for standardised benchmarks

**System Patterns**
- [[langsmith_llm_observability|LangSmith LLM Observability]] — tracing and monitoring companion to evaluation
- [[rag_pipeline_pattern|RAG Pipeline Pattern]] — the pipeline most often evaluated with this workflow

**End-to-End Examples**
- [[rag_qa_system|RAG Q&A System]] — system where this evaluation pipeline is applied
