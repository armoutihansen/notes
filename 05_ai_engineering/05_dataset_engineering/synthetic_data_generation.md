---
layer: 05_ai_engineering
type: engineering
tool: general
status: growing
tags: [synthetic-data, self-instruct, evol-instruct, data-flywheel, llm-judge]
created: 2026-03-05
---

# Synthetic Data Generation

## Purpose

Generating training data programmatically using LLMs has become one of the most impactful techniques in modern AI engineering. High-quality human annotation is expensive ($10–50/example), slow, and difficult to scale. Synthetic generation can produce millions of examples at cents per thousand, enabling rapid iteration, targeted capability coverage, and data flywheels that improve models continuously. The critical constraint: synthetic data is only as reliable as its verification pipeline.

Synthetic data generation spans a spectrum from near-ground-truth quality (verified math and code problems with programmatic solution checking) to higher-risk applications (factual QA, where errors from the generating model propagate silently). The appropriate strategy depends heavily on whether verification is possible.

## Architecture

**Self-Instruct (Wang et al., 2022)**

The foundational synthetic data generation algorithm:

1. Start with a seed pool of ~175 human-written (instruction, input, output) triplets
2. Sample 8 tasks from the pool; prompt an LLM to generate `n` new tasks in the same format
3. Filter generated tasks: remove those with ROUGE-L overlap > 0.7 against existing tasks (near-duplicates), invalid format, or length < 3 words
4. Add surviving tasks to the pool; repeat

This bootstrapped GPT-3.5 with 52K examples (Alpaca dataset) at minimal cost. The core insight: a capable LLM can generate a diverse instruction distribution given only a small seed — the LLM knows far more instruction types than the seed demonstrates.

**Evol-Instruct (Xu et al., 2023 — WizardLM)**

Increases instruction *complexity* rather than quantity via two evolution operators:

- **In-depth evolution**: add constraints, increase reasoning steps, concretize abstractions, add domain specificity
- **In-breadth evolution**: generate a new instruction on a different but related topic

Applied iteratively, simple instructions become progressively more challenging. WizardLM trained on Evol-Instruct data substantially outperformed Alpaca on complex reasoning benchmarks. WizardCoder extended this to code generation.

**Constitutional AI Data Generation (Anthropic)**

1. Red-team the model to generate potentially harmful responses to adversarial prompts
2. Ask the model to critique each response against a list of principles (the "constitution")
3. Ask the model to revise the response based on its critique
4. Use (original prompt, revised response) pairs for SFT; then preference pairs for RLAIF

Generates safety alignment data without human labelers rating harmful content.

**Backtranslation for SFT**

Take a corpus of high-quality *output* text (e.g., well-written Wikipedia articles, expert forum answers). Prompt a strong LLM to generate an instruction that would plausibly produce each output: `"What question or instruction would produce the following response?"` Use (generated instruction, original output) pairs for fine-tuning. Effective for response-style alignment when you have many examples of the target output style but no corresponding instructions.

**Verified Synthetic Data for Math and Code**

The highest-quality synthetic pipeline:
1. Generate candidate problems using an LLM (with diversity controls)
2. Generate candidate solutions using the same or a stronger LLM
3. Verify solutions programmatically: unit tests (code), symbolic solvers (algebra), numerical evaluation (arithmetic)
4. Only include (problem, solution) pairs that pass verification

This produces near-ground-truth training data at scale. DeepSeek-R1, Qwen-Math, and similar reasoning models rely heavily on this approach. The key insight: verification decouples generation quality from training data quality.

## Implementation Notes

**Self-Instruct filtering:**
```python
from rouge_score import rouge_scorer

scorer = rouge_scorer.RougeScorer(["rougeL"], use_stemmer=True)

def is_duplicate(new_instruction, existing_pool, threshold=0.7):
    for existing in existing_pool:
        score = scorer.score(new_instruction, existing)["rougeL"].fmeasure
        if score > threshold:
            return True
    return False

def is_valid(instruction):
    words = instruction.split()
    return (
        len(words) >= 3
        and instruction[0].isalpha()  # starts with letter
        and not instruction.startswith("Image")  # filter image instructions
    )
```

**LLM judge quality scoring:**
```python
JUDGE_PROMPT = """Rate the following instruction-response pair on a scale of 1-5 for:
1. Instruction clarity (is it unambiguous?)
2. Response quality (accurate, helpful, complete?)
3. Diversity value (does it cover a task type not already well-represented?)

Instruction: {instruction}
Response: {response}

Output JSON: {{"clarity": N, "quality": N, "diversity": N}}"""

# Filter: include only pairs where all scores >= 3
```

**Verified code data pipeline:**
```python
def generate_and_verify(problem_prompt: str, n_attempts: int = 5):
    """Generate a coding problem with verified solution."""
    problem = llm.generate(problem_prompt)
    
    for _ in range(n_attempts):
        solution = llm.generate(f"Solve this problem:\n{problem}\nInclude test cases.")
        test_results = run_in_sandbox(solution)
        
        if test_results.all_passed:
            return {"problem": problem, "solution": solution}
    
    return None  # discard unverifiable examples
```

**Data flywheel architecture:**
```
Production traffic
    → sample 1% of queries
    → run through quality filter (perplexity, length, safety)
    → route high-quality examples to human review queue
    → human approves/edits response
    → add to training set
    → periodic fine-tune → deploy → repeat
```

This compounds over time: better model → better production responses → higher-quality training data → better model.

**Mixing synthetic and human data:**
- Never train 100% on synthetic data; maintain a 20–40% human-curated baseline
- Monitor for distribution collapse: if synthetic data dominates, models may lose response diversity
- Use held-out human-annotated evaluation sets — never evaluate on synthetic data alone

## Trade-offs

**Verified vs. unverified synthetic data:**

| Type | Quality | Scale | Use case |
|---|---|---|---|
| Verified (math/code) | Near ground truth | Large | Reasoning, coding tasks |
| Unverified factual | Risky (propagates errors) | Very large | Avoid for facts |
| Synthetic style/format | Safe | Very large | Format, tone, structure |
| Backtranslation | High (real outputs) | Medium | Response style alignment |

**Self-distillation risk:**
When a model generates its own training data and then trains on it, it can amplify its own errors and biases — a form of distribution collapse. Mitigations: use a *stronger* model for generation (never self-generate for the same model tier), apply rigorous filtering, and always include a human-curated anchor.

**Evol-Instruct complexity vs. distribution:**
Progressive complexity evolution produces harder examples, but can drift toward unusual or unrealistic instruction types if not controlled. Include human evaluation checkpoints every few evolution iterations.

**Scalability vs. quality ceiling:**
Synthetic generation is infinitely scalable; quality is upper-bounded by the generating model's capabilities. For tasks at the frontier of model capability, synthetic data quickly degrades. For tasks well within model capability (format following, basic QA, code in common languages), synthetic data quality is excellent.

**Data flywheel dependencies:**
Building a flywheel requires production traffic, annotation infrastructure, and a training pipeline. Effective at scale (millions of queries/day); less effective for niche deployments with low traffic volume.

## References

- Wang et al. (2022) — *Self-Instruct: Aligning Language Models with Self-Generated Instructions* (arXiv:2212.10560)
- Xu et al. (2023) — *WizardLM: Empowering Large Language Models to Follow Complex Instructions* (Evol-Instruct)
- Bai et al. (2022) — *Constitutional AI: Harmlessness from AI Feedback* (Anthropic)
- Li et al. (2023) — *Self-Alignment with Instruction Backtranslation*
- Zhou et al. (2023) — *LIMA: Less Is More for Alignment*
- Taori et al. (2023) — *Alpaca: A Strong, Replicable Instruction-Following Model*

## Links
- [[instruction_data_design|Instruction Data Design]]
- [[finetuning_strategies|Fine-tuning Strategies]]
- [[rl_finetuning|Reinforcement Learning Fine-tuning]]
- [[dataset_versioning|Dataset Versioning]]
