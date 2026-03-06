---
layer: 05_ai_engineering
type: engineering
tool: general
status: growing
tags: [prompting, icl, chain-of-thought, react, few-shot, llm]
created: 2026-03-05
---

# Prompting Strategies

## Purpose

Prompting strategies are techniques for eliciting desired behavior from LLMs without modifying model weights. Rather than fine-tuning, prompting exploits the model's pre-existing capabilities by carefully structuring inputs. This is the primary engineering lever for most production AI systems — cheap to iterate, zero training cost, and immediately applicable to any model. Understanding the landscape of prompting techniques allows practitioners to match the right strategy to the task's complexity, latency budget, and quality requirements.

## Architecture

### In-Context Learning (ICL)
- **Zero-shot**: No examples provided. Relies entirely on the model's instruction-following capability. Works well for simple, well-defined tasks with frontier models.
- **One-shot**: A single example included in the prompt. Useful when the output format is non-obvious or domain-specific.
- **Few-shot**: Multiple demonstrations (typically 3–10). The model generalizes from the pattern of examples. Particularly effective for tasks with non-standard output formats, specialized reasoning, or domain terminology.

### Chain-of-Thought (CoT)
CoT prompting instructs the model to produce intermediate reasoning steps before the final answer, improving accuracy on multi-step tasks.
- **Zero-shot CoT**: Append "Let's think step by step." to the prompt. Surprisingly effective — triggers reasoning behavior without demonstrations.
- **Few-shot CoT**: Provide demonstrations that include explicit reasoning traces. Higher quality than zero-shot CoT; demonstrations encode the expected reasoning style.
- **Self-Consistency**: Generate multiple independent CoT completions (e.g., k=5–20) at higher temperature, then take a majority vote on the final answer. Substantially improves performance at the cost of k× inference calls.

### Tree of Thoughts (ToT)
Extends CoT to a tree-search paradigm. The model proposes multiple candidate next steps, evaluates them (via another LLM call or heuristic), and searches the best path. Appropriate for problems that benefit from lookahead: math proofs, planning, complex puzzles. High latency and implementation complexity.

### ReAct (Reasoning + Acting)
Interleaves `Thought:` (reasoning trace) and `Action:` (tool call) steps in the prompt. After each action, an `Observation:` is appended and the model continues reasoning. Foundational pattern for agentic systems. See [[agentic_loop|Agentic Loop]] and [[function_calling|Function Calling]].

### Role Prompting and Persona Setting
Setting a system prompt with a specific role ("You are a senior software engineer specializing in distributed systems") shapes tone, vocabulary, and reasoning style. Useful for calibrating output register and domain focus.

### Prompt Chaining and Decomposition
Decompose complex tasks into a pipeline of smaller prompts, each handling one subtask. The output of one step becomes the input to the next. Reduces errors from asking a single prompt to do too much; each step can be validated independently.

## Implementation Notes

**Few-shot example selection**
- Select examples similar in structure to the query (semantic similarity retrieval from an example bank can help at scale).
- Maintain diversity — examples covering different sub-types of the task improve generalization.
- Ensure consistent formatting across all examples; inconsistency degrades performance.

**Prompt formatting**
- Use markdown headers, numbered lists, or XML tags (`<document>`, `<instructions>`) to create clear structural boundaries.
- Delimiters (triple backticks, `---`, `<user_input>`) separate untrusted content from instructions, which also aids injection resistance (see [[prompt_injection_and_guardrails|Prompt Injection and Guardrails]]).

**System vs user vs assistant roles**
- In chat models, use the `system` role for persistent instructions and persona; `user` for the task input; `assistant` for pre-filled reasoning scaffolds when needed.
- The model treats system instructions as higher priority, though this is a soft constraint.

**Temperature**
- CoT and reasoning tasks: use moderate temperature (0.5–0.9) when sampling multiple paths (Self-Consistency).
- Deterministic extraction and classification: temperature = 0 for reproducibility.

**Prompt length management**
- Context-window costs scale linearly; long few-shot prompts increase latency and cost.
- Use DSPy (see [[dspy_systematic_prompting|DSPy — Systematic Prompt Optimization]]) to automate example selection and reduce manual prompt bloat.

**Structured outputs**
- When the task requires a specific output format, combine prompting with constrained decoding or schema validation. See [[structured_outputs|Structured Outputs]].

## Trade-offs

| Strategy | Quality | Cost | Latency | Notes |
|---|---|---|---|---|
| Zero-shot | Baseline | 1× | Lowest | Simple tasks only |
| Few-shot ICL | +10–20% | 1.2–2× | Low | Context window usage |
| Zero-shot CoT | +15–25% | 1.1× | Slight increase | Best bang-for-buck |
| Few-shot CoT | +20–35% | 1.5–3× | Moderate | Needs good demonstrations |
| Self-Consistency (k=20) | +5–15% over CoT | 20× | High | Only for high-stakes tasks |
| Tree of Thoughts | +25–40% | Very high | Very high | Niche; planning/math |
| ReAct | Task-dependent | Varies | High | Enables tool use |

Context window consumption is the main cost driver for few-shot approaches. On tasks where quality ceiling matters (e.g., code generation, complex reasoning), Self-Consistency or ToT may be worth the cost.

## References

- Wei et al. (2022). *Chain-of-Thought Prompting Elicits Reasoning in Large Language Models*. NeurIPS.
- Wang et al. (2023). *Self-Consistency Improves Chain of Thought Reasoning in Language Models*. ICLR.
- Yao et al. (2023). *Tree of Thoughts: Deliberate Problem Solving with LLMs*. NeurIPS.
- Yao et al. (2022). *ReAct: Synergizing Reasoning and Acting in Language Models*. ICLR 2023.
- Brown et al. (2020). *Language Models are Few-Shot Learners*. NeurIPS.

## Links
- [[structured_outputs|Structured Outputs]]
- [[prompt_injection_and_guardrails|Prompt Injection and Guardrails]]
- [[dspy_systematic_prompting|DSPy — Systematic Prompt Optimization]]
- [[agentic_loop|Agentic Loop]]
- [[function_calling|Function Calling]]
