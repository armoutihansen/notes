---
layer: 05_ai_engineering
type: engineering
tool: general
status: growing
tags: [agents, agentic-loop, react, planning, tools, memory, llm]
created: 2026-03-05
---

# Agentic Loop

## Purpose

The agentic loop is the core execution pattern for LLM-based autonomous agents. It enables an LLM to accomplish multi-step tasks by iteratively observing the environment, reasoning about the next action, executing that action via a tool, and incorporating the result into its subsequent reasoning. This transforms the LLM from a single-turn request-response system into a goal-directed process that can operate over extended time horizons, decompose complex tasks, and interact with external systems.

## Architecture

### Observe → Think → Act Loop

```
Goal / User query
    ↓
[Observation: task state, prior results, tool outputs]
    ↓
[Think: LLM reasoning — what is the current state? what is the next action?]
    ↓
[Act: select tool, fill arguments, execute]
    ↓
[Observation: tool result appended to context]
    ↓
[Termination check: goal achieved? max steps?]
    ↓ (if not done)
[back to Think]
    ↓ (if done)
Final answer
```

### Components

**(1) Planner (LLM)**
The LLM decides the next action given: the goal, the accumulated history of thoughts and observations, and the available tool definitions. Produces either a tool call (structured) or a final answer. In ReAct format, the LLM outputs alternating `Thought:` and `Action:` blocks; in function-calling format (see [[function_calling|Function Calling]]), it emits a structured JSON tool call.

**(2) Tool Executor**
Receives the tool call, validates arguments, executes the tool (API call, code execution, database query, web search, file read/write), and returns a structured observation. Tool results must be formatted for inclusion in the LLM context — include status (success/error), result data, and any metadata needed for downstream reasoning.

**(3) Memory**
- **Short-term / working memory**: The context window. All thoughts, observations, and tool results accumulated in the current run. Bounded by the model's context limit.
- **Long-term memory**: A vector store or key-value store that persists across runs or accumulates knowledge during a long run. The agent queries it when the context window would overflow or when episodic memory is needed.
- **Episodic memory**: Stored summaries of past agent runs, enabling learning from previous attempts.

**(4) Agent Supervisor / Termination Condition**
Guards against infinite loops and runaway cost. Enforces:
- `max_iterations`: hard cap on loop count (typically 10–30).
- Goal-completion detection: the LLM signals completion with a special token or the presence of a `FinalAnswer` action.
- Human-in-the-loop checkpoints: pause and request human confirmation before high-stakes actions (file deletion, API writes, financial transactions).

### Planning Strategies

**Single-step (function calling)**: The LLM selects one tool call per turn. Simple, low latency, works well for tasks that decompose naturally into linear steps.

**Multi-step chain-of-thought**: The LLM reasons through the full plan in a scratchpad before taking action (plan-then-execute). Useful when the task requires upfront decomposition.

**Hierarchical planning**: An orchestrator LLM decomposes the goal into subtasks and delegates to worker agents, each running their own agentic loop. See [[multi_agent_systems|Multi-Agent Systems]].

### ReAct Pattern
```
Thought: I need to find the population of Tokyo.
Action: web_search(query="Tokyo population 2024")
Observation: Tokyo's population is approximately 13.96 million (2024).
Thought: I now have the answer.
Action: final_answer("Tokyo's population is approximately 13.96 million.")
```
The interleaving of reasoning and observation creates a traceable audit trail that aids debugging and interpretability.

## Implementation Notes

**Tool definition**
Define tools as JSON schema (name, description, parameters). Description quality is critical — it is the primary signal the LLM uses to choose the right tool. Include: what the tool does, when to use it, what each parameter means, what the output looks like. See [[function_calling|Function Calling]] for detailed tool definition guidance.

**Max iterations**
Always set a hard iteration cap. Without it, a confused or adversarially prompted agent can loop indefinitely, running up API costs. Typical values: 10–20 for simple tasks, up to 50 for complex research tasks.

**Structured error handling**
Tool failures should return structured error observations, not raise exceptions that crash the loop:
```
Observation: {"status": "error", "error": "FileNotFoundError: config.json does not exist", "suggestion": "list_files() to see available files"}
```
Include actionable hints to help the model recover.

**Scratchpad pattern**
Allow the model to reason in a private `<scratchpad>` block that is not shown to the user. This inner monologue can be stripped from the final output, preserving reasoning quality without exposing raw chain-of-thought to end users.

**Checkpoint and state persistence**
For long-running tasks (>5 minutes), serialize the full agent state (messages, tool call history, memory references) to a durable store after each iteration. Enables resumption after failures without restarting from scratch.

**Context length management**
As the conversation grows, old observations may need to be summarized and replaced with a compact summary to stay within the context window. Implement a sliding window with periodic summarization for long-horizon tasks.

## Trade-offs

**Autonomy vs. safety**: Fully autonomous agents are powerful but unpredictable. Human-in-the-loop checkpoints (before irreversible actions) significantly reduce risk at the cost of throughput.

**Cost**: Each iteration = at least one LLM API call plus tool execution costs. A 10-iteration agent using GPT-4o with long contexts can cost $0.10–$1.00 per run. Evaluate whether multi-step agentic execution is necessary vs. a single well-structured prompt.

**Latency**: Agentic loops are inherently sequential (each action depends on the prior observation). A 10-step loop with 500ms LLM latency per step = 5 seconds minimum. Parallel tool execution within a single step (when tools are independent) can partially mitigate this.

**Error propagation**: A bad tool result early in the loop can send the agent down an incorrect reasoning path for many subsequent steps. Validation of tool outputs before they enter the context reduces this risk.

## References

- Yao et al. (2022). *ReAct: Synergizing Reasoning and Acting in Language Models*. ICLR 2023.
- Shinn et al. (2023). *Reflexion: Language Agents with Verbal Reinforcement Learning*. NeurIPS.
- Significant Gravitas (2023). *AutoGPT*. https://github.com/Significant-Gravitas/AutoGPT
- LangChain Agent documentation: https://python.langchain.com/docs/modules/agents

## Links
- [[function_calling|Function Calling]]
- [[multi_agent_systems|Multi-Agent Systems]]
- [[rag_architecture|RAG Architecture]]
- [[prompting_strategies|Prompting Strategies]]
- [[agentic_coding|Agentic Coding]]
