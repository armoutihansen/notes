---
layer: 06_ai_engineering
type: engineering
tool: general
status: evergreen
tags: [evaluation, generation]
created: 2026-03-05
---

# Evaluating Code Generation Models

## Purpose

Code generation is one of the highest-value LLM applications — and one of the few where correctness can be evaluated **objectively and automatically** via program execution. Rather than relying on semantic similarity to a reference solution, execution-based evaluation runs generated code against unit tests and reports a binary pass/fail signal.

This makes code evaluation qualitatively different from text generation evaluation: there is an objective correctness criterion, and evaluation is reproducible without human judgement. The pass@k metric captures both the quality of individual completions and the diversity of a model's output distribution.

Engineering teams building code assistants (Copilot-style) or automated coding agents should use these benchmarks to: (1) select base models; (2) validate fine-tunes and prompt changes; (3) set realistic expectations for task difficulty and success rates.

## Architecture

### Pass@k Metric

The standard code evaluation metric. For each problem in the benchmark:

1. Generate `n` independent completions (typically `n = 20` to `n = 200`).
2. Execute each completion against the problem's hidden unit test suite.
3. Record `c` = number of completions that pass all tests.
4. Compute `pass@k` using the unbiased estimator:

$$
\text{pass@k} = 1 - \frac{C(n - c, k)}{C(n, k)}
$$

where `C(n, k)` is the binomial coefficient. This is an unbiased estimator of the probability that at least one of `k` independently sampled completions passes.

Key variants:
- **pass@1**: single attempt; use greedy decoding (T=0) or low temperature (T=0.2). Most commonly reported; measures single-shot reliability.
- **pass@10**: 10 attempts; higher temperature (T=0.8); measures whether the model can produce a correct solution with light exploration.
- **pass@100**: extensive sampling; upper bound on model capability for the task.

### Benchmarks

**HumanEval** (Chen et al., OpenAI 2021):
- 164 Python function-level programming problems.
- Each problem: docstring + function signature; model generates the function body.
- Test suite: 7–8 unit tests per problem (hidden from model).
- Limitation: Python-only; relatively short functions; may be contaminated in large training corpora.

**MBPP** (Mostly Basic Programming Problems, Austin et al., Google 2021):
- 500 Python problems sourced from crowd-sourced programming challenges.
- More diverse difficulty distribution than HumanEval.
- 3 test cases per problem.

**MultiPL-E** (Cassano et al., 2022):
- HumanEval and MBPP prompts **machine-translated** into 18 programming languages: JavaScript, TypeScript, Java, C++, Go, Rust, Scala, Julia, Lua, R, Perl, Swift, PHP, Ruby, Haskell, D, Racket, Elixir.
- Measures cross-language generalisation; useful for multilingual code models (StarCoder, DeepSeek Coder).

**SWE-bench** (Jimenez et al., 2023):
- 2,294 real GitHub issues from 12 Python repositories (Django, scikit-learn, NumPy, etc.).
- Task: given the issue description and repository state, generate a patch that resolves the issue.
- Evaluation: apply patch; run repository test suite.
- Extremely hard: SWE-bench Verified pass@1 ≈ 18–50% for best models (2024).
- SWE-bench Lite (300 problems) is the standard reporting subset.

**LiveCodeBench** (Jain et al., 2024):
- Rolling benchmark: continuously adds new problems from LeetCode, AtCoder, and CodeForces.
- Contamination-resistant by design (problems added after most training cutoffs).
- Covers code generation, code execution prediction, and code repair subtasks.

**Execution Pipeline Architecture**

```
Input prompt
     │
     ▼
Model generates n completions
     │
     ▼
Post-processing (extract function body, strip markdown fences)
     │
     ▼
Sandbox execution (subprocess / Docker container with timeout)
     │
     ▼
Test suite runs against each completion
     │
     ▼
pass/fail per test → aggregate pass@k
```

Sandbox isolation is mandatory: generated code may attempt filesystem access, network calls, or infinite loops. Use `subprocess` with `timeout`, or a Docker container with network disabled and resource limits.

## Implementation Notes

### BigCode Evaluation Harness

The `bigcode-evaluation-harness` (HuggingFace BigCode Project) extends EleutherAI's harness with code-specific tasks:

```bash
pip install bigcode-evaluation-harness

# HumanEval pass@1 (greedy)
accelerate launch main.py \
  --model bigcode/starcoder2-7b \
  --tasks humaneval \
  --n_samples 1 \
  --temperature 0 \
  --allow_code_execution \
  --save_generations \
  --metric_output_path results/humaneval.json

# MultiPL-E: HumanEval in JavaScript
accelerate launch main.py \
  --model bigcode/starcoder2-7b \
  --tasks humaneval-js \
  --n_samples 20 \
  --temperature 0.8 \
  --allow_code_execution
```

The `--allow_code_execution` flag is required and explicit — the harness does not execute code by default as a safety measure.

### Sampling Strategy for pass@k

| Target metric | `n` samples | Temperature |
|---|---|---|
| pass@1 | 1 | 0 (greedy) |
| pass@10 | 20–50 | 0.8 |
| pass@100 | 200 | 0.8 |

Higher temperature increases output diversity (beneficial for k>1) but reduces per-sample quality (hurts pass@1). For production systems, pass@1 at low temperature reflects deployment reality; pass@10 measures headroom for agentic retry loops.

### Sandboxing Code Execution

Never execute untrusted generated code without isolation. Options by security level:

```python
# Minimal: subprocess with timeout (vulnerable to resource exhaustion)
import subprocess, tempfile, os

def run_code(code: str, timeout: int = 10) -> tuple[bool, str]:
    with tempfile.NamedTemporaryFile(mode='w', suffix='.py', delete=False) as f:
        f.write(code)
        fname = f.name
    try:
        result = subprocess.run(
            ['python', fname],
            capture_output=True, text=True, timeout=timeout
        )
        return result.returncode == 0, result.stdout + result.stderr
    except subprocess.TimeoutExpired:
        return False, "Timeout"
    finally:
        os.unlink(fname)

# Production: Docker container with --network none --memory 256m --cpus 0.5
# or: modal.com / AWS Lambda for serverless isolation
```

### Test Suite Quality

Test coverage quality is as important as generation quality for meaningful pass@k numbers. A problem with only 1–2 trivially satisfied test cases will have inflated pass@k. HumanEval has 7–8 tests per problem, including edge cases; MBPP has only 3. SWE-bench uses the repository's full existing test suite, which is the most realistic signal.

When building custom code evaluation for a product, invest in test suite quality: property-based testing, boundary cases, and semantically distinct test cases.

### Temperature and Post-Processing

LLMs generate code in markdown code fences, with prose explanations, or with boilerplate that the benchmark doesn't expect. Post-processing rules:

```python
import re

def extract_function(generation: str, entry_point: str) -> str:
    # Strip markdown fences
    code = re.sub(r'```\w*\n', '', generation).replace('```', '')
    # Extract only the target function (HumanEval convention)
    lines = code.split('\n')
    in_fn, result = False, []
    for line in lines:
        if f'def {entry_point}' in line:
            in_fn = True
        if in_fn:
            result.append(line)
            if line and not line[0].isspace() and 'def ' in line and entry_point not in line:
                break
    return '\n'.join(result)
```

## Trade-offs

**Execution-based evaluation**: objective, reproducible, no human or judge required. Requires test suites (which must be maintained); only applicable to constrained output formats; execution infrastructure adds complexity and security burden.

**Function-level benchmarks (HumanEval, MBPP)**: easy to run; well-understood; widely used for comparison. Limited scope: real software engineering requires multi-file reasoning, debugging, and repository navigation. Pass@1 scores on HumanEval (>80% for top models) are approaching saturation — discriminating power is decreasing.

**Real-world benchmarks (SWE-bench)**: much more predictive of actual utility for software engineers; requires real repository checkout and full test suite execution. Extremely slow and expensive to run (minutes per problem; requires cloning dozens of repos). Hard to implement without the official infrastructure.

**LiveCodeBench (contamination-resistant)**: best for current capability assessment since problems post-date most training cutoffs. Narrower scope (competitive programming) than general software engineering.

## References

- Chen et al. (2021). *Evaluating Large Language Models Trained on Code* (HumanEval). OpenAI. arXiv:2107.03374.
- Austin et al. (2021). *Program Synthesis with Large Language Models* (MBPP). Google Brain. arXiv:2108.07732.
- Cassano et al. (2022). *MultiPL-E: A Scalable and Polyglot Approach to Benchmarking Neural Code Generation*. IEEE TSE.
- Jimenez et al. (2023). *SWE-bench: Can Language Models Resolve Real-World GitHub Issues?* ICLR 2024.
- Jain et al. (2024). *LiveCodeBench: Holistic and Contamination Free Evaluation of Large Language Models for Code*. arXiv:2403.07974.

## Links

- [[benchmarks_and_harness|Benchmarks and Evaluation Harness]]
- [[llm_evaluation_overview|LLM Evaluation Overview]]
- [[foundation_model_overview|Foundation Model Overview]]
